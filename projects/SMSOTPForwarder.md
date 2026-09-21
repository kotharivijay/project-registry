# SMS OTP Forwarder

| Field | Value |
|---|---|
| System | HP Laptop (Windows 11) |
| Version | 1.0 (versionCode 1) |
| Status | Working build; SMS-command authorization gap open |
| Package | `com.vijay.otpforwarder` |
| Location | `C:\Users\HP\SMSOTPForwarder` |
| Release APK | `C:\Users\HP\SMSOTPForwarder\app\build\outputs\apk\release\app-release.apk` |
| Last updated | 2026-09-21 |

## Description

Android app that watches incoming SMS and automatically forwards any message
containing an OTP / security code to a configured target number, using the
phone's native SMS (no internet needed). Can also be turned on/off remotely by
sending SMS commands. Built originally for a textile business use case.

## Complete info

**Tech stack:** Java, native Android SDK, SQLite (local log). compileSdk 34,
minSdk 21, targetSdk 34. No third-party backend — fully offline.

**Source components** (`app/src/main/java/com/vijay/otpforwarder/`):
- `MainActivity.java` — UI: target number, on/off toggle, logs, stats.
- `SmsReceiver.java` — broadcast receiver for incoming SMS; keyword detection + forwarding.
- `SmsObserver.java` — content observer path (second way it reads new SMS).
- `SmsForwarder.java` — sends the forwarded SMS (splits long messages).
- `DatabaseHelper.java` — SQLite log of forwarded/failed messages + stats.
- `KeywordsActivity.java` — manage detection keywords.
- `ForegroundService.java` — keeps the app alive in the background.
- `BootReceiver.java` — restarts monitoring after phone reboot.

**Permissions:** RECEIVE_SMS, SEND_SMS, READ_SMS, READ_PHONE_STATE,
FOREGROUND_SERVICE, FOREGROUND_SERVICE_SPECIAL_USE, POST_NOTIFICATIONS,
RECEIVE_BOOT_COMPLETED, REQUEST_IGNORE_BATTERY_OPTIMIZATIONS, WAKE_LOCK.

**Features:**
- Auto-detects OTP/password/verification keywords + 4–8 digit codes.
- Forwards matching SMS to the saved target number; logs every attempt.
- Banking-OTP guard: banking messages skipped unless "force forward all" is on.
- SMS remote control: text `SMS FORWARDER ON` / `OFF` / `STATUS` to the phone;
  it toggles forwarding and replies with the current state.

**Known issues / next steps:**
- ⚠️ **SMS command has no sender authorization** — anyone can text
  `SMS FORWARDER OFF`/`STATUS` and control it or make it reply with the target
  number. A `KEY_AUTHORIZED_NUMBER = "authorizedNumber"` constant exists in
  `SmsReceiver.java` but is never used. Needs a check against an authorized
  number, applied in **both** `SmsReceiver` and `SmsObserver` (logic is duplicated).
- OTP/banking keyword logic is copy-pasted between `SmsReceiver` and
  `SmsObserver` — consider extracting to one shared helper.

**Build / install:**
- Build APK: open in Android Studio → Build > Build APK(s), or `./gradlew assembleRelease`.
- Install (phone connected by USB, debugging on):
  `adb install -r app\build\outputs\apk\release\app-release.apk`.
- After install, on the phone: grant SMS permissions, set target number,
  toggle ON, and set Battery → Unrestricted.

**Other copies:** older version without the SMS-command feature at
`C:\Users\HP\Downloads\SMSOTPForwarder-FIXED\SMSOTPForwarder`.
