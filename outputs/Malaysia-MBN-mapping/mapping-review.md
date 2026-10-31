# Malaysian MBN mapping review

Read-only inventory of installed SOG09 modem bundle and live EFS selection database. No profile selection or EFS changes made during this review.

| SIM operator | Home PLMN evidence | Bundled profile | Carrier index | Decision |
|---|---|---|---:|---|
| Yes | 502-152, live SIM and previously tested rule | YTL_Commercial | 45 | Existing priority mapping, tested SA/IMS/NR call |
| U Mobile | 502-18, profile IMS and ePDG domains mnc018.mcc502 | UMobile_Commercial_Malaysia | 61 | Suitable mapping candidate; needs SIM test |
| Maxis | 502-12, profile IMS/XCAP domains mnc012.mcc502 | Maxis_Commercial | 87 | Suitable mapping candidate; needs SIM test |
| Unifi / legacy TM-P1 | Exact current SIM and profile relation unresolved | TM and P1_Malaysia | 65 / 57 | Do not choose based on directory labels alone |
| Celcom / Digi / MVNOs | Exact SIM identity needed, especially MVNOs | No explicitly named profile found in full installed generic software tree | — | Preserve fallback; do not substitute Yes/Maxis blindly |

## Current precedence

Live database retains PriorityOrder customid first. Our Yes rule matches both customid7 and502-152 before the general KDDI customid7 rule. Maxis502-12 appears only in a lower-priority rule, index87. Preserving European rule text alone does not remove the higher-priority Japan custom-ID problem. To obtain global SIM selection, Japan-forcing rules must be separately removed or qualified using verified syntax; this was not changed in this audit. Do not delete all Japanese profiles or ROW.

## NR voice persistence issue

Stock U Mobile, Maxis and P1 records explicitly set nr5g_full_voice_support to00. The YTL profile omits this item in this bundle, allowing the separately saved01 trial to remain. TM also omits it. Automatic profile changes may therefore reset the NR voice item. Neither mapping nor forcing01 universally establishes support for VoNR, emergency voice or handover.

## Proposed scope

Keep ROW as fallback. Add only evidence-backed SIM-specific rules and test each operator. Preserve non-Japan international mappings, validate precedence, and keep backups. Shared serving PLMNs and roaming networks are not SIM-home identifiers; do not map every broadcast502 PLMN to a carrier MBN. No modem or firmware patch can guarantee every Malaysian SIM/service from this inventory alone.
