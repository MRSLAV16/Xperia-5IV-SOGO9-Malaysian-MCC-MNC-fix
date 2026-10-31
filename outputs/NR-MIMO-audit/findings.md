# NR MIMO / CA read-only audit — 2026-10-31

Scope: seven saved PDX224 hardware MBN variants, ROW/YTL software-profile record inventories, original EFS export, previous UE capability capture, and 29 allowlisted current EFS paths. No modem settings were changed. This is not a complete reverse engineering of modem firmware or numeric RFNV items.

## Current EFS findings

All RRC paths below start with `/nv/item_files/modem/nr5g/RRC/`.

| Item | Live result | Interpretation / limit |
|---|---|---|
| cap_limit_rf_mimo and subscription copy | ENOENT (2), absent | Directly MIMO-named override exists only in J4 among examined hardware profiles (44 bytes). No such live override found; absent does not mean hardware-unlimited. |
| cap_feature_phy and subscription copy | Absent | N4 has a five-byte record; field meanings unverified. |
| cap_limit_bw and subscription copy | Both 816 bytes | Retain known n28 bandwidth patch. No evidence this record sets spatial layer counts. |
| cap_per_plmn_limit_bw and subscription copy | Absent | Present in N4/S4 profiles; no current explicit file override found. |
| cap_control_mrdc_f_plus_t_band_combos and copy | 01 | Matches K4; MR-DC-named combination control, exact bit schema unverified. |
| cap_control_mrdc_t_plus_t_band_combos and copy | 01 01 00 | Matches K4/S4; not a verified SISO switch. |
| cap_control_nrca_f_plus_f / f_plus_t / t_plus_t_band_combos, each with copy | All absent | Other variants contain some records. Absence is not evidence NR CA is disabled; captured capability advertises NR combinations. |
| cap_ctrl_combo_validation | Absent | Subscription01 copy exists: 01 01 01, matching K4/J4/N4/S4. No evidence of a layer limit. |
| llcdb_ampr_info | Absent | A2US profile contains record; schema unverified. |

`/mdb/nr/plmn2cacombos_nr.mdb`: 693 live bytes, exactly stock K4 payload after its one-byte MBN record wrapper. Verified zlib streams yield readable band-combination text. Examples include `n28A[2]A[1]-n78A[4]-n78A[4]` and `n41A[4]-n78A[4]A[2]`. These visibly encode per-combination numbers consistent with layer-related notation, but the proprietary notation, PLMN lookup, whitelist/blacklist role and applicability to Yes are not verified. They must not be interpreted as a universal n28 two-layer limit.

`/mdb/nr/plmn2features.mdb`: 273 live bytes, matches K4/S4 payload after wrapper. Zlib decompression succeeds; binary feature identifiers remain undecoded. No claimed layer mapping.

`/nv/item_files/modem/nr5g/L2/mac/ul_tx_backoff` and copy: 16 bytes, identical across all seven profiles. Name suggests transmit backoff, not proof of a MIMO limit. Power/backoff fields were not modified.

`/nv/item_files/modem/lte/ML1/asdiv_ulca_disable`: 01, identical across all seven profiles. This is an LTE-path item; do not equate it to NR uplink MIMO disable.

`/nv/item_files/modem/lte/ML1/asdiv_per_band_cfg` and `asdiv_ca_band_combo`: absent. The similarly named `.conf` files in the original export merely contain these target paths; they are not antenna configuration payloads.

`/policyman/band_set_01.xml`: current 9353-byte Malaysia band-policy version. Band availability policy is established; a spatial layer-control role is not established.

ROW/YTL do not contain the named NR/MIMO controls identified above. Their absence from named records does not rule out numeric NV items or modem defaults.

## Capability evidence and next diagnostic boundary

The previous filtered UE capability exchange advertised four DL layers for the n78 and n28 feature sets, two UL layers for the n78 feature set and one UL layer for n28, plus n28+n78 and n78+n78 combinations. This is historical advertised capability, not proof of current active rank, antenna health, or four layers in every CA combination. A filtered request can also limit the returned list.

No verified current EFS override forcing NR globally to SISO was found. The modem firmware, numeric RFNV calibration/hardware routing, thermal/coexistence state, network configuration and channel conditions remain outside this decoded file audit. A fresh complete capability exchange and PDSCH/PUSCH layer/rank counters during sustained DL and UL traffic are required to explain current rank 1. Do not substitute CSI-RS port counts or TCI-state counts for spatial layer counts.

Evidence: live-read.txt, live-comparison.json, profile-records.json, backup-records.json, extracted live and profile payloads in this directory. Standard capability definitions: ETSI TS 138 306, sections on band combinations and FeatureSetDownlinkPerCC / FeatureSetUplinkPerCC.
