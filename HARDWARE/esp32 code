// ESP32 Mine-Detecting Rover with Touch-Responsive Web UI, PWM Speed Control, LCD Status

#include <WiFi.h>
#include <TinyGPSPlus.h>
#include <HardwareSerial.h>
#include <HTTPClient.h>
#include <WebServer.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <time.h>

// WiFi Credentials (MASKED)
const char* ssid = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";

// Firebase Host (MASKED)
const String firebaseHost = "https://your-project-id.firebaseio.com";

// Pins
const int metalPin = 15;
const int a1 = 32, a2 = 14, b1 = 12, b2 = 13;
const int ena = 25, enb = 26;

// GPS
TinyGPSPlus gps;
HardwareSerial gpsSerial(1); // RX=16, TX=17
float currentLat = 0.0, currentLng = 0.0, lastLat = 0.0, lastLng = 0.0;
bool metalDetected = false;
String lastTimestamp = "";

// LCD
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Web server
WebServer server(80);
String motorState = "stop";
int speedPWM = 180;
bool buttonPressed = false;
bool forwardMode = false;

// Time
const char* ntpServer = "pool.ntp.org";
const long gmtOffset_sec = 19800;
const int daylightOffset_sec = 0;

String getTimeString() {
  struct tm timeinfo;
  if (!getLocalTime(&timeinfo)) return "N/A";
  char buffer[30];
  strftime(buffer, sizeof(buffer), "%Y-%m-%d %H:%M:%S", &timeinfo);
  return String(buffer);
}

void updateLCD(String line1, String line2) {
  lcd.clear();
  lcd.setCursor(0, 0); lcd.print(line1);
  lcd.setCursor(0, 1); lcd.print(line2);
}

void stopMotors() {
  digitalWrite(a1, LOW); digitalWrite(a2, LOW);
  digitalWrite(b1, LOW); digitalWrite(b2, LOW);
  analogWrite(ena, 0);
  analogWrite(enb, 0);
}

void controlMotors() {
  if (!buttonPressed && !forwardMode) {
    stopMotors();
    return;
  }

  analogWrite(ena, speedPWM);
  analogWrite(enb, speedPWM);

  if (motorState == "forward") {
    digitalWrite(a1, HIGH); digitalWrite(a2, LOW);
    digitalWrite(b1, HIGH); digitalWrite(b2, LOW);
  } else if (motorState == "backward") {
    digitalWrite(a1, LOW); digitalWrite(a2, HIGH);
    digitalWrite(b1, LOW); digitalWrite(b2, HIGH);
  } else if (motorState == "left") {
    digitalWrite(a1, LOW); digitalWrite(a2, HIGH);
    digitalWrite(b1, HIGH); digitalWrite(b2, LOW);
  } else if (motorState == "right") {
    digitalWrite(a1, HIGH); digitalWrite(a2, LOW);
    digitalWrite(b1, LOW); digitalWrite(b2, HIGH);
  } else {
    stopMotors();
  }
}

void handleMove() {
  if (server.hasArg("dir")) {
    motorState = server.arg("dir");
    buttonPressed = (motorState != "stop");
  }
  server.send(200, "text/plain", "OK");
}

void handleSpeed() {
  if (server.hasArg("val")) {
    speedPWM = server.arg("val").toInt();
  }
  server.send(200, "text/plain", "Speed set");
}

void handleStatus() {
  String status = "{";
  status += "\"lat\":" + String(currentLat, 6) + ",";
  status += "\"lng\":" + String(currentLng, 6) + ",";
  status += "\"metal\":" + String(metalDetected ? 1 : 0) + ",";
  status += "\"time\":\"" + getTimeString() + "\",";
  status += "\"last\":\"" + lastTimestamp + "\"";
  status += "}";
  server.send(200, "application/json", status);
}

void handleSwitch() {
  forwardMode = !forwardMode;
  server.send(200, "text/plain", forwardMode ? "Auto Forward ON" : "Auto Forward OFF");
}

