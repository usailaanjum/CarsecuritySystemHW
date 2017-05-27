
// This #include statement was automatically added by the Particle IDE.
#include <LiquidCrystal.h>
#include <math.h>
// This #include statement was automatically added by the Particle IDE.
#include <Keypad_I2C.h>

// This #include statement was automatically added by the Particle IDE.
//#include <Keypad.h>

// This #include statement was automatically added by the Particle IDE.
#include <Adafruit_MCP23017.h>

#include <TinyGPS++/TinyGPS++.h>

//#include <SoftwareSerial.h>
/*
   This sample code demonstrates the normal use of a TinyGPS++ (TinyGPSPlus) object.
   It requires the use of SoftwareSerial, and assumes that you have a
   4800-baud serial GPS device hooked up on pins 4(rx) and 3(tx).
*/
//static const int RXPin = 4, TXPin = 3;
static const uint32_t GPSBaud = 9600;

// The TinyGPS++ object
TinyGPSPlus gps;

const char *password = "153841";

// The serial connection to the GPS device
//SoftwareSerial ss(RXPin, TXPin);

const byte ROWS = 4; //four rows
const byte COLS = 3; //three columns

char* I2CTYPE = "Adafruit_MCP23017";

char keys[ROWS][COLS] = {
  {'1','2','3'},
  {'4','5','6'},
  {'7','8','9'},
  {'*','0','#'}
};

byte rowPins[ROWS] = {3, 4, 5, 6}; //connect to the row pinouts of the keypad
byte colPins[COLS] = {0, 1, 2}; //connect to the column pinouts of the keypad

Keypad keypad = Keypad( makeKeymap(keys), rowPins, colPins, ROWS, COLS, I2CTYPE );

LiquidCrystal lcd(D2, D3, D4, D5, D6, D7);
char pin[21];
char txtmsg[80];
char serverval[50];
 volatile byte half_revolutions;
 double rpm;
 unsigned long told;
 double ms;
 int mcontrol;

enum state {WAITING, DONE,SMS} current_state = WAITING;
enum task {DISPLAY,PIN,ALERT} display_state = DISPLAY;

void pinEncrypt();
void sendSMS();
void motorControl(int input);
void measureRPM();
void magnet_detect(); 

int tries = 0;    




Timer tm (15000,pinEncrypt);
Timer msg(30000,sendSMS);


void setup()
{
      
  Serial.begin(115200);
  
pinMode(B1,OUTPUT);
pinMode(B2,OUTPUT);
pinMode(B3,OUTPUT);
pinMode(B4,OUTPUT);
pinMode(B5,OUTPUT);
pinMode(A4,OUTPUT);
   attachInterrupt(B0, magnet_detect, RISING);//Initialize the intterrupt pin (Arduino digital pin 2)
   half_revolutions = 0;
   ms = 0.0;
   rpm = 0;
   told = 0;
   mcontrol = 0;
  
  lcd.begin(20,4);
  lcd.clear();
 Serial1.begin(GPSBaud);
 Particle.variable("val", serverval);
 
 keypad.addEventListener(keypadEvent);
 tm.start();
   
   
}

void loop()
{
motorControl(mcontrol);
sprintf(serverval,"%f %f %f %.2f",gps.location.lat(),gps.location.lng(),rpm,ms);

check:
switch(display_state){
    case DISPLAY:
        lcd.setCursor(0, 0);
        lcd.print("Lat: ");
        if(gps.location.lat() >= 0 && gps.location.lng() < 0)
        {
            lcd.setCursor(6,0);
        }
        lcd.print(gps.location.lat(),6);
        lcd.setCursor(0, 1); 
        lcd.print("Lon: ");
        if(gps.location.lat() < 0 && gps.location.lng() >= 0)
        {
            lcd.setCursor(6,1);
        }
        lcd.print(gps.location.lng(),6);
        lcd.setCursor(0,2);
        if((fmod((((double)gps.time.hour()/24)+(double(-4)/double(24))),1)*24) < 10){
        lcd.print("0");}            
        lcd.print(double(fmod((((double)gps.time.hour()/24)+(double(-4)/double(24))),1)*24),0);
        lcd.print(":");
        if(gps.time.minute()<10){
        lcd.print("0");}
        lcd.print(double(gps.time.minute()),0);
        lcd.print(":");
        if(gps.time.second()<10){
        lcd.print("0");}
        lcd.print(gps.time.second(),10);
        lcd.setCursor(8,2);
        lcd.print(" ");
        
        
        lcd.setCursor(10,2);
        lcd.print(double(gps.date.month()),0);
        lcd.print("/");
        lcd.print(double(gps.date.day()),0);
        lcd.print("/");
        lcd.print(double(gps.date.year()),0);
        
        lcd.setCursor(0,3);
        lcd.print("RPM:");
        if(mcontrol == 0){
            lcd.setCursor(4,3);
            lcd.print("0");
        }
         if (half_revolutions >= 20) { 
        measureRPM();
        }
        break;
    case PIN:
        pintry:  
        lcd.clear();
        memset(&pin[0],0,sizeof(pin));
        lcd.print("Enter encryption pin");
        lcd.setCursor(0,1);
        Serial.println("Before all of that");
        while(current_state != DONE){
            delay(50);
            memset(&serverval[0],0,sizeof(serverval));
            sprintf(serverval,"%f %f %f %.2f",gps.location.lat(),gps.location.lng(),rpm,ms);
            Serial.println("Before Key");
            char key = keypad.getKey();
            Serial.println("After Key");
            switch(current_state) {
                case WAITING:
                    if (key != NO_KEY) {
                    strncat(pin,&key,1);    
                    lcd.print("*");
                   }
                   
                   break;
                case DONE:
                  lcd.setCursor(0,3);
                  if(strcmp(pin,password) != 0)
                  {
                      tries++;
                      if(tries < 3){
                      current_state = WAITING;
                      tm.reset();
                      goto pintry;
                      }
                      else{
                        mcontrol = 0;  
                        display_state = ALERT;              
                        tm.stop();
                        lcd.clear();
                        msg.start();
                        goto check;
                      }
                      
                  }
                  tries = 0;
                  mcontrol = 1;
                  
                  


                    break;
                  }  
                  
        }
        
        current_state = WAITING;
        display_state = DISPLAY;
        lcd.clear();
        tm.reset();
        break;
    case ALERT:
        
        lcd.setCursor(0, 0);
        lcd.print("Intruder Alert");
        if(current_state == SMS)
        {
            memset(&txtmsg[0],0,sizeof(txtmsg));
            msg.stop();
            
            sprintf(txtmsg,"SECURITY BREACH\n\nLat: %f\nLon: %f\n%02d:%02d:%02d EST %02d/%02d/%d",gps.location.lat(),gps.location.lng(),(int)(fmod((((double)gps.time.hour()/24)+(double(-4)/double(24))),1)*24),gps.time.minute(),gps.time.second(),gps.date.month(),gps.date.day(),gps.date.year());
            //Serial.print(txtmsg);
            Particle.publish("sms/4169487670", txtmsg, PRIVATE);
            delay(6000);
            lcd.clear();
            msg.start();
            current_state = WAITING;
            
        }
    break;
}


  
  smartDelay(50);
  
}

