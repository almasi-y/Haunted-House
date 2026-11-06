// Pins
const int ENA = 5, IN1 = 7, IN2 = 8;
const int ENB = 6, IN3 = 9, IN4 = 10;
const int TRIG = 3, ECHO = 2;

const int SPEED = 160;          // 0-255; below ~100 most motors stall
const int STOP_DISTANCE = 20;   // cm

void setup() {
  pinMode(ENA, OUTPUT); pinMode(ENB, OUTPUT);
  pinMode(IN1, OUTPUT); pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT); pinMode(IN4, OUTPUT);
  pinMode(TRIG, OUTPUT); pinMode(ECHO, INPUT);
  Serial.begin(9600);
}

long readDistanceCm() {
  digitalWrite(TRIG, LOW);  delayMicroseconds(2);
  digitalWrite(TRIG, HIGH); delayMicroseconds(10);   // 10 µs pulse starts a measurement
  digitalWrite(TRIG, LOW);
  long duration = pulseIn(ECHO, HIGH, 30000);        // timeout ~5 m
  if (duration == 0) return 999;                     // no echo = nothing close
  return duration * 0.034 / 2;                       // sound speed, there and back
}

void setMotors(int leftDir, int rightDir, int speed) {
  // dir: 1 = forward, -1 = reverse, 0 = stop
  digitalWrite(IN1, leftDir == 1);  digitalWrite(IN2, leftDir == -1);
  digitalWrite(IN3, rightDir == 1); digitalWrite(IN4, rightDir == -1);
  analogWrite(ENA, speed);
  analogWrite(ENB, speed);
}

void loop() {
  long d = readDistanceCm();
  Serial.println(d);

  if (d > STOP_DISTANCE) {
    setMotors(1, 1, SPEED);      // clear: drive forward
  } else {
    setMotors(0, 0, 0);          // obstacle: stop
    delay(200);
    setMotors(-1, -1, SPEED);    // back up
    delay(400);
    setMotors(1, -1, SPEED);     // spin in place to turn
    delay(450);
    setMotors(0, 0, 0);
    delay(100);
  }
  delay(50);
}
