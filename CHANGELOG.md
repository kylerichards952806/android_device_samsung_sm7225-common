# LineageOS 24.0 — Qualcomm sm7225 platform

## Highlights

* SELinux now runs fully enforcing on the sm7225 platform, with every denial raised during bring-up resolved in the device trees.
* Builds are signed with private keys, replacing the shared AOSP test keys and closing the security risk that came with them.
* Lockscreen PINs now survive a reboot, and brightness, power hints, thermal reporting and charging all behave as they should.

## Device trees

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

## ROM

* lineage-sdk: LiveDisplay no longer blocks `system_server` during boot. `LineageHardwareManager` uses non-blocking service lookups instead of waiting indefinitely for a service that never registers, which previously left the device on a black screen after a data wipe.
* SystemUI: the biometric prompt message is no longer drawn underneath the navigation bar on devices whose under-display sensor sits close to the bottom edge of the panel.

## Kernel

* No kernel changes in this release.

## Build and signing

* vbmeta and recovery are signed with a private 4096-bit AVB key (`keys/avb.pem`), replacing the public `external/avb/test/data/testkey_rsa4096.pem`.
* All APK signing keys (platform, shared, media, networkstack, sdk_sandbox, bluetooth, nfc, cts_uicc_2021) are private.
* Because the signing keys changed, the first flash over a publicly-keyed build requires wiping data.
