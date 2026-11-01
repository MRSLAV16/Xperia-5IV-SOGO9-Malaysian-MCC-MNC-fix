# Sony Xperia 5 IV SOG09: 5G SA, VoNR and Malaysian Carrier Fixes

## Investigation update — 1 November 2026

This update supersedes earlier persistence assumptions where noted. Results apply to the tested SOG09 and firmware; they are not universal carrier compatibility guarantees.

### LTE B40 and carrier aggregation

Live `/policyman/band_set_01.xml` excluded LTE B40 from the Malaysia list. Adding zero-based LTE index **39** to `rf_bands_malaysia`, preserving the NR settings, was followed by reported working B40 service. Fresh capability responses subsequently included B40. Individual band support does not establish support for a particular aggregation combination.

An experimental Malaysia CA rule retained the original 10 entries and added exact entries from global A4/C2 policies, including B20/B40 strings from European operator rules. The added combinations did **not** appear in the captured LTE capabilities. This trial is not a verified CA fix.

A controlled removal probe then removed only the two B3+B1 variants from the Malaysia rule. Two fresh capability responses dropped from **10 to 8 CA entries**, and both B3+B1 variants disappeared. This demonstrates that the edited rule executes. The missing additions are constrained downstream of that rule; the exact control and whether it reflects firmware or RF limitations remain unidentified. The probe was subsequently restored to the exact pre-probe CA trial file with readback verification.

With B20/B40 band locking, responses advertised the bands individually without a CA combination list. Removing the lock restored the original CA entries. No active B20+B40 CA was established. Earlier NR capability exchanges advertised n28+n78 and n78+n78; those advertisements are not proof of active NR CA. No verified NR CA binary patch was produced.

### Malaysian profile selection

The saved selection database now places these exact home-PLMN rules before the ROW fallback:

| SIM home PLMN | Bundled profile | Carrier index | Automatic NR voice repair scope |
|---|---|---:|---|
| 502-152 | YTL | 45 | Included |
| 502-18 | UMobile | 61 | Included |
| 502-12 | Maxis | 87 | Excluded |
| 502-153 | P1 / legacy Unifi profile | 57 | Excluded |

UMobile and P1 were loaded as unmodified stock profiles; Maxis was confirmed already available. The mapping database was backed up and independently read back. UMobile activation was verified, and subsequent field feedback reported YTL/UMobile selection and successful IMS registration. Future swaps and other subscriptions still require verification. The P1 profile is a legacy candidate, not a claim that every current Unifi service is validated.

ROW remains the fallback for unmatched SIMs. No Celcom/Digi-specific profile was identified in the inspected SEA set. The bundled profile named `TM` identifies **Taiwan Mobile**, not Telekom Malaysia, and was not substituted for Unifi. Sub-brands/MVNOs must be matched by home PLMN and, where necessary, SIM-specific identifiers; shared towers or corporate ownership do not prove shared IMS provisioning.

### VoNR reset and root recovery limitations

The live `/nv/item_files/modem/mmode/nr5g_full_voice_support` byte was observed reset to **00** during the SIM/profile-switch investigation. Restoring **01**, with readback and a modem restart, was followed by reported working calls. Both stock ROW and UMobile profiles contain a record for this setting with encoded payload `07 00`; YTL does not contain the same named record. This supports provisioning as a reset source, but the exact writer/timing was not traced. Avoiding ROW alone does not prevent the UMobile profile from provisioning 00.

A root module was developed locally for the tested Yes/YTL and UMobile SIM/profile pairs. The initial implementation had two material problems: a fixed PDC port failed after reboot when port 33 became port 30, and background diagnostic checks could interfere with Cellular-Pro. The fixed-port problem was corrected, but Cellular-Pro later reported missing signalling and UL/DL counters. Disabling the module and restarting diagnostic backends did not immediately restore all features, so the complete cause of that regression remains unresolved.

The current **v1.4 candidate** uses a single guarded late-boot pass, approximately 45 seconds after Android boot completion, then releases DIAG and exits. A boot-completed hook and zygote-session duplicate guard were added for the exploit's soft-reboot flow. There is no continuous diagnostic polling or automatic SIM-change watcher. Manual one-shot execution has been verified; **v1.4 cold-/soft-reboot startup remains unverified** because root was unavailable at the subsequent boot check. This module is not presented here as a production-ready, root-free persistence fix. Module binaries and raw logs from this follow-up are not published with this README update.

Root must exist when recovery runs. Installing a root module does not itself enable exploit autorun or guarantee KernelSU bootstrap. A modem reload may still be needed after repairing the EFS byte. No automatic modem restart was added.

### VoLTE versus VoNR icon

The inspected SystemUI supports a separate VoNR icon controlled partly by CarrierConfig `vonr_enabled_bool`. NR-SA voice/data registration can coexist with a VoLTE icon when that setting is false. A persistent one-key CarrierConfig override changed the visible icon to VoNR in live tests. Overrides are subscription-specific: an override for subscription 1 did not cover subscription 3, and Android rejected attempts to configure inactive subscription records without an assigned phone slot.

The module's one-shot worker handles currently active subscriptions using the persistent CarrierConfig API. Saved values can remain effective without root, but applying values to newly active subscriptions requires another rooted pass. This flag may have telephony consumers beyond the icon; it is not a purely cosmetic guarantee. A VoNR icon, IMS registration, and a successful two-way call are distinct observations. SIP `200 OK` confirms IMS registration only when it is the response to `REGISTER`.

### Exploit and deployment status

The previously verified APK was DirtyFrag v1.16. A later installed v1.17 (versionCode 34) produced a saved successful bootstrap log and working root. The failed autorun attempt's private run log was not recovered before the successful run overwrote it. Boot timing remains a hypothesis, not an established cause. AVB continued to report green; this does not validate the security of the running kernel or establish that runtime changes are harmless.

Official deployment information checked during this investigation:

- [Yes 5G Advanced](https://www.yes.my/yes-5g-advanced/) advertises VoNR.
- [U Mobile SA support](https://www.u.com.my/en/personal/support/self-help/5g-sa) describes SA and VoNR, subject to subscription/device requirements.
- [Maxis SA FAQ](https://www.maxis.com.my/en/faq/network/5g-network/5g-sa/) states that consumer mobile plans remain NSA, while selected business plans support SA. Do not infer universal Maxis VoNR availability.

The observed restrictions establish configuration behaviour, not a motive. Carrier certification, regional testing, product policy and commercial considerations are possible explanations; the evidence does not establish deliberate restriction for profit.

---

**Report date:** 1 November 2026, Asia/Kuala_Lumpur.

**Test platform:** Sony Xperia 5 IV SOG09, KDDI/K4, build `64.2.D.2.261`.

This report documents the investigation of Malaysian 5G SA, IMS, Wi-Fi and modem diagnostics on the Japanese KDDI Xperia 5 IV SOG09. It records applicable restrictions, revision-specific corrections, validation results, rollback procedures and unresolved conditions.

The investigation established n28 and n78 SA data service, n28 operation at 20 MHz, successful NR-only call establishment in recorded trials, and offline NR RRC decoding on the device. These results apply to the examined phone and file revisions; they do not establish complete equivalence with global firmware.

The linked evidence includes intermediate investigation reports. Results reported before a controlled change must be distinguished from subsequent validation. Where the exact recovery sequence or reset writer was not isolated, the report retains that limitation.

### Symptoms covered

This report addresses the Japanese KDDI **Sony Xperia 5 IV SOG09** and the following connectivity symptoms:

- **5G not working / only 4G available** despite n28/n78 coverage.
- **5G SA not detected or not registering**, including MIB/SIB1 reception without registration.
- **VoNR calls failing in NR-only mode** while LTE calling works.
- `ROW_Commercial` selected instead of `YTL_Commercial` for a Yes SIM.
- n28 20 MHz compatibility, carrier-profile persistence and IMS provisioning in Malaysia.

The tested corrections require root and revision-matched modem files. The report separates verified results from unresolved behavior; it is not a universal fix for every Xperia variant or operator.

### Report contents

- [External tools and provenance](#external-tools-and-provenance)
- [1. Device and scope](#1-device-and-scope)
- [2. Interpretation criteria](#2-interpretation-criteria)
- [3. MBN access and analysis](#3-mbn-access-and-analysis)
- [4. Selecting YTL instead of ROW](#4-selecting-ytl-instead-of-row)
- [5. Regional PolicyMan NR block in Malaysia](#5-regional-policyman-nr-block-in-malaysia)
- [6. n28 bandwidth correction](#6-n28-bandwidth-correction)
- [7. Band-menu startup and registration recovery](#7-band-menu-startup-and-registration-recovery)
- [8. IMS, VoNR and the NR voice item](#8-ims-vonr-and-the-nr-voice-item)
- [9. Neutral SIM selection and Malaysian mappings](#9-neutral-sim-selection-and-malaysian-mappings)
- [10. Carrier-policy reference and accessible EFS inventory](#10-carrier-policy-reference-and-accessible-efs-inventory)
- [11. Wi-Fi investigation: actual blockers](#11-wi-fi-investigation-actual-blockers)
- [12. TW override and working 5 GHz hotspot](#12-tw-override-and-working-5-ghz-hotspot)
- [13. Automatic root, reboot and kernel stability](#13-automatic-root-reboot-and-kernel-stability)
- [14. AVB coverage](#14-avb-coverage)
- [15. Global-behavior audit](#15-global-behavior-audit)
- [16. Banking and debugging state](#16-banking-and-debugging-state)
- [17. GNSS and temporary display settings](#17-gnss-and-temporary-display-settings)
- [18. SISO/MIMO investigation](#18-sisomimo-investigation)
- [19. Direct DIAG capture: problems and working procedure](#19-direct-diag-capture-problems-and-working-procedure)
- [20. Diagnostic implementation and SOG09 Signal Lab](#20-diagnostic-implementation-and-sog09-signal-lab)
- [21. Persistence assessment](#21-persistence-assessment)
- [22. Reproducibility and rollback](#22-reproducibility-and-rollback)
- [23. Remaining work](#23-remaining-work)
- [24. Technical references](#24-technical-references)

## External tools and provenance

The investigation used the following external projects in addition to the repository's diagnostic helpers. Installed software, source references and experimental adaptations are distinguished below. Package names are included to avoid confusing similarly named tools.

| Tool or project | Recorded version | Role in this investigation | Upstream material |
|---|---|---|---|
| DirtyFrag | `1.16`, package `df.root`, version code 33 | Temporary root bootstrap for privileged ADB, modem diagnostics and root-module tests | [Exact release](https://github.com/mitschud/DirtyFrag/releases/tag/v1.16), [source revision](https://github.com/mitschud/DirtyFrag/tree/205ceee98612908cc2d52e7529c91498e4fc7969) |
| DFRoot | Engine 4.5, as attributed by DirtyFrag v1.16 release notes | Upstream root-bootstrap implementation; not a second separately verified APK installation | [DFRoot project](https://github.com/diabl0w/DFRoot) |
| ReSukiSU / KernelSU | Installed manager `v4.2.0-rc3`, package `com.resukisu.resukisu`, version code 35171 | Per-application root grants, SU daemon and module management | [ReSukiSU documentation](https://resukisu.org/), [historical project and migration notice](https://github.com/ReSukiSU), [KernelSU](https://github.com/tiann/KernelSU) |
| Qualcomm Band Menu | App `7.0.0`, package `dev.qcom.bandmenu`; examined backend `5.0.0` | RAT/band selection, serving-cell inspection and startup-triggered recovery analysis | [qcom-band-menu](https://github.com/Fronsipswu/qcom-band-menu) |
| Qualcomm EFS Browser | `1.4.7`, package `dev.qcom.efs`, version code 15 | Manual modem EFS export, targeted file installation and backup/restore workflow | [qcom-efs-browser](https://github.com/catxeon/qcom-efs-browser) |
| Cellular-Pro | `2.0.16`, package `make.more.r2d2.play.cellular_pro` | Signal measurements, signalling displays, screenshots and initial exported logs | [Package listing](https://play.google.com/store/apps/details?id=make.more.r2d2.play.cellular_pro); [inspection summary](#20-diagnostic-implementation-and-sog09-signal-lab) |
| Xperia Convenience Module | Examined `US20251125` variant | Wi-Fi country/property implementation reference; a bounded MY adaptation was tested and rolled back | [Upstream project](https://github.com/htcmagehoho/Xperia-Convenience-Module), [adaptation results](outputs/wifi-audit/github-review/ADAPTATION-RESULT.md) |
| Android Platform Tools | ADB; exact installed release not archived | Device connection, binary transfer, package inspection and Android service queries | [Platform Tools](https://developer.android.com/tools/releases/platform-tools), [ADB documentation](https://developer.android.com/tools/adb) |
| Android SDK / NDK | Build Tools `35.0.0`, platform `android-35`, NDK `28.2.13676358` | Compile native helpers, inspect Android packages and build/sign Signal Lab | [SDK Build Tools](https://developer.android.com/tools/releases/build-tools), [NDK](https://developer.android.com/ndk), [Signal Lab source](outputs/SOG09-SignalLab/SOG09-SignalLab-source.zip) |
| libqmi | Schema reference; no CLI release pinned | Interpret QMI NAS/DMS/PDC fields used by the local QRTR helpers | [Primary source and schemas](https://github.com/linux-mobile-broadband/libqmi) |
| SCAT | Parser source reference; not established as a standalone installed capture application | Verify Qualcomm diagnostic framing and NR RRC packet layout | [NR parser](https://github.com/fgsect/scat/blob/master/src/scat/parsers/qualcomm/diagnrlogparser.py) |
| pycrate | `0.8.1` | ASN.1 UPER decoding on the workstation and inside Signal Lab | [Project](https://github.com/pycrate-org/pycrate) |
| BeeWare Android Python support | `3.10-b2` | Embedded ARM64 Python runtime used by the Signal Lab prototype | [Exact runtime release](https://github.com/beeware/Python-Android-support/releases/tag/3.10-b2) |
| AOSP avbtool | Exact source revision not archived | Read-only parsing of active-slot AVB metadata | [Primary source](https://android.googlesource.com/platform/external/avb/+/refs/heads/main/avbtool.py) |

The installed EFS Browser package/version matched its upstream build declarations. That is a package-level identification, not a claim of a byte-for-byte release match. Proprietary Cellular-Pro components were inspected but were not redistributed or incorporated into Signal Lab. The complete Xperia Convenience Module was not installed as the successful Wi-Fi fix.

### DirtyFrag release and vulnerability revision

The archived installed APK matched the official DirtyFrag v1.16 release asset exactly:

```text
Release: v1.16
Release date: 2026-10-15
Source commit: 205ceee98612908cc2d52e7529c91498e4fc7969
Installed APK and official asset SHA-256:
fb0f8e253a0d3c15f01e489922c9a42baef5dd9ce7b61cac8ec0660cfde445d3
Upstream engine attribution: DFRoot 4.5
```

The relevant Android root primitive is **CVE-2026-43284**, the Linux XFRM/ESP shared-page decryption issue. The Linux-assigned CVE record is titled `xfrm: esp: avoid in-place decrypt on shared skb frags`. The reviewed record was published on 8 May 2026 and updated on 15 October 2026.

Sources: [machine-readable CVE record](https://cveawg.mitre.org/api/cve/CVE-2026-43284), [CVE entry](https://www.cve.org/CVERecord?id=CVE-2026-43284), [Android implementation](https://github.com/diabl0w/DFRoot).

The original [Dirty Frag research](https://github.com/V4bel/dirtyfrag) also describes an RxRPC primitive, **CVE-2026-43500**. That separate Linux chain must not be presented as another demonstrated exploit on this Xperia: use of the RxRPC path was not established in the recorded Android tests.

### Recorded root-test platform

```text
Android: 14
Security patch level: 2025-10-01
Kernel: 5.10.236-android12-9-00014-gb97c62c4e7d1-ab13436420
Selected exploit module KMI: android12-5.10
```

These values came from the saved root-run banner. A security-patch label or kernel version alone is not a complete vulnerability test because vendor backports can differ.

DirtyFrag supplied temporary root access. ReSukiSU/KernelSU supplied the root-management environment; neither was the modem compatibility correction. EFS provisioning could survive root-free operation even when the exploit itself had to be rerun after reboot. Automatic re-rooting failed in the controlled startup trial and was not treated as permanent root. No replacement boot image was flashed for this workflow.

The provenance record contains release metadata and identity checks without distributing the root APK: [tool-provenance.json](outputs/tooling/tool-provenance.json). This documentation update did not execute an exploit or change phone provisioning.

## 1. Device and scope

| Item | Details |
|---|---|
| Phone | Sony Xperia 5 IV SOG09 |
| Original customization | Japan / KDDI, K4 |
| Observed build | `64.2.D.2.261` |
| Main SIM | Yes, home PLMN `502-152` |
| Other carrier comparison | U Mobile SIM; Maxis/other profiles inspected |
| Root | Temporary exploit-based root with ReSukiSU/KernelSU management |
| Main tools | ADB, Qualcomm DIAG/libdiag, QMI over QRTR, EFS Explorer, Cellular-Pro, Qualcomm Band Menu, pycrate |
| Main objective | Malaysian SA/IMS compatibility, persistence, Wi-Fi investigation and usable modem diagnostics |

Confirmed outcomes include Yes n28/n78 SA, n28 20 MHz operation, successful NR-only calling in recorded trials, working 5 GHz 80 MHz hotspot and a dedicated rooted signalling app. This did not convert every Japanese-specific Android component into global firmware.

## 2. Interpretation criteria

- Receiving MIB/SIB1 proves reception of a broadcast; it does not prove registration, a working uplink or an active data session.
- `YTL_Commercial` being selected does not itself establish that every policy, band limit and IMS setting is suitable.
- IMS registration does not by itself establish successful calling, VoNR, two-way audio or emergency calling.
- A 5G icon or a “VoLTE” status label is not authoritative evidence of the actual call RAT.
- Advertised capability, network configuration and scheduled spatial layers are different measurements.
- Downlink `4 × 4` displayed alongside rank one does not mean four active layers.
- An EFS read timeout is not equivalent to `ENOENT`/file absence.
- Reboot persistence, root-free operation and firmware-update persistence are separate claims.

## 3. MBN access and analysis

### Problem

Ordinary `adb pull` returned permission denied. A root-assisted copy followed by `chmod` also failed at the permission-change step.

The files of interest were:

```text
/vendor/firmware_mnt/image/modem_pr/mcfg/configs/mcfg_sw/generic/SEA/YTL/Commercial/mcfg_sw.mbn
/vendor/firmware_mnt/image/modem_pr/mcfg/configs/mcfg_sw/generic/common/ROW/Commercial/mcfg_sw.mbn
```

### Method

1. Confirm ADB transport and root independently.
2. Obtain the exact binary files through a root-readable path/export.
3. Compare size, SHA-256, ELF segment boundaries, MCFG records and readable strings.
4. Preserve the originals; do not trim padding, rewrite headers or treat arbitrary bytes as a documented Qualcomm schema.

The following binary-safe extraction pattern avoids Windows text-output conversion. It is an example method rather than a reconstruction of the original export command:

```python
from pathlib import Path
import subprocess

remote = "/absolute/root-readable/path/to/file"
result = subprocess.run(
    [r"C:\platform-tools\adb.exe", "-s", "<ADB_SERIAL>",
     "exec-out", "su", "-c", "cat " + remote],
    check=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE,
)
Path("original.bin").write_bytes(result.stdout)
```

Check the resulting size and hash against an independent read. Do not use text pipelines for MBN or EFS binaries.

### MBN findings

| Profile | Actual file size | SHA-256 |
|---|---:|---|
| YTL | 52,080 bytes | `4f2c91edb9b5860b14d5e6d967e5b7253f5776cf0a1b3150b882830fbef7c3b4` |
| ROW | 58,892 bytes | `bf82d8a85481124dfd30cd7b094bdfe02f533fe4d10fae44881695a8e2962c6e` |

- Both are ELF-wrapped MCFG containers; they are not ordinary Android executables.
- Maximum declared segment ends matched the actual file sizes.
- The MCFG header begins at file offset `0x2000` in these files.
- The local parser identified 96 YTL records and 117 ROW records. Small trailing bytes after the parsed record sequence were not automatically classified as corruption.
- YTL contained a preconfigured URSP policy referencing `yesnet` and `ims`; ROW contained generic `ims`/`sos` policy content.
- A name/string match alone was not treated as proof of SA eligibility.
- The comparison did not independently authenticate Qualcomm signatures or decode every numeric NV record.

## 4. Selecting YTL instead of ROW

### Problem

Sony's service menu initially reported `ROW_Commercial` on slot 0. YTL was available but not automatically selected for Yes.

### Steps performed

1. Discover the current PDC service endpoint instead of assuming its QRTR port is fixed.
2. Query installed, selected, active and pending configurations.
3. Identify the YTL configuration by its returned name, size, version and ID.
4. Select and activate YTL through PDC.
5. Query again to verify the active profile and pending state.
6. Compare registration/data before and after activation.

Recorded YTL identity:

```text
Name: YTL_Commercial
Size: 52080
Version: 0a012d01
Configuration ID: 65589df5df646cc41682cba03103a6460fa8744c
```

### Initial result

YTL activation succeeded and LTE data returned, but an NR-only trial still failed to register. This demonstrated that missing YTL selection was only one part of the problem.

### Early persistence workaround

An early controlled test changed slot-0 PDC selection mode from HYBRID `3` to automatic-selection-disabled `0`, selected YTL and rebooted. YTL remained active afterward.

This pinned the slot and was not the final neutral-SIM solution. A manually pinned profile can be unsuitable after a SIM change. Later work used the selection database and automatic selection instead.

### Rollback principle

Restore the original selection behavior and re-identify the desired stock configuration before activating it. Do not reuse stale service ports or assume a previously observed ROW ID is still the current one. ROW was retained rather than deleted.

Evidence: [initial activation](outputs/YTL-activation-results.txt), [early persistence test](outputs/YTL-persistence-verified.txt).

## 5. Regional PolicyMan NR block in Malaysia

### Confirmed blocker

The live policy file was:

```text
/policyman/band_set_01.xml
```

Its “Other Roaming” country rule applied to Malaysia MCC `502`, which was outside the enumerated Japanese/test/FCC/European/local-service groups. The selected roaming band list contained:

```xml
<nr5g_sa_bands base="none" />
<nr5g_nsa_bands base="none" />
<nr5g_nrdc_bands base="none" />
```

The master `policies.xml` loaded this policy. Live persisted MCC values independently confirmed country 502. This was a country-policy condition, not the Android roaming icon.

### Fix steps

1. Export the original live `band_set_01.xml` through modem EFS.
2. Preserve a byte-for-byte original and hash.
3. Add a Malaysia MCC list and Malaysia-specific band list.
4. Preserve the phone's existing band capabilities instead of enabling every possible band.
5. Add a Malaysia selection rule and exclude Malaysia from the generic NR-empty roaming rule.
6. Validate the XML and compare the exact changes.
7. Write only the target EFS policy using the verified tool/manual EFS workflow.
8. Read it back and compare the complete payload.
9. Refresh/restart the modem as required, then inspect actual registration and data.

The preserved NR band set was n3/n28/n41/n77/n78. Policy list representations use their own verified encoding; do not assume every number inside the XML is a literal 3GPP band number.

### Result

The broader troubleshooting sequence achieved SA registration and data. Later n28 work identified a separate bandwidth restriction. Removing the country block was not sufficient evidence that every remaining modem condition was fixed.

Final recorded policy:

```text
Size: 9353 bytes
SHA-256: a6a097ca3afa9e4ec77ad4a4801dcd3270d591453eea8f194e2af63655f07bc8
```

### Rollback

Restore the matching original policy, read it back and refresh the modem. Do not delete the master policy database or unrelated persisted items.

Files: [final read-back policy](outputs/current-camping-audit/band_set_01.READBACK.xml), [matching prepared policy](outputs/Malaysia-NR-policy-candidate/band_set_01.EXPERIMENTAL.xml), [original policy](outputs/EFS-Malaysia-policy/band_set_01.ORIGINAL.xml), [blocker report](outputs/SA-regional-blocker-report.md). The separate 9155-byte file in `EFS-Malaysia-policy` is an earlier artifact, not the final 9353-byte payload identified above.

## 6. n28 bandwidth correction

### Evidence

The phone could receive n28 broadcasts but was unreliable or unable to register. The captured SIB1 described a 20 MHz carrier: 106 PRBs at 15 kHz. The earlier UE capability advertised a 10 MHz n28 limit.

Two live files matched the stock K4 hardware-profile payload:

```text
/nv/item_files/modem/nr5g/RRC/cap_limit_bw
/nv/item_files/modem/nr5g/RRC/cap_limit_bw_Subscription01
```

Both were 816 bytes. Band-associated groups and capability bitmaps were cross-checked across regional profiles and against [TS 38.306](https://www.etsi.org/deliver/etsi_ts/138300_138399/138306/16.05.00_60/ts_138306v160500p.pdf). This supported a narrow bandwidth-mask interpretation; it was not presented as a complete official schema.

### Exact change on this revision

```text
Zero-based file offset: 76 decimal / 0x4C
Original byte: 02
Modified byte: 0A
```

The change added the empirically identified 20 MHz allowance while retaining the existing bit. It was applied to both corresponding files.

### Fix steps

1. Export both live files.
2. Confirm the exact original length and complete content/hash.
3. Compare them with the saved K4 hardware-profile records.
4. Patch only the verified byte in copies of these exact payloads.
5. Confirm the two patched copies have the expected full hash and no other differences.
6. Install both through modem EFS; the prepared files were also installed through EFS Explorer.
7. Read back both files independently.
8. Refresh/restart the modem and test n28 registration/data.
9. Capture a fresh UE capability exchange and verify that 20 MHz is advertised.

### Integrity checks

| Payload | SHA-256 |
|---|---|
| Original 816-byte files | `0005dcc581fb65adb74886120bd2de5798b86708fa910b1b706c6bdfec380e7c` |
| Modified 816-byte files | `10c7c4a3f5c156644f2fb7d17c6a6cf973bcf3159a8fb4f0982495540d3bb39c` |

### Result

- n28 SA registered successfully during later testing.
- Cellular-Pro reported 20 MHz uplink and downlink.
- A fresh direct capability capture confirmed `supportedBandwidthUL fr1 : mhz20`.
- These results support the bandwidth fix for this phone/file revision.
- They do not establish two-layer uplink or compatibility with a foreign complete hardware MBN.

### Rollback

Restore both original 816-byte payloads from the same backup set, verify their hashes and refresh the modem. Restoring the limit can also restore the original 20 MHz compatibility problem.

Files: [original backups](outputs/n28-20MHz-trial/n28-original-backups.zip), [prepared trial files](outputs/n28-20MHz-trial/n28-experimental-EFS-files.zip), [initial investigation](outputs/n28-bandwidth-investigation/n28-bandwidth-findings.md).

The initial investigation correctly says its own controlled write had not yet happened. Its historical “not tested” status is superseded by the subsequent successful trials and fresh capability capture.

## 7. Band-menu startup and registration recovery

### Symptom

Opening the menu sometimes brought back n28/n78 service without pressing Apply. Final readable band preferences appeared unchanged.

### Investigation

The installed backend matched the saved binary. Its startup capability-query routine also sent the existing SA/NSA masks back through NAS Set System Selection Preference, including independent-mask TLVs and a preference duration.

### Interpretation and result

Opening the menu was not purely read-only. Reapplying the same preferences was a concrete likely reacquisition trigger. Other startup operations were not individually isolated, so it was not proven to be the sole cause.

A recovery observation included HOME registration on Yes n28 and a successful 3/3 ping test. No periodic band-reset loop was established as a permanent fix. The underlying stuck-registration cause remains unresolved.

Evidence: [menu recovery investigation](outputs/SA-menu-trigger-investigation/menu-recovery-findings.md).

## 8. IMS, VoNR and the NR voice item

### Symptom

LTE calling worked, while NR-only calling failed immediately. IMS could remain registered during the failure.

### Investigated setting

```text
/nv/item_files/modem/mmode/nr5g_full_voice_support
```

The tested working payload was the single binary byte `01`, not the text characters “01”. The original/trial payloads were backed up separately. Other profiles explicitly contained `00`, so profile changes can affect this item; that did not prove which writer caused each observed reset.

### Steps used

1. Compare LTE automatic-mode calling with NR-only calling to separate a general IMS failure from an NR voice issue.
2. Verify the active MBN and IMS registration.
3. Export and preserve the current NR voice item.
4. Apply the narrowly scoped `01` trial.
5. Read back the binary byte rather than trusting a UI toggle.
6. Refresh/restart the modem using the working manual EFS/modem workflow.
7. Call an ordinary known number; inspect call establishment and actual cellular RAT.
8. Repeat after a full phone reboot without automatic exploit/root-recovery scripts.

### Results and limits

- A logged call reached ACTIVE with NR radio technology at the start and end; cellular rather than Wi-Fi calling was reported.
- Successful calling after a later root-free reboot was reported.
- Earlier root-free tests had failed and showed the item reverted to `00`.
- Android's supported VoNR setter was also inspected/tested, but an enabled preference did not prevent an earlier failure.
- The exact reset writer and a universally reproducible root-free persistence mechanism were not conclusively isolated.
- Emergency calling, handover and every operator were not validated.

Do not equate a successful ordinary NR call with complete certification of all voice services.

### Rollback

Restore the original byte from that phone's backup and use normal automatic 5G/4G mode if needed for working LTE voice. Do not force NR-only as the only recovery option.

Files: [verified call record](outputs/VoNR-viability/nr-voice-trial/verified-call-result.json), [trial files](outputs/VoNR-viability/nr-voice-trial/NR-voice-trial-file.zip), [backup](outputs/VoNR-viability/nr-voice-trial/NR-voice-trial-backup.zip).

### Digest deletion: tested, but not an established fix

The isolated controlled deletion concerned:

```text
/nv/item_files/mcfg/mcfg_sw_config_digest_version
```

After a manual modem restart, the same 32-byte value regenerated. Those bytes also existed in the stock YTL MBN at offset 52039.

**Conclusion:** its unusual appearance was not corruption evidence. The experiment did not establish it as the persistence culprit or justify further deletion. A working call after this sequence does not isolate the deletion as the cause.

Evidence: [correction](outputs/VoNR-viability/persistence-investigation/TEST-RESULT-CORRECTION.txt).

## 9. Neutral SIM selection and Malaysian mappings

### Problem

Higher-priority Japanese custom-ID rules could override ordinary SIM-specific selection. Merely adding a lower-priority Malaysian rule was insufficient.

### File format

The modem selection database was not a plain XML file despite its suffix:

```text
/nv/item_files/mcfg/mcfg_sel_db.xml
```

It used an MCFG wrapper, a stored length and a valid zlib stream beginning at byte 8. The complete wrapped file and decoded XML were preserved separately.

### Steps performed

1. Back up the complete database plus relevant modem/root-module state.
2. Decode and validate the XML without discarding its wrapper format.
3. Remove the blanket forcing rules identified as `DCM13`, `KDDI7`, `KDDI_SA239` and `SBM28`.
4. Use home-SIM PLMN conditions for the verified Malaysian candidates.
5. Preserve unrelated international rules and ROW fallback.
6. Rebuild the wrapped binary in the original format.
7. Apply it, independently read it back and compare the complete hash.
8. Enable automatic selection and test without root-recovery modules.

| Home SIM PLMN | Profile | Carrier index in this bundle | Validation |
|---|---|---:|---|
| `502-152` | YTL | 45 | Yes service tested |
| `502-18` | U Mobile Malaysia | 61 | Mapping backed by profile data; complete SIM/service retest not performed |
| `502-12` | Maxis | 87 | Mapping backed by profile data; complete SIM/service retest not performed |

Shared serving PLMNs are not necessarily the home SIM identity. Unifi/P1/TM, Celcom/Digi and MVNOs were not blindly mapped to Yes or Maxis.

Final recorded database:

```text
Size: 4023 bytes
SHA-256: 0d6970b188558bf5feda7e3646db8612432d91fd57fc177f86afa48e092ab42a
```

### Result

Automatic YTL selection and SA/IMS registration were observed in root-free testing. One intermediate neutral-selection boot still failed calling and reverted the voice item to `00`; later successful voice tests must not be attributed to the mapping alone.

### Rollback

Restore the original wrapped database, not just its decoded XML. Restore the matching selection mode if it was changed, refresh the modem, then verify the active configuration and real service.

Files: [mapping review](outputs/Malaysia-MBN-mapping/mapping-review.md), [XML diff](outputs/Malaysia-MBN-mapping/neutral-selection.diff), [recovery package](outputs/Malaysia-MBN-mapping/SOG09-neutral-selection-recovery.zip), [backup exclusion record](outputs/PUBLICATION-MANIFEST.json).

## 10. Carrier-policy reference and accessible EFS inventory

The search was expanded beyond guessed paths because `policies.xml` referenced `carrier_policy` while direct queries returned not found.

Steps:

1. Recursively inventory accessible modem EFS directories.
2. Read accessible files using bounded, read-only helpers with CRC/length checks.
3. Search names and content separately.
4. Compare the live results with the original ZIP export and named MBN records.
5. Record inaccessible/protected paths separately from absent ones.

Recorded coverage: 189 directories, 1376 entries, 1188 non-directory entries and 1187 successfully read files, totaling approximately 1.31 MB. Protected areas and one unsupported entry prevented a claim of reading every physical modem file.

Findings:

- No live `carrier.xml`/`carrier_policy.xml` was found at the investigated paths.
- The literal reference appeared in live `policies.xml`.
- The older backup contained a 900-byte carrier policy matching the ROW named record after its wrapper byte.
- YTL lacked corresponding named PolicyMan records in the examined inventory.
- Compiled/default fallback behavior was not fully reverse engineered.

The ROW policy was not blindly restored into YTL.

Evidence: [full search](outputs/efs-carrier-search/findings.md), [location review](outputs/carrier-policy-location/findings.md).

## 11. Wi-Fi investigation: actual blockers

Investigated the installed framework, Sony overlays, driver channel rules, generated INI, SoftAP behavior and runtime country state.

Important paths:

```text
/vendor/bin/init.qcom.early_boot.sh
/vendor/etc/wifi/WCNSS_qcom_cfg.ini
/mnt/vendor/persist/wifi/WCNSS_qcom_cfg.ini
/vendor/etc/wifi/regdb.bin
/vendor_dlkm/lib/modules/qca_cld3_wlan.ko
/product/overlay/ServiceWifiRes-Nagara-Overlay.apk
/apex/com.android.wifi/javalib/service-wifi.jar
```

Findings:

- The early-boot script regenerated the persist INI from the vendor source and appended `gWifi6eCCList` from Sony's property.
- Editing the persist copy alone was therefore not a reliable permanent boot change.
- The MY regulatory/channel state and Sony ACS/DFS behavior limited the available 5 GHz SoftAP channels.
- 6 GHz remained disabled in the tested driver state.
- The framework country override was memory state. A saved default country was only a fallback and did not necessarily outrank SIM-derived MY.

### Xperia Convenience Module review and failed MY adaptation

The upstream scripts were read rather than blindly executed. A bounded Wi-Fi-only adaptation tried JP→JP,MY in the Sony country list, MY framework/property settings and Wi-Fi recovery.

Result: no usable 5 GHz SAP list in that MY trial, 6 GHz still disabled and the requested 5 GHz AP failed. The trial was rolled back.

Original persist INI hash:

```text
2921b3cbd451ce27158c8cffe35c1b546606828099f9ab334157c07ceae8062a
```

The adaptation was not a successful installation of the entire upstream module.

Evidence: [adaptation result](outputs/wifi-audit/github-review/ADAPTATION-RESULT.md).

## 12. TW override and working 5 GHz hotspot

TW was applied as a forced test country. This does not establish that its regulatory domain applies at the operating location.

### Working runtime command

```sh
su -c 'cmd wifi force-country-code enabled TW'
```

### Verification steps

1. Read framework/driver country state.
2. Apply the override with root.
3. Start the normal hotspot and connect a client.
4. Inspect actual SoftAP channel and bandwidth.
5. Compare the client's negotiated connection details.

Verified outcome:

| Item | Result |
|---|---|
| Band/channel | 5 GHz / 153 |
| Frequency | 5765 MHz |
| Wi-Fi protocol | 802.11ax |
| Channel width | 80 MHz |
| Client-reported link rate | 1201/1201 Mbps |

The 1201 Mbps values were negotiated rates, not application throughput. Neither 160 MHz nor 6 GHz was established.

### Root module

Installed module location:

```text
/data/adb/modules/sony_tw_wifi/
```

Version 1.2 used a bounded boot/apply helper with SOG09 checks and disable/remove checks. It applied the runtime country command rather than modifying verified vendor files.

### Bug found and fixed

Directly redirecting `cmd wifi` output into the module log passed that file descriptor through Binder. SELinux prevented `system_server` from appending to the root-module log, causing a failed transaction.

The fix captured command output through a pipe and let the root shell write the log afterward:

```sh
RESULT=$(cmd wifi force-country-code enabled TW 2>&1)
RC=$?
# The root shell writes RESULT to its log after the command returns.
```

No SELinux policy relaxation was required. A manual/module action test verified a successful return and TW state. New startup behavior was not proven by another automatic-root reboot.

### Rollback

Disable/remove the TW module through the root manager and disable the runtime override:

```sh
su -c 'cmd wifi force-country-code disabled'
```

Recheck the resulting native country and channels. A currently active hotspot may need a normal restart before it reflects the new state.

Evidence: [v1.2 fix](outputs/wifi-audit/TW-HELPER-V12-RESULT.md), [persistence analysis](outputs/wifi-audit/TW-PERSISTENCE-AVB.md).

## 13. Automatic root, reboot and kernel stability

The exploit/root bootstrap was separate from modem provisioning. Earlier unexpected reboots and an exploit-related kernel fault were investigated through pstore/Dropbox/kernel records. One saved fault implicated module insertion during the root-loading path.

A controlled single test enabled `df.root`'s boot receiver and rebooted:

1. Boot completed.
2. Verified boot still reported green.
3. Automatic root was unavailable.
4. Wi-Fi country returned to MY.
5. The boot receiver was disabled again.
6. Root was subsequently restored manually without a further reboot.

The manual run overwrote the private last-run log, so the exact automatic-start failure could not be recovered from it. Bounded crash records did not show a new panic, which is not proof that every possible fault was excluded.

Automatic root was left disabled after the trial. Disabled experimental Wi-Fi/YTL recovery modules were marked for removal; the enabled TW module and existing unrelated modules were not indiscriminately deleted.

**Result:** automatic root/TW after boot remains unresolved. Manual root availability and successful modem persistence do not establish reliable exploit auto-start.

Evidence: [controlled boot test](outputs/wifi-audit/AUTO-BOOT-TRIAL-RESULT.md).

## 14. AVB coverage

The active `_b` slot metadata was read and parsed using AOSP avbtool. This was descriptor/mount inspection, not a full independent hash verification of every partition.

| AVB relationship | Partitions/components |
|---|---|
| Main vbmeta direct hashes | `boot`, `dtbo`, `vendor_boot` |
| Main vbmeta hashtrees | `odm`, `vendor`, `vendor_dlkm` |
| Chained metadata | `recovery`, `vbmeta_system` |
| vbmeta_system hashtrees | `product`, `system`, `system_ext` |
| Additional OEM path | Separate verity mapping/key configuration observed; not fully parsed |

Observed state: locked bootloader, green verified boot, enforcing verity, vbmeta flags 0.

Consequences:

- Vendor scripts, vendor Wi-Fi INI/regulatory files, driver module and product overlay are protected paths.
- Editing those partition bytes is not an AVB-preserving solution on this locked trust chain.
- `/data` and persist were outside the descriptors parsed here; that does not mean every modem/persist format is unsigned or unchecked.
- APEX components have additional signature/payload constraints.
- Runtime commands and userdata root modules can leave verified partition bytes unchanged, but require root when applied.

No bootloader unlock, vbmeta patch, disable-verity operation, full global crossflash or protected-partition rewrite was performed as the fix.

Evidence: [AVB audit](outputs/avb-audit/findings.md).

## 15. Global-behavior audit

Read-only inspection identified active KDDI overlays controlling forced camera sound, multi-user UI limits, power-menu/default resources and TCP buffer resources. The actual SMS role was Google Messages despite a KDDI default resource.

The examined global/KDDI profile comparison found selected regional differences, but several tempting files were identical across variants. No blanket KDDI overlay disable or model spoof was applied.

**Result:** specific differences were documented. Full global firmware equivalence, extra RF hardware, eSIM support and universal carrier compatibility were not established.

Evidence: [global comparison](outputs/global-comparison/findings.md).

## 16. Banking and debugging state

Banking operation was reported during a root-free state. A later static BIMB Mobile audit examined installed APKs without opening banking screens, reading credentials/account databases or intercepting requests.

Found code references included:

- Developer-options, USB ADB and wireless ADB setting observers
- Root-related paths/packages
- Configurable native root-detection/Play Integrity flags
- Native warning strings

Code presence did not establish every active check or server-side policy. No bypass, trick, patched banking APK or attestation spoof was produced.

ADB/DIAG were disabled during an earlier banking-oriented test and subsequently enabled for further diagnostics. The earlier disabled state is not the final recorded configuration.

Evidence: [static audit](outputs/BIMB-static-audit/findings.md).

## 17. GNSS and temporary display settings

GNSS was checked with Maps open near a window. Location service activity and weak satellite signals were observed, but no confirmed satellite fix or L5 reception was established during the recorded interval.

Time/PSDS injection requests were issued through the supported location command path; request acceptance was not treated as proof of successful download or fix. No GPS configuration was changed.

A temporary longer screen timeout/stay-awake setting was used for diagnosis and restored to the recorded original timeout of 60,000 ms and stay-awake value 0. Signal Lab later uses an activity-scoped keep-screen-on flag rather than permanently changing the system timeout.

## 18. SISO/MIMO investigation

### Named files inspected

```text
/nv/item_files/modem/nr5g/RRC/cap_limit_rf_mimo
/nv/item_files/modem/nr5g/RRC/cap_feature_phy
/nv/item_files/modem/nr5g/RRC/cap_limit_bw
/nv/item_files/modem/nr5g/RRC/cap_per_plmn_limit_bw
/nv/item_files/modem/nr5g/RRC/cap_control_mrdc_f_plus_t_band_combos
/nv/item_files/modem/nr5g/RRC/cap_control_mrdc_t_plus_t_band_combos
/mdb/nr/plmn2cacombos_nr.mdb
/mdb/nr/plmn2features.mdb
/nv/item_files/modem/nr5g/L2/mac/ul_tx_backoff
/nv/item_files/modem/lte/ML1/asdiv_ulca_disable
```

Subscription copies and other combination controls were also checked. No verified global force-SISO override was found. LTE `asdiv_ulca_disable` was not reinterpreted as an NR MIMO switch. Power/backoff and RF calibration were not modified.

### n28 capability and configured rank

| Evidence layer | Observed result |
|---|---|
| Phone's fresh capability | 20 MHz UL, one PUSCH layer, one SRS port |
| Network setup/reconfiguration | `maxRank 1`, one SRS port |
| Cellular-Pro instantaneous display | Rank 1, Layers Num 1, SISO |

The screenshot also showed poor signal quality, about −3.9 dB SINR, 22 dBm reported PUSCH power and only two UL resource blocks at that instant. It was not a maximum-throughput measurement.

**Conclusion:** one-layer n28 operation is consistent with both current advertised and configured limits. The cause of the phone's advertised limit—RF design, firmware or an unidentified policy—remains unresolved. No validated two-layer n28 patch exists from this work.

### n78 initial configuration and evidence limits

- An earlier capability advertised two UL layers for the n78 feature set.
- A direct n78 capture on Yes ARFCN 627360/PCI 238 showed initial `maxRank 1` and one SRS port.
- Signal was approximately −121 dBm.
- A fresh n78 capability response and later rank-changing configuration were not captured in that interval.

The initial setup is not proof of a permanent one-layer ceiling. Stronger-signal capture during sustained upload remains the next useful test.

Some earlier measurements instead saw U Mobile ARFCN 634080 in emergency-only service while the SIM was Yes. Those were not treated as registered Yes upload tests.

Evidence: [file audit](outputs/NR-MIMO-audit/findings.md), [fresh n28 capability](outputs/NR-MIMO-audit/reconnect-decoded/fresh-UE-NR-Capability.txt), [n78 initial setup](outputs/NR-MIMO-audit/n78-decoded/0016-DL_CCCH_Message.txt).

## 19. Direct DIAG capture: problems and working procedure

### Problems encountered

- Cellular-Pro, EFS tools and Qualcomm Band Menu backends could contend for diagnostic routing.
- A logging-mode call could fail with `-22`.
- A successful connection could still receive nothing because the relevant log mask was zero.
- EFS/decode conflict messages were not assumed to be modem corruption.

### Working capture steps

1. Close competing diagnostic applications and verify their backends are not still running.
2. Load the phone's vendor `libdiag` and initialize a callback connection.
3. Read the current equipment-11 logging mask.
4. Preserve the exact original mask.
5. Temporarily add NR RRC logs B821/B825/B826.
6. Confirm the modem acknowledges the mask change.
7. Capture a bounded interval; reconnecting can elicit setup/capability messages, but a capability response is network-request-dependent.
8. Validate HDLC CRC, wrapper, packet version and declared PDU lengths.
9. Decode only supported layouts using ASN.1 UPER definitions.
10. Restore the original mask, query it again and compare the readback.
11. Return diagnostic routing and deinitialize cleanly.

This workflow records signalling rather than changing EFS provisioning. It does not mean that every packet emitted by this modem is understood.

## 20. Diagnostic implementation and SOG09 Signal Lab

### Inspection findings

The installed Cellular-Pro ARM64 libraries included `libd005`, `libd006` and `libd007` with `uper_decode`/ASN.1-related symbols. That supports a native ASN.1 UPER decoder approach but does not reveal the full architecture.

No proprietary Cellular-Pro decoder libraries, assets or entitlement logic were incorporated in the new application.

### App built

```text
Name: SOG09 Signal Lab
Package: dev.sog09.signallab
Version: 0.1
```

Architecture:

- Java Android UI
- Narrow native libdiag capture helper
- Embedded Python runtime
- pycrate 0.8.1 for offline ASN.1 RRC decoding
- Capture first, decode afterward

### Features

- Root capture restricted by UI model check to SOG09
- 60-second limit, Stop flag and raw-data cap
- Exact logging-mask snapshot, restoration and readback verification
- Recovery snapshot for interrupted captures and a process lock
- NR RRC diagnostic version-17 decoding
- Nested CellGroupConfig and UE-NR capability decoding
- Bandwidth/rank/SRS highlights
- Selectable full-message details
- Raw/JSON/text export through Android's document picker
- Explicit unsupported-PDU labels
- No Internet permission, EFS-writing controls, boot hooks or band-locking controls

### Build/debug steps and fixes

1. Bundle ARM64 Python and pycrate with the app.
2. Compile the capture helper and embedded interpreter runner using the Android NDK.
3. Compile Java, generate DEX, package/align/sign the APK and install through ADB.
4. Replay a known trace to validate decoding on the phone.
5. Fix the missing `pycrate_asn1c` dependency discovered by replay.
6. Add a sanitized capability/reconfiguration sample without subscriber NAS packets.
7. Grant the new app root in ReSukiSU/KernelSU; ADB root permission does not automatically grant app root.
8. Fix app-originated `libdiag unavailable` by explicitly providing the vendor dependency search path:

```sh
LD_LIBRARY_PATH=/vendor/lib64:/system/lib64 /path/to/native/capture-helper OUTPUT STOP_FLAG
```

The library existed. The failure was the app-launched helper's loader environment, not proof that the phone lacked DIAG.

9. Test the Capture button from the actual app, then inspect exported output and restoration status.

### Validation results

| Test | Result |
|---|---|
| APK installation | Passed |
| Embedded Python startup | Passed |
| Original trace replay | 44 decoded messages, zero decoder errors |
| Native stop/restore test | Passed; original mask readback matched |
| App-originated root/DIAG connection | Passed after permission/path fixes |
| App live capture | 10,618 bytes |
| App live output | 46 decoded messages, six explicitly unsupported entries |
| Decoder errors | Zero exceptions in the recorded live test |
| Logging-mask restoration | `RESTORE VERIFIED` |

Unsupported entries are not counted as successfully decoded messages. They remain in the raw capture for future support.

### How to use

1. Enable root access for Signal Lab in ReSukiSU/KernelSU.
2. Close Cellular-Pro/NSG/EFS Explorer/Qualcomm Band Menu.
3. Open Signal Lab and press **Capture 60 seconds**.
4. Wait for **CAPTURE READY** before reconnecting or starting the intended traffic.
5. Keep the app visible. Stop manually or allow the timer to expire.
6. Confirm restoration succeeded.
7. Tap messages to inspect the decoded content.
8. Save the latest session through the document picker.

### Troubleshooting

| Symptom | Check/fix |
|---|---|
| `su` missing | Grant this app root access; check root is actually active |
| DIAG library load failure | Use the updated APK with explicit vendor library path and loader-error reporting |
| Cannot read log mask | Close competing diagnostic apps/backends; retry after they release DIAG |
| Empty capture | Confirm mask acknowledgement, supported log format and that relevant signalling occurred |
| No UE capability message | The network may not request it on every connection; absence is not capability evidence |
| Unsupported PDU/version | Preserve raw capture; do not guess an ASN.1 layout |
| Restoration failed/interrupted | Keep app data/root available; the next capture attempts saved-mask recovery |

### Prototype limitations

No live PHY rank counters, throughput decoder, NAS/LTE decoding, maps or band locking are included. The app uses an archived embedded Python runtime, Android target 28 and a local test signing key. It needs lifecycle/runtime modernization and broader format testing before general distribution. A hard kill/kernel crash cannot guarantee immediate cleanup; the recovery snapshot is a next-run mechanism, not protection against every failure.

Artifacts: [APK](outputs/SOG09-SignalLab/SOG09-SignalLab-0.1.apk), [source archive](outputs/SOG09-SignalLab/SOG09-SignalLab-source.zip), [app instructions](outputs/SOG09-SignalLab/README.md), [hashes](outputs/SOG09-SignalLab/SHA256.json).

## 21. Persistence assessment

| Item | What was demonstrated | What remains unproven |
|---|---|---|
| YTL manual selection | Survived an early reboot | Universal behavior after SIM/firmware changes |
| Neutral SIM database | Readback and root-free SA/IMS selection tests | Every Malaysian SIM and future profile refresh |
| Malaysia band policy | Applied/read back; SA subsequently worked | Firmware-update retention |
| n28 bandwidth patch | Readback and fresh 20 MHz capability/live operation | Other file revisions/phones |
| NR voice byte | Working NR calls, including reported root-free reboot trial | Exact reset writer and unconditional persistence |
| TW override | Manual/root-module runtime application | Root-free or reliable automatic-root startup |
| 80 MHz hotspot | Actual AP/client verification | 160 MHz and 6 GHz |
| Signal Lab | Installed; offline replay and app live capture passed | Modern runtime, broader modem formats and general release readiness |

## 22. Reproducibility and rollback

Before repeating any modem modification:

1. Identify the phone/build and active profile afresh.
2. Export both the current file and original reference; record length/hash.
3. Distinguish Android filesystem paths from modem EFS paths. `/policyman` and modem `/nv` are not ordinary directories to overwrite using `adb push`.
4. Apply only a revision-matched, reviewed payload.
5. Read back the full file independently.
6. Refresh the modem only when needed and record the actual method used.
7. Verify registration, data, call establishment and actual RAT independently.
8. Test full reboot/root-free behavior separately if persistence is the objective.
9. Restore the original file if the isolated test regresses service.

For NR-only failures, returning to the phone's previously working normal 5G/4G mode is a useful service recovery step. Do not delete entire NV/EFS trees, replace RF calibration, flash foreign hardware profiles or erase modem partitions based on this report.

Some historical EFS writes were performed manually through EFS Explorer. No universal modem-restart command is asserted: an Android-supported restart attempt was denied during the investigation, and a manual EFS-tool workflow was used instead. A full phone reboot also re-enters the separate exploit/root-startup behavior.

## 23. Remaining work

- Identify the underlying n28 one-layer capability source without inventing an undocumented EFS switch.
- Capture n78 capability, PUSCH/SRS configuration and actual UL scheduling under stronger signal and sustained load.
- Test U Mobile/Maxis mappings with their actual SIMs and complete voice/data validation.
- Isolate the NR voice reset writer if it recurs.
- Resolve automatic root startup independently from modem provisioning.
- Establish whether 160 MHz/6 GHz is supported by the complete hardware/firmware/regulatory stack.
- Expand Signal Lab with verified packet layouts, NAS/LTE support, modern embedded runtime and robust foreground-service handling.

## 24. Technical references

- [AOSP AVB implementation and documentation](https://android.googlesource.com/platform/external/avb/)
- [ETSI TS 38.306 — UE radio access capabilities](https://www.etsi.org/deliver/etsi_ts/138300_138399/138306/16.05.00_60/ts_138306v160500p.pdf)
- [ETSI TS 38.214 — physical-layer data procedures](https://www.etsi.org/deliver/etsi_ts/138200_138299/138214/16.06.00_60/ts_138214v160600p.pdf)
- [KDDI SOG09 implemented frequency list](https://www.au.com/content/dam/au-com/mobile/product/smartphone/sog09/pdf/sog09_syuuhasu.pdf)
- [libqmi primary schemas](https://github.com/linux-mobile-broadband/libqmi)
- [SCAT Qualcomm diagnostic parser](https://github.com/fgsect/scat/blob/master/src/scat/parsers/qualcomm/diagnrlogparser.py)
- [pycrate](https://github.com/pycrate-org/pycrate)
- [Archived BeeWare Android Python runtime](https://github.com/beeware/Python-Android-support/releases/tag/3.10-b2)

Raw logs and backups can contain subscriber identifiers and network metadata. The published set excludes complete device backups and raw subscriber traces. The publication manifest identifies excluded and redacted material.

### Related SOG09 problem reports

These links provide independently reported symptoms and search context. They do not establish that another device has the same cause or validate every workaround in the discussion.

- [Japanese Xperia 5 IV: 5G not working, only 4G](https://www.reddit.com/r/SonyXperia/comments/1l6mwss/i_have_sony_5_iv_japanese_model_5g_not_working/).
- [Xperia 5 IV SOG09 cannot connect to 5G in France](https://www.reddit.com/r/SonyXperia/comments/1lbt2mk/xperia_5_iv_sog09_wont_connect_to_5g_in_france/).
- [Sony: understanding 5G SA on Xperia smartphones](https://www.sony.ee/electronics/support/articles/00371610).
- [Sony Malaysia: VoLTE support, subscription and software requirements](https://www.sony.com.my/electronics/support/mobile-phones-tablets-mobile-phones/xperia-5-iv-256gb/articles/SX669701).
