//https://arduino.stackexchange.com/questions/49395/arduino-vending-machine-to-monitor-coin-slot-input-while-waiting-for-user-input

#include <Wire.h>
#include <LiquidCrystal_I2C.h> //LCD Library
#include <I2c_bv.h> //bv i2c library
#include <bv4627_I.h> //bv i2c relais 
#include <Arduino.h>
#include <SoftwareSerial.h>

SoftwareSerial mySerial(2,3); // RX, TX

// Constants

bool userSelect = false;
const int coinPin = 3;      // Defined as the receiving pin from the coin machine.
int coinState = 0;          // Determine the status of coinPin
volatile int credit = 0;    // Define a variable to display the credit.
float credits = 0;
float money = 0;            // Define a variable to display the money
volatile int pulses = 0;    // Define a variable to display the pulses.
volatile long timeLastPulse = 0;
int delayTime = 0;          // Define delay variable for number of onionskin feed duration
int resetPin = 12;          // Define reset pin for resetting after complete function
uint8_t address;            //Define adress for PCF8574

// Initializing PCF8574 IO
void WriteIo(uint8_t bits)
{
  Wire.beginTransmission(address);
  Wire.write(bits);
  Wire.endTransmission();
}
uint8_t ReadIo()
{  
  WriteIo(B11111111); // PCF8574 require us to set all outputs to 1 before doing a read.
  
  Wire.beginTransmission(address);
  Wire.requestFrom((int)address, 1); // Ask for 1 byte from slave
  uint8_t bits = Wire.read(); // read that one byte
  Wire.endTransmission();
  
  return bits;
}

//I2C configuration 
LiquidCrystal_I2C lcd(0x27, 2, 1, 0, 4, 5, 6, 7, 3, POSITIVE); // I2C Display
BV4627_I rly(0x32);

//CoinInterrupt counter function
void coinInterrupt() {

  // Each time a pulse is sent from the coin acceptor, interrupt main loop to add 1 cent and flip on the LED
  credits += 1;
  pulses += 1;
  }

//serving function
void shortSkinServe() {
  lcd.clear();
  lcd.setCursor(4, 0);
  lcd.print("FACH OEFFNET:");
  lcd.setCursor(0, 1);
  lcd.print("FACH 1");
  rly.on(1,1);
  delay(2250);
  rly.off(1,0);
  lcd.clear();
  lcd.setCursor(3, 0);
  lcd.print("Danke!");
  lcd.setCursor(0, 1);
  lcd.print("Besuche uns bald wieder!");
  delay(3000);
  digitalWrite(resetPin, LOW);
  credit = 0;
  credits = 0;
  userSelect = false;
}

/////////////////////
// VOID SETUP
/////////////////////
void setup() {
  address = 0x20; // define PCF8574 address
  Serial.begin(9600);  // Begin Serial Connection to PC
  
  // set up the LCD's number of columns and rows:
  lcd.begin(20,4);  // initialize the lcd for 20 chars 4 lines, turn on backlight
  lcd.print("Startvorgang...");
  
  // OLD Coin acceptor Pulse interrupt
  //attachInterrupt(digitalPinToInterrupt(coinPin), coinInterrupt, RISING);

  //NEW Coin Acceptor Serial Read
  mySerial.begin(9600);

  //Set Reset PIN as OUTPUT (needed?)
  pinMode(resetPin, OUTPUT);  
  
  // RESET i2c Relay
  for (int i=0; i<8; i++){
  rly.off(i,0);
  }

  // Clear LCD
  lcd.clear();
  lcd.print("fertig...");
  delay(1000);
}

/////////////////
// Main loop
/////////////////
void loop() {
//----- NEW Coin recognition ------
    int i;

      if (mySerial.available()>0) {
        i=mySerial.read();
        //ignore 255 values
        if (i != 255) {
          credits = credit + i ;
      money = credits / 2.0;
      String stringOne = "Kredite 1: ";
      String stringThree = stringOne + money;
      Serial.println(stringThree);
          }
        
 
  
  if (credits == 0) {//not yet running, there is no credit yet
    lcd.clear();
    lcd.print("   Tollense Honig");
    lcd.setCursor(0, 1);
    lcd.print("   aus der Region");    
    lcd.setCursor(20, 1);
    lcd.write("-[GELD   EINWERFEN]-");
    delay(1000);
    lcd.clear();
    lcd.print("4,50EUR = 500g Glas");
    lcd.setCursor(0, 1);
    lcd.write("2x2EUR und 1x50Cent");
    lcd.setCursor(20, 1);
    
    lcd.write("[Nicht ueberzahlen!]");
    Serial.println("Noch keine Einzahlung ... Warte");
    delay (2000);

    //----- OLD Coin recognition ------
    while (credits > 0 && userSelect == false) { //if coin inserted stay in servo enable loop
    
    
    // Output Display
    lcd.clear();            //Clear command all LCD screen
    lcd.print("Kredit:");   //Display LCD Display Message
    lcd.setCursor(11, 0);   //Set the cursor's starting position at position(8,0)
    lcd.print(money);     //Displays the value of variable "credit" on the LCD screen.
    lcd.setCursor(15, 0);   //Set the cursor's starting position at position(8,0)
    lcd.print("Cents");     //Display screen text LCD
    lcd.setCursor(0, 1);


     // Serielle Ausgabe
      String stringOne = "Kredite: ";
      String stringThree = stringOne + credits;
      Serial.println(stringThree);
      String stringPulse = "Pulse: ";
      String stringFour = stringPulse + pulses;
      Serial.println(stringFour);
      
      /*Serial.println("Kredit:\n");
      Serial.print(credits);
      Serial.println("Kredit Raw:\n");
      Serial.print(pulses);*/


      
      uint8_t bits = ReadIo(); 

      if (credits >= 5 && userSelect == false) {
      lcd.print("Bitte ein Fach ");
      lcd.setCursor(0, 2);
      lcd.print("auswaehlen und Taste Druecken.");
      lcd.setCursor(0, 3);
          
      //-- Don't do anything unless they press a switch --
      if ( bits != B11111111) // Unless they're all high...
      {
      //-- Find lowest pressed switch --
      for (byte bitIndex = 0; bitIndex < 8; bitIndex++)
      if (bitRead(bits, bitIndex) == 0) {
        userSelect = true;
          shortSkinServe();
        exit;
      }    
      }

      delay (500); // for while loop LCD refresh speed control

      }
    
    }
}
}
}
