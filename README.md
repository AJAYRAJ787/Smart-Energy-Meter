#include <ThingSpeak.h>
#include <WiFi.h>
#include "ThingSpeak.h"
// always include thingspeak header file after other
header files and custom macros
#include <WiFiClient.h>
#include <LiquidCrystal.h>
#include "EmonLib.h"
#define SECRET_SSID "abcdefg"
// replace MySSID with your WiFi network name
#define SECRET_PASS "12345678" // replace MyPassword with your WiFi password
#define SECRET_CH_ID 1787819
// replace 0000000 with your channel number
#define SECRET_WRITE_APIKEY "N7KR54GL7CW0495N"
// replace XYZ with your channel write API Key
char ssid[] = SECRET_SSID;
// your network SSID (name)
char pass[] = SECRET_PASS;
// your network password
unsigned long myChannelNumber = SECRET_CH_ID;
const char * myWriteAPIKey = SECRET_WRITE_APIKEY;
int keyIndex = 0;
// your network key Index number (needed only for
WEP)
WiFiClient client;
LiquidCrystal lcd(13, 12, 14, 27, 26, 25);
EnergyMonitor emon;
#define vCalibration 83.3
#define currCalibration 0.50
#define EXE_INTERVAL 20000
unsigned long LEM = 0;
// vairable to save the last executed time
float kWh = 0;
float V_RMS = 0;
float I_RMS = 0;
float APPARENT_POWER = 0;
unsigned long lastmillis = millis();
String myStatus = "";
void updateValues()
{
emon.calcVI(20, 2000);
kWh = kWh + emon.apparentPower * (millis() - lastmillis) / 3600000000.0;
yield();
Serial.print("Vrms: ");
Serial.print(emon.Vrms, 2);
Serial.print("V");
Serial.print("\tIrms: ");
Serial.print(emon.Irms, 4);
Serial.print("A");
Serial.print("\tPower: ");
Serial.print(emon.apparentPower, 4);
Serial.print("W");
Serial.print("\tkWh: ");
Serial.print(kWh, 5);
Serial.println("kWh");
lcd.clear();
lcd.setCursor(0, 0);
lcd.print("Vrms:");
lcd.print(emon.Vrms, 2);
lcd.print("V");
lcd.setCursor(0, 1);
lcd.print("Irms:");
lcd.print(emon.Irms, 4);
lcd.print("A");
delay(2500);
lcd.clear();
lcd.setCursor(0, 0);
lcd.print("Power:");
lcd.print(emon.apparentPower, 4);
lcd.print("W");
lcd.setCursor(0, 1);
lcd.print("kWh:");
lcd.print(kWh, 4);
lcd.print("W");
delay(2500);
lastmillis = millis();
V_RMS = emon.Vrms;
I_RMS = emon.Irms;
APPARENT_POWER = emon.apparentPower;
}
void setup()
{
Serial.begin(115200);
WiFi.mode(WIFI_STA);
ThingSpeak.begin(client);
// Initialize ThingSpeak
lcd.begin(16, 2);
emon.voltage(35, vCalibration, 1.7);
// Voltage: input pin, calibration, phase_shift
emon.current(34, currCalibration);
// Current: input pin, calibration.
lcd.setCursor(3, 0);
lcd.print("IoT Energy");
lcd.setCursor(5, 1);
lcd.print("Meter");
delay(3000);
lcd.clear();
}
void loop()
{
unsigned long cM = millis();
if (cM - LEM >= EXE_INTERVAL) {
LEM = cM;
// save the last executed time
if (WiFi.status() != WL_CONNECTED) {
Serial.print("Attempting to connect to SSID: ");
Serial.println(SECRET_SSID);
while (WiFi.status() != WL_CONNECTED) {
WiFi.begin(ssid, pass);
// Connect to WPA/WPA2 network. Change
this line if using open or WEP network
Serial.print(".");
delay(5000);
}
Serial.println("\nConnected.");
}
updateValues();
// set the fields with the values
ThingSpeak.setField(1, V_RMS);
ThingSpeak.setField(2, I_RMS);
ThingSpeak.setField(3, APPARENT_POWER);
ThingSpeak.setField(4, kWh);
int x = ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey);
if (x == 200) {
Serial.println("Channel update successful.");
}
else {
Serial.println("Problem updating channel. HTTP error code " + String(x));
}
}

