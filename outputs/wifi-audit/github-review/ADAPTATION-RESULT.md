# Xperia Convenience Module adaptation trial

Read full US20251125module from htcmagehoho/Xperia-Convenience-Module. No upstream scripts executed. SourceZIP and module-scripts.txt saved locally.

Applied adaptation live: Sony6e propertyJP->JP,MY; generatedINI countrylistJP->JP,MY; wifi_country_code property and globalsettingMY; frameworkcountryoverrideMY; supportedWiFirecovery. No11be/WiFi7claims or changes, noforeigncountryselection, noaudio,IMS,bootstatus tweaks. NoKSUmoduleinstalled.

ConfirmedlivepropertyJP,MY andnativeMY. Native5GHzSAPchannel list still empty;6GHz5955disabled; nativeMYdomain still contains five2.4/5GHzrules andno6GHzrule. Requested5GHzmax80MHzSoftAPfailed. These results establish this live adaptation didnotfixit; rereadingeveryconfigfieldafterrecoverywasnotindependentlyproved. TheupstreamTW/UScountryforcingwasnottestedandcompatibilitywithSOG09isnotestablished.

Rolledback: INIoriginalSHA2562921b3cbd451ce27158c8cffe35c1b546606828099f9ab334157c07ceae8062a verified;SonypropertyJP;wifi_country_codepropertydeletedandglobalsettingnull;countryoverride disabled;WiFirecoveryrequested;WiFioffrestored;testAPstopped. ADBenabled. Modem/IMSunmodified.

Do not describe this as installing or fully testing the original module. It is a Malaysia-specific WiFi-only adaptation trial, with no lasting unlock.
