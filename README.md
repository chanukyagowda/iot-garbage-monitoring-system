# iot-garbage-monitoring-system
iot garbage monitoring system with gsm AND GPS


#include <LiquidCrystal.h>
#include <SoftwareSerial.h>
SoftwareSerial sim(2,3);
char phone_no[] = "------------"; // replace with your phone no.
String data[5];
#define DEBUG true
String state,timegps,latitude,longitude;
int _timeout;
String _buffer;
String number = "--------------";//replace with your phone no.
LiquidCrystal lcd(A0,A1,A2,A3,A4,A5); 
#define trigPin 7
#define echoPin 6
#define led 13
#define led2 12
#define led3 11
#define led4 10
#define led5 9
#define led6 8

int sound = 250;


void setup() {
  lcd.begin(16,2); 
   delay(5000); //delay for 7 seconds to make sure the modules get the signal
  Serial.begin(9600);
  _buffer.reserve(50);
  lcd.setCursor(0,0); 
  lcd.print("System Started...");
  sim.begin(9600);
  delay(1000);
  Serial.begin (9600);
   sim.print("AT+CSMP=17,167,0,0");  // set this parameter if empty SMS received
 delay(100);
 sim.print("AT+CMGF=1\r"); 
 delay(400);
  
 sendData("AT+CGNSPWR=1",1000,DEBUG);
 delay(50);
 sendData("AT+CGNSSEQ=RMC",1000,DEBUG);
 delay(150);

  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(led, OUTPUT);
  pinMode(led2, OUTPUT);
  pinMode(led3, OUTPUT);
  pinMode(led4, OUTPUT);
  pinMode(led5, OUTPUT);
  pinMode(led6, OUTPUT);
  

}

void loop() {
  long duration, distance;
  digitalWrite(trigPin, LOW); 
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  duration = pulseIn(echoPin, HIGH);
  distance = (duration/2) / 29.1;


  if (distance <= 22) {
    digitalWrite(led, HIGH);
    sound = 250;
}
  else {
    digitalWrite(led,LOW);
  }
  if (distance < 17) {
      digitalWrite(led2, HIGH);
      sound = 260;
}
  else {
      digitalWrite(led2, LOW);
  }
  if (distance < 13) {
      digitalWrite(led3, HIGH);
      sound = 270;
} 
  else {
    digitalWrite(led3, LOW);
  }
  if (distance < 8) {
    digitalWrite(led4, HIGH);
    sound = 280;
}
  else {
    digitalWrite(led4,LOW);
  }
  if (distance < 6) {
    digitalWrite(led5, HIGH);
    sound = 290;
}
  else {
    digitalWrite(led5,LOW);
  }
  if (distance < 5) {
    digitalWrite(led6, HIGH);
    sound = 300;
}
  else {
    digitalWrite(led6,LOW);
  }
   int a=distance,n;
   if(a>=23){
    n=1;
   }
     if(a<=17){
    n=2;
   }
     if(a<=13){
    n=3;
   }
     if(a<=8){
    n=4;
   }
     if(a<=4){
    n=5;
   }
switch(n)
{
  case 1:
  lcd.setCursor(0,0); 
  lcd.print("dustbin is     ");
  lcd.setCursor(0,1);
  lcd.print("empty        ");
  break;

  case 2:
  lcd.setCursor(0,0); 
  lcd.print("dustbin is    ");
  lcd.setCursor(0,1);
  lcd.print("25% filled    ");
  break;

  case 3:
  lcd.setCursor(0,0); 
  lcd.print("dustbin is    ");
  lcd.setCursor(0,1);
  lcd.print("50% filled    ");
  break;

  case 4:
  lcd.setCursor(0,0); 
  lcd.print("dustbin is    ");
  lcd.setCursor(0,1);
  lcd.print("75% filled    ");
  break;


  case 5:
  lcd.setCursor(0,0); 
  lcd.print("dustbin is    ");
  lcd.setCursor(0,1);
  lcd.print("100% filled   ");
  gps_location();
  SendMessage();
  break;
  
 default:
  lcd.setCursor(0,0); 
  lcd.print("loading please");
  lcd.setCursor(0,1);
  lcd.print("wait          ");
  break;
}
 delay(500);
   if (sim.available() > 0)
    Serial.write(sim.read());
 
}
void SendMessage()
{
  //Serial.println ("Sending Message");
  sim.println("AT+CMGF=1");    //Sets the GSM Module in Text Mode
  delay(500);
  //Serial.println ("Set SMS Number");
  sim.println("AT+CMGS=\"" + number + "\"\r"); //Mobile phone number to send message
  delay(500);
  String SMS = "Chanukya your garbage is filled";
  sim.println(SMS);
  delay(100);
  sim.println((char)26);// ASCII code of CTRL+Z
  delay(500);
  _buffer = _readSerial();
}
String _readSerial() {
  _timeout = 0;
  while  (!sim.available() && _timeout < 12000  )
  {
    delay(13);
    _timeout++;
  }
  if (sim.available()) {
    return sim.readString();
  }
}
void gps_location()
{
    sendTabData("AT+CGNSINF",1000,DEBUG);
  if (state !=0) {
    Serial.println("State  :"+state);
    Serial.println("Time  :"+timegps);
    Serial.println("Latitude  :"+latitude);
    Serial.println("Longitude  :"+longitude);

    sim.print("AT+CMGS=\"");
    sim.print(phone_no);
    sim.println("\"");
    
    delay(300);

    sim.print("http://maps.google.com/maps?q=loc:");
    sim.print(latitude);
    sim.print(",");
    sim.print (longitude);
    delay(200);
    sim.println((char)26); // End AT command with a ^Z, ASCII code 26
    delay(200);
    sim.println();
    delay(20000);
    sim.flush();
    
  } else {
    Serial.println("GPS Initialising...");
  }
}

void sendTabData(String command , const int timeout , boolean debug){

  sim.println(command);
  long int time = millis();
  int i = 0;

  while((time+timeout) > millis()){
    while(sim.available()){
      char c = sim.read();
      if (c != ',') {
         data[i] +=c;
         delay(100);
      } else {
        i++;  
      }
      if (i == 5) {
        delay(100);
        goto exitL;
      }
    }
  }exitL:
  if (debug) {
    state = data[1];
    timegps = data[2];
    latitude = data[3];
    longitude =data[4];  
  }
}
String sendData (String command , const int timeout ,boolean debug){
  String response = "";
  sim.println(command);
  long int time = millis();
  int i = 0;

  while ( (time+timeout ) > millis()){
    while (sim.available()){
      char c = sim.read();
      response +=c;
    }
  }
  if (debug) {
     Serial.print(response);
     }
     return response;

}

