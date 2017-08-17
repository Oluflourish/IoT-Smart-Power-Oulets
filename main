float Voltage1 = 0;
double VRMS1 = 0;
double AmpsRMS1 = 0;
double totalVRMS1 = 0; //Total voltage consumed
double totalAmpsRMS1 = 0; //Total current consumed

float Voltage2 = 0;
double VRMS2 = 0;
double AmpsRMS2 = 0;
double totalVRMS2 = 0; //Total voltage consumed
double totalAmpsRMS2 = 0; //Total current consumed

float Voltage3 = 0;
double VRMS3 = 0;
double AmpsRMS3 = 0;
double totalVRMS3 = 0; //Total voltage consumed
double totalAmpsRMS3 = 0; //Total current consumed

float Voltage4 = 0;
double VRMS4 = 0;
double AmpsRMS4 = 0;
double totalVRMS4 = 0; //Total voltage consumed
double totalAmpsRMS4 = 0; //Total current consumed

int mVperAmp = 185; // use 100 for 20A Module and 66 for 30A Module

// Caleibrate the current sensors
// Calibrate using the initial reading of each of the current sensors
// This is becuase the current is not stable due to variation in frquency of the ac source
const double calibrate1 = 0.00;
const double calibrate2 = 0.00;
const double calibrate3 = 0.04;
const double calibrate4 = 0.00;

//Power rating for each plan
const double powerSub1 = 0.06;  //Starter Plan
const double powerSub2 = 0.08;  //Premium Plan
const double powerSub3 = 0.20;  //Plus Plan
const double powerSub4 = 0.32;  //Pro Plus Plan

#include <SPI.h>
#include <Ethernet.h>
#include <SD.h>

#include <Key.h>
#include <Keypad.h>

#include "Timer.h"
#define relay_on 0
#define relay_off 1
Timer t;


#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 20, 4); // set the LCD address to 0x27 for a 16 chars and 2 line display

const byte numRows = 4; //number of rows on the keypad
const byte numCols = 4; //number of columns on the keypad
int count = 0;
//keymap defines the key pressed according to the row and columns just as appears on the keypad
char keymap[numRows][numCols] =
{
  {'1', '2', '3', 'A'},
  {'4', '5', '6', 'B'},
  {'7', '8', '9', 'C'},
  {'*', '0', '#', 'D'}
};

//Code that shows the the keypad connections to the arduino terminals
byte rowPins[numRows] = {32, 34, 36, 38}; //Rows 0 to 3
byte colPins[numCols] = {40, 42, 44, 46}; //Columns 0 to 3

//initializes an instance of the Keypad class
Keypad myKeypad = Keypad(makeKeymap(keymap), rowPins, colPins, numRows, numCols);

// it's a 16x2 LCD so...
int screenWidth = 16;
int screenHeight = 2;

// just some reference flags for LCD Scroll
int stringStart, stringStop = 0;
int scrollCursor = screenWidth;

File myFile;
char buf[10];
String nameOfFile = "test.txt";
String nameOfPowerFile = "asd.txt";
//int checkWebTime = 0.5 * 60 * 1000;   //In seconds
int checkWebTime = 20 * 1000;   //In seconds

char attempt[10] = {}; // used for comparison OR for Initialization
int z = 0;
unsigned int trial = 0;
String pinOutput = "";

unsigned int socket1 = 43;
unsigned int socket2 = 41;
unsigned int socket3 = 39;
unsigned int socket4 = 37;

//Relay Switch is Opposite that of a normal switch (0 => LOW, 1 => HIGH)
//int relay_on = 0;
//int relay_off = 1;

byte mac[] = {0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED}; //needs to be replaced with your own Ethernet shield mac address
//The following is based on your configuration of the DNS and the DHCP of the Windows Server 2003, you have to change the data below
//according to your configurations

//192,168,43,1:8080

IPAddress myserver {192, 168, 1, 100};
IPAddress ipAddress {192, 168, 1, 101};
IPAddress myDNS {8, 8, 8, 8};
IPAddress myGateway {192, 168, 1, 1};
IPAddress mySubnet {255, 255, 255, 0};

