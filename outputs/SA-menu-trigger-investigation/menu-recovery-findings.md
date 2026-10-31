# Qualcomm Band Menu recovery investigation

Opening dev.qcom.bandmenu version7.0.0 was followed by recovery from emergency-only50218 with a suspended cellular network to HOME registration onYes502152, n28 ARFCN152670, and connected/validated data. A direct ping returned3/3 packets with average44.968ms. The readable NAS system-selection preferences before/after opening were identical.

The installed native backend matches the local binary exactly: SHA25606a345e50ace458c1eed3af9158befa9ed0dd6f5631b0d28ecba113b220cae1f, backend version5.0.0. Its associated source calls query_nr_independent_capability at startup. That routine sends the existing SA and NSA masks back using NAS SetSystemSelectionPreference0x0033, independent mask TLVs0x2f/0x30 plus duration1. Thus opening the app is not purely read-only, even when the user does not press Apply.

This provides a concrete likely recovery trigger: reapplying existing radio preferences can provoke reacquisition while leaving final preference values unchanged. Startup operations such as subscription binding were not individually isolated, so the probe is not proven the sole cause. Why registration becomes stuck remains unresolved.

A one-time, narrowly scoped post-radio-initialization reapply could be evaluated as a workaround. It should preserve exact current masks, avoid hardcoded service ports, wait for root and the proper SIM, verify readback and registration, and avoid repeated resets or a periodic loop. No such boot automation was installed during this investigation. The known root-loader panic must remain a separate issue rather than being concealed by a retry loop.

Rank1 alone is not a failed-registration indicator. It describes a radio transmission configuration and can coexist with working SA data. These checks establish registration and packet delivery, not maximum achievable MIMO rank or throughput.

No EFS edits, band changes, reboot or manual exploit invocation occurred in this turn. The menu itself may issue its normal same-mask startup writes. Evidence files include before/after NAS frames, verified source excerpt, installed backend binary, registration and ping snapshots.
