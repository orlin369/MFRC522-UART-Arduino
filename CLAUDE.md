# CLAUDE.md

This file provides guidance for AI assistants working with the MFRC522-UART-Arduino library codebase.

## Project Overview

The **MFRC522-UART-Arduino** library is a lightweight C++ library for Arduino microcontrollers that enables communication with MFRC522 RFID reader modules via Serial UART interface. Unlike the common SPI-based libraries, this uses a serial protocol for simplified wiring.

- **License:** MIT
- **Original Author:** zodier (2014)
- **Communication:** Serial UART at 9600 baud

## Directory Structure

```
MFRC522-UART-Arduino/
├── MFRC522.h             # Class definition and method declarations
├── MFRC522.cpp           # Implementation of all class methods
├── examples/
│   └── Basic-example.ino # Basic Arduino card detection example
├── the_pill/
│   └── mfrc522_read.shelly.js  # Shelly script for card reading
├── README.md             # Method documentation
├── CLAUDE.md             # AI assistant guidance
└── LICENSE               # MIT License
```

## Architecture

### Single Main Class: `MFRC522`

The library centers around a single class with methods organized by functionality:

| Category | Key Methods |
|----------|-------------|
| **Initialization** | `MFRC522()`, `begin()` |
| **Card Detection** | `available()`, `wait()` |
| **Card Identity** | `readCardSerial()`, `getCardSerial()` |
| **Block Operations** | `getBlock()`, `writeBlock()` |
| **Low-level I/O** | `communicate()`, `read()`, `write()` |

### Class Members

**Public Methods:**
- `begin(HardwareSerial *serial)` - Initialize with serial port, sets 9600 baud
- `available()` - Returns true if card detected (data available on serial)
- `wait()` - Put MFRC522 in wait/listen mode
- `readCardSerial()` - Read 4-byte card serial into buffer
- `getCardSerial()` - Returns pointer to 4-byte serial array
- `getBlock(block, keyType, *key, *returnBlock)` - Read 16-byte block with authentication
- `writeBlock(block, keyType, *key, *data)` - Write 16-byte block with authentication
- `communicate(command, *sendData, sendDataLength, *returnData, *returnDataLength)` - Raw protocol communication

**Private Members:**
- `HardwareSerial *_Serial` - Serial port reference
- `byte cardSerial[4]` - Card serial number buffer

## Protocol Specification

### Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `MFRC522_HEADER` | `0xAB` | Protocol header byte |
| `COMMAND_WAIT` | `0x02` | Enter wait mode |
| `COMMAND_READBLOCK` | `0x03` | Read block command |
| `COMMAND_WRITEBLOCK` | `0x04` | Write block command |
| `MIFARE_KEYA` | `0x00` | MIFARE Key A authentication |
| `MIFARE_KEYB` | `0x01` | MIFARE Key B authentication |
| `STATUS_OK` | `1` | Operation successful |
| `STATUS_ERROR` | `0` | Operation failed |

### Packet Structure

**Request format:**
```
[Header 0xAB] [Length] [Command] [Data...]
```

**Response format:**
```
[Header] [Length] [Command Result] [Data...]
```

### Block Operations

- **Read block:** Sends 10 bytes (block number + key type + 6-byte key)
- **Write block:** Sends 26 bytes (block number + key type + 6-byte key + 16-byte data)
- Default key: `0xFF 0xFF 0xFF 0xFF 0xFF 0xFF`

## Coding Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Classes | PascalCase | `MFRC522` |
| Methods | camelCase | `readCardSerial()` |
| Constants | UPPER_CASE | `COMMAND_READBLOCK` |
| Private members | Underscore prefix | `_Serial` |

## Hardware Setup

### Wiring

The MFRC522 UART module connects via serial:

```
Arduino TX → MFRC522 RX
Arduino RX → MFRC522 TX
Arduino GND → MFRC522 GND
Arduino 5V → MFRC522 VCC
```

### Platform Notes

- **Arduino Uno:** Uses primary `Serial` port (requires disconnecting for upload)
- **Arduino Mega/Due:** Can use `Serial1`, `Serial2`, or `Serial3` for dedicated RFID communication
- **ESP32/ESP8266:** Use `Serial1` or `Serial2`; level shifter recommended (3.3V vs 5V logic)
- **Shelly Gen2/Gen3 (The Pill):** Uses built-in UART; requires external 5V power for MFRC522

## Common Tasks

### Basic Card Detection

