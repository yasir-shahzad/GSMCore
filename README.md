# GSMCore

[![Discord](https://img.shields.io/discord/1349656889981407283?style=social&logo=discord&label=COMMUNITY)](https://discord.gg/aqafn5wj)
[![Ko-fi](https://img.shields.io/badge/Support%20on%20Ko--fi-F16061?style=flat&logo=kofi&logoColor=white&labelColor=%23FF5E5B)](https://buymeacoffee.com/yasirshahzad)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![GitHub issues](https://img.shields.io/github/issues/yasir-shahzad/GSMCore.svg)](https://github.com/yasir-shahzad/GSMCore/issues)

GSMCore is an Arduino library for communicating with GSM modules using AT commands over `SoftwareSerial`. It provides helpers for sending and receiving SMS messages, making calls, and managing phonebook entries.

The project targets GSMCore-compatible boards and shields. The source lists SIM800L, SIM800C, SIM900, and SIM808 modules; check your board's voltage, serial interface, and modem support before connecting it.

## Features

- Initialize a modem and configure SMS text mode.
- Send SMS messages and read or search stored messages.
- Delete one or all stored messages.
- Make a call to a phone number or a SIM phonebook position.
- Read and manage phonebook entries.
- Read modem network time.

## Installation

Install the library using either method:

1. **Arduino IDE:** In Library Manager, search for **GSMCore** and install it, if available.
2. **Manual:** Download or clone this repository and place the `GSMCore` folder in your Arduino `libraries` directory. Restart the IDE.

The library uses the Arduino `SoftwareSerial` library. Some board cores provide it separately; install the version appropriate for your board if it is not already available.

## Wiring

Connect the modem's UART to the pins passed to `GSMCore`:

| Arduino | GSM module |
| --- | --- |
| Transmit pin (TX) | Receive pin (RX) |
| Receive pin (RX) | Transmit pin (TX) |
| GND | GND |

Use a suitable external supply for the GSM module and share ground with the Arduino. GSM modules can require high current bursts; follow the power and logic-level requirements for your specific board. Do not assume the modem can be powered safely from an Arduino GPIO or 5 V pin.

`GSMCore` takes the Arduino transmit pin first and receive pin second. For example, `GSMCore modem(11, 10);` sends on pin 11 and receives on pin 10.

## Quick start: send an SMS

```cpp
#include <GSMCore.h>

const int modemTxPin = 11;
const int modemRxPin = 10;

GSMCore modem(modemTxPin, modemRxPin);

void setup() {
  Serial.begin(115200);

  if (!modem.begin(19200)) {
    Serial.println("GSM modem initialization failed");
    return;
  }

  char number[] = "+1234567890";
  char message[] = "Hello from GSMCore";

  if (modem.SendSMS(number, message)) {
    Serial.println("SMS sent");
  } else {
    Serial.println("SMS could not be sent");
  }
}

void loop() {
}
```

Replace the example phone number with the recipient's number in international format. The baud rate must match the modem's UART configuration; the examples in this repository use `19200`.

## Examples

Open an example from the Arduino IDE's **File > Examples > GSMCore** menu, or browse [`examples/`](./examples/):

- [`SendSMS`](./examples/SendSMS/SendSMS.ino) — send a text message.
- [`ReceiveSMS`](./examples/ReceiveSMS/ReceiveSMS.ino) — check for and read an SMS.
- [`MakeVoiceCall`](./examples/MakeVoiceCall/MakeVoiceCall.ino) — place a voice call.

Update the serial pins, baud rate, and destination number in each sketch to match your hardware.

## API overview

| Method | Description |
| --- | --- |
| `GSMCore(int transmitPin, int receivePin)` | Create a modem interface using the specified Arduino pins. |
| `int begin(long baud_rate)` | Start the modem serial connection and initialize modem settings. |
| `char SendSMS(char *number, char *message)` | Send an SMS. |
| `int IsSMSPresent(char *phone_number, char *SMS_text, byte max_SMS_len)` | Check for an SMS and copy its sender and text into the provided buffers. |
| `char GetSMS(byte position, char *phone_number, char *SMS_text, byte max_SMS_len)` | Read an SMS by storage position. |
| `char IsSMSPresent(byte required_status)` | Search stored SMS messages by status (`SMS_UNREAD`, `SMS_READ`, or `SMS_ALL`). |
| `char DeleteSMS(byte position)` / `bool DeleteAll()` | Delete one stored SMS or all stored SMS messages. |
| `void Call(char *number)` / `void Call(int sim_position)` | Call a number or a phonebook entry. |
| `char GetPhoneNumber(byte position, char *phone_number)` | Read a phonebook entry. |
| `char WritePhoneNumber(byte position, char *phone_number)` | Write a phonebook entry. |
| `char DelPhoneNumber(byte position)` | Delete a phonebook entry. |
| `char GetTIME(char *year, char *month, char *day, char *hour, char *minute)` | Read the modem's network time into caller-provided buffers. |

See [`src/GSMCore.h`](./src/GSMCore.h) for the full API and return-value definitions.

## Initialization behavior

`begin()` configures the modem, including sending `AT&F` (restore factory settings), enabling network time synchronization, and selecting SMS text mode. This may overwrite modem settings previously configured outside the library. Initialization also uses the `RESET_PIN` defined in `src/GSMCore.h` (default pin 2) when it does not receive a modem response. Review the pin definition and your board's reset wiring before use.

## License

This project is distributed under the GNU General Public License v3.0. See the [GPL-3.0 license](https://www.gnu.org/licenses/gpl-3.0) for details.
