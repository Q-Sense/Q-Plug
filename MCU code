#include <NRF24.h>
#include <SPI.h>
#include <RFclient.h>
#include <EEPROM.h>



RFclient wifi;

const int voltSens = A5;
const double curSens_1 = A6;
const double curSens_2 = A0;

const int relay_1 = A4;
const int relay_2 = A3;
const int intResis = A2;

const int blueLED = 7;
const int greenLED = 6;
const int redLED = 5;
const int detectSW = 9;
const int btn = 2;


double curSenVal_1 = 0;
double curSenVal_2 = 0;
double dataArray[100];
double dataArray2[100];
double calc1 = 0;
double calc2 = 0;
double sumcalc1 = 0;
double sumcalc2 = 0;


int i, j = 0;
int mincur = 600;
int mincur2 = 600;
int tempMin = 1023;
int voltSensVal, minVolt = 0;
int SUM = 0;
int data = 0;
int tempres = 0;
int watts = 0;
int powLimit = 0;
int powLimCount;

uint8_t command;
boolean hasError = false;

void setup() {

  Serial.begin(9600);

  if ( wifi.initialization() )
    Serial.println("Init successful");
  else Serial.println("Init failed");
  pinMode(A3, OUTPUT);
  pinMode(A4, OUTPUT);
  pinMode(13, OUTPUT);
  pinMode(7, OUTPUT);
  pinMode(6, OUTPUT);
  pinMode(5, OUTPUT);
  pinMode(9, INPUT);
  pinMode(2, INPUT);
  pinMode(A2, INPUT);
  digitalWrite(9, HIGH);
  digitalWrite(2, HIGH);
  digitalWrite(A2, LOW);
  delay(200);


  i = 0;
  do {
    curSenVal_1 = analogRead(curSens_1);
    curSenVal_2 = analogRead(curSens_2);
    if (mincur > curSenVal_1) mincur = curSenVal_1;
    if (mincur2 > curSenVal_2) mincur2 = curSenVal_2;
    delay(0.1);
    i++;
  } while (i < 1000);

  // powLimit = 60;
  // writeEP();
  delay(100);
  readEP();
  Serial.println("Power Limit: ");
  Serial.println(powLimit);
}


void loop() {

  if (wifi.available())
  {
    wifi.recvPacket(&command, &data);
    Serial.print("Data: ");
    Serial.println(data);

    Serial.print("Command: ");
    Serial.print(command, HEX);
    checkMessage();
  }
  if ((digitalRead(detectSW) == LOW) && hasError == false)
    if ((measIntRes() == false) || (hasError == true)) errorState();
    else do {
        if (digitalRead(relay_1) == LOW)
          goState();
        measStartVolt();
        measCurr();
        powLimCheck();
        Serial.print(watts); Serial.println(" W");
        wifi.sendPacket(0x02, watts, "12345");
        if (digitalRead(btn) == LOW) btnPress();
      } while ((digitalRead(detectSW) == LOW) && (hasError == false));
  if (digitalRead(detectSW) == HIGH) offState();
  if (digitalRead(btn) == LOW) btnPress();
}


boolean leakCurrCheck() {
  double difference = 0;
  if (sumcalc1 < 5.0) {
    if ((sumcalc1 > sumcalc2)) {
      difference = sumcalc1 - sumcalc2;
    }
    else {
      difference = sumcalc2 - sumcalc1;
    }
    if (difference > 0.20) {
      errorState();
      wifi.sendPacket(0x08, difference, "12345");
      Serial.print("LEAK!!!");
    }
  }
}

boolean powLimCheck() {
  if (watts > powLimit) {
    powLimCount ++;
    if (powLimCount > 2) {
      errorState();
      // wifi.sendPacket(0x07, watts, "12345");
      Serial.println("POWER!!!");
    }
  }
  else  {
    powLimCount = 0;
  }
}


boolean btnPress() {
  if (hasError == HIGH) {
    hasError = LOW;
    digitalWrite(redLED, LOW);
  }
  else
    lampState();
}

boolean measIntRes() {
  delay (500);
  tempres = 0;
  for (int i = 0; i < 5; i++) {
    tempres += analogRead(intResis);
    //Serial.println(tempres);
    delay(300);
  }
  if (((tempres / 3) < 10) && digitalRead(detectSW) == LOW)
    return true;
  else
    return false;
}