```cpp
#include "MFRC522.h"

MFRC522 rfid;

void setup() {
    rfid.begin(&Serial);
}

void loop() {
    if (rfid.available()) {
        rfid.readCardSerial();
        byte *serial = rfid.getCardSerial();
        // Use serial[0..3] for card ID
        rfid.wait();
    }
}
```

### Reading a Block

```cpp
byte key[6] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};
byte blockData[16];

if (rfid.getBlock(4, 0x00, key, blockData)) {
    // blockData contains 16 bytes from block 4
}
```

### Writing a Block

```cpp
byte key[6] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};
byte data[16] = {0x01, 0x02, 0x03, ...};

if (rfid.writeBlock(4, 0x00, key, data)) {
    // Block 4 written successfully
}
```

### Adding a New Command

1. Add command byte constant to `MFRC522.cpp` defines section
2. Add method declaration to public section in `MFRC522.h`
3. Implement in `MFRC522.cpp` using `communicate()` method
4. Update `README.md` documentation

## Important Files to Read First

1. `MFRC522.h` - Class interface and public API
2. `MFRC522.cpp` - Full implementation with protocol details
3. `examples/Basic-example.ino` - Arduino usage pattern
4. `the_pill/mfrc522_read.shelly.js` - Shelly device usage pattern

## Shelly Scripts (The Pill)

The `the_pill/` directory contains JavaScript scripts for Shelly Gen2/Gen3 devices.

### Script Naming Convention

- Files use `.shelly.js` extension
- Names are lowercase with underscores: `mfrc522_read.shelly.js`

### Available Scripts

| Script | Description |
|--------|-------------|
| `mfrc522_read.shelly.js` | Basic card serial reading with callbacks |

### Shelly Script Architecture

Scripts follow these patterns:
- `CONFIG` object at top for settings
- `MFRC522` object with API methods
- `init()` function for initialization
- Callback-based event handling

### Shelly UART API

```javascript
// Get UART handle
var uart = UART.get();

// Configure UART
uart.configure({ baud: 9600, mode: '8N1' });

// Send data
uart.write(String.fromCharCode(0x02));

// Receive data (callback-based)
uart.recv(function(data) {
    // Process received bytes
});
```

### Shelly Wiring

```
Shelly TX  → MFRC522 RX
Shelly RX  → MFRC522 TX
Shelly GND → MFRC522 GND
External 5V → MFRC522 VCC (Shelly cannot power the module)
```

**Note:** Level shifter may be required as Shelly uses 3.3V logic.

## Git Workflow

### Branching Strategy

- **master**: Production-ready code, only receives merges from dev
- **dev**: Development branch, created from master, where integration happens
- **feature branches**: Created from dev for each new feature or change

### Branch Naming

- Feature branches: `feature/<short-description>` (e.g., `feature/add-ultralight-support`)
- Bug fixes: `fix/<short-description>` (e.g., `fix/timeout-handling`)

### Commit Workflow (Step by Step)

1. **Checkout dev branch:**
   ```bash
   git checkout dev
   ```

2. **Create feature branch from dev:**
   ```bash
   git checkout -b feature/<short-description>
   ```

3. **Stage and commit changes with descriptive message:**
   ```bash
   git add <file>
   git commit -m "$(cat <<'EOF'
   Short summary of changes

   - Detailed bullet point 1
   - Detailed bullet point 2
   - Detailed bullet point 3

   Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
   EOF
   )"
   ```

4. **Test the feature before merging:**
   - For software-only changes: Verify code compiles in Arduino IDE
   - For hardware-dependent changes: **ASK the user to test manually**
   - Never merge untested code into dev

5. **ASK before merging to dev:**
   - Always ask the user for approval before merging feature into dev
   - Example: "Feature is ready and committed. May I merge to dev and master?"

6. **Merge feature branch to dev (with --no-ff to preserve branch history):**
   ```bash
   git checkout dev
   git merge feature/<short-description> --no-ff -m "Merge feature/<short-description> into dev"
   ```

7. **Merge dev to master (with --no-ff to preserve branch history):**
   ```bash
   git checkout master
   git merge dev --no-ff -m "Merge dev into master"
   ```

8. **Push both branches and clean up:**
   ```bash
   git push origin master
   git push origin dev
   git branch -d feature/<short-description>
   ```

### Important: Always Use --no-ff

Always use `--no-ff` (no fast-forward) when merging to create merge commits. This preserves the branch topology and makes the history visible in GitLens.

### Commit Message Format

```
Short summary (imperative mood, max 50 chars)

- Bullet point describing change 1
- Bullet point describing change 2
- Bullet point describing change 3

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```