EthernetClient client;

void setup()
{
  Serial.begin(9600);

  //Set Sockets to OUTPUT PINS and HIGH(off for relays)
  for (int i = 37; i <= 43; i++) {
    pinMode(i, OUTPUT);
    digitalWrite(i, HIGH);
  }
  
  t.every(2000, getVPP1);
  t.every(2000, getVPP2);
  t.every(2000, getVPP3);
  t.every(2000, getVPP4);
  t.every(checkWebTime, getNewPIN);
  t.every(checkWebTime + 5000, getPower);

  Serial.print("Checks website database for New PINS every ");
  Serial.print(checkWebTime / 1000);
  Serial.print(" seconds");

  

  lcd.init();                      // initialize the lcd
  lcd.backlight();
  lcd.begin(screenWidth, screenHeight);
  //  keypad.addEventListener(keypadEvent);
  while (!Serial) {
    ; // wait for serial port to connect. Needed for Leonardo only
  }

  Ethernet.begin(mac, ipAddress, myDNS, myGateway, mySubnet);

  Serial.print("Initializing SD card... \n");

  if (!SD.begin(4)) {
    Serial.println("SD card Initialization Failed!");
    return;
  }

  Serial.println("SD card Initialization Done.");

  //if the function returns 1 is moves to enterPIN else if goes to the nested block
  if (myScroller("Project Title:", "SMART POWER OUTLETS WITH ONLINE MANAGEMENT SYSTEM") != 1) {
    if (myScroller("Project By:", "OGUNDIPE ABDUL-LATEEF[164690] & FAKOLUJO OLUWAPELUMI[164275]") != 1) {
      lcd.clear();
      lcd.print("Supervised By:");
      lcd.setCursor(0, 1);
      lcd.print("MR. O.O OLUFAJO");
      delay(5000);
      lcd.clear();
      enterPIN();
    } else {
      enterPIN();
    }
  } else {
    enterPIN();
  }

}


int myScroller(String line1, String line2) {
  int countMovement = 0;
  char key = myKeypad.getKey();
  for (int countMovement = 0; countMovement < (line2.length() + screenWidth ); countMovement++) {
    //for (int countMovement = 0; countMovement < (line2.length() + screenWidth) && (myKeypad.getKey() == NO_KEY); countMovement++) {
    //Check if any key is pressed on the keypad
    char key = myKeypad.getKey();
    switch (key)
    {
      case '#':
        return 1;
        break;
      case '*':
        return 2;
        break;
    }

    lcd.setCursor(scrollCursor, 1);
    lcd.print(line2.substring(stringStart, stringStop));
    lcd.setCursor(0, 0);
    lcd.print(line1);
    delay(300);
    lcd.clear();
    if (stringStart == 0 && scrollCursor > 0) {
      scrollCursor--;
      stringStop++;
    } else if (stringStart == stringStop) {
      stringStart = stringStop = 0;
      scrollCursor = screenWidth;
    } else if (stringStop == line2.length() && scrollCursor == 0) {
      stringStart++;
    } else {
      stringStart++;
      stringStop++;
    }
  }
}

