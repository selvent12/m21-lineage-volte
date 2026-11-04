# LineageOS 23.2 with VoLTE for Galaxy M21 (SM-M215F)

Unofficial LineageOS 23.2 build for the Samsung Galaxy M21 (`m21`, Exynos 9611)
with working VoLTE, using the open-source IMS stack by krazey (ImsStack,
ImsMedia, CarrierSettings) instead of Samsung's proprietary IMS.

Tested on Vodafone RO (MCC/MNC 226-01), stock firmware base M215FXXS3CWK1.

## Status

| Feature | State |
|---|---|
| IMS registration (VoLTE) | works |
| Calls over IMS, outgoing and incoming (AMR-WB) | work |
| SMS over IMS, send and receive | works |
| Call audio on loudspeaker | usable, quality can improve |
| Call audio on earpiece | **quiet, poor quality** (in progress) |
| Call audio dropping mid-call | **known issue** (in progress) |

Why the audio needs work: ImsMedia runs call audio on the application
processor. Samsung's audio HAL routes that as a VoIP call
(`MODE_IN_COMMUNICATION`), which does not get the modem's call tuning. The
call path is quieter and unprocessed as a result.

## What changed versus upstream

All changes are on branch `volte` of these forks:

- [android_device_samsung_universal9611-common](https://github.com/selvent12/android_device_samsung_universal9611-common/commits/volte)
  - IMS integration: switch to ImsStack, telephony config and audio policy
    (cherry-picked from betelqeyza)
  - CarrierSettings as standalone package, as the ImsStack README describes
  - Removed an invisible U+200E character from a directory name
  - sepolicy: defined the missing `proc_irq` type
  - Default ImsMedia call gain (`persist.radio.imsmedia.{rx,tx}_gain_db`)
- [android_vendor_samsung_universal9611-common](https://github.com/selvent12/android_vendor_samsung_universal9611-common/commits/volte)
  - `libsec-ril.so` without the SMSC nulling patch. With that patch,
    `GET_SMSC_ADDRESS` returns an empty string and SMS over IMS fails.
- [android_device_samsung_m21](https://github.com/selvent12/android_device_samsung_m21/commits/volte)
  and [android_vendor_samsung_m21](https://github.com/selvent12/android_vendor_samsung_m21/commits/volte)
  - Import the `hardware/samsung` Soong namespace (libteecl)
- [ImsMedia](https://github.com/selvent12/ImsMedia/commits/volte)
  - Optional per-direction digital gain for call audio, set through
    `persist.radio.imsmedia.rx_gain_db` / `tx_gain_db` (0 to 24 dB)

## Build

```bash
mkdir lineage && cd lineage
repo init -u https://github.com/LineageOS/android.git -b lineage-23.2 --git-lfs
mkdir -p .repo/local_manifests
curl -L -o .repo/local_manifests/m21-volte.xml \
  https://raw.githubusercontent.com/selvent12/m21-lineage-volte/main/m21-volte.xml
repo sync -c --no-clone-bundle --no-tags
source build/envsetup.sh
lunch lineage_m21-bp4a-userdebug
m bacon
```

The output is `out/target/product/m21/lineage-23.2-*-UNOFFICIAL-m21.zip`.

## Install

1. **Back up EFS first** (`efs`, `sec_efs`, `cpefs`, `persistent`), for
   example with `dd` from a recovery with adb root. If you lose these
   partitions, you lose IMEI and modem calibration.
2. Unlock the bootloader. This wipes the phone.
3. Flash `recovery.img` (Odin or heimdall), then `vbmeta.img`.
4. In recovery: Factory reset → Format data, then
   `adb sideload lineage-23.2-*-UNOFFICIAL-m21.zip`.

## Tuning call volume without rebuilding

ImsMedia reads the gains when a call's audio starts:

```bash
adb shell setprop persist.radio.imsmedia.rx_gain_db 6   # what you hear
adb shell setprop persist.radio.imsmedia.tx_gain_db 6   # what the other side hears
```

## Credits

- [Exynos9611Development](https://github.com/Exynos9611Development) for the device, vendor and kernel trees
- [krazey](https://github.com/krazey) for ImsStack, ImsMedia and CarrierSettings
- betelqeyza for the ImsStack device integration
- [Macktherealonereally/galaxy-s21-exynos-lineageos-fixes](https://github.com/Macktherealonereally/galaxy-s21-exynos-lineageos-fixes) as a reference for VoLTE on Exynos
