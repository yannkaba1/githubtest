# githubtest
#include <SPI.h>
#include <MFRC522.h>

// Define pins for the RC522 module
#define RST_PIN 9  // Reset pin
#define SS_PIN 10  // Slave select pin

// Define pins for LED and buzzer
#define LED_PIN 7
#define BUZZER_PIN 8

MFRC522 rfid(SS_PIN, RST_PIN); // Create an instance of the MFRC522 class

// Predefined list of authorized UIDs
const byte authorizedUIDs[][4] = {
    {0xDE, 0xAD, 0xBE, 0xEF}, // Example UID 1
    {0x12, 0x34, 0x56, 0x78}  // Example UID 2
};
const int numAuthorizedUIDs = sizeof(authorizedUIDs) / sizeof(authorizedUIDs[0]);

void setup() {
    Serial.begin(9600); // Initialize serial communication
    SPI.begin();        // Initialize SPI bus
    rfid.PCD_Init();    // Initialize the RFID reader

    // Initialize LED and buzzer
    pinMode(LED_PIN, OUTPUT);
    pinMode(BUZZER_PIN, OUTPUT);

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
    byte currentUID[4];
    for (byte i = 0; i < rfid.uid.size; i++) {
        Serial.print(rfid.uid.uidByte[i], HEX); // Print each byte in hexadecimal
        Serial.print(" ");
        currentUID[i] = rfid.uid.uidByte[i]; // Store UID in memory
    }
    Serial.println();

    // Compare the UID with the predefined list of authorized cards
    if (isAuthorized(currentUID)) {
        Serial.println("Access Granted!");
        digitalWrite(LED_PIN, HIGH); // Turn on LED
        tone(BUZZER_PIN, 1000, 200); // Short beep
    } else {
        Serial.println("Access Denied!");
        digitalWrite(LED_PIN, LOW); // Turn off LED
        tone(BUZZER_PIN, 500, 500); // Long beep
    }

    // Halt the card to stop communication
    rfid.PICC_HaltA();
}

// Function to check if the UID is authorized
bool isAuthorized(byte *uid) {
    for (int i = 0; i < numAuthorizedUIDs; i++) {
        bool match = true;
        for (int j = 0; j < 4; j++) {
            if (uid[j] != authorizedUIDs[i][j]) {
                match = false;
                break;
            }
        }
        if (match) {
            return true;
        }
    }
    return false;
}