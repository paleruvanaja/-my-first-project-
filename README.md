# -smart locking system-
 A door locking system with biometric authentication is a modern security solution
designed to provide enhanced access control by using biological characteristics such as
fingerprints, facial recognition, or iris scans. Traditional locking mechanisms, such as
mechanical locks and password-based systems, have been widely used for securing
residential and commercial properties. However, these conventional methods have significant
limitations, including the risk of key duplication, password breaches, or unauthorized access.
As security threats continue to evolve, there is an increasing need for more reliable and
foolproof authentication systems. Biometric authentication addresses these concerns by
offering a highly secure and user-friendly method for verifying an individual’s identity before
granting access.
#include <Adafruit_Fingerprint.h> 

#include <LiquidCrystal_I2C.h> 

#include <SoftwareSerial.h>

// Pins

#define RELAY_PIN 4

#define RX_PIN 2

#define TX_PIN 3

// LCD settings

// Relay connected to pin 4

// RX pin for fingerprint sensor

// TX pin for fingerprint sensor

LiquidCrystal_I2C lcd(0x27, 16, 2); // Set the LCD addressto 0x27 for a 16 chars 

and 2 line display

// Fingerprint sensor

SoftwareSerial mySerial(RX_PIN, TX_PIN); Adafruit_Fingerprint 

finger = Adafruit_Fingerprint(&mySerial);

// Constants

const unsigned long DOOR_OPEN_TIME= 5000; // Doorstays open for 5 seconds

// Variables

unsigned long doorOpenTime = 0; bool 

doorOpen = false;

