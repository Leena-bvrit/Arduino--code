# Arduino--code
#include <SD.h> //Load SD card library
#include<SPI.h> //Load SPI Library
#include <Wire.h>
#include "RTClib.h"

RTC_DS1307 RTC;
int chipSelect = 4; //chipSelect pin for the SD card Reader
File mySensorData; //Data object you will write your sesnor data to
 
int pulsePin = 0;                 // Pulse Sensor purple wire connected to analog pin 0
int blinkPin = 7;                // pin to blink led at each beat
int fadePin = 5;                  // pin to do fancy classy fading blink at each beat
int fadeRate = 0;                 // used to fade LED on with PWM on fadePin

volatile int BPM;                   // int that holds raw Analog in 0. updated every 2mS
volatile int Signal;                // holds the incoming raw data
volatile int IBI = 600;             // int that holds the time interval between beats! Must be seeded! 
volatile boolean Pulse = false;     // "True" when User's live heartbeat is detected. "False" when not a "live beat". 
volatile boolean QS = false;        // becomes true when Arduoino finds a beat.

static boolean serialVisual = false;   // Set to 'false' by Default.  Re-set to 'true' to see Arduino Serial Monitor ASCII Visual Pulse 

int val;
int tempPin = 2;

void setup(){
  pinMode(blinkPin,OUTPUT);         // pin that will blink to your heartbeat!
  pinMode(fadePin,OUTPUT);          // pin that will fade to your heartbeat!
  Serial.begin(9600);             // we agree to talk fast!
   Wire.begin();
    RTC.begin();
  interruptSetup();                 // sets up to read Pulse Sensor signal every 2mS
 
Serial.println("Initializing SD card...");

  if (!SD.begin(4)) {
    Serial.println("initialization failed!");
    return;
  }
  Serial.println("initialization done."); 
  //reading process start
  mySensorData= SD.open("PTData.txt");
  Serial.println("Printing the file PTData.txt");
  if (mySensorData) {
    Serial.println("PTData.txt:");

    // read from the file until there's nothing else in it:
    while (mySensorData.available()) {
      Serial.write(mySensorData.read());
    }
    // close the file:
    mySensorData.close();
  } else {
    // if the file didn't open, print an error:
    Serial.println("error opening test.txt");
  }
  //reading process send
}
void loop(){
   DateTime now = RTC.now();
    Serial.println("\nDate & Time: ");
    Serial.print(now.day(), DEC);
    Serial.print('/');
    Serial.print(now.month(), DEC);
    Serial.print('/');
    Serial.print(now.year(), DEC);
    Serial.print(' ');

     if (now.hour()<10)
    Serial.print('0');
    Serial.print(now.hour(), DEC);
    Serial.print(':');
     if (now.minute()<10)
    Serial.print('0');
    Serial.print(now.minute(), DEC);
    Serial.print(':');
    if (now.second()<10)
    Serial.print('0');
    Serial.print(now.second(), DEC);
    Serial.print("\n");

delay(1000);
    serialOutput() ;       
    
  if (QS == true){     //  A Heartbeat Was Found
                       // BPM and IBI have been Determined
                       // Quantified Self "QS" true when arduino finds a heartbeat
        digitalWrite(blinkPin,HIGH);     // Blink LED, we got a beat. 
        fadeRate = 255;         // Makes the LED Fade Effect Happen
                                // Set 'fadeRate' Variable to 255 to fade LED with pulse
        serialOutputWhenBeatHappens();   // A Beat Happened, Output that to serial.     
        
        QS = false;                      // reset the Quantified Self flag for next time    
       } 
      else { 

      digitalWrite(blinkPin,LOW);            // There is not beat, turn off pin 13 LED
      }
     
   ledFadeToBeat();                      // Makes the LED Fade Effect Happen 
  delay(20);                             //  take a break



 val = analogRead(tempPin);
  float mv = ( val/1024.0)*5000; 
  float cel = mv/10;
  float farh = (cel*9)/5 + 32;
  mySensorData = SD.open("PTData.txt", FILE_WRITE);
  
  if (mySensorData) {
Serial.print("TEMPERATURE = ");
Serial.print(farh);
Serial.print("*F");
Serial.println("");
Serial.println("Data logging in SD card");
delay(500); //Pause between readings.

//Writing data to SD card

mySensorData.println("BPM Reading:");
mySensorData.print(BPM);  //write temperature data to card                           
mySensorData.println("");
mySensorData.println("Temperature Reading:");
mySensorData.print(farh);  //write temperature data to card                           
mySensorData.println("");
mySensorData.close();//close the file
}
  

}



