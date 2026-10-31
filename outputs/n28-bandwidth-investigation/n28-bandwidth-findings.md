# n28 bandwidth investigation

## Finding

The live EFS files `/nv/item_files/modem/nr5g/RRC/cap_limit_bw` and `/nv/item_files/modem/nr5g/RRC/cap_limit_bw_Subscription01` were read successfully. Both are 816 bytes and have SHA256 `0005dcc581fb65adb74886120bd2de5798b86708fa910b1b706c6bdfec380e7c`.

Their complete payload is identical to both corresponding cap_limit_bw records embedded in the stock PDX224_K4 hardware MBN on this phone (the subscription record is independently checked in the companion verification). ROW and YTL software-profile inventories do not contain this named override. The restriction therefore has a directly identified stock hardware-profile counterpart; it was not introduced by the Malaysia band-list XML edit.

## Decoding evidence and limits

The live blob contains a value5 at byte44 followed by five24-byte groups beginning at byte48. Their first32-bit values are3,28,41,77,78. The group beginning with28 has next32-bit value2. These observations alone do not establish an official schema.

Cross-check against the user-pasted capability provides independent support for an interpretation as band plus per-SCS bandwidth masks:

| Band | Capability field | Observed corresponding word | Cross-check |
|---|---|---|---|
| n3 |15kHz `30 00` |0x0c|15/20MHz bits correspond |
| n28 |15kHz `40 00` |0x02|10MHz bit set,20MHz bit clear |
| n41 |30kHz `04 00` |0x20|30MHz bit corresponds |

Under TS38.306, FR1 channelBWs-DL base bitmap bits from left represent5,10,15,20,25,30,40,50,60,80MHz. Hence n28 `40 00` (six padding bits unused) represents `0100000000`, with10MHz set and20MHz clear. This conclusion concerns the supplied capability report, not unverified physical RF limits. Other extension fields and requested capability context were not supplied in full.

The Yes n28 SIB1 supplied by the user reports106PRBs at15kHz and initialBWP RIV28875 (using275 as reference RB count, start0/length106). This is consistent with a20MHz carrier and initial bandwidth part. The 10-versus20MHz mismatch is therefore a strong specific explanation for n28 failure, but no controlled live test removing the limit was performed.

## Hardware variant comparison

The stock K4/J4/S4 profiles have apparent n28 word0x02; N4 has0x03. The A4/A2US/C2 profiles have a different wider n28 value0x3490002f, including the low bit corresponding to20MHz under the inferred interpretation. They also differ in many other entries. This does not establish cross-variant RF compatibility or justify flashing another hardware profile.

## Verification status

- Exact live-to-K4 payload equality: verified.
- Original software profiles lack named cap_limit_bw: verified from their record inventories.
- Bit mapping in supplied NR capability: standard-defined.
- Proposed interpretation of proprietary EFS groups: empirically cross-checked, not officially documented.
- Physical support for n28 20MHz on SOG09: not established.
- Successful n28 registration after changing the limit: not tested.

No EFS/NV files, band settings, firmware or boot configuration were written during this investigation. No reboot or root exploit was run. Temporary diagnostic helpers were removed afterward.

## Next step

A fresh full capability exchange on the current configuration would establish whether n28 still advertises10MHz only. A source-verified definition or compatible reference configuration is needed before a narrowly scoped change can be called validated. Any eventual experiment should back up both live files, preserve unrelated bands, verify readback, compare resulting capability and n28 access, and retain an exact rollback. Do not replace the full hardware MBN or delete cap_limit_bw blindly.

Sources: local EFS exports and stock hardware MBNs in this directory; user-pasted capability/SIB1; ETSI TS38.306 https://www.etsi.org/deliver/etsi_ts/138300_138399/138306/16.05.00_60/ts_138306v160500p.pdf ; TS38.104 table5.3.2-1 https://tec.gov.in/pdf/3gpp/29714%202025.pdf
