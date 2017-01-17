// libraries
    #include "WS2801/WS2801.h"
    #include "spark-dallas-temperature/spark-dallas-temperature.h"
    #include "OneWire/OneWire.h"
    #include "Arduino/Arduino.h"
    #include "application.h"


// Pins
    #define DEPTH A1                                // Analog pin 1 to measure the water depth
    //LED Clock (green) = A3                        // Analog Pin 3 to clock the LED strip
    //LED Data (yellow) = A5                        // Analog Pin 5 to send data to the LED strip
    #define ONE_WIRE_BUS D1                         // Digital Pin 1 to read the DS18S20 temperature
    int Fan = D3;                                   // Digital Pin 3 to relay on/off the Fan
    int Pump = D4;                                  // Digital Pin 4 to relay on/off water Pump
    int Valve = D5;                                 // Digital Pin 5 to relay on/off the refill water Valve

// Definitions
    double tempc;                                   // The displayed real-time Temperature
    double resistance;                              // The displayed waterdepth resistance; can comment out once set up
    double waterdepth;                              // The displayed waterdepth in percentage
    int LEDs;
    int iteration;                                  // Integer 0-99 to operate Water Pump based on % time (0-99 loops)
    boolean LEDOp = true;                           // Flag for feedback on whether or not the LED strip is operating
    boolean FanOp = false;                          // Flag for feedback on whether or not the Fan is operating
    boolean PumpOp = false;                         // Flag for feedback on whether or not the Pump is operating
    boolean ValveOp = false;                        // Flag for feedback on whether or not the Valve is operating
    OneWire oneWire(ONE_WIRE_BUS);                  // Setup a oneWire instance to communicate with Thermometer
    DallasTemperature sensors(&oneWire);            // Pass our oneWire reference to Dallas Temperature.
    #define SERIESRESISTOR 560                      // Depth meter resistor
    #define ZERO_VOLUME_RESISTANCE    298.00        // Resistance value (in ohms) when no liquid is present.
    #define CALIBRATION_RESISTANCE    1050.00       // Resistance value (in ohms) when liquid is at max line.
    #define CALIBRATION_VOLUME        100.00        // Volume (in any units) when liquid is at max line.
    const int numPixel = 11;                        // Number of LEDs on the Tower
    Adafruit_WS2801 strip = Adafruit_WS2801(numPixel);
 

void setup(void) {
    sensors.begin();                                // IC defaults to 9 bit. If you have trouble consider changing to 12. 
    Serial1.begin(9600);                            // initialize 16x2 LCD Display
        Serial1.write(0xFE);
        Serial1.write(0xD1);
        Serial1.write(16);                            // 16 columns
        Serial1.write(2);                             // 2 rows
      delay(10);
        Serial1.write(0xFE);
        Serial1.write(0x52);                          // autoscroll off
      delay(10);  
        Serial1.write(0xFE);    
        Serial1.write(0x50);
        Serial1.write(200);                           // Contrast
      delay(10);  
        Serial1.write(0xFE);    
        Serial1.write(0x99);
        Serial1.write(255);                           // Brightness
      delay(10);      
      
    Particle.variable("Resistance", &resistance, DOUBLE);// Temp Resistance exported to internet; for set up only, comment out otherwise
    Particle.variable("Depth", &waterdepth, DOUBLE);    // Water Depth exported to internet
    Particle.variable("Celsius", &tempc, DOUBLE);       // Temperature exported to internet  
    Particle.variable("Fan", &FanOp, BOOLEAN);          // Fan Operation exported to internet  
    Particle.variable("Rain Shower", &PumpOp, BOOLEAN); // Pump Operation exported to internet  
    Particle.variable("Refill", &ValveOp, BOOLEAN);     // Valve Operation exported to internet 
    Particle.variable("LEDs", &LEDOp, BOOLEAN);         // LED Operation exported to internet 

    pinMode(LEDs, OUTPUT);
        Particle.function("LEDs",ledToggle);
        
    pinMode(Fan, OUTPUT);
        Particle.function("Fan",fanToggle);
    
    pinMode(Pump, OUTPUT);
        Particle.function("Pump",pumpToggle);
    
    strip.begin();
        Particle.function("Color", color);
            color("[000,255,000]");
}                                           


