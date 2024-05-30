# Disaster-preparedness-amd-management-actitvity


Flood Level.txt
import #include <LiquidCrystal.h>
// #include <LiquidCrystal_PCF8574.h>
#include <SoftwareSerial.h>
SoftwareSerial ard_node(2, 3);
// LiquidCrystal lcd(13, 12, 11, 10, 9, 8);
LiquidCrystal lcd(8, 9, 10, 11, 12, 13);
// LiquidCrystal_PCF8574 lcd(0x27);
#define splash splash1
#define gsm_g Serial
String smsmsg, text;
String usernum1 = "09443947483";
String usernum2 = "09963770185";
int sms_rst = 0;
#define buz A0
#define wat A2
#define wat2 A3
#define rel 6

int dis_t = 2;
int disp = dis_t;
int alert_Status;
int wat_r, wat2_r, wat_fn;
void setup() {
 // initialize serial communications at 9600 bps:
 pinMode(rel, OUTPUT);
 pinMode(buz, OUTPUT);
 digitalWrite(buz, LOW);
 digitalWrite(rel, LOW);
 pinMode(wat, INPUT);
 pinMode(wat2, INPUT);
 Serial.begin(115200);
 ard_node.begin(115200);
 LcDSet();
 delay(2000);
 lcd.clear();
 LcDSet();
 
 SendSmS(usernum1, "GSM Initialized!!!", "Sending SMS.........");
}

void LcDSet() {
 lcd.begin(16, 2);
 lcd.clear();
 // lcd.home();
 // lcd.setBacklight(1);
 splash(0, "Flood");
 splash(1, "Monitoring");
 delay(2000);
 lcd.clear();
}
void loop() {
 getSensor();
 setDisplay();
 delay(100);
}
void getSensor() {
 wat_r = analogRead(wat);
 wat2_r = analogRead(wat2);
 if (wat_r > 650) {
 wat_r = 650;
 }
 if (wat2_r > 700) {

 wat2_r = 700;
 }
 wat_fn = map(wat_r, 1, 650, 0, 50);
 if (wat_fn > 49) {
 wat_fn = map(wat2_r, 1, 700, 50, 100);
 }
 if (wat_fn > 60) {
 digitalWrite(buz, HIGH);
 splash(1, "Flood Alert ");
 } else {
 // digitalWrite(buz, LOW);
 splash(1, "");
 }
 if (wat_fn > 60) {
 alert(" Flood Alert ");
 digitalWrite(rel, HIGH);
 }
}
void setDisplay() {
 disp++;
 if (disp > dis_t) {
lcd.setCursor(0, 0);
 lcd.print("F: % ");
 lcd.setCursor(3, 0);
 lcd.print(wat_fn);
 ard_node.print(wat_fn);
 ard_node.print(",");
 disp = 0;
 }
}
void alert(String msg) {
 sms_rst += 1;
 if (sms_rst >= 130) {
 alert_Status = 0;
 sms_rst = 0;
 }
 if (alert_Status == 0) {
 smsmsg = "Emergency ";
 smsmsg += msg;
 smsmsg += String(wat_fn);
 smsmsg += "%";
 Serial.println(smsmsg);
 SendSmS(usernum1, smsmsg, "Sending SMS.........");
 SendSmS(usernum2, smsmsg, "Sending SMS.........");
 alert_Status = 1;
 }
}

Iot Node.txt




#include <ESP8266WiFi.h>
#include <SoftwareSerial.h>
#include "ThingSpeak.h"
SoftwareSerial ard_node(D5, D6); //red/org
int a, b;
// Flooddetection123
char ssid1[] = "Project";
char password1[] = "1234567890";
unsigned long channelID = 2466588;
const char* writeAPIKey = "EZZ5RN46THWFSSM1"; // write API key for your ThingSpeak Channel
const char* server = "api.thingspeak.com";
String th;
int wat_fn;
int tup = 20;
WiFiClient client;
36
void setup() {
 // put your setup code here, to run once:
 Serial.begin(115200);
 ard_node.begin(115200);
 ard_node.println("");
 delay(200);
 setupWifi();
 Serial.print("Connecting to ThingSpeak");
 ThingSpeak.begin(client);
}
void setupWifi() {
 Serial.println();
 Serial.println();
 Serial.print(ssid1);
 WiFi.begin(ssid1, password1);
 Serial.print("Connecting to ");
 delay(2000);
 if (WiFi.status() != WL_CONNECTED) {
 delay(800);
 Serial.print(".");
 Serial.println("");
 Serial.println("WiFi connected");
 Serial.println("IP address: ");
 Serial.println(WiFi.localIP());
 delay(1000);
 }

}
void loop() {
 getArd();
 UpdateServer();
 delay(50);
}
void getArd() {
 String SS = "";
 String SSs = "";
 while (ard_node.available() > 0) {
 char S = ard_node.read();
 SS += S;
 delay(5);
 }
 if (SS.length() > 0) {
 Serial.println(SS);
 String a = getSplitValue(SS, ',', 0);
 wat_fn = a.toInt();
 ard_node.flush();
 }
}

void UpdateServer() {
 tup++;
 if (tup >= 250) {
 if (client.connect(server, 80)) {
 ThingSpeak.setField(1, String(wat_fn));
 // write to the ThingSpeak channel
 int x = ThingSpeak.writeFields(channelID, writeAPIKey);
 if (x == 200) {
 Serial.println("Channel update successful.");
 } else {
 Serial.println("Problem updating channel. HTTP error code " + String(x));
 }
 }
 tup = 0;
 }
}
