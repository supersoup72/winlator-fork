# Winlator Box86 Fork

Fork of [brunodev85/winlator](https://github.com/brunodev85/winlator) v11.0
with native Box86 integration for true 32-bit Windows app support on Android.

---

## What's different from upstream Winlator 11.0

| Feature | Upstream 11.0 | This Fork |
|---|---|---|
| 64-bit Windows apps | ✅ box64 + wine64 | ✅ box64 + wine64 |
| 32-bit Windows apps | ⚠️ WoW64 fallback only | ✅ Native box86 + wine32 |
| Installs alongside 11.0 | ❌ same package name | ✅ different package name |
| Package name | `com.winlator` | `com.winlator.box86fork` |

---

## How 32/64-bit selection works

```
Launch game.exe
       │
       ▼
  Read PE header
       │
  ┌────┴────┐
  │ PE32+?  │  → 64-bit → box64 + wine64
  │ PE32?   │  → 32-bit ──┐
  └─────────┘              │
                     box86 available?
                     rootfs32 present?
                           │
                    YES ───┼──→ box86 + wine32 (native 32-bit)
                    NO  ───┴──→ box64 + WoW64 (fallback)
```

---

## Building

### Option A: GitHub Actions (recommended)

1. Fork this repo on GitHub
2. Push to `main` — Actions will automatically:
   - Cross-compile Box86 for ARM32
   - Build the debug APK
   - Upload it as a build artifact (no signing needed for sideloading)
3. Download the APK from Actions → your workflow run → Artifacts

### Option B: Local build

```bash
# Prerequisites: Android Studio, NDK, JDK 17

git clone https://github.com/YOUR_NAME/winlator-box86-fork
cd winlator-box86-fork

# Build debug APK (no keystore needed)
./gradlew assembleDebug

# APK will be at:
# app/build/outputs/apk/debug/app-debug.apk
```

---

## How to integrate with upstream Winlator source

The upstream source is published up to v7.1. To build a full working app:

1. Clone the upstream source: `git clone https://github.com/brunodev85/winlator`
2. Copy all files from `app/src/main/kotlin/com/winlator/` into the upstream source
3. Change `applicationId` in `build.gradle` to `com.winlator.box86fork`
4. Find the container launch code in the upstream (look for where `box64` or `wine`
   is exec'd — likely in `ContainerManager` or `XServerView`)
5. Replace the launch call with `ContainerLauncher(context).launch(exePath)`

---

## 32-bit rootfs setup (rootfs32)

Box86 needs a 32-bit ARM library environment. This is NOT bundled in the APK.

You need to set up `rootfs32` at:
```
/data/data/com.winlator.box86fork/files/rootfs32/
```

The easiest way is to use the rootfs from an older Winlator version (pre-8.0)
that still shipped with 32-bit support, or build one with:

```bash
# On a Linux machine with multiarch support
sudo apt-get install debootstrap
sudo debootstrap --arch=armhf --foreign bullseye ./rootfs32-build
# Then copy to device via adb:
adb push rootfs32-build /data/data/com.winlator.box86fork/files/rootfs32
```

---

## Device compatibility

| Device type | 32-bit support |
|---|---|
| Snapdragon 7xx / older 8xx | ✅ box86 works |
| Snapdragon 8 Gen 1+ (Prime cores) | ⚠️ WoW64 fallback only |
| MediaTek / Exynos / Tensor | Depends on kernel 32-bit support |

---

## Credits

- [brunodev85](https://github.com/brunodev85) — original Winlator
- [ptitSeb](https://github.com/ptitSeb) — Box86 / Box64
- Wine project — Windows compatibility layer
- Mesa / Turnip / VirGL — GPU translation
