# VE-Display

VE-Display is an iOS and Apple CarPlay app for monitoring Victron Energy devices through Bluetooth Low Energy advertisement packets.

https://apps.apple.com/it/app/ve-display/id6781215472?l=en-GB

The app reads Victron BLE Instant Readout data locally on the iPhone, decodes the encrypted manufacturer data, and presents the most relevant electrical values on the phone. On Apple CarPlay, VE-Display uses a low-distraction device list: select a device and the app announces its key values through text-to-speech.

> This project is independent and is not affiliated with, endorsed by, or sponsored by Victron Energy.

## Features

- Victron BLE Instant Readout decoding.
- Local AES-CTR decoding using user-provided Victron advertisement keys.
- No cloud service, no account, no external server.
- Automatic BLE scan at app launch.
- Manual start/stop scan control.
- Configurable advertisement keys from the iPhone app.
- Device list with automatic removal of stale BLE devices.
- Apple CarPlay list view with large device names and device-type icons.
- Apple CarPlay voice announcements when a device is selected.
- CarPlay speech routed through the vehicle audio system when available.
- iPhone portrait-only interface.
- Launch splash screen and About screen using the VE-Display app icon.

## Supported Victron Devices

The current implementation supports the Victron Instant Readout record families used by the following devices tested during development:

| Device family | Exact device verified so far | Decoded values |
| --- | --- | --- |
| SmartSolar | Victron SmartSolar MPPT 75/15 | State, charger error, battery voltage, battery current, yield today, PV power, load current |
| SmartShunt | Victron SmartShunt 500A/50mV | Battery voltage, battery current, state of charge, consumed Ah, time-to-go, alarm reason, auxiliary input |
| Smart Battery Sense | Victron Smart Battery Sense | Battery voltage, temperature/auxiliary value when advertised |
| Orion XS | Victron Orion XS 12/12-50A | State, charger error, input voltage, output voltage, input current, off reason |
| AC Charger / Smart Charger | Victron Smart IP43 / Smart Charger BLE Instant Readout record | State, charger error, battery voltage, charger current |

The parser also recognizes Victron record type metadata for other Instant Readout families, but the app only displays fields that are currently decoded with confidence. Unknown or not-yet-mapped records are intentionally not converted into guessed values.

## Important Notes About Values

- Smart Charger current may differ from the rounded value shown in VictronConnect. VE-Display displays the value decoded from the BLE Instant Readout packet.
- Orion XS output current is not currently shown because the observed Orion XS BLE payload exposes input current reliably but does not expose a separate confirmed output-current field in the same way.
- BLE scanning in the background is limited by iOS. When the iPhone screen is off, iOS may suspend or reduce BLE scanning even if CarPlay is active.

## How It Works

Victron BLE Instant Readout packets are broadcast as BLE manufacturer data using Victron company ID `0x02E1`.

VE-Display parses the manufacturer data layout as follows:

| Offset | Field |
| --- | --- |
| `0...1` | Victron company ID, little endian: `0x02E1` |
| `2` | Manufacturer record type: `0x10` |
| `3...4` | Product ID, little endian |
| `5` | Readout type |
| `6` | Victron record type |
| `7...8` | Data counter / nonce, little endian |
| `9` | Advertisement key check byte |
| `10...` | AES-CTR encrypted payload |

The encrypted payload starts at byte `10`. The app decrypts it with AES-CTR using the Victron advertisement key and a nonce derived from the data counter.

## Requirements

- iPhone with iOS 26.5 or later, according to the current Xcode project settings.
- Xcode 26.5 or later.
- Bluetooth permission enabled for the app.
- Victron devices with BLE Instant Readout enabled.
- Advertisement key for each Victron device.
- Apple CarPlay entitlement/capability if building and running the CarPlay interface on real hardware.

## Getting the Victron Advertisement Key

Each Victron device has its own advertisement key. VE-Display cannot decode encrypted Instant Readout packets without the correct key.

The path in VictronConnect may vary slightly by app version and product firmware, but the key is normally found from the device information screens:

1. Open VictronConnect.
2. Connect to the Victron device.
3. Open the device settings using the gear icon.
4. Open `Product info`.
5. Look for the Instant Readout / Bluetooth advertisement information.
6. Copy or reveal the `Advertisement key`.
7. Enter the key in VE-Display using the key button on the iPhone app.

Use the key exactly as shown by VictronConnect. It should be a 16-byte key represented as 32 hexadecimal characters.

## Device-Specific Key Setup

Add one key for each physical Victron device:

| Device | VictronConnect path |
| --- | --- |
| SmartSolar MPPT 75/15 | Device -> Settings -> Product info -> Instant Readout / Advertisement key |
| SmartShunt 500A/50mV | Device -> Settings -> Product info -> Instant Readout / Advertisement key |
| Smart Battery Sense | Device -> Settings -> Product info -> Instant Readout / Advertisement key |
| Orion XS 12/12-50A | Device -> Settings -> Product info -> Instant Readout / Advertisement key |
| Smart IP43 / Smart Charger | Device -> Settings -> Product info -> Instant Readout / Advertisement key |

The app stores configured keys locally in app preferences. Keys are not transmitted outside the device.

## Using the iPhone App

1. Launch VE-Display.
2. Tap the key button.
3. Add one or more Victron advertisement keys.
4. Return to the main screen.
5. BLE scanning starts automatically.
6. Discovered devices appear in the device list.
7. Stop and restart scanning if needed. Restarting the scan clears the current device list and starts fresh.

## Using CarPlay

1. Install and run VE-Display on the iPhone.
2. Connect the iPhone to CarPlay.
3. Open VE-Display from the CarPlay launcher.
4. Select a discovered Victron device from the CarPlay list.
5. VE-Display announces the selected device values through text-to-speech.
6. Select the same device again to repeat the announcement.

The CarPlay UI is intentionally compact and low-distraction: large device names, small device-type icons, no RSSI values and no debug clutter. The complete visual value list remains available on the iPhone.

Current CarPlay voice announcements include:

| Device | Announced values |
| --- | --- |
| SmartShunt | Battery voltage, current, state of charge, consumed Ah, time-to-go |
| SmartSolar | Battery voltage, solar power, charge current, state |
| Orion XS | Input voltage, output voltage, input current, state |
| Smart Battery Sense | Battery voltage, temperature |
| Smart Charger | Battery voltage, charge current, state |

## Privacy

VE-Display decodes BLE advertisement packets locally on the iPhone.

- No personal data collection.
- No telemetry.
- No cloud synchronization.
- No external API calls.
- Advertisement keys remain local to the app preferences.

## Known Limitations

- iOS may suspend BLE scanning when the iPhone screen is off.
- Orion XS output current is not decoded from a confirmed dedicated output-current field.
- Only the devices listed above have been verified with real logs so far.
- Additional Victron devices may work if they use already-supported Instant Readout record types, but they should be validated with real BLE logs before being listed as supported.

## Credits

VE-Display is based on the Victron BLE Instant Readout manufacturer data format and on practical validation against real Victron Energy devices.

Thanks to the open-source Victron BLE ecosystem and community research around Victron Instant Readout decoding.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE).