//Accepts inputted PIN as a char array and check against the PIN file
void xCheck(char *attempt) {
  // re-open the file for reading:
  myFile = SD.open(nameOfFile);
  Serial.println(nameOfFile);

  if (myFile) {
    int pinMatch = 0; //Initialize the variable with zero

    // read from the file until there's nothing else in it:
    //loops through the pins in the sd card and stops on seeing a pin that matches
    while (myFile.available() && pinMatch == 0) {
      myFile.read(buf, 10);

      Serial.print("keyPadPIN :");
      Serial.println(atol(attempt));
      if (strncmp(&buf[0], attempt, 5) == 0)
      {
        pinMatch = 1;      // Assigns to pin to indicate that a correct pin has been found

        //Extracts the START TIME and DURATION
        String durationStr = String(buf).substring(5, 6);
        String socketPinStr = String(buf).substring(6, 7);
        String powerPlanStr = String(buf).substring(7, 8);

        //Converts the Strings to Integers
        int duration = durationStr.toInt();
        int socketNumber = socketPinStr.toInt();
        int powerPlan = powerPlanStr.toInt();

        String planName;
        switch (powerPlan)
        {
          case 1:
            planName = " Starter";
            break;
          case 2:
            planName = "Premium";
            break;
          case 3:
            planName = "Plus Plan";
            break;
          case 4:
            planName = "Pro Plus";
            break;
        }

        correctPIN(duration, socketNumber, planName);

        Serial.print("Duration: ");
        Serial.println(duration);
        Serial.print("Socket no.: ");
        Serial.println(socketNumber);
        Serial.print("Power Plan: ");
        Serial.println(planName);

        //Activates socket relay and timer for the specified duration
        String buffPIN = String(buf);
        onSocketWithTimer(socketNumber, duration, powerPlan, buffPIN);

        /*
           METHODS TO BE CALLED
           Split PIN input to extract Socket Port, Time and Duration
           Activate Circuitry for the selected port
           Write to another TEXT file to notify webServer of Login OR Probably communicate with the Webserver directly
        */
      } else {
        enterPIN();
      }
    }

    if (pinMatch == 1)
    {
      Serial.println("Match!");
      Serial.println("--------------------------------------");
      Serial.println();
    } else {
      Serial.println("Pin does not Match!");
      Serial.println("--------------------------------------");
      Serial.println();
    }

    // close the file:
    myFile.close();
  } else {
    // if the file didn't open, print an error:
    Serial.println("Error Opening " + nameOfFile);
  }
  myFile.close();
}

void enterPIN() {
  lcd.clear();
  pinOutput = ""; //Clears previous pin Output on Screen
  lcd.print("  Enter PIN...");
  lcd.setCursor(0, 1);
  lcd.print("*=Clear  #=Enter");
}

void correctPIN(int duration, int socketNumber, String planName) // do this if correct PIN entered
{
  lcd.clear();
  lcd.print("* Correct PIN *");
  delay(1000);
  lcd.clear();

  lcd.print("Socket No. = ");
  lcd.print(socketNumber);
  lcd.setCursor(0, 1);
  lcd.print("Duration = ");
  lcd.print(duration * 5);
  lcd.print("sec");

  delay(2500);
  lcd.clear();

  lcd.print("Socket No. = ");
  lcd.print(socketNumber);
  lcd.setCursor(0, 1);
  lcd.print("Power:");
  lcd.print(planName);
  delay(2500);

  enterPIN();
}

void incorrectPIN() // do this if incorrect PIN entered
{
  trial++;

  lcd.setCursor(0, 0);
  String triall = String(trial) + "/10";
  String trialOutput = "Attempt(s): ";
  trialOutput.concat(triall);   // Concatenates a string and a variable
  lcd.print(trialOutput);

  lcd.setCursor(0, 1);
  lcd.print(" * Try again *");

  delay(3000);  //Displays Error Screen for 3 seconds
  lcd.clear();
  pinOutput = ""; //Clears previous pin Output on Screen
  enterPIN();
}

void clearPIN() {
  lcd.clear();
  pinOutput = ""; //Clears previous pin Output on Screen
  enterPIN();
}


void checkPIN() {
  int correct = 0;
  int i;

  //Converts Character Array of Input PIN to String and Long Integer
  long pinLong = atol(attempt);

  //Passes the PIN char array for checking in the SD card
  xCheck(attempt);

  for (int zz = 0; zz < 6; zz++) {
    attempt[zz] = '0';
  }
}



void readKeypad()
{
  char key = myKeypad.getKey();
  if (key != NO_KEY)
  {
    Serial.print(key);
    lcd.clear();
    lcd.print("Pin: ");
    //lcd.cursor();
    //lcd.blink();
    pinOutput += key; // Concatenate previous key(s) to new key
    lcd.print(pinOutput);
    lcd.setCursor(0, 1);
    lcd.print("*=Clear  #=Enter");

    attempt[z] = key;
    z++;
    switch (key)
    {
      case '*':
        z = 0;
        clearPIN();
        break;
      case '#':
        z = 0;
        delay(100); // for extra debounce
        lcd.clear();
        checkPIN();
        break;
    }
  }
}

