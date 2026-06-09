---
title: "Hoverboard Motor Control"
date: 2026-06-09
tags: ["motors", "arduino", "pid", "prototype", "raspberry-pi"]
summary: "Using a cheap hoverboard motor and a $25 controller as the drive system for the RoBug prototype, with a PID speed controller running on an Arduino and a live tuning dashboard on the Pi."
---

## Why hoverboard motors

For the first prototype, I want to test the full navigation and control stack — the software that will eventually run on the real RoBug — without committing to the final hardware. Hoverboard motors are cheap, well-documented, and easy to get. I picked one up off Facebook Marketplace for next to nothing and have been playing around with it for the past week or so.

The goal is simple: accept speed commands from the Raspberry Pi, spin the motor at that speed. Once that works, I can wire up ROS2, test the navigation stack, and figure out what needs fixing before I build the real thing.

## The motor controller

{{< figure src="https://m.media-amazon.com/images/I/61+IpQ4KQWL._AC_SL1500_.jpg" alt="ZS-X11H hoverboard motor controller" >}}

I picked up a [ZS-X11H controller from Amazon](https://www.amazon.ca/dp/B087M2378D) for about $25. These are popular in the hoverboard hacking community because they're so cheap and take a standard PWM input — which means an Arduino can drive them directly. The controller handles all the brushless commutation; I just send it a PWM duty cycle and it does the rest.

This video is a good overview of how the whole thing works:

{{< youtube 1E_WDIyZ6Y0 >}}

## Speed control

A raw PWM signal gets you open-loop speed control, which is fine on a bench but useless on a robot. Load changes — terrain, slope, payload — will throw the speed off and the controller has no way to compensate. I need closed-loop control, which means a PID controller.

The hoverboard motor has hall effect sensors that output a pulse train as the motor spins. I use an interrupt on the Arduino to measure the time between pulses and convert that to RPM. The PID loop runs at 50Hz and adjusts the PWM output to hold the target speed.

The control architecture is feedforward + PID correction. The feedforward term uses the motor's KV rating to calculate a baseline PWM for a given target RPM, and the PID trims the error on top of that. This means the PID only has to correct small residual errors rather than doing all the work, which keeps the gains small and the response clean.

Direction control uses the ZS-X11H's DIR input pin. When a direction change is requested, the controller ramps speed down to zero first, flips the direction pin, then ramps back up — so there's no hard reversal shock to the motor.

## The tuning dashboard

To tune the PID gains without reflashing the Arduino every time, I built a small Flask app that runs on the Raspberry Pi. It reads the Arduino's serial output and streams it to a browser over SSE. There are sliders for target RPM and the three PID gains, two live charts (RPM/setpoint and PWM/correction), and all changes send serial commands back to the Arduino in real time.

{{< youtube 1E_WDIyZ6Y0 >}}

## The code

<details>
<summary>Arduino sketch</summary>

```cpp
#include <PID_v1.h>

const int S_PIN = 2;
const int P_PIN = 9;
const int DIR_PIN = 4;
const float PULSES_PER_REV = 90.0;
const float SUPPLY_V = 12.7;
const float KV = 17.2;
const unsigned long STALE_TIMEOUT_US = 500000;
const unsigned long LOOP_INTERVAL = 20;
const unsigned long PRINT_INTERVAL = 200;
const double RAMP_STEP = 50.0;

volatile unsigned long pulseInterval = 0;
volatile unsigned long lastPulseTime = 0;

void onPulse() {
  unsigned long now = micros();
  pulseInterval = now - lastPulseTime;
  lastPulseTime = now;
}

float getRPM() {
  noInterrupts();
  unsigned long interval = pulseInterval;
  unsigned long lastPulse = lastPulseTime;
  interrupts();
  if (micros() - lastPulse > STALE_TIMEOUT_US) return 0;
  if (interval == 0) return 0;
  return 60000000.0 / (interval * PULSES_PER_REV);
}

double requestedRPM = 0, targetRPM = 0, currentRPM = 0, pidCorrection = 0;
double Kp = 0.5, Ki = 0.7, Kd = 0.0;
bool forward = true;
PID myPID(&currentRPM, &pidCorrection, &targetRPM, Kp, Ki, Kd, DIRECT);

void setDirection(bool fwd) {
  if (fwd == forward) return;
  forward = fwd;
  digitalWrite(DIR_PIN, forward ? HIGH : LOW);
  myPID.SetMode(MANUAL);
  pidCorrection = 0;
  myPID.SetMode(AUTOMATIC);
}

void handleSerial() {
  if (!Serial.available()) return;
  char cmd = Serial.peek();
  if (cmd == 'p' || cmd == 'i' || cmd == 'd') {
    Serial.read();
    double val = Serial.parseFloat();
    while (Serial.available()) Serial.read();
    if (cmd == 'p') Kp = val;
    else if (cmd == 'i') Ki = val;
    else if (cmd == 'd') Kd = val;
    myPID.SetTunings(Kp, Ki, Kd);
    Serial.print("Kp="); Serial.print(Kp);
    Serial.print(" Ki="); Serial.print(Ki);
    Serial.print(" Kd="); Serial.println(Kd);
  } else {
    requestedRPM = Serial.parseFloat();
    while (Serial.available()) Serial.read();
    Serial.print("Target: "); Serial.println(requestedRPM);
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(S_PIN, INPUT_PULLUP);
  pinMode(P_PIN, OUTPUT);
  pinMode(DIR_PIN, OUTPUT);
  digitalWrite(DIR_PIN, HIGH);
  analogWrite(P_PIN, 0);
  attachInterrupt(digitalPinToInterrupt(S_PIN), onPulse, CHANGE);

  myPID.SetMode(AUTOMATIC);
  myPID.SetOutputLimits(-80, 80);
  myPID.SetSampleTime(LOOP_INTERVAL);

  Serial.println("Commands: <rpm> (negative=reverse), p<val>, i<val>, d<val>");
}

unsigned long lastLoop = 0;
unsigned long lastPrint = 0;

void loop() {
  handleSerial();

  unsigned long now = millis();
  if (now - lastLoop < LOOP_INTERVAL) return;
  lastLoop = now;

  double absReq = abs(requestedRPM);
  bool reqFwd = requestedRPM >= 0;

  if (reqFwd != forward && targetRPM > 0) {
    targetRPM = max(0.0, targetRPM - RAMP_STEP);
    if (targetRPM == 0) setDirection(reqFwd);
  } else {
    setDirection(reqFwd);
    if (targetRPM < absReq) targetRPM = min(absReq, targetRPM + RAMP_STEP);
    else if (targetRPM > absReq) targetRPM = max(absReq, targetRPM - RAMP_STEP);
  }

  currentRPM = getRPM();
  myPID.Compute();

  float pwm_ff = (targetRPM / KV) / SUPPLY_V * 255.0;
  int pwm = (int)constrain(pwm_ff + pidCorrection, 0, 255);
  if (targetRPM <= 0) pwm = 0;

  analogWrite(P_PIN, pwm);

  if (now - lastPrint >= PRINT_INTERVAL) {
    lastPrint = now;
    float signedRPM = forward ? currentRPM : -currentRPM;
    int signedPWM = forward ? pwm : -pwm;
    Serial.print("RPM: "); Serial.print(signedRPM, 1);
    Serial.print(" | FF: "); Serial.print((int)pwm_ff);
    Serial.print(" | Corr: "); Serial.print((int)pidCorrection);
    Serial.print(" | PWM: "); Serial.println(signedPWM);
  }
}
```

</details>

<details>
<summary>Flask tuning dashboard (runs on the Pi)</summary>

```python
from flask import Flask, Response, request, jsonify
import serial, threading, time, json, queue

app = Flask(__name__)

ser = serial.Serial('/dev/ttyUSB0', 115200, timeout=1)
time.sleep(2)

state = {'rpm': 0.0, 'pwm': 0, 'corr': 0, 'target': 0.0, 'kp': 0.5, 'ki': 0.7, 'kd': 0.0}
clients = []
clients_lock = threading.Lock()

def serial_reader():
    while True:
        try:
            line = ser.readline().decode().strip()
            if line.startswith('RPM:'):
                parts = line.split()
                # format: RPM: x | FF: x | Corr: x | PWM: x
                state['rpm'] = float(parts[1])
                state['corr'] = int(parts[7])
                state['pwm'] = int(parts[-1])
                data = json.dumps({'rpm': state['rpm'], 'pwm': state['pwm'], 'corr': state['corr'], 'target': state['target']})
                with clients_lock:
                    for q in list(clients):
                        q.put(data)
        except Exception:
            pass

threading.Thread(target=serial_reader, daemon=True).start()

@app.route('/')
def index():
    return HTML.replace('__STATE__', json.dumps(state))

@app.route('/stream')
def stream():
    q = queue.Queue()
    with clients_lock:
        clients.append(q)
    def generate():
        try:
            while True:
                try:
                    data = q.get(timeout=15)
                    yield f'data: {data}\n\n'
                except queue.Empty:
                    yield ': keepalive\n\n'
        finally:
            with clients_lock:
                if q in clients:
                    clients.remove(q)
    return Response(generate(), mimetype='text/event-stream')

@app.route('/set', methods=['POST'])
def set_value():
    data = request.json
    cmd = None
    for key in ('kp', 'ki', 'kd', 'target'):
        if key in data:
            val = float(data[key])
            state[key] = val
            cmd = f"{key[1]}{val}\n" if key != 'target' else f"{val}\n"
            break
    if cmd:
        ser.write(cmd.encode())
    return jsonify({'ok': True})

HTML = '''<!DOCTYPE html>
<html>
<head>
<title>Motor PID</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4"></script>
<style>
  * { box-sizing: border-box; }
  body { font-family: monospace; background: #151515; color: #ddd; margin: 0; padding: 20px; }
  h1 { margin: 0 0 16px; font-size: 1.2em; color: #fff; }
  .charts { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; margin-bottom: 16px; }
  .chart-wrap { background: #1e1e1e; border-radius: 8px; padding: 16px; }
  .controls { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }
  .card { background: #1e1e1e; border-radius: 8px; padding: 16px; }
  .card h3 { margin: 0 0 12px; font-size: 0.9em; color: #aaa; text-transform: uppercase; letter-spacing: 1px; }
  .slider-row { margin: 10px 0; }
  .slider-row label { display: flex; justify-content: space-between; margin-bottom: 4px; font-size: 0.9em; }
  .slider-row label span { color: #7df; }
  input[type=range] { width: 100%; accent-color: #7df; }
  .stats { display: flex; gap: 24px; margin-bottom: 16px; font-size: 0.95em; }
  .stat { color: #888; }
  .stat span { font-weight: bold; }
  #stat-rpm { color: #4af; }
  #stat-target { color: #fa4; }
  #stat-pwm { color: #c8f; }
  #stat-corr { color: #f84; }
</style>
</head>
<body>
<h1>Motor PID Dashboard</h1>
<div class="stats">
  <div class="stat">RPM <span id="stat-rpm">0</span></div>
  <div class="stat">Setpoint <span id="stat-target">0</span></div>
  <div class="stat">PWM <span id="stat-pwm">0</span></div>
  <div class="stat">Corr <span id="stat-corr">0</span></div>
</div>
<div class="charts">
  <div class="chart-wrap">
    <canvas id="chart-rpm" height="120"></canvas>
  </div>
  <div class="chart-wrap">
    <canvas id="chart-pwm" height="120"></canvas>
  </div>
</div>
<div class="controls">
  <div class="card">
    <h3>Target RPM</h3>
    <div class="slider-row">
      <label>RPM <span id="target-val">0</span></label>
      <input type="range" id="target" min="-200" max="200" step="1" value="0" oninput="updateTarget()">
    </div>
  </div>
  <div class="card">
    <h3>PID Gains</h3>
    <div class="slider-row">
      <label>Kp <span id="kp-val">0.50</span></label>
      <input type="range" id="kp" min="0" max="5" step="0.01" value="0.5" oninput="updateGain(\'kp\')">
    </div>
    <div class="slider-row">
      <label>Ki <span id="ki-val">0.70</span></label>
      <input type="range" id="ki" min="0" max="2" step="0.01" value="0.7" oninput="updateGain(\'ki\')">
    </div>
    <div class="slider-row">
      <label>Kd <span id="kd-val">0.000</span></label>
      <input type="range" id="kd" min="0" max="1" step="0.001" value="0" oninput="updateGain(\'kd\')">
    </div>
  </div>
</div>
<script>
const MAX = 120;
const labels = [], rpmData = [], targetData = [], pwmData = [], corrData = [];

const chartOpts = (datasets, scales) => ({
  type: \'line\',
  data: { labels, datasets },
  options: {
    animation: false,
    interaction: { mode: \'index\', intersect: false },
    scales,
    plugins: { legend: { labels: { color: \'#aaa\', boxWidth: 20 } } }
  }
});

const chartRPM = new Chart(document.getElementById(\'chart-rpm\').getContext(\'2d\'), chartOpts(
  [
    { label: \'RPM\',      data: rpmData,    borderColor: \'#4af\', borderWidth: 1.5, tension: 0.2, pointRadius: 0 },
    { label: \'Setpoint\', data: targetData, borderColor: \'#fa4\', borderWidth: 1.5, tension: 0,   pointRadius: 0, borderDash: [6,3] },
  ],
  {
    x: { display: false },
    y: { min: -250, max: 250, title: { display: true, text: \'RPM\', color: \'#4af\' }, ticks: { color: \'#888\' }, grid: { color: \'#2a2a2a\' } }
  }
));

const chartPWM = new Chart(document.getElementById(\'chart-pwm\').getContext(\'2d\'), chartOpts(
  [
    { label: \'PWM\',  data: pwmData,  borderColor: \'#c8f\', borderWidth: 1.5, tension: 0.2, pointRadius: 0 },
    { label: \'Corr\', data: corrData, borderColor: \'#f84\', borderWidth: 1.5, tension: 0.2, pointRadius: 0 },
  ],
  {
    x: { display: false },
    y: { min: -255, max: 255, title: { display: true, text: \'PWM / Corr\', color: \'#c8f\' }, ticks: { color: \'#888\' }, grid: { color: \'#2a2a2a\' } }
  }
));

let t = 0;
const es = new EventSource(\'/stream\');
es.onmessage = e => {
  const d = JSON.parse(e.data);
  labels.push(t++);
  rpmData.push(d.rpm);
  targetData.push(d.target);
  pwmData.push(d.pwm);
  corrData.push(d.corr);
  if (labels.length > MAX) { labels.shift(); rpmData.shift(); targetData.shift(); pwmData.shift(); corrData.shift(); }
  chartRPM.update(\'none\');
  chartPWM.update(\'none\');
  document.getElementById(\'stat-rpm\').textContent    = d.rpm.toFixed(1);
  document.getElementById(\'stat-target\').textContent = d.target.toFixed(1);
  document.getElementById(\'stat-pwm\').textContent    = d.pwm;
  document.getElementById(\'stat-corr\').textContent   = d.corr;
};

const s = __STATE__;
[\'kp\',\'ki\',\'kd\'].forEach(k => {
  document.getElementById(k).value = s[k];
  document.getElementById(k+\'-val\').textContent = s[k].toFixed(k===\'kd\'?3:2);
});
document.getElementById(\'target\').value = s.target;
document.getElementById(\'target-val\').textContent = s.target;

const timers = {};
function updateGain(g) {
  const val = parseFloat(document.getElementById(g).value);
  document.getElementById(g+\'-val\').textContent = val.toFixed(g===\'kd\'?3:2);
  clearTimeout(timers[g]);
  timers[g] = setTimeout(() => post({[g]: val}), 150);
}

function updateTarget() {
  const val = parseFloat(document.getElementById(\'target\').value);
  document.getElementById(\'target-val\').textContent = val;
  clearTimeout(timers[\'target\']);
  timers[\'target\'] = setTimeout(() => post({target: val}), 150);
}

function post(body) {
  fetch(\'/set\', { method: \'POST\', headers: {\'Content-Type\':\'application/json\'}, body: JSON.stringify(body) });
}
</script>
</body>
</html>'''

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5053, threaded=True)
```

</details>

## Where we are

The motor accepts speed commands over serial, runs closed-loop at 50Hz, and handles direction changes smoothly. Next up is wiring this into ROS2 so the Pi can drive it from the navigation stack.