boolean checkMessage() {
  if (command == 0x01) {
    if ((digitalRead(detectSW) == LOW) && hasError == false) {
      wifi.sendPacket(0x01, 1, "12345");
    }
  }
  else {
  }
  if (command == 0x02) {
    wifi.sendPacket(0x02, watts, "12345");
    Serial.println("02");
  }
  if (command == 0x03) {
    if (data >= 0 && data <= 3500 ) {
      powLimit = data;
      writeEP();
      Serial.println("03");
    }
  }
  if (command == 0x04) {
    readEP();
    wifi.sendPacket(0x04, powLimit , "12345");
    Serial.println("04");
  }
  if (command == 0x05) {
    if ((digitalRead(detectSW) == LOW) && hasError == false && digitalRead(relay_1) == LOW) {
      goState();
      wifi.sendPacket(0x01, 1, "12345");
    }
    else {
      offState();
      wifi.sendPacket(0x01, 0, "12345");
    }
    Serial.println("05");
  }
  if (command == 0x06) {
    lampState();
    Serial.println("06");
  }
  if (command == 0x85) {
    errorState();
    Serial.println("85");
  }
}


void errorState() {
  digitalWrite(relay_1, LOW);
  digitalWrite(relay_2, LOW);
  digitalWrite(greenLED, LOW);
  digitalWrite(blueLED, LOW);
  digitalWrite(redLED, HIGH);
  hasError = true;
}

void goState() {
  softStart();
  digitalWrite(relay_1, HIGH);
  digitalWrite(relay_2, HIGH);
  delay(300);
  digitalWrite(greenLED, HIGH);
  digitalWrite(blueLED, LOW);
}

void softStart() {
  do {

  } while (analogRead(voltSens) > minVolt + 10);
  do {
  } while (analogRead(voltSens) < 1023);
  delay(6);
}


void offState() {
  digitalWrite(relay_1, LOW);
  digitalWrite(relay_2, LOW);
  digitalWrite(greenLED, LOW);
  digitalWrite(redLED, LOW);
  hasError = false;
  delay (200);
}


void lampState() {

  if (hasError == false) {
    if (digitalRead(blueLED) == LOW)
    {
      digitalWrite(blueLED, HIGH);
    }
    else if (digitalRead(blueLED) == HIGH && digitalRead(greenLED) == HIGH) {
      digitalWrite(greenLED, LOW);
      //digitalWrite(blueLED, LOW);
    }
    else if (digitalRead(blueLED) == HIGH && digitalRead(greenLED) == LOW) {
      digitalWrite(greenLED, LOW);
      digitalWrite(blueLED, LOW);
    }
  }


  else {
    digitalWrite(blueLED, LOW);
    //    digitalWrite(greenLED, HIGH);
    //    digitalWrite(redLED, HIGH);
  }
  do {
  } while (digitalRead(btn) == LOW);
  delay(500);
}

void measStartVolt() {
  do {
    voltSensVal = analogRead(voltSens);
  } while (voltSensVal < 500);
  do {
    voltSensVal = analogRead(voltSens);
  } while (voltSensVal > 140);
}


void measCurr() {
  do {
    voltSensVal = 0;
    calc1 = 600;
    i = 0;
    do {
      voltSensVal = analogRead(voltSens);
      //Serial.println(voltSensVal);
      delay(0.1);
    } while (voltSensVal > 140);
    i = 0;
    while (i < 30 || voltSensVal > 140) {

      voltSensVal = analogRead(voltSens);
      dataArray[i] = analogRead(curSens_1);
      dataArray2[i] = analogRead(curSens_2);
      i++;
      delay(0.1);
    }
    j = i;
    do {
      j--;
      //517
      dataArray[j] = abs(dataArray[j]) - 516;
      dataArray2[j] = abs(dataArray2[j]) - 518;
    } while (j != 0);

    calc1 = 0;
    j = i;
    do {
      j--;
      dataArray[j] = sq(dataArray[j]);
      dataArray2[j] = sq(dataArray2[j]);
      calc1 += dataArray[j];
      calc2 += dataArray2[j];
    } while (j != 0);

    calc1 = sqrt(calc1) / i;
    calc2 = sqrt(calc2) / i;
    double temp = (calc1 / 0.025) - 10;
    sumcalc1 += calc1;
    sumcalc2 += calc2;

    SUM++;
    if (SUM >= 10) {
      sumcalc1 = sumcalc1 / SUM;
      sumcalc2 = sumcalc2 / SUM;
      if (sumcalc1 > 0.15)
        watts = sumcalc1 / 0.04;
      else watts = 0;
      SUM = 0;
    }
  } while (SUM != 0 );
}


void readEP() {
  int i = 0;
  powLimit = 0;
  do {
    powLimit = powLimit + EEPROM.read(i);
    i++;
    delay(100);
  } while (EEPROM.read(i) != 0);
}


void writeEP() {
  int i = 0 ;
  do {
    EEPROM.write(i, 0);
    i++;
  } while (EEPROM.read(i) != 0);
  if (powLimit > 255) {
    int divider = powLimit / 255;
    for ( i = 0; i <= divider - 1; i++)
      EEPROM.write(i, 255);
    EEPROM.write(i, (powLimit - (divider * 255)));
  }
  else
    EEPROM.write(0, powLimit);
}
