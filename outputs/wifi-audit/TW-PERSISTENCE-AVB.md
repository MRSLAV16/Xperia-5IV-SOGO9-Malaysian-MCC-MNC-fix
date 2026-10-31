# TW persistence and AVB — installed-code findings

Working override is stillTW. No reboot or WiFi restart was performed during this inspection.

Exactinstalled /apex/com.android.wifi/javalib/service-wifi.jar was pulled and disassembled. WifiCountryCode.setOverrideCountryCode writes mOverrideCountryCode then requests update; no store-write in that method. pickCountryCode checks overridefirst, telephonysecond, then conditionaldriver/framework sources, with savedWIFI_DEFAULT_COUNTRY_CODE asfallback. setDefaultCountryCode persists through WifiSettingsConfigStore but cannot outrank activeSIMMY. Focusedcode saved country-persistence-installed-code.txt.

Actual configwriter: /vendor/bin/init.qcom.early_boot.sh lines416–423, copying /vendor/etc/wifi/WCNSS_qcom_cfg.ini to /mnt/vendor/persist/wifi/WCNSS_qcom_cfg.ini and appending gWifi6eCCList from ro.vendor.sony.wlan.6e_cc_list. This countrylist governs6GHzcapability, not the Android override priority.

/vendor and/product are read-onlydevice-mapper mounts; AVBcurrentlygreen. /mnt/vendor/persist isrwseparatepartition butINIgeneratoroverwritesit. CurrentAndroidpersistentconfigstore /data/misc/apexdata/com.android.wifi/WifiConfigStore.xml isuserdata, not a supportedpersistentoverridefile; no editing wasperformed. Do not store a country override there byinventingschemakeys.

Available approach preservingphysicalverifiedpartitions: a rootstartup module invokingcmdwifi force-country-code enabledTW. Itrequiresrootavailableatstartup; the user'slateCVErootandrootfreebootcannotguaranteethat. Existingmoduleautolaunchshouldnotbereenabledblindly. NonrootpersistentTWoverridehasnotbeenestablished. ChangingvendorbootcodeorframeworkAPEXwouldrequireaddressingverifiedpartition/signatureconstraintsandisnotproposedasAVBgreencompatible.

Screenshotuserprovidedreports5GHzchannel153,802.11ax,WPA3and1201/1201Mbpsnegotiatedlinkrate. This isactualclientconnectionevidence, notmeasuredthroughputorproof160MHz. No widthclaimmade.
