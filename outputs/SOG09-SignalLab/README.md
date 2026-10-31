# SOG09 Signal Lab 0.1

An offline, phone-specific prototype for rooted Sony Xperia 5 IV SOG09.

## What works

- Root DIAG capture through the installed Qualcomm libdiag callback interface.
- Reads the current equipment-11 log mask, adds only NR RRC log IDs B821/B825/B826, then restores and reads back the exact previous mask.
- Saves a recovery mask before enabling logging. If interrupted, the next capture first restores the saved mask. A process lock prevents simultaneous captures from this app.
- Stop button and a 60-second capture limit; 64 MiB raw-data cap.
- Local decoding of CRC-validated NR RRC OTA diagnostic packet version 17 using pycrate 0.8.1. Supports BCCH, CCCH, DCCH and paging message containers; decodes nested CellGroupConfig and NR capability containers.
- Selectable full message details, maxRank/SRS-port/capability highlights, raw capture plus JSON/text export through Android's document picker.
- No Internet permission, EFS writes, radio-mode changes, boot hooks or verified-partition changes.

## Use

1. Enable root access for **SOG09 Signal Lab** in ReSukiSU/KernelSU. This manager can hide su from apps which have not been granted access.
2. Close Cellular-Pro, Network Signal Guru, EFS Explorer and Qualcomm Band Menu. Their diagnostic backends can conflict with capture.
3. Open Signal Lab and press **Capture 60 seconds**. Wait for **CAPTURE READY** before reconnecting the radio or running traffic. The app does not toggle airplane mode itself.
4. Keep the app visible. Press Stop or let the timer finish. Decoding runs after capture and may take several seconds.
5. Tap a message to inspect its decoded structure. Save the latest session to Downloads using the document picker.

Raw signalling can contain subscriber and temporary network identifiers. Export only when intended.

## Limits

This is a prototype, not a Cellular-Pro replacement. No live PHY scheduling, MIMO rank counters, throughput measurements, LTE decoder, NAS decoder, band locking or map is included. B825/B826 remain available in raw data but are not decoded. Unknown diagnostic packet versions/PDU IDs are labelled unsupported rather than guessed. CRC-invalid frames are not decoded. A network capability request can filter the capability response; a missing field is not proof of missing hardware support.

Capture requires working root and the SOG09 libdiag interface. Root-free decoding of recorded data works; capture is disabled for other model names. There is no persistent background service; the activity requests that its screen stay awake only while visible. Normal stop/destruction and SIGTERM attempt cleanup; a hard kill, root loss or kernel crash cannot guarantee immediate cleanup. The next capture uses the saved recovery snapshot. Do not clear app storage if recovery is pending.

The bundled embedded Python is BeeWare's archived 3.10-b2 Android support runtime, used to prove offline decoding on this device. This is a local development build, signed with a local test key, with Android target 28 for that runtime. A maintained modern runtime, foreground-service lifecycle, broader modem format support and a security review are needed before general distribution.

## Validation on the user's SOG09

- APK installation and embedded interpreter startup passed.
- Original 39,322-byte n28 trace decoded on-device: **44 RRC messages, 0 errors**. NR capability confirms 20 MHz and one UL layer; setup/reconfiguration shows maxRank 1.
- Bundled replay uses only two sanitized RRC packets (capability and reconfiguration); timestamp/cell header identifiers were removed. No subscriber NAS packets are included in the APK fixture.
- Native capture backend connected, accepted the temporary mask, responded to stop, and printed **RESTORE VERIFIED** after comparing the readback with the original mask. Updated snapshot-capable backend also passed a short start/stop restoration test.
- App-originated root capture reached CAPTURE READY after setting LD_LIBRARY_PATH=/vendor/lib64:/system/lib64 for the native helper. App-originated live capture completed: 10,618 bytes; 52 NR entries including explicitly unsupported PDU types; zero decoder exceptions. Original mask restoration readback passed.

## Source and build

`app/MainActivity.java`, `app/capture.c`, `app/pythonrunner.c`, `app/assets/decode.py`, `app/AndroidManifest.xml`, and `build.py` are the application source. The source archive includes the offline runtime and native libraries so the test build can be reproduced locally. Configure the SDK/NDK/JDK paths in build.py if needed. Run `python build.py` from the original workspace or adjust its output path. A new local signing key is generated if absent; the private key used on this phone is not included in the source archive.

## Third-party components and inspection

- pycrate 0.8.1, LGPL-2.1-or-later: https://github.com/pycrate-org/pycrate . License included in assets/licenses. The bundled pure Python files are readable source and may be replaced/repacked.
- CPython and BeeWare Android support 3.10-b2: https://github.com/beeware/Python-Android-support/releases/tag/3.10-b2 . Python license included. Upstream archive includes its native dependency libraries, including OpenSSL 1.1, SQLite, libffi, libbz2 and liblzma. See their upstream licenses before redistribution.
- Qualcomm NR diagnostic layout reference: https://github.com/fgsect/scat/blob/master/src/scat/parsers/qualcomm/diagnrlogparser.py . The app only decodes the locally verified version-17 layout and checks packet lengths/CRC.
- Cellular-Pro 2.0.16 was inspected locally. Its ARM64 libraries libd005/libd006/libd007 contain uper_decode and ASN.1-related symbols. This supports identifying an ASN.1 UPER native-decoder approach, but does not establish its complete architecture. No Cellular-Pro code, decoder libraries, assets, entitlement checks or subscription bypasses are incorporated in Signal Lab. The APKs are not included in the source archive.