void onSocketWithTimer (int socketNumber, int duration, int powerPlan, String buffPIN)
{
  int relayPinn;
  char sensorPin;
  switch (socketNumber)
  {
    case 1:
      relayPinn = socket1;
      sensorPin = A1;
      break;
    case 2:
      relayPinn = socket2;
      sensorPin = A2;
      break;
    case 3:
      relayPinn = socket3;
      sensorPin = A3;
      break;
    case 4:
      relayPinn = socket4;
      sensorPin = A4;
      break;
  }
  t.pulseImmediate(relayPinn, duration * 5000, LOW); // on for 2 minutes
  sendHTTP(1, buffPIN);
  Serial.print("Timer turned ON for ");
  Serial.print(duration * 5);
  Serial.println(" seconds");
  //Serial.println(relayPinn);
}

void getNewPIN()
{
  sendGET("/final_year_project/pages/data_output.php", nameOfFile);
}

void getPower()
{
  sendGET("/final_year_project/pages/power_output.php", nameOfPowerFile);
}

void sendGET(String url, String nameOfTxtFile) //client function to send/receive GET request data.
{
  if (client.connect(myserver, 80)) {  //starts client connection, checks for connection
    Serial.println("connected");
    client.println("GET " + url);
    //Assign data recieved to a variable and write it to a text

    //client.println("GET /~shb/arduino.txt HTTP/1.0"); //download text
    client.println(); //end of get request
  }
  else {
    Serial.println("connection failed"); //error message if no client connect
    Serial.println();
  }

  String d = "";

  while (client.connected() && !client.available()) delay(1); //waits for data
  while (client.connected() || client.available()) { //connected or data available
    char c = client.read(); //gets byte from ethernet buffer
    d += c; // Appends each of the char the previous one

    //Serial.println(c); //prints byte to serial monitor
    //Serial.println(d);
  }

  Serial.println(d);

  writeToFile(d, nameOfTxtFile);

  Serial.println();
  Serial.println("disconnecting.");
  Serial.println("==================");
  Serial.println();
  client.stop(); //stop client

}

void writeToFile(String data2write, String nameOfTxtFile)
{

  //SD.remove(nameOfFile);
  //myFile = SD.open(nameOfFile, FILE_WRITE);
  //O_WRITE | O_CREAT | O_TRUNC Will open the file for write, Creating it if it does not exist, and truncating it's length to 0 if it does RESPECTIVELY
  myFile = SD.open(nameOfTxtFile, O_WRITE | O_CREAT | O_TRUNC);

  // if the file opened okay, write to it:
  if (myFile) {
    Serial.print("Writing to ");
    Serial.print(nameOfTxtFile);
    //myFile.flush();
    myFile.println(data2write);
    // close the file:
    myFile.close();
    Serial.println("done.");
  } else {
    // if the file didn't open, print an error:
    Serial.print("Error opening ");
    Serial.println(nameOfTxtFile);
  }
  myFile.close();

  Serial.println();
  // re-open the file for reading:
  myFile = SD.open(nameOfTxtFile);
  if (myFile) {
    Serial.print("Contents of ");
    Serial.println(nameOfTxtFile);

    // read from the file until there's nothing else in it:
    while (myFile.available()) {
      Serial.write(myFile.read());
    }
    // close the file:
    myFile.close();
  } else {
    // if the file didn't open, print an error:
    Serial.print("Error opening ");
    Serial.println(nameOfTxtFile);
  }
}

void loop()
{
  t.update();
  readKeypad();
  //sock();
}

int getMaxPower(int soook, int ppower) {
  return ppower;
}

void sock() {
  getVPP1();
  getVPP2();
  getVPP3();
  getVPP4();
}