void motorControl(int input)
{
    if(input == 1){
        
    digitalWrite(B3,LOW);
    digitalWrite(B2,HIGH);
    analogWrite(B1,255);
    
    digitalWrite(B4,LOW);
    digitalWrite(B5,HIGH);
    analogWrite(A4,255);
    }
    else{
        
    digitalWrite(B3,LOW);
    digitalWrite(B2,HIGH);
    analogWrite(B1,0);
    
    digitalWrite(B4,LOW);
    digitalWrite(B5,HIGH);
    analogWrite(A4,0);
    }
}

void measureRPM()
{
     rpm = 30*1000/(millis() - told)*half_revolutions;
     ms = 0.10472*rpm*0.03;
     told = millis();
     half_revolutions = 0;
     lcd.setCursor(4, 3);
     lcd.print(rpm,0);
     delay(50);

}
 void magnet_detect()//This function is called whenever a magnet/interrupt is detected by the arduino
 {
   half_revolutions++;
  
 }


void pinEncrypt()
{
    display_state = PIN;
 
}
void sendSMS()
{
   current_state = SMS; 
}

void keypadEvent(KeypadEvent key){
  switch (keypad.getState()){
    case PRESSED:
      switch (key){
        case '#':
            current_state = DONE; break; // enter the function with "#"
        case '1': 
        case '2':
        case '3':
        case '4':
        case '5':
        case '6':
        case '7':
        case '8':
        case '9':
        case '0':
            current_state = WAITING; break; // exit the function with EXITKEY
      }
    break;

  }
}





static void smartDelay(unsigned long ms)
{
  unsigned long start = millis();
  do 
  {
    while (Serial1.available())
      gps.encode(Serial1.read());
  } while (millis() - start < ms);
}

static void printFloat(float val, bool valid, int len, int prec)
{
  if (!valid)
  {
    while (len-- > 1)
      Serial.print('*');
    Serial.print(' ');
  }
  else
  {
    Serial.print(val, prec);
    int vi = abs((int)val);
    int flen = prec + (val < 0.0 ? 2 : 1); // . and -
    flen += vi >= 1000 ? 4 : vi >= 100 ? 3 : vi >= 10 ? 2 : 1;
    for (int i=flen; i<len; ++i)
      Serial.print(' ');
  }
  smartDelay(0);
}

static void printInt(unsigned long val, bool valid, int len)
{
  char sz[32] = "*****************";
  if (valid)
    sprintf(sz, "%ld", val);
  sz[len] = 0;
  for (int i=strlen(sz); i<len; ++i)
    sz[i] = ' ';
  if (len > 0) 
    sz[len-1] = ' ';
  Serial.print(sz);
  smartDelay(0);
}

static void printDateTime(TinyGPSDate &d, TinyGPSTime &t)
{
  if (!d.isValid())
  {
    Serial.print(F("********** "));
  }
  else
  {
    char sz[32];
    sprintf(sz, "%02d/%02d/%02d ", d.month(), d.day(), d.year());
    Serial.print(sz);
  }
  
  if (!t.isValid())
  {
    Serial.print(F("******** "));
  }
  else
  {
    char sz[32];
    sprintf(sz, "%02d:%02d:%02d ", t.hour()-5, t.minute(), t.second());
    Serial.print(sz);
  }

  printInt(d.age(), d.isValid(), 5);
  smartDelay(0);
}

static void printStr(const char *str, int len)
{
  int slen = strlen(str);
  for (int i=0; i<len; ++i)
    Serial.print(i<slen ? str[i] : ' ');
  smartDelay(0);
}

