# this is for the code with the icloud
//#include <Wire.h>
//#include "rgb_lcd.h"
#include "IotHttpClient.h"
#include "IotUtils.h"

#include <LWiFi.h>
#include <string>

// ********************************************
// ****** START user defined definitions ******
// ********************************************
#define WIFI_SSID                       "mawchai"
#define WIFI_PASSWORD                   "terrorblade"
// ******************************************
// ****** END user defined definitions ******
// ******************************************

//#define DELAY 60000

//rgb_lcd lcd;

//const int colorR = 0;
//const int colorG = 255;
//const int colorB = 0;

// ******************************************
// ****** setup()****************************
// ******************************************

#include <Wire.h>
#include "rgb_lcd.h"

//connect Touch to digital pin ?
#define pinTouch 2
//connect LED to digital pin ?
#define pinLed 3
#define pinBuzzer 8


rgb_lcd lcd;
const int colorR = 0;
const int colorG = 255;
const int colorB = 0;

int length = 15;         /* the number of notes */
char notes[] = "ccggaagffeeddc ";
int beats[] = { 1, 1, 1, 1, 1, 1, 2, 1, 1, 1, 1, 1, 1, 2, 4 };
int tempo = 300;

  
void setup()
{
    Serial.begin(9600);
    LWiFi.begin();
    delay(6000);
    
     lcd.setRGB(colorR, colorG, colorB);
  Serial.begin(9600);
  pinMode(pinTouch, INPUT);
  pinMode(pinLed, OUTPUT);
  pinMode(pinBuzzer, OUTPUT);
  lcd.begin(16, 1);
    
     int state = digitalRead(pinTouch);
  if (state == 1) {
    lcd.begin(16, 1);
    lcd.display();
    lcd.print("HELP CUSTOMER");
    delay(100);
    Serial.println("Touch detected");
    digitalWrite(pinLed, HIGH);
    delay(100);
    lcd.setRGB(255, 0, 0);
  }
   else {
    lcd.begin(16, 1);
    lcd.display();
    Serial.println("Touch not detected");
    digitalWrite(pinLed, LOW);
    lcd.setRGB(0, 255, 0);
  }

 
  if (state == 1) {
    Serial.println("Touch detected");
    digitalWrite(pinLed, HIGH);
    
      for(int i = 0; i < length; i++) {
        if(notes[i] == ' ') {
            delay(beats[i] * tempo);
        } else {
            playNote(notes[i], beats[i] * tempo);
        }
        delay(tempo / 2);    /* delay between notes */
    }
    
    delay(100);
  }
  else {
    Serial.println("Touch not detected");
    digitalWrite(pinLed, LOW);
    digitalWrite(pinBuzzer, LOW);
  }
  
}

/* play tone */
void playTone(int tone, int duration) {
  for (long i = 0; i < duration * 1000L; i += tone * 2) {
    digitalWrite(pinBuzzer, HIGH);
    delayMicroseconds(tone);
    digitalWrite(pinBuzzer, LOW);
    delayMicroseconds(tone);
  }
}
void playNote(char note, int duration) {
  char names[] = { 'c', 'd', 'e', 'f', 'g', 'a', 'b', 'C' };
  int tones[] = { 1915, 1700, 1519, 1432, 1275, 1136, 1014, 956 };
  // play the tone corresponding to the note name
  for (int i = 0; i < 8; i++) {
    if (names[i] == note) {
      playTone(tones[i], duration);
     
  }
  
  }
    
}

// ******************************************
// ****** loop() ****************************
// ******************************************
void loop()
{
    Serial.print("\nSearching for Wifi AP...\n");

    if ( LWiFi.connectWPA(WIFI_SSID, WIFI_PASSWORD) != 1 )
    {
        Serial.println("Failed to Connect to Wifi.");
    }
    else
    {
        Serial.println("Connected to WiFi");
        
     //   lcd.setCursor(0, 0);
       // lcd.print("Connected WiFi");
        
        // Generate some random data to send to Azure cloud.
        srand(vm_ust_get_current_time());

        int device_id = 1 + (rand() % 50);
        int temperature = (rand() % 100);

        // Construct a JSON data string using the random data.
        std::string json_iot_data;
        
        json_iot_data += "{ \"DeviceId\":" + IotUtils::IntToString(device_id);
        json_iot_data += ", \"Temperature\":" + IotUtils::IntToString(temperature);
        json_iot_data += " }";

        // Send the JSON data to the Azure cloud and get the HTTP status code.
        IotHttpClient     https_client;
        
        int http_code = https_client.SendAzureHttpsData(json_iot_data);

        Serial.println("Print HTTP Code:" + String(http_code));
        //lcd.setCursor(0, 1);
        //lcd.print("Code:" + String(http_code));
    }
  //  Serial.println("Disconneting HTTP connection");
    LWiFi.disconnect();
    
    // Sleeps for a while...
    delay(60000);
}
and 
#ifndef IOTHTTPCLIENT_H
#define IOTHTTPCLIENT_H

#include <arduino.h>
#include <LWiFi.h>
#include <LWiFiClient.h>

#include <string>

// ********************************************
// ****** START user defined definitions ******
// ********************************************
#define AZURE_SERVICE_BUS_NAME_SPACE    "mawchaidevices"
#define AZURE_EVENT_HUB_NAME            "ehdevices"
#define AZURE_POLICY_NAME               "RootManageSharedAccessKey"

//#define AZURE_KEY                       "AhBB/d6/tJj/awnV9MPgj1UzXjeboHNzszF5ShcD2FY="
#define AZURE_KEY                       "XC494RUnsL6fx9epZQfHOWDCisrPN2PBc+PypOBFxho="



// ******************************************
// ****** END user defined definitions ******
// ******************************************

#define AZURE_HOST                      AZURE_SERVICE_BUS_NAME_SPACE".servicebus.windows.net"
#define AZURE_URL                       "/"AZURE_EVENT_HUB_NAME"/messages"

// Set to year 2020 so that it will not expire.

#define AZURE_UTC_2020_01_01            "1577836800"

/* HttpClient implementation to be used on the Mtk device. */
class IotHttpClient
{
    LWiFiClient client;
    
public:
    IotHttpClient();
    /* Send http request and return the response. */
    char* send(const char *request, const char* serverUrl, int port);
    int SendAzureHttpsData(std::string);
    
protected:
    char* sendHTTP(const char *request, const char* serverUrl, int port);
    char* sendHTTPS(const char *request, const char* serverUrl, int port);
    
public:
    static boolean sendHTTPS_remotecall(void*);
    static void sendHTTPS_ssl_callback(VMINT handle, VMINT event);
};

#endif
