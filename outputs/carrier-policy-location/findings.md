# Carrier policy location

Live /policyman directory was enumerated completely, not only probed: carrier.xml, carrier_policy.xml and carrier_policy.xml_Subscription01 are absent. policies.xml is present (2829 bytes) and explicitly references /policyman/carrier_policy.xml with execute_for=subs,load_for=subs,refresh_on_sim_change=true. Exactdirectreads returnedENOENT2 for allthreecarrierpaths. NoEFSwrites.

Original efs-backup.zip contains both carrier_policy.xml copies, each900bytes SHA2566b1b2bac212caa313003efd32357321d5595d91780a159b4f904ecd0048c0b6b. ROW-live.mbn contains /policyman/carrier_policy.xml recordat its recordedoffset,901bytes: onebyte07recordprefixfollowedbytheexact900byteexportedXML. YTL-live.mbn hasno/policyman/records inparsedinventory. This establishesoriginalpolicy'ssourceinROWandcurrentabsence;doesnotprovetheexactdeletioneventorwhetherpolicymanusesafirmwarefallback.

ExportedXMLpolicynamegeneric,version55.1.8,home/operatorPLMN001-01;actionssetIMS_PREF voicedomainwithconditions. NotestablishedasWiFicountrypolicy. DonotrestoreROWfileintoactiveYTLjustbecausegloballoaderreferencesit. Exactcurrentloaderbehaviorforabsentfile/fallbackremainsunverified.

Evidence:live-directory.txt,live-read.txt,live-policies.xml,backup-carrier_policy.xml,ROW-embedded-carrier-policy.bin. Userclarifiedfileisunderpolicyman;livefilenamecarrier.xmlnotseen.