double readPower(int num, int type)
{
  char character;
  String data, powerSub, timeSlot;
  double maxPower, powerSoc;
  int pp;

  myFile = SD.open(nameOfPowerFile);
  if (myFile) {
    while (myFile.available()) {
      character = myFile.read();
      data = data + character;
    }
    // close the file:
    myFile.close();

    switch (num)
    {
      case 1:
        powerSub = data.substring(0, 1);
        timeSlot = data.substring(1, 3);
        //return powerSub.toInt();
        break;
      case 2:
        powerSub = data.substring(3, 4);
        timeSlot = data.substring(4, 6);
        break;
      case 3:
        powerSub = data.substring(6, 7);
        timeSlot = data.substring(7, 9);
        break;
      case 4:
        powerSub = data.substring(9, 10);
        timeSlot = data.substring(10, 12);
        break;
    }

    pp = powerSub.toInt();

    switch (pp)
    {
      case 1:
        powerSoc = powerSub1;
        break;
      case 2:
        powerSoc = powerSub2;
        break;
      case 3:
        powerSoc = powerSub3;
        break;
      case 4:
        powerSoc = powerSub4;
        break;
    }

    maxPower = powerSoc * timeSlot.toInt();

    if (type == 1) {          //For Unit Power
      return powerSoc;
    } else if (type == 0) {   //For Whole Power
      return maxPower;
    }
    
  }
  //myFile.close();
}

double getUnitPower(int num)
{
  return readPower(num, 1);  
}

double getWholePower(int num)
{
  return readPower(num, 0);
}

float getVPP1()
{
  int socketNum1 = 1;
  char sensorPin1 = A1;

  int readValue1;             //value read from the sensor
  int maxValue1 = 0;          // store max value here
  int minValue1 = 1024;          // store min value here

  uint32_t start_time1 = millis();
  //while ((millis() - start_time) < 1000) //sample for 1 Sec
  while ((millis() - start_time1) < 50) //sample for 1 Sec
  {
    readValue1 = analogRead(sensorPin1);
    // see if you have a new maxValue1
    if (readValue1 > maxValue1)
    {
      maxValue1 = readValue1;   /*record the maximum sensor value*/
    }
    if (readValue1 < minValue1)
    {
      minValue1 = readValue1;   /*record the maximum sensor value*/
    }
  }

  // Subtract min from max
  Voltage1 = ((maxValue1 - minValue1) * 5.0) / 1024.0;

  VRMS1 = (Voltage1 / 2.0) * 0.707;
  totalVRMS1 = totalVRMS1 + VRMS1;  //Increments OR Adds up the voltage1 consumed
  AmpsRMS1 = ((VRMS1 * 1000) / mVperAmp) -calibrate1;
  if (AmpsRMS1 >= 0.00) {
    totalAmpsRMS1 = totalAmpsRMS1 + AmpsRMS1;  //Increments OR Adds up the current consumed
  } else {
    totalAmpsRMS1 = 0; //ReIntializes the total current consumed to measure
  }

  //Turns socket off when consumed power is greater than subscribed power
  if (getWholePower(1) > 0) {
    if (AmpsRMS1 >= getUnitPower(1)) {
      digitalWrite(socket1, HIGH);
      //Sends data to the website (Recommended Device not used)
      sendHTTP(2, "1");       //sendHTTP(msgType, value=>socketNo);
    } else if (totalAmpsRMS1 >= getWholePower(1)) {
      digitalWrite(socket1, HIGH);
      //Sends data to the website (Energy subsribed for has been exhausted)
      sendHTTP(3, "1");       //sendHTTP(msgType, value=>socketNo);
    }
  }

  Serial.print("Socket ");
  Serial.print(socketNum1);
  Serial.print("[");
  Serial.print(getWholePower(1));
  Serial.print("]: ");
  Serial.print(AmpsRMS1);
  Serial.print(" per sec | ");
  Serial.print("Total Consumed: ");
  Serial.println(totalAmpsRMS1);
}

