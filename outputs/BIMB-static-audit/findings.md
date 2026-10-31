# BIMB Mobile static security audit

Analyzed installed package `com.bankislam.bimbmobile`, version 2.8.1 (81), target SDK 35. All five installed APK splits were copied from the phone. No banking account data, app-private databases, credentials, or network tokens were read. No banking screens were opened, requests intercepted, or application code modified.

## Confirmed bytecode evidence

- `classes3.dex`, obfuscated class `v2.W`, method `e`: calls `Settings.Global.getUriFor` and `ContentResolver.registerContentObserver` for `development_settings_enabled`, `adb_enabled`, and `adb_wifi_enabled`. This supports monitoring changes after startup; observer registration alone does not establish the final blocking decision.
- `classes.dex`, class `Y6.g`: contains root-related path/package arrays including su locations and Magisk paths. Executable code reads OEM-unlock-related global settings. These checks are within bundled code; exact ownership and invocation in every banking flow remain unresolved.
- `classes.dex`, class `Z6.a`, method `a(Properties)`: reads configurable `Use-native-rootdetection`, `Enable-Root-Detection-Logging`, and `USE_Play_Integrity` flags. Code presence is not evidence those flags are enabled on this installation.
- `classes.dex`, class `Y6.D`: contains PlayIntegrityCheck retry code. Full server policy and current attestation verdict are not established.
- ARM64 `libapp.so` contains English and Malay warnings referring to Developer Mode, OEM Unlocking, or Bootloader. The Flutter AOT implementation is not fully decompiled in this audit.
- Bundled native iProov library contains root/hooking-related strings. These may serve identity-verification SDK checks rather than BIMB's startup block. Strings alone are not treated as executed checks.

## Phone state after the rootless-use test

- New boot ID: daf02842-e1d0-43d7-bdea-8886a1b55dbe; reported boot reason reboot,userrequested.
- Bootloader locked; AVB green. `/sys/fs/pstore` empty at inspection; this does not exclude every historical or unrecorded fault.
- YTL profile active; manual selection mode 0; IMS registered with voice/SMS capability.
- Mobile data ping 3/3 received; RTT 113/1051/128ms, so connectivity works but latency was variable.
- Root active again. Developer settings and USB ADB flags both 1 during this inspection; earlier banking success in the rootless state does not establish acceptance in this debug-enabled state.
- YTL recovery log shows successful activation this boot. Two startup invocations were logged; a per-boot atomic directory lock was added to prevent concurrent instances. Shell syntax verified. This lock change has not been reboot-tested.

## Limits

This is a partial static audit, not a complete reverse engineering of protected native and Flutter code. It does not prove why a particular banking session was accepted, enumerate all checks, or establish bank-side risk policy. No bypass or patched APK was produced.

## APK hashes

| File | SHA-256 |
|---|---|
| base.apk | `9f2e2127676f42a8fda3ea20274fc414687356d9eaba312c8f738e5665f9ff23` |
| split_config.arm64_v8a.apk | `a61b077b310b9fd0ecaa673f954586f534e096854c00e409552cd7fbababd1a4` |
| split_config.en.apk | `31e99b42de408993a28d25f25a856d81a692fba45f1c6969e799e39cfe52bdbe` |
| split_config.ms.apk | `0b29c23aa5b0cdd2df4cc9ab5f4a42723fc497810ac397151ebffe79011c34fe` |
| split_config.xxhdpi.apk | `e161ce05158e7ccb4d38de4f9133ae914193e86474afa9f61800205761348dec` |
