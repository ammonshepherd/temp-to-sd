# Temperature to SD card

This sketch takes a temperature reading every X milliseconds and stores the
Celsius and Fahrenheit degrees on an SD card.

## Physical Setup

The physical setup is like this, but with a Wifi shield, or other shield with SD
card reader, stacked on top.

![temperature](temperature-tmp36.png)

(image source: https://github.com/rwaldron/johnny-five/blob/master/docs/temperature-tmp36.md)

## Hardware

- Arduino Uno
- Wifi Shield or other shield with an SD card reader.
  - This sketch uses a Wifi shield with SD card. This one uses the Adafruit
    CC3000 shield. 
  - Instructions for installation here: https://learn.adafruit.com/adafruit-cc3000-wifi/cc3000-shield
  - Tips for installing male headers or stackable headers: https://learn.sparkfun.com/tutorials/arduino-shields#installing-headers-preparation
- SD card
- Temperature sensor
  - https://www.sparkfun.com/products/10988
- Battery pack
  - This set up works well with an external battery pack.
  - https://www.adafruit.com/products/248?&main_page=product_info&products_id=248

## Code

```c++
// SPI and SD libraries. SPI for connecting SD card to SPI bus.
#include <SPI.h>
#include <SD.h>
const int sdPin = 4;

// Temperature pin set to analog 0
const int temPin = 0;

// Delay time. How often to take a temperature reading, in miliseconds
// 20 minutes = 1200000 milliseconds
const int delayTime = 1200000;

// File variable
File tempsFile;



void setup() {
  // Serial output for when connected to computer
  Serial.begin(9600);
  while (!Serial) {
    ; // wait for serial port to connect. Needed for native USB port only
  }
  
  Serial.print("Initializing SD card...");
  if(!SD.begin(sdPin)) {
    Serial.println("initialization failed!");
    return;
  }
  Serial.println("Initialization done.");

  tempsFile = SD.open("temps.txt", FILE_WRITE);
  
  if (tempsFile) {
    Serial.println("Printing temperatures");
    tempsFile.println("Printing temperatures:");
    tempsFile.close();
    Serial.println("Done.");
  } else {
    Serial.println("Error opening file in setup.");
  }
  
}

void loop() {
  /********************/
  // Open SD card for writing
  tempsFile = SD.open("temps.txt", FILE_WRITE);
  
  if (tempsFile) {
    // Temperature readings
    float voltage, degreesC, degreesF;
    voltage = getVoltage(temPin);
    degreesC = (voltage - 0.5) * 100.0;
    degreesF = degreesC * (9.0/5.0) + 32.0;
    
    // write temps to Serial
    Serial.print("Celsius: ");
    Serial.print(degreesC);
    Serial.print(" Fahrenheit: ");
    Serial.println(degreesF);
    
    // write temps to SD card
    tempsFile.print("Celsius: ");
    tempsFile.print(degreesC);
    tempsFile.print(" Fahrenheit: ");
    tempsFile.println(degreesF);
    
    // close the file
    tempsFile.close();
  } else {
    Serial.println("Error opening file in loop.");
  }
  

  delay(delayTime);
  
}

float getVoltage(int pin)
{
  return (analogRead(pin) * 0.004882814);
}
```