float getVPP2()
{
  int socketNum2 = 2;
  char sensorPin2 = A2;

  int readValue2;             //value read from the sensor
  int maxValue2 = 0;          // store max value here
  int minValue2 = 1024;          // store min value here

  uint32_t start_time2 = millis();
  //while ((millis() - start_time) < 1000) //sample for 1 Sec
  while ((millis() - start_time2) < 50) //sample for 1 Sec
  {
    readValue2 = analogRead(sensorPin2);
    // see if you have a new maxValue2
    if (readValue2 > maxValue2)
    {
      maxValue2 = readValue2;   /*record the maximum sensor value*/
    }
    if (readValue2 < minValue2)
    {
      minValue2 = readValue2;   /*record the maximum sensor value*/
    }
  }

  // Subtract min from max
  Voltage2 = ((maxValue2 - minValue2) * 5.0) / 1024.0;

  VRMS2 = (Voltage2 / 2.0) * 0.707;
  totalVRMS2 = totalVRMS2 + VRMS2;  //Increments OR Adds up the voltage2 consumed
  AmpsRMS2 = ((VRMS2 * 1000) / mVperAmp) - calibrate2;
  if (AmpsRMS2 >= 0.00) {
    totalAmpsRMS2 = totalAmpsRMS2 + AmpsRMS2;  //Increments OR Adds up the current consumed
  } else {
    totalAmpsRMS2 = 0; //ReIntializes the total current consumed to measure
  }

  //Turns socket off when consumed power is greater than subscribed power
  if (getWholePower(2) > 0) {
    if (AmpsRMS2 >= getUnitPower(2)) {
      digitalWrite(socket2, HIGH);
      //Sends data to the website (Recommended Device not used)
      sendHTTP(2, "2");       //sendHTTP(msgType, value=>socketNo);
    } else if (totalAmpsRMS2 >= getWholePower(2)) {
      digitalWrite(socket2, HIGH);
      //Sends data to the website (Energy subsribed for has been exhausted)
      sendHTTP(3, "2");       //sendHTTP(msgType, value=>socketNo);
    }
  }

  Serial.print("Socket ");
  Serial.print(socketNum2);
  Serial.print("[");
  Serial.print(getWholePower(2));
  Serial.print("]: ");
  Serial.print(AmpsRMS2);
  Serial.print(" per sec | ");
  Serial.print("Total Consumed: ");
  Serial.println(totalAmpsRMS2);
}

float getVPP3()
{
  int socketNum3 = 3;
  char sensorPin3 = A3;

  int readValue3;             //value read from the sensor
  int maxValue3 = 0;          // store max value here
  int minValue3 = 1024;          // store min value here

  uint32_t start_time3 = millis();
  //while ((millis() - start_time) < 1000) //sample for 1 Sec
  while ((millis() - start_time3) < 50) //sample for 1 Sec
  {
    readValue3 = analogRead(sensorPin3);
    // see if you have a new maxValue3
    if (readValue3 > maxValue3)
    {
      maxValue3 = readValue3;   /*record the maximum sensor value*/
    }
    if (readValue3 < minValue3)
    {
      minValue3 = readValue3;   /*record the maximum sensor value*/
    }
  }

  // Subtract min from max
  Voltage3 = ((maxValue3 - minValue3) * 5.0) / 1024.0;

  VRMS3 = (Voltage3 / 2.0) * 0.707;
  totalVRMS3 = totalVRMS3 + VRMS3;  //Increments OR Adds up the voltage3 consumed
  AmpsRMS3 = ((VRMS3 * 1000) / mVperAmp) - calibrate3;
  if (AmpsRMS3 >= 0.00) {
    totalAmpsRMS3 = totalAmpsRMS3 + AmpsRMS3;  //Increments OR Adds up the current consumed
  } else {
    totalAmpsRMS3 = 0; //ReIntializes the total current consumed to measure
  }

  //Turns socket off when consumed power is greater than subscribed power
  if (getWholePower(3) > 0) {
    if (AmpsRMS3 >= getUnitPower(3)) {
      digitalWrite(socket3, HIGH);
      //Sends data to the website (Recommended Device not used)
      sendHTTP(2, "3");       //sendHTTP(msgType, value=>socketNo);
    } else if (totalAmpsRMS3 >= getWholePower(3)) {
      digitalWrite(socket3, HIGH);
      //Sends data to the website (Energy subsribed for has been exhausted)
      sendHTTP(3, "3");       //sendHTTP(msgType, value=>socketNo);
    }
  }


  Serial.print("Socket ");
  Serial.print(socketNum3);
  Serial.print("[");
  Serial.print(getWholePower(3));
  Serial.print("]: ");
  Serial.print(AmpsRMS3);
  Serial.print(" per sec | ");
  Serial.print("Total Consumed: ");
  Serial.println(totalAmpsRMS3);
}