void setup() {

// Initialize Serial Monitor

Serial.begin(9600); while

(!Serial);

// Initialize LCD 

lcd.init(); 

lcd.backlight(); 

lcd.clear(); 

lcd.setCursor(0, 0);

lcd.print("Fingerprint Lock"); 

lcd.setCursor(0, 1); 

lcd.print("Initializing...");

// Initialize fingerprint sensor 

finger.begin(57600); delay(5);

// Check if fingerprint sensoris found if

(finger.verifyPassword()) { lcd.clear();

lcd.setCursor(0, 0); lcd.print("Sensor 

found!");

Serial.println("Found fingerprint sensor!");

} else { 

lcd.clear();

lcd.setCursor(0, 0); lcd.print("Sensor 

not found");

Serial.println("Did not find fingerprint sensor :("); while

(1) { delay(1); }

}

// Initialize relay pin 

pinMode(RELAY_PIN, OUTPUT);
digitalWrite(RELAY_PIN, HIGH); // Ensure door is locked initially

// Display readymessage 

delay(2000);

lcd.clear(); 

lcd.setCursor(0, 0);

lcd.print("Ready to scan"); 

lcd.setCursor(0, 1); 

lcd.print("Place finger");

// Printmenu 

printMenu();

}

void loop() {

// Check if the doorshould be locked after the open time

if (doorOpen&& (millis() - doorOpenTime > DOOR_OPEN_TIME)){ lockDoor();

}

// Check for commands from Serial Monitor if 

(Serial.available() > 0) {

char command = Serial.read();

switch (command) {

case 'e': // Enroll a fingerprint 

enrollFingerprint();

break;

case 'd': // Delete a fingerprint 

deleteFingerprint();

break;

case '?': // Print menu 

printMenu();

break;

}

}

// Check for fingerprint scan if

(!doorOpen) {

int fingerprintID = getFingerprintID();

if (fingerprintID != -1) {

// Valid fingerprint detected

lcd.clear(); lcd.setCursor(0,

0);

lcd.print("Access Granted!"); 

lcd.setCursor(0,1); lcd.print("ID 

#"); lcd.print(fingerprintID);

// Unlock the door 

unlockDoor();

}

}
delay(100); // Small delay to avoid excessive CPU usage

}

void printMenu() {

Serial.println("\n========= Fingerprint Door Lock System ========="); 

Serial.println("Commands:");

Serial.println("e - Enroll a new fingerprint"); 

Serial.println("d - Delete a fingerprint"); Serial.println("?

- Print this menu"); 

Serial.println("================================================");

}

void unlockDoor() {

digitalWrite(RELAY_PIN, LOW); // Activate relay to unlock door 

doorOpen = true;

doorOpenTime = millis(); 

Serial.println("Door unlocked!");

}

void lockDoor() {

digitalWrite(RELAY_PIN, HIGH); // Deactivate relay to lock door 

doorOpen = false; Serial.println("Door

locked!");

// Update LCD 

lcd.clear(); 

lcd.setCursor(0, 0);

lcd.print("Ready to scan"); 

lcd.setCursor(0, 1); 

lcd.print("Place finger");

}

int getFingerprintID() {

uint8_t p = finger.getImage();

if (p != FINGERPRINT_OK)return -1; 

p = finger.image2Tz();

if (p != FINGERPRINT_OK)return -1; 

while (true) {

while (!Serial.available());

// Get the ID from Serial Monitor id

= Serial.parseInt();

if (id > 0 && id < 128) { 

break;

} else {

Serial.println("ID must be between 1-127");

}

}

Serial.print("Enrolling ID #");
Serial.println(id);

// Update LCD lcd.clear(); 

lcd.setCursor(0, 0); 

lcd.print("Enrolling ID #"); 

lcd.print(id);

// Get the first fingerprint image

while (getFingerprintEnroll(id, 1) != true);

// Get the second fingerprint image while 

(getFingerprintEnroll(id, 2) != true);

// Store the fingerprint in the database 

storeFingerprint(id);

// Return to main state 

lcd.clear(); lcd.setCursor(0,

0); lcd.print("Ready to

scan"); lcd.setCursor(0, 1); 

lcd.print("Place finger");

}

bool getFingerprintEnroll(int id, int step) { 

int p = -1;

if (step == 1) {

lcd.setCursor(0, 1); 

lcd.print("Place finger");

Serial.println("Place your finger on the sensor...");

} else {

lcd.setCursor(0, 1); lcd.print("Place 

same finger");

Serial.println("Place same finger again...");

}

while (p != FINGERPRINT_OK) {

p = finger.getImage(); 

switch (p) {

case FINGERPRINT_OK:

Serial.println("Image taken"); 

break;

case FINGERPRINT_NOFINGER:

Serial.print("."); 

break;

case FINGERPRINT_PACKETRECIEVEERR:

Serial.println("Communication error"); 

break;

case FINGERPRINT_IMAGEFAIL:

Serial.println("Imaging error");
break;

default:

Serial.println("Unknown error"); 

break;

}

}

// OK success!

p = finger.image2Tz(step); 

switch (p) {

case FINGERPRINT_OK:

Serial.println("Image converted"); break; 

case FINGERPRINT_IMAGEMESS:

Serial.println("Image too messy"); 

returnfalse;

case FINGERPRINT_PACKETRECIEVEERR:

Serial.println("Communication error"); 

return false;

case FINGERPRINT_FEATUREFAIL:

Serial.println("Could not find fingerprint features"); 

return false;

case FINGERPRINT_INVALIDIMAGE:

Serial.println("Could not find fingerprint features"); 

return false;

default:

Serial.println("Unknown error"); 

return false;

}

if (step == 1) { 

Serial.println("Remove finger");

lcd.setCursor(0, 1); 

lcd.print("Remove finger "); 

delay(2000);

}

return true;

}

void storeFingerprint(int id) { int 

p = finger.createModel();

if (p == FINGERPRINT_OK) {

Serial.println("Prints matched!");

} else if (p == FINGERPRINT_PACKETRECIEVEERR){

Serial.println("Communication error"); 

return;

} else if (p == FINGERPRINT_ENROLLMISMATCH){

Serial.println("Fingerprints did not match"); 

lcd.setCursor(0, 1);
lcd.print("Prints didn't match"); 

delay(2000);

return;

} else {

Serial.println("Unknown error"); 

return;

}

p = finger.storeModel(id);

if (p == FINGERPRINT_OK) {

Serial.println("Fingerprint stored!"); 

lcd.setCursor(0, 1); 

lcd.print("Fingerprint stored");

delay(2000);

} else if (p == FINGERPRINT_PACKETRECIEVEERR) {

Serial.println("Communication error"); 

return;

} else if (p == FINGERPRINT_BADLOCATION) {

Serial.println("Could notstore in that location"); 

return;

} else if (p == FINGERPRINT_FLASHERR) {

Serial.println("Error writing to flash"); return;

} else {

Serial.println("Unknown error"); 

return;

}

}

void deleteFingerprint() { 

int id = 0;

Serial.println("Please type in the ID # (1-127)you want to delete...");

while (true) {

while (!Serial.available());

// Get the ID from Serial Monitor id

= Serial.parseInt();

if (id > 0 && id < 128) { 

break;

} else {

Serial.println("ID must be between 1-127");

}

}

Serial.print("Deleting ID #"); 

Serial.println(id);

// Update LCD

lcd.clear(); 

lcd.setCursor(0, 0); 

lcd.print("Deleting ID #");
lcd.print(id);

uint8_t p = finger.deleteModel(id); 

if (p == FINGERPRINT_OK) {

Serial.println("Fingerprint deleted!");

lcd.setCursor(0, 1); lcd.print("Fingerprint 

deleted"); delay(2000);

} else if (p == FINGERPRINT_PACKETRECIEVEERR){

Serial.println("Communication error");

} else if (p == FINGERPRINT_BADLOCATION) {

Serial.println("Could not delete from that location");

} else if (p == FINGERPRINT_FLASHERR){

Serial.println("Error writing to flash");

} else {

Serial.print("Unknown error: 0x"); Serial.println(p, HEX);

}

// Return to main state 

lcd.clear(); lcd.setCursor(0, 

0); lcd.print("Ready to

scan"); lcd.setCursor(0, 1);

lcd.print("Place finger");

}
