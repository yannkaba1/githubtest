# githubtest
#include <SPI.h>
#include <MFRC522.h>

// Define pins for the RC522 module
#define RST_PIN 9  // Reset pin
#define SS_PIN 10  // Slave select pin

MFRC522 rfid(SS_PIN, RST_PIN); // Create an instance of the MFRC522 class

void setup() {
    Serial.begin(9600); // Initialize serial communication
    SPI.begin();        // Initialize SPI bus
    rfid.PCD_Init();    // Initialize the RFID reader
    Serial.println("Place your RFID card near the reader...");
}

void loop() {
    // Check if a new card is present
    if (!rfid.PICC_IsNewCardPresent()) {
        return; // No card detected
    }

    // Check if the card can be read
    if (!rfid.PICC_ReadCardSerial()) {
        return; // Failed to read the card
    }

    // Print the UID of the card
    Serial.print("Card UID: ");
    for (byte i = 0; i < rfid.uid.size; i++) {
        Serial.print(rfid.uid.uidByte[i], HEX); // Print each byte in hexadecimal
        Serial.print(" ");
    }
    Serial.println();

    // Halt the card to stop communication
    rfid.PICC_HaltA();
}