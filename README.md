# libhoshikv_jni

Native jni/c++ ioctl libs including **FOD, SOFOD, AOD, DT2W**, and full xiaomi touch controls to **oplus port with HyperOS vendor**.

Targets **VNDK34+ / newer AIDL devices**. Works on a HyperOS vendor, or an AOSP vendor with `mfp-daemon` ported from a HyperOS vendor/ODM.

Author: **[@hoshikv](https://t.me/kvports)** · Telegram: [t.me/kvports](https://t.me/kvports)

---

## Features

- **FOD** — Fingerprint including fingerprint animation
- **SOFOD** — working sofod on doze_suspend/doze_brightness
- **AOD** — always on display/doze brightness without delay
- **DT2W** — connected xiaomi-dt2w to oplus dt2w settings
- **Touch suite** — game, active, aim, expert, edge filter, tap, tolerance — via `persist.sys.kv*` props (`0` = off, `1` = on)
- **HTSR** — forces 500Hz touch report rate `/sys/class/*/*/switch_report_rate

## Requirements

- HyperOS vendor/using `mfp-daemon`
- VNDK34+ or AIDL-based touch/display hal

## Files

| File | Purpose |
|---|---|
| `libhoshikv.so` | Prebuilt native library |
| `libhoshikv-sfd.cpp` | Native source |
| `classes*.dex` | `hoshi.kvfod` bridge class |
| `PATCH.smali` | Smali patch for SystemUI |
| `vendor_sepolicy.cil` | SELinux rules |

## Installation

**1. Push the library**
0644 0 0
```
/system_ext/lib64/libhoshikv.so
```

**2. Patch SystemUI**

Decompile SystemUI and put invoke/sput object lines based on `PATCH.smali`

- `SystemUIApplication.onCreate()` → add before `return-void`:
  ```
  invoke-static {}, Lhoshi/kvfod;->initkv()V
  ```
- `OplusBiometricAuthController` → add on show/hideudfpsoverlay after `registers`:
  ```
  # hideUdfpsOverlay()
  invoke-static {}, Lhoshi/kvfod;->v()V

  # showUdfpsOverlay(I)
  invoke-static {}, Lhoshi/kvfod;->k()V
  ```
- `OnScreenFingerprintUIMech` constructor init → add before `return-void`:
  ```
    sput-object p0, Lhoshi/kvfod;->fpInstance:Lcom/oplus/systemui/biometrics/finger/udfps/OnScreenFingerprintUiMech;
    invoke-static/range {p0 .. p0}, Lhoshi/kvfod;->setFpInstance(Ljava/lang/Object;)V
    invoke-static {}, Lhoshi/kvfod;->init()V
  
    return-void
.end method
  ```

Then merge the provided `classes*.dex` into the SystemUI APK, recompile, sign, and push.

**3. Enable the vendor flag**

Add to `/vendor/build.prop`:
```
ro.hoshikv.support=1
```

**4. Add selinux policy**
add on the end of `/vendor/sepolicy.cil`:
```
(typeattributeset mlstrustedobject (touchfeature_device))
(allow platform_app touchfeature_device (chr_file (read write open ioctl getattr map)))
(allowx platform_app touchfeature_device (ioctl chr_file (range 0x0000 0xffff)))
```
on enforcing, if you don't add this manually `libhoshikv.so` will get denied even an automated sepolicy-merge script won't add it for you

## System Properties
| Property | Function |
|---|---|
| `persist.sys.kvhtsrg` | Force 500Hz touch report rate |
| `persist.sys.kve` | Edge filter |
| `persist.sys.kvaim` | Aim mode |
| `persist.sys.kvga` | Game mode |
| `persist.sys.kvex` | Expert mode |
| `persist.sys.kvact` | Active mode |
| `persist.sys.kvto` | Tolerance |
| `persist.sys.kvtap` | Tap mode |

Example of usage:
```bash
setprop persist.sys.kv* 1/0
```

## Notes
**[@hoshikv](https://t.me/kvports)** (カヴェ kv) — credit required if you use this.
