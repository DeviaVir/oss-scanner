# Blockstream App (Green iOS) Threat Model

Blockstream App for iOS (`Blockstream/green_ios`) is a self-custodial Bitcoin, Liquid, and Lightning wallet for iOS written in Swift (UIKit / SwiftUI). Because it targets iOS and depends on Xcode and Apple frameworks (`Security`, `LocalAuthentication`, `CoreBluetooth`, `UIKit`), this project cannot be compiled on Linux and is provided for source-level security analysis.

## Assets

- **Wallet secrets**: mnemonics, BIP39 passphrases, Keychain-stored PIN/biometric credentials, Lightning/Greenlight device keys & certs (`core/`, `lightning/`).
- **Transaction signing integrity**: users must only sign transactions whose destinations, amounts, Liquid asset IDs, and fees match what is shown in the UI (`gaios/Sendflow/`, `gaios/TxDetails/`).
- **App lock and privacy**: Face ID / Touch ID / PIN lock (`gaios/ScreenLocker.swift`, `gaios/ScreenLockWindow.swift`), background privacy screen, and avoiding secret leakage into iOS logs, crash reports, pasteboard, or `NotificationService/`.

## Trust boundaries and where untrusted input enters

1. **Deep links, Universal Links, and custom URL schemes (`gaios/AppDelegate.swift`, `gaios/SceneDelegate.swift`, `gaios/Info.plist`)**:
   - `bitcoin:`, `liquidnetwork:`, `lightning:`, `lnurl:`, and custom links opened from Safari or third-party apps. Links must never bypass send/auth confirmations or expose secrets.
2. **QR codes, PSBTs/PSETs, and imported wallet data (`gaios/Sendflow/`, `gaios/Receive/`, `gaios/WOFlow/`, `core/`)**:
   - BIP21 URIs, BOLT11/BOLT12 invoices & offers, LNURL payloads, animated BC-UR QR payloads, PSBTs/PSETs, and descriptors.
3. **Hardware wallet BLE communication (`hw/Jade/`, `hw/Ledger/`, `hw/HWResolver.swift`, `gaios/HWFlow/`, `gaios/GenuineCheck/`)**:
   - CoreBluetooth (`AsyncBluetooth`) responses, CBOR/APDU parsing, genuine-check attestation, and firmware update verification.
4. **GDK, LWK, and Greenlight Lightning backends (`core/gdk/`, `core/backend/`, `lightning/GreenlightSdk.swift`, `lightning/LightningSdk.swift`, `gaios/LTFlow/`)**:
   - Electrum, Green backend, Breez/Greenlight, Boltz swap, and asset-registry responses.
5. **Keychain, Biometrics, and Extension boundaries (`core/Managers/`, `core/Repository/`, `NotificationService/`)**:
   - Keychain accessibility flags (`kSecAttrAccessible*`), biometric evaluation bypasses, and `ScreenLocker` state transitions.

## Components that matter most / least

- **Matter most**: `core/`, `hw/`, `lightning/`, `NotificationService/`, `gaios/Sendflow/`, `gaios/HWFlow/`, `gaios/LTFlow/`, `gaios/OnBoard/`, `gaios/Onboarding/`, `gaios/Recovery/`, `gaios/GenuineCheck/`, `gaios/ScreenLocker.swift`, `gaios/AppDelegate.swift`, `gaios/SceneDelegate.swift`.
- **Matter least**: `gaios/Assets.xcassets`, `gaios/*.lproj`, `gaios/Rive/`, `gaiosUITests/`, visual layout/theming.

## How to exercise / report findings

- This repository requires macOS/Xcode to build and run XCTest suites (`gaiosTests/`).
- For any finding, provide the exact file path and line numbers, the attacker model and reachability path, and (where possible) a minimal Swift test case in the style of `gaiosTests/` that we can run in Xcode.

## How we rate severity

- **Critical**: Theft of funds or private keys/mnemonics; silent signing of altered destinations, amounts, assets, or fees; Keychain/biometric/PIN lock bypass exposing secrets; arbitrary code execution via deep link, QR, or server data.
- **High**: Deep-link/URL-handler bypass of confirmation flows; hardware wallet verification bypass; Lightning/swap/PSBT logic flaw causing loss of funds; logging or leaking mnemonics/keys/certs.
- **Medium**: App crash/DoS on untrusted input (malformed QR, URI, BLE packet, or backend JSON); unintended privacy leak of addresses/xpubs.
- **Low**: Issues requiring a jailbroken device or an attacker who already holds the unlocked device and passcode.

## Anything to leave alone

- Issues assuming a jailbroken device or physical attacker with the device passcode.
- `lightning/greenlight_device_cert.txt` / test fixtures or non-security UI layout/localization issues.
