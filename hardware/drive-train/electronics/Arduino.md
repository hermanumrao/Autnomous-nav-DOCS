```c++
#include <IBusBM.h>
#include <Servo.h>

// --- RC setup ---
IBusBM ibus;

// --- Motor Pins (BTS7960) ---
const int RPWM = 6;
const int LPWM = 5;
const int R_EN = 4;
const int L_EN = 3;

// --- Servo Pins ---
const int SERVO1_PIN = 8;
const int SERVO2_PIN = 9;
Servo servo1;
Servo servo2;

const int SERVO_DEADBAND = 50;
int lastServo1 = 1500;
int lastServo2 = 1500;

// --- Heartbeat Watchdog ---
unsigned long lastCommandTime = 0;
const unsigned long timeoutMs = 1000;   // stop motors if no command for 1s

// --- Mode control ---
bool rosMode = false; // false = RC mode, true = ROS mode

void setup() {
  Serial.begin(115200);    // ROS serial
  Serial1.begin(115200);   // IBUS serial (use Serial1 on Arduino Mega / hardware UART)
  ibus.begin(Serial1);

  pinMode(RPWM, OUTPUT);
  pinMode(LPWM, OUTPUT);
  pinMode(R_EN, OUTPUT);
  pinMode(L_EN, OUTPUT);

  digitalWrite(R_EN, HIGH);
  digitalWrite(L_EN, HIGH);

  servo1.attach(SERVO1_PIN);
  servo2.attach(SERVO2_PIN);

  stopMotor();
  lastCommandTime = millis();
  Serial.println("BTS7960 + Servo + RC/ROS Switch Ready");
}

void loop() {
  ibus.loop();

  // --- Check CH5 for mode switch ---
  int ch5 = ibus.readChannel(4); // CH5 index=4
  rosMode = (ch5 > 1500);        // >1500 µs = ROS mode, else RC mode

  if (rosMode) {
    handleROS();
  } else {
    handleRC();
  }

  // --- Heartbeat Timeout Check (only for ROS mode) ---
  if (rosMode && (millis() - lastCommandTime > timeoutMs)) {
    stopMotor();
    Serial.println("Motor Stopped (watchdog timeout)");
    lastCommandTime = millis();  // reset to avoid spamming
  }
}

// ================= ROS MODE =================
void handleROS() {
  if (Serial.available() > 0) {
    String command = Serial.readStringUntil('\n');
    command.trim();

    if (command.startsWith("F")) {
      int speed = command.substring(1).toInt();
      forwardMotor(speed);
      Serial.print("Forward, Speed: ");
      Serial.println(speed);
    }
    else if (command.startsWith("B")) {
      int speed = command.substring(1).toInt();
      backwardMotor(speed);
      Serial.print("Backward, Speed: ");
      Serial.println(speed);
    }
    else if (command == "STOP") {
      stopMotor();
      Serial.println("Motor Stopped (STOP cmd)");
    }
    else if (command == "HEARTBEAT") {
      Serial.println("Heartbeat OK");
    }
    else if (command.startsWith("SERVO1")) {
      int angle = command.substring(6).toInt();
      angle = constrain(angle, 0, 180);
      servo1.write(angle);
      Serial.print("Servo1 angle: ");
      Serial.println(angle);
    }
    else if (command.startsWith("SERVO2")) {
      int angle = command.substring(6).toInt();
      angle = constrain(angle, 0, 180);
      servo2.write(angle);
      Serial.print("Servo2 angle: ");
      Serial.println(angle);
    }

    lastCommandTime = millis();  // reset watchdog
  }
}

// ================= RC MODE =================
void handleRC() {
  // CH3 -> Power
  int powerRaw = ibus.readChannel(2);
  int power = map(powerRaw, 1000, 2000, 0, 255);

  // CH2 -> Direction
  int dirRaw = ibus.readChannel(1);
  int direction = map(dirRaw, 1000, 2000, -3, 3);

  int pwm = (power > 0) ? (power * direction / 3) : 0;

  if (pwm != 0) {
    driveMotor(pwm);
  } else {
    stopMotor();
  }

  updateServo1();
  updateServo2();
}

// ================= MOTOR CONTROL =================
void forwardMotor(int speed) {
  speed = constrain(speed, 0, 255);
  analogWrite(RPWM, 0);
  analogWrite(LPWM, speed);
}

void backwardMotor(int speed) {
  speed = constrain(speed, 0, 255);
  analogWrite(LPWM, 0);
  analogWrite(RPWM, speed);
}

void driveMotor(int pwm) {
  if (pwm > 0) {
    analogWrite(RPWM, pwm);
    analogWrite(LPWM, 0);
  } else if (pwm < 0) {
    analogWrite(RPWM, 0);
    analogWrite(LPWM, -pwm);
  } else {
    stopMotor();
  }
}

void stopMotor() {
  analogWrite(RPWM, 0);
  analogWrite(LPWM, 0);
}

// ================= SERVO CONTROL =================
void updateServo1() {
  int ch1Raw = ibus.readChannel(0); // CH1
  if (abs(ch1Raw - lastServo2) > SERVO_DEADBAND) {
    servo2.writeMicroseconds(ch1Raw);
    lastServo2 = ch1Raw;
  }
}

void updateServo2() {
  int ch4Raw = ibus.readChannel(3); // CH4
  if (abs(ch4Raw - lastServo1) > SERVO_DEADBAND) {
    servo1.writeMicroseconds(ch4Raw);
    lastServo1 = ch4Raw;
  }
}

```

Burn this code
youshould know the methods
for understanding refer [circuitry](circuitry.md)
