# libhoshikv

Native jni/c++ ioctl libs including **FOD, SOFOD, AOD, DT2W, HTSR**, and more xiaomi touch controls to **oplus port with HyperOS vendor** with just 3 SystemUI hooks.

Targets **VNDK34+ / newer AIDL devices**. Works on HyperOS vendor or an AOSP vendor with `mfp-daemon` ported from a HyperOS vendor/ODM.

You can use this on **HIDL** Xiaomi Device for **SOFOD + FODANIM + AOD** by combining trydun fod method + libhoshikv.so patch

READ UNTIL THE END.

Author: **[@hoshikv](https://t.me/hoshikv)** ·
Tg Channel: [@kvports](https://t.me/kvports)

---

## Features

- **FOD** — fingerprint including fingerprint animation
- **SOFOD** — working screen off fod on doze_suspend/doze_brightness
- - **FODANIM** — oplus fodanim connected to xiaomi fingerdown
- **AOD** — always on display/doze brightness without delay
- **DT2W** — connected xiaomi-dt2w to oplus dt2w settings
- **Other Touch** — game, active, aim, expert, edge filter, tap, tolerance — via `persist.sys.kv*` props (`0` = off, `1` = on)
- **HTSR** — forces 500Hz touch report rate `/sys/class/*/*/switch_report_rate`
you need chmod script that i provide there htsrchmod and htsrpermission.rc(optional)

## Requirements

- HyperOS vendor / using `mfp-daemon`
- VNDK34+ or AIDL-based touch/display HAL

## Files

| File | Purpose |
|---|---|
| `libhoshikv.so` | Prebuilt native JNI/ioctl library |
| `classes*.dex` | `hoshi.kvfod` bridge class |
| `vendor_sepolicy.cil` | SELinux rules |
| `htsrchmod` | script for chmod htsr node (optional) |
| `htsrpermission.rc` | rc for runs htsrchmod script 1 time (optional) |


## Installation

**1. Add the library**

Permissions `0644 0 0`, path:

```
/system_ext/lib64/libhoshikv.so
```

**2. Patch SystemUI**

Decompile SystemUI and add the invoke/sput lines.

`SystemUIApplication` OnCreate() method → add this invoke static before `return-void`:

```smali
    invoke-static {}, Lhoshi/kvfod;->initkv()V

    return-void
.end method
```

`OplusBiometricAuthController` → add inside `showUdfpsOverlay`/`hideUdfpsOverlay`, right after `.registers`:

```smali
.method public final hideUdfpsOverlay()V
    .registers 4

    invoke-static {}, Lhoshi/kvfod;->v()V
...

.method public final showUdfpsOverlay(I)V
    .registers 4

    invoke-static {}, Lhoshi/kvfod;->k()V
...
```

`OnScreenFingerprintUIMech` constructor init method → add before `return-void`:

```smali
    sput-object p0, Lhoshi/kvfod;->fpInstance:Lcom/oplus/systemui/biometrics/finger/udfps/OnScreenFingerprintUiMech;
    invoke-static/range {p0 .. p0}, Lhoshi/kvfod;->setFpInstance(Ljava/lang/Object;)V
    invoke-static {}, Lhoshi/kvfod;->init()V

    return-void
.end method
```

Then merge the provided `classes*.dex` into the SystemUI if last dex is classes8.dex add it as classes9.dex

**3. add to build.prop**

Add to `/vendor/build.prop`:

```
ro.hoshikv.support=1
persist.vendor.fingerprint.optical.support=1
persist.vendor.fingerprint.sensor_type=optical
vendor.fingerprint.aidl.support=1
```

**4. Add SELinux policy**

Add at the end of `/vendor/sepolicy.cil`:

```cil
(typeattributeset mlstrustedobject (touchfeature_device))
(allow platform_app touchfeature_device (chr_file (read write open ioctl getattr map)))
(allowx platform_app touchfeature_device (ioctl chr_file (range 0x0000 0xffff)))
(typeattributeset mlstrustedobject (vendor_displayfeature_device))
(allow platform_app vendor_displayfeature_device (chr_file (read write open ioctl getattr map)))
(allowx platform_app vendor_displayfeature_device (ioctl chr_file (range 0x0000 0xffff)))
```

On enforcing, if you don't add this manually, `libhoshikv.so` will get denied — even an automated sepolicy-merge script won't add it for you.

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

Example usage:

```bash
setprop persist.sys.kv* 1/0
```
you can make this props inbuit to settings toggle/your apk

## Notes

**[@hoshikv](https://t.me/hoshikv) (カヴェ kv) — you should give credits if you using this, as FOD+AOD+DT2W fix, put this on somewhere on your changelogs or credits page.**

**using this for paid roms is not allowed except you have permissions from me**

i will notice every single thing if you dont giving credits because it has my name hardcoded on it

Report to me on **[@kvportschat](https://t.me/kvelementarychat)** if you found bugs
