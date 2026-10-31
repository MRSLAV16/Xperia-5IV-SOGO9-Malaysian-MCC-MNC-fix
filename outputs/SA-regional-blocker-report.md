# Explicit regional 5G restriction found in live modem EFS

Read-only investigation on 31 October 2026, Sony Xperia 5 IV SOG09, active YTL_Commercial.

## Finding

The live modem file `/policyman/band_set_01.xml` (policy version 44.2.2) contains an Other Roaming country rule that selects `rf_bands_roam`. Malaysia MCC 502 matches all the rule's exclusions: it is outside the configured Japanese, test, FCC, European and local-service MCC groups. The modem's current_mcc and last_mcc both contain little-endian 502; the serving operator property is 502152.

The selected list permits LTE bands but specifies:

```xml
<nr5g_sa_bands base="none" />
<nr5g_nsa_bands base="none" />
<nr5g_nrdc_bands base="none" />
```

The master `/policyman/policies.xml` explicitly loads and executes this band policy for subscriptions. `global:default_bands` in global_defines.xml uses base=current for NR; it does not itself grant extra NR bands. There is no Malaysia/Yes exception in the inspected country lists or this rule. The rule is based on serving country, not the Android SIM roaming icon.

This is an explicit applicable configured blocker for 5G in Malaysia, independent of the user SA band preference and carrier MBN selection. It is consistent with LTE working, accepted NR/n28 preferences and no NR service. Runtime policy execution traces and a removal/retest were not performed, so additional blockers are not excluded and no success after modification is promised.

## What was ruled out or clarified

- DMS Get Band Capabilities advertises NR bands 3, 28, 41, 77, 78. This does not separately prove every SA combination.
- Android carrier configuration reports carrier_nr_availabilities_int_array [1,2] and Sony default-SA flag true.
- The phone's own extphonelib NrConfig constants define 0 as COMBINED_SA_NSA, 1 as NSA and 2 as SA. Thus nr_mode_1=0 is not an SA-disable setting.
- Sony FiveGSaEnableController contains an e-label SA veto, but its e-label check is triggered for model SOG06. This phone identifies as SOG09; that code path is not evidence of an SOG09 veto.
- The live selection database is wrapped with an MCFG header and a valid zlib stream at byte 8. Decompression produces valid XML identical to the earlier backed-up rules; its stored digest matches the SHA-256 of the actual compressed file, rather than the decoded XML. No Yes selection entry was introduced by pinning PDC manually.

## Access and integrity

The policy files were read from the modem itself using the installed libdiag callback interface. A DIAG version request first proved the transport. The EFS reader allowlists only OPEN with flags=0 (read-only), READ and CLOSE, and only the named policy/MCFG files. Each frame's HDLC CRC was checked; read lengths and offsets were validated. Files were not modified. Diagnostic routing was returned to USB mode and the library deinitialized after each run. No diagnostic log masks, EFS contents, selection modes, NV items, band preferences or radio modes were changed during this investigation.

The files were absent from the earlier ZIP export and not mounted as ordinary Android paths, but are readable directly through modem EFS. Absence from that export did not mean the modem lacked them.

## Evidence files

- live-modem-policies/policyman__band_set_01.xml: rf_bands_roam at line 69; empty NR lists at lines 78-80; Other Roaming rule at line 246 and selection at line 259.
- live-modem-policies/policyman__plmn_mcc_supported_01.xml: country groups at lines 10-14.
- live-modem-policies/policyman__policies.xml: loads band_set_01.xml at line 18.
- live-modem-policies/policyman__persisted_items__current_mcc: f6 01 00 00, or 502.
- SA-regional-blocker-evidence.json: hashes and checked matching conditions.

## Next decision

Investigate a narrowly scoped, reversible policy correction for Malaysia while preserving the original rule for other regions. That requires preparing and validating the exact edit, verifying file-write/restore behavior, and accounting for PolicyMan cached state. This investigation did not perform such a change or clear any caches.

