# Echo Pay

This project implements a simple RFID-based payment/credit update system using an **ESP32**, **MFRC522 RFID reader**, and an **Async Web Server** for communication with external applications.
The ESP32 reads an RFID card, extracts the UID, retrieves an amount from an HTTP endpoint, updates the stored balance on the card, and exposes useful JSON endpoints.

---

## Features

- Reads RFID card UID using MFRC522.
- Stores and updates balance on a MIFARE Classic card.
- Connects to WiFi and runs an asynchronous web server.
- Provides REST API endpoints for:
  - Fetching card UID and ESP ID.
  - Fetching status flag.
  - Receiving an amount to update the stored balance.

- Uses block 4 on the RFID card to store a numeric balance value.
- Automatically writes the updated balance back to the card.

---

## Hardware Requirements

- ESP32 Development Board
- MFRC522 RFID Reader
- Jumper Wires
- Power Supply (USB or external)

---

## Pin Configuration

### MFRC522 to ESP32

| MFRC522 Pin | ESP32 GPIO |
| ----------- | ---------- |
| SDA (SS)    | 21         |
| RST         | 22         |
| SCK         | 18         |
| MISO        | 19         |
| MOSI        | 23         |
| VCC         | 3.3V       |
| GND         | GND        |

---

## How It Works

### 1. WiFi Connection

The ESP32 connects to the configured WiFi network using the credentials in the code.

### 2. Web Server Endpoints

The device exposes three endpoints via HTTP:

#### **GET /uid**

Returns the scanned RFID card UID and ESP ID.
Response example:

```json
{
  "uid": "a1b2c3d4",
  "espid": "1010"
}
```

#### **GET /call**

Returns a flag value. This is used to notify external systems about card state.

```json
{ "flag": "0" }
```

#### **GET /1010?response=VALUE**

Used by external systems to send a transaction amount.
The ESP32 reads `response` as an integer and stores it in `Tamount`.

Example:

```
/1010?response=50
```

---

## Balance Reading and Updating

1. When a new card is detected, its UID is retrieved.
2. The ESP32 authenticates using Key A (default FF FF FF FF FF FF).
3. Block 4 is read and interpreted as the current stored balance.
4. The new balance is calculated as:

   ```
   newBalance = oldBalance + Tamount
   ```

5. The updated value is written back to block 4.
6. Verification is performed by reading the block again.
7. The transaction amount (`Tamount`) is reset to 0 after processing.

---

## Important Variables

| Variable           | Description                                |
| ------------------ | ------------------------------------------ |
| `cardUID`          | UID of the scanned RFID card               |
| `espid`            | Device identifier                          |
| `Tamount`          | Amount to be added to card balance         |
| `resp`             | Status flag returned by `/call`            |
| `balanceCheckFlag` | Used to detect specific balance conditions |

---

## JSON Interaction Summary

External clients can poll `/uid` to detect when a card is present, then call `/1010` to send a transaction amount, and finally use `/call` to check system status.

---

## Setup Instructions

1. Install Arduino IDE or PlatformIO.
2. Install required libraries:
   - `ESPAsyncWebServer`
   - `AsyncTCP`
   - `MFRC522`

3. Replace `serverIP` with your server's address (if required).
4. Flash the code into the ESP32.
5. Open the serial monitor at 115200 baud.
6. Scan an RFID card to test balance read/write operations.
7. Use HTTP requests from browser, Postman, or another ESP to interact.

---

## Notes

- Block 4 stores balance as plain text. Ensure that it contains a numeric string (e.g., `"100"`).
- MIFARE Classic cards must use Key A `FF FF FF FF FF FF` unless changed.
- The ESP32 resets the transaction amount after each card update.
- The system currently performs only additive operations; modify logic if deductions or authentication are required.

---

## Future Improvements

- Add authentication for HTTP endpoints.
- Encrypt the balance stored on the RFID card.
- Add support for debit or multi-application wallets.
- Improve status reporting, error codes, or logging.
- Store transaction history on the server.