int getVPP4()
{
  int socketNum4 = 4;
  char sensorPin4 = A4;

  int readValue4;             //value read from the sensor
  int maxValue4 = 0;          // store max value here
  int minValue4 = 1024;          // store min value here

  uint32_t start_time4 = millis();
  //while ((millis() - start_time) < 1000) //sample for 1 Sec
  while ((millis() - start_time4) < 50) //sample for 1 Sec
  {
    readValue4 = analogRead(sensorPin4);
    // see if you have a new maxValue4
    if (readValue4 > maxValue4)
    {
      maxValue4 = readValue4;   /*record the maximum sensor value*/
    }
    if (readValue4 < minValue4)
    {
      minValue4 = readValue4;   /*record the maximum sensor value*/
    }
  }

  // Subtract min from max
  Voltage4 = ((maxValue4 - minValue4) * 5.0) / 1024.0;

  VRMS4 = (Voltage4 / 2.0) * 0.707;
  totalVRMS4 = totalVRMS4 + VRMS4;  //Increments OR Adds up the voltage4 consumed
  AmpsRMS4 = ((VRMS4 * 1000) / mVperAmp) - calibrate4;
  if (AmpsRMS4 >= 0.00) {
    totalAmpsRMS4 = totalAmpsRMS4 + AmpsRMS4;  //Increments OR Adds up the current consumed
  } else {
    totalAmpsRMS4 = 0; //ReIntializes the total current consumed to measure
  }

  //Turns socket off when consumed power is greater than subscribed power
  if (getWholePower(4) > 0) {
    if (AmpsRMS4 >= getUnitPower(4)) {
      digitalWrite(socket4, HIGH);
      //Sends data to the website (Recommended Device not used)
      sendHTTP(2, "4");       //sendHTTP(msgType, value=>socketNo);
    } else if (totalAmpsRMS4 >= getWholePower(4)) {
      digitalWrite(socket4, HIGH);
      //Sends data to the website (Energy subsribed for has been exhausted)
      sendHTTP(3, "4");       //sendHTTP(msgType, value=>socketNo);
    }
  }

  Serial.print("Socket ");
  Serial.print(socketNum4);
  Serial.print("[");
  Serial.print(getWholePower(4));
  Serial.print("]: ");
  Serial.print(AmpsRMS4);
  Serial.print(" per sec | ");
  Serial.print("Total Consumed: ");
  Serial.println(totalAmpsRMS4);
}

void sendHTTP(int msgType, String dataTosend) {

  //Codes for messages & data sent to the website
  // 1 - Client has login in
  // 2 - Maximum power exceeeded (Recommended Device not used)
  // 3 - Energy subsribed for has been exhausted 

  // Connect to the server (your computer or web page)
  if (client.connect(myserver, 80)) {
    Serial.println("Connected");
    //Arduino now sending DATA continuously in this void loop below
    Serial.println("Arduino now sending DATA continuously to the Database");

    client.print("GET /final_year_project/pages/data_input.php?msgType=");
    client.print(msgType);
    client.print("&value=");
    client.print(dataTosend);
    //Serial.print(dataTosend);
    client.println(" HTTP/1.1"); // Part of the GET request
    client.println("Host: 192.168.1.100"); // IMPORTANT: If you are using XAMPP you will have to find out the IP address of your computer and put it here (it is explained in previous article). If you have a web page, enter its address (ie.Host: "www.yourwebpage.com")
    client.println("Connection: close"); // Part of the GET request telling the server that we are over transmitting the message
    client.println(); // Empty line
    client.println(); // Empty line
    client.stop();    // Closing connection to server
  }

  else {
    // If Arduino can't connect to the server (your computer or web page)
    Serial.println("--> connection failed\n");
  }

  // Give the server some time to recieve the data and store it. I used 10 seconds here. Be advised when delaying. If u use a short delay, the server might not capture data because of Arduino transmitting new data too soon.
  //delay(10000);
}