void handleRoot() {
  String html = R"rawliteral(
  <!DOCTYPE html><html><head><meta charset='UTF-8'>
  <meta name='viewport' content='width=device-width, initial-scale=1.0'>
  <style>
    body { font-family: sans-serif; background: #111; color: white; text-align: center; padding: 20px; }
    h2 { color: #0af; }
    .btn {
      display: inline-block; padding: 15px 25px; margin: 10px;
      font-size: 18px; background: #007BFF; color: white;
      border: none; border-radius: 8px; cursor: pointer;
      transition: 0.3s;
    }
    .btn:hover { background: #0056b3; }
  </style>
  <script>
    function bindHoldButton(id, startUrl, stopUrl) {
      const btn = document.getElementById(id);
      btn.addEventListener('mousedown', () => { fetch(startUrl); });
      btn.addEventListener('mouseup', () => { fetch(stopUrl); });
      btn.addEventListener('mouseleave', () => { fetch(stopUrl); });
      btn.addEventListener('touchstart', (e) => { e.preventDefault(); fetch(startUrl); });
      btn.addEventListener('touchend', (e) => { e.preventDefault(); fetch(stopUrl); });
    }
    window.onload = () => {
      bindHoldButton('forward', '/move?dir=forward', '/move?dir=stop');
      bindHoldButton('backward', '/move?dir=backward', '/move?dir=stop');
      bindHoldButton('left', '/move?dir=left', '/move?dir=stop');
      bindHoldButton('right', '/move?dir=right', '/move?dir=stop');
    }
  </script>
  </head><body>
  <h2>🚗 Rover Control Panel</h2>
  <button id="forward" class="btn">↑ Forward</button><br>
  <button id="left" class="btn">← Left</button>
  <button onclick="fetch('/switch')" class="btn">⏯ Auto Mode</button>
  <button id="right" class="btn">→ Right</button><br>
  <button id="backward" class="btn">↓ Backward</button>
  <br><br>
  <input type='range' min='100' max='255' value='180' id='speed' oninput="fetch('/speed?val='+this.value)"><br>
  <div id='info' style='margin-top:20px'></div>
  <script>
    setInterval(()=>{
      fetch('/status')
        .then(r=>r.json())
        .then(d=>{
          document.getElementById('info').innerHTML =
            `<p>Lat: ${d.lat}</p><p>Lng: ${d.lng}</p><p>Metal: ${d.metal}</p><p>Time: ${d.time}</p><p>Last Sent: ${d.last}</p>`;
        });
    }, 2000);
  </script>
  </body></html>
  )rawliteral";
  server.send(200, "text/html", html);
}

// 🔄 Upload current position every 2 seconds
unsigned long lastUploadMillis = 0;
void uploadRoverPosition() {
  if (!gps.location.isValid()) return;
  String json = "{";
  json += "\"latitude\":" + String(currentLat, 6) + ",";
  json += "\"longitude\":" + String(currentLng, 6) + ",";
  json += "\"timestamp\":\"" + getTimeString() + "\"";
  json += "}";
  HTTPClient http;
  http.begin(firebaseHost + "/rover_position.json");
  http.addHeader("Content-Type", "application/json");
  http.PUT(json);
  http.end();
}

void setup() {
  Serial.begin(115200);
  lcd.init(); lcd.backlight(); updateLCD("Booting...", "WiFi Scan...");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500); lcd.print(".");
  }
  updateLCD("WiFi OK", WiFi.localIP().toString()); delay(2000);
  configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);

  gpsSerial.begin(9600, SERIAL_8N1, 16, 17);
  pinMode(metalPin, INPUT_PULLDOWN);
  pinMode(a1, OUTPUT); pinMode(a2, OUTPUT); pinMode(b1, OUTPUT); pinMode(b2, OUTPUT);
  pinMode(ena, OUTPUT); pinMode(enb, OUTPUT);

  server.on("/", handleRoot);
  server.on("/move", handleMove);
  server.on("/speed", handleSpeed);
  server.on("/status", handleStatus);
  server.on("/switch", handleSwitch);
  server.begin();
}

void loop() {
  server.handleClient();

  while (gpsSerial.available()) {
    gps.encode(gpsSerial.read());
  }

  if (gps.location.isValid()) {
    currentLat = gps.location.lat();
    currentLng = gps.location.lng();
    updateLCD("Searching...", String(currentLat, 6) + "," + String(currentLng, 6));
  } else {
    updateLCD("Waiting GPS...", "...");
  }

  // 🔁 Every 2 seconds, upload position
  if (millis() - lastUploadMillis >= 2000) {
    uploadRoverPosition();
    lastUploadMillis = millis();
  }

  int metalRaw = digitalRead(metalPin);
  delayMicroseconds(50);
  int confirmRead = digitalRead(metalPin);
  metalDetected = (metalRaw == HIGH && confirmRead == HIGH);

  if (metalDetected && gps.location.isValid()) {
    float distanceMoved = TinyGPSPlus::distanceBetween(currentLat, currentLng, lastLat, lastLng);
    if (distanceMoved > 3.0) {
      lastLat = currentLat;
      lastLng = currentLng;
      lastTimestamp = getTimeString();

      updateLCD("Mine Detected", "Sending...");
      HTTPClient http;
      String json = "{";
      json += "\"latitude\":" + String(currentLat, 6) + ",";
      json += "\"longitude\":" + String(currentLng, 6) + ",";
      json += "\"timestamp\":\"" + lastTimestamp + "\"";
      json += "}";

      http.begin(firebaseHost + "/mines.json");
      http.addHeader("Content-Type", "application/json");
      int code = http.POST(json);
      http.end();

      updateLCD(code > 0 ? "Post Success" : "Post Failed", "");
      delay(2000);
    }
  }

  controlMotors();
  delay(100);
}
