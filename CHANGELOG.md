# Infinity-X 4.2 — Qualcomm sm7225 platform

## Infinity-X 4.2 — 16 November 2026

### Boot

* Removed the `vendor.lineage.livedisplay` AIDL HAL declaration from `configs/manifest.xml`. The declaration had no backing service: the only livedisplay daemon in the tree is the HIDL 2.0 one, and nothing ever registers `IAdaptiveBacklight`, `IDisplayColorCalibration`, `IDisplayModes` or `ISunlightEnhancement`. The blocking `waitForDeclaredService()` call in `LineageHardwareManager` wedged the `android.display` thread inside `finishBooting()`, which kept user 0 in `BOOTING`, hid the launcher and made the Watchdog restart `system_server` every ~66 seconds.

### SELinux

* `crash_dump` can ptrace `vold` on `user` builds, so vold crashes produce usable tombstones.
* `system_server` can read `binderfs_logs_transactions` on `user` builds, so ANR and watchdog reports include binder transactions.
* Silenced the SystemUI read of `ro.usb.uvc.enabled` with a `dontaudit`; the `appdomain` neverallow still denies it by design.

# LineageOS 24.0 — Qualcomm sm7225 platform

## Highlights

* SELinux now runs fully enforcing on the sm7225 platform, with every denial raised during bring-up resolved in the device trees.
* Builds are signed with private keys, replacing the shared AOSP test keys and closing the security risk that came with them.
* Lockscreen PINs now survive a reboot, and brightness, power hints, thermal reporting and charging all behave as they should.
* The speaker loads its TAS25xx factory calibration again, and brightness recovers after the under-display fingerprint reader's high-brightness mode instead of staying stuck.

## Device trees

* sm7225-common: enabled Always On Display for the A42 panel and turned screen-off fingerprint unlock on by default, so the optical UDFPS reader can authenticate while the display is off.
* sm7225-common: added `seclabel` entries for the Samsung fingerprint, LiveDisplay and vibrator HAL services, so each one starts in its own domain instead of falling back to `init`.
* sm7225-common: added the supporting sepolicy for those HALs, covering audio, bluetooth, camera, fingerprint, keymaster, health, power, sensors, fastbootd, fsck and storage.
* sm7225-common: removed the stale `lights-service.example` entry from the vendor `file_contexts`, which produced duplicate labels for the light HAL and broke image builds.
* sm7225-common: dropped the unused SamsungCallManager package from the product.
* sm7225-common: AOSP Messaging is now built in, so the platform ships a default SMS app instead of only the MMS transport.
* sm7225-common: the lockscreen PIN is now accepted after a reboot. The security service could not read its stored credential, so it enrolled a new key on every boot and every PIN became invalid.
* sm7225-common: fixed the light HAL not being allowed to write the HBM and panel nodes, which could leave brightness and high brightness mode changes unapplied.
* sm7225-common: fixed the power hints pointing at the wrong frequency nodes. Every write failed, so touch and app launch boosts never applied. The big cluster hints now also target the A77 cores instead of a little core, so launch boosts actually reach the performance cluster. This is the biggest scrolling improvement in this build.
* sm7225-common: charger and OTG type detection is fixed, the health HAL can read those nodes again.
* sm7225-common: added the QTI thermal HAL, so the framework can read temperatures and apply thermal limits.
* sm7225-common: the vibrator HAL now reports a real resonant frequency instead of an unsupported value, which removes the invalid frequency profile warning.
* sm7225-common: zram now uses lz4 on every RAM variant, so memory compression costs less CPU under pressure.
* sm7225-common: cleaned up the remaining SELinux denials covering the security service, light and power HAL sysfs writes, GNSS, suspend statistics and early init.
* sm7225-common: allowed the audio HAL to read the TAS25xx calibration files in `/efs/tas25xx`, so the speaker protection and thermal calibration load instead of being skipped.
* sm7225-common: labelled the hall IC wakeup source, closing the remaining suspend-related denial.
* sm7225-common: added a narrow dontaudit for KSUNext's benign /data/local/tmp/tests probe; the test path stays inaccessible to untrusted apps.

## ROM
* a42xq: disabled notification-shade blur with persist.sysui.disableBlur=true; QS animation jank fell from 5.32% to 1.1–1.2% and median frame time from 18 ms to 10–11 ms.

* lineage-sdk: LiveDisplay no longer blocks `system_server` during boot. `LineageHardwareManager` uses non-blocking service lookups instead of waiting indefinitely for a service that never registers, which previously left the device on a black screen after a data wipe.
* SystemUI: the biometric prompt message is no longer drawn underneath the navigation bar on devices whose under-display sensor sits close to the bottom edge of the panel.

## Upstream (LineageOS 24.0)

* Sources synced to LineageOS 24.0 as of 9 November 2026, with the November 2026 Android security patch (`2026-11-01`).
* SystemUI: the profiles tile requests `MODIFY_PROFILES`, so toggling profiles works instead of being silently denied.
* Settings: turning the Bluetooth A2DP and LE audio offload toggles off and choosing reboot now applies the change; the confirmation was being dropped, so the phone only rebooted.
* Contacts: exporting to an existing vCard file truncates it first, so stale contacts from a previous export can no longer be recovered from the file.
* lineage-sdk: profiles use type-checked parcel reads.
* Translation imports for Dialer, Messaging, Jelly, Desk Clock and Updater.
* Security fixes in the media codec (stack buffer overflow in `LWOmxNode::getParameter`) and the input dispatcher (out-of-bounds read and use-after-free in `flushSensor`).

No upstream platform commits landed between the 14/11 and 15/11 builds.

## Kernel

* display: added a five-second watchdog for the UDFPS finger-mask HBM state. If the mask is left on after an unlock, the watchdog clears it so normal brightness updates resume instead of the panel staying at the fingerprint brightness level.

## Build and signing

* vbmeta and recovery are signed with a private 4096-bit AVB key (`keys/avb.pem`), replacing the public `external/avb/test/data/testkey_rsa4096.pem`.
* All APK signing keys (platform, shared, media, networkstack, sdk_sandbox, bluetooth, nfc, cts_uicc_2021) are private.
* Because the signing keys changed, the first flash over a publicly-keyed build requires wiping data.