void loop(void) {

// Start loop count for Water Pump On percentage time through 100% (0-99)
    if (iteration < 99) {
        
        
    // Measure the water depth
        float depthvalue = analogRead(DEPTH);       // convert depth reading to resistance
        depthvalue = ( depthvalue / 1023.0) - 1.0;
        resistance = SERIESRESISTOR / depthvalue;
        if ((ZERO_VOLUME_RESISTANCE - CALIBRATION_RESISTANCE) == 0.0) {
            waterdepth = 0.0;
            }
        else { 
            float scale = ((ZERO_VOLUME_RESISTANCE - resistance) / (ZERO_VOLUME_RESISTANCE - CALIBRATION_RESISTANCE));
            waterdepth = (CALIBRATION_VOLUME * scale);
            }
    
    // Measure the temperature of the water
        sensors.requestTemperatures();          // Send the command to get temperatures from all sensors on the one wire bus
        tempc = sensors.getTempCByIndex(0);     // 0 refers to the first IC on the wire
        while (tempc == -127.0) {               // if the anomalous -127C occurs, then get another temp reading without registering the -127
            tempc = sensors.getTempCByIndex(0);
        }
        
    // Display the temp and water level on the LCD Display
        Serial1.print("Temp ");   
        Serial1.print(tempc,1);
        Serial1.println("\337C");
        Serial1.print("Level ");   
        Serial1.print(waterdepth,0);
        Serial1.print("% "); 
        Serial1.println(iteration);

    
    // Flag the Fan on if temp over 24C, Flag off when below 20C
        if (tempc > 24.0) {
            FanOp = true;
        }    
        if (tempc < 20.0) {
            FanOp = false;
        }

    // Fan operation based off of flag FanOp
        if (FanOp) {
            digitalWrite(Fan,HIGH);
        }
        else {
            digitalWrite(Fan,LOW);
        }

    // Flag the Valve on if Waterlevel under 25%, Flag off when above 85%
        if (waterdepth < 25.0) {
            ValveOp = true;
        }    
        if (waterdepth > 85.0) {
            ValveOp = false;
        }
        
    // Valve control based off of flag ValveOp
        if (ValveOp) {
           digitalWrite(Valve,HIGH);
        }
        else {
            digitalWrite(Valve,LOW);
        }
        
    // Flag the Pump on if Iterations are = %, Flag off when not
        if (iteration < 33) {
            PumpOp = true;
        }
        else {
            PumpOp = false;
        }
        
    // Pump control based off of flag PumpOp
        if (PumpOp) {
            digitalWrite(Pump,HIGH);
        }
        else {
            digitalWrite(Pump,LOW);
        }

    // LED control based off of flag LEDOp
        if (LEDOp) {
        // Rainbow lights 
            int i, j;
            for (j=0; j < 256; j++) {     // 25 colors in the wheel
            for (i=0; i < strip.numPixels(); i++) {
          // tricky math! we use each pixel as a fraction of the full 96-color wheel
          // (thats the i / strip.numPixels() part)
          // Then add in j which makes the colors go around per pixel
          // the % 96 is to make the wheel cycle around
                strip.setPixelColor(i, Wheel( ((i * 256 / strip.numPixels()) + j) % 256) );
                Serial.println(j);
            }  
            strip.show();   // write all the pixels out
            delay(20);
            }            
        }
        else {
            color("[000,000,000]");
            delay(1000);
        }


    // Increment the iteration; if the count goes above 99, then reset to zero      
    iteration++;    
    }
    else {
        iteration = 0;
    }
    delay(500);
}



// Internet control of the LEDs
    int ledToggle(String command) {
        if (command=="on") {
            LEDOp = true;
        }
        else if (command=="off") {
            LEDOp = false;
        }
    }
    
// Internet control of the Pump
    int pumpToggle(String command) {
        if (command=="on") {
            PumpOp = true;
        }
        else if (command=="off") {
            PumpOp = false;
        }
    }
    
// Internet control of the Fan
    int fanToggle(String command) {
        if (command=="on") {
            FanOp = true;
        }
        else if (command=="off") {
            FanOp = false;
        }
    }
    
// Internet control of the LED Tower lights
    int color(String command) {
        int red = command.substring(1,4).toInt();
        int green = command.substring(5,8).toInt();
        int blue = command.substring(9,12).toInt();
        int i;
        for (i=0; i < strip.numPixels(); i++) {
            strip.setPixelColor(i, red, green, blue);
            strip.show();
            delay(50);
        }
    }

// Rainbow Cycle for LEDs function
    void rainbowCycle(uint8_t wait) {
        int i, j;
        for (j=0; j < 256 * 5; j++) {     // 5 cycles of all 25 colors in the wheel
        for (i=0; i < strip.numPixels(); i++) {
      // tricky math! we use each pixel as a fraction of the full 96-color wheel
      // (thats the i / strip.numPixels() part)
      // Then add in j which makes the colors go around per pixel
      // the % 96 is to make the wheel cycle around
            strip.setPixelColor(i, Wheel( ((i * 256 / strip.numPixels()) + j) % 256) );
            Serial.println(j);
        }  
        strip.show();   // write all the pixels out
        delay(wait);
        }
    }

    // Color sub-function for Rainbow cycle
        uint32_t Color(byte r, byte g, byte b) {
            uint32_t c;
            c = r;
            c <<= 8;
            c |= g;
            c <<= 8;
            c |= b;
            return c;
        }

    // Wheel sub-function for Rainbow Cycle
        uint32_t Wheel(byte WheelPos) {
            if (WheelPos < 85) {
                return Color(WheelPos * 3, 255 - WheelPos * 3, 0); } 
            else if (WheelPos < 170) {
                WheelPos -= 85;
                return Color(255 - WheelPos * 3, 0, WheelPos * 3); } 
            else {
                WheelPos -= 170; 
                return Color(0, WheelPos * 3, 255 - WheelPos * 3);}
        }
