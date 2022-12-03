#include <ATX2.h>	// ATX2 Board
extern volatile unsigned long timer0_millis;
unsigned long last_time = 0;
int sensorsValue[] = {5} ; // do not use 0 index
int lineV = 0;
int groundV = 0;
int Ref = 500;
int N = 4;
int motorSpeed;
int baseSpeed = 100;
int rightSpeed, leftSpeed;
int maxSpeed = 100;
int sum_error = 0;

// PID
int error = 0;
int pre_error = 0;
int Kp = 0;
int Kd = 0;
int Ki = 0;

bool B(int n) {
  if (n < Ref) { // is black
    return true;
  } else {
    return false;
  }
}

bool W(int n) {
  if (n >= Ref) { // is white
    return true;
  } else {
    return false;
  }
}

void Tl() {

  for (int i = 0; i < N; i++) {
    sensorsValue[i] = analog(i);
  }

  if ( W(sensorsValue[0]) && W(sensorsValue[1]) && W(sensorsValue[2]) && W(sensorsValue[3]) && B(sensorsValue[4]) ) {
    error = 4;
  } else if ( W(sensorsValue[0]) && W(sensorsValue[1]) && W(sensorsValue[2]) && B(sensorsValue[3]) && B(sensorsValue[4]) ) {
    error = 3;
  } else if ( W(sensorsValue[0]) && W(sensorsValue[1]) && W(sensorsValue[2]) && B(sensorsValue[3]) && W(sensorsValue[4]) ) {
    error = 2;
  } else if ( W(sensorsValue[0]) && W(sensorsValue[1]) && B(sensorsValue[2]) && B(sensorsValue[3]) && W(sensorsValue[4]) ) {
    error = 1;
  } else if ( W(sensorsValue[0]) && W(sensorsValue[1]) && B(sensorsValue[2]) && W(sensorsValue[3]) && W(sensorsValue[4]) ) {
    error = 0;
  } else if ( W(sensorsValue[0]) && B(sensorsValue[1]) && B(sensorsValue[2]) && W(sensorsValue[3]) && W(sensorsValue[4]) ) {
    error = -1;
  } else if ( W(sensorsValue[0]) && B(sensorsValue[1]) && W(sensorsValue[2]) && W(sensorsValue[3]) && W(sensorsValue[4]) ) {
    error = -2;
  } else if ( B(sensorsValue[0]) && B(sensorsValue[1]) && W(sensorsValue[2]) && W(sensorsValue[3]) && W(sensorsValue[4]) ) {
    error = -3;
  } else if ( B(sensorsValue[0]) && W(sensorsValue[1]) && W(sensorsValue[2]) && W(sensorsValue[3]) && W(sensorsValue[4]) ) {
    error = -4;
  }
  /// check WWWWW
  else if ( W(sensorsValue[0]) && W(sensorsValue[1]) && W(sensorsValue[2]) && W(sensorsValue[3]) && W(sensorsValue[4]) ) {
    error = pre_error;
  }
  motorSpeed = Kp * error + Kd * (error - pre_error) + Ki * (sum_error);
  leftSpeed = baseSpeed + motorSpeed;
  rightSpeed = baseSpeed - motorSpeed;

  if (leftSpeed > maxSpeed) leftSpeed = maxSpeed;
  if (rightSpeed > maxSpeed) rightSpeed = maxSpeed;

  if (leftSpeed < -maxSpeed) leftSpeed = -maxSpeed;
  if (rightSpeed < -maxSpeed) rightSpeed = -maxSpeed;

  motor(1, leftSpeed);
  motor(2, rightSpeed);

  pre_error = error;
  sum_error += error;
}

void setup()
{

}
void loop()
{

  Tl();
}
bool timer_robot(unsigned long period)
{
  if (millis() - last_time > period)
  {
    last_time = 0;
    timer0_millis = 0;
    return true;
  }
  return false;
}
void RunTimer(int motorSpeed, unsigned long period)
{
  timer0_millis = 0;
  while (true)
  {
    motorSpeed = Kp * error + Kd * (error - pre_error) + Ki * (sum_error);
    leftSpeed = baseSpeed + motorSpeed;
    rightSpeed = baseSpeed - motorSpeed;

    if (leftSpeed > maxSpeed) leftSpeed = maxSpeed;
    if (rightSpeed > maxSpeed) rightSpeed = maxSpeed;

    if (leftSpeed < -maxSpeed) leftSpeed = -maxSpeed;
    if (rightSpeed < -maxSpeed) rightSpeed = -maxSpeed;

    motor(1, leftSpeed);
    motor(2, rightSpeed);

    pre_error = error;
    sum_error += error;
    if (timer_robot(period))
    {
      break;
    }
  }
  AO();
}
