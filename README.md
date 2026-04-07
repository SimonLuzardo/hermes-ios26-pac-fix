# Hermes iOS 26 PAC Fix



**Fix by Simon Luzardo** — Indie developer, Houston TX  
🐦 X: [@SimonLuzardoV] https://x.com/SimonLuzardoV  
💼 LinkedIn: https://linkedin.com/in/simon-luzardo-305323252  
📧 Contact: fairpay25@gmail.com 
🏗️ Building: [Stella](https://github.com/SimonLuzardo) — AI chat app | [FairPay](https://fairpay.it.com) — group payments app

---
 
This workaround resolves the iOS 26 crash by recompiling Hermes with arm64e support. The root cause is a React Native threading issue (facebook/hermes#1966), not PAC. This workaround is effective until the official React Native fix is released.

---

## What is this?

A fix for the critical crash affecting **all React Native / Expo apps** on physical iOS 26 devices — iPhones and iPads.

iOS 26 hardened ARM64 PAC (Pointer Authentication Codes) enforcement. Hermes performs raw pointer arithmetic in `PointerBase.h` that violates PAC, causing a `KERN_PROTECTION_FAILURE` crash on every cold start on physical devices.

**Crash signature:**
Exception Type: EXC_BAD_ACCESS (KERN_PROTECTION_FAILURE)
hermesvm JSObject::getNamedDescriptorUnsafe()
hermesvm HiddenClass::findProperty()

## Status

- ✅ Tested on iPhone 17 Pro Max, iOS 26.4
- ✅ React Native 0.83.4 + Expo SDK 54
- ✅ Hermes v0.14.1
- ✅ Stella (production React Native app) running successfully on iOS 26
- ⬜ iPad — should work, not yet tested

## The Fix

In `include/hermes/VM/PointerBase.h`:

1. Added PAC macros using Apple's `ptrauth` intrinsics
2. Fixed `basedToPointerNonNull` to sign returned pointer on arm64e
3. Fixed `pointerToBasedNonNull` to authenticate pointer before offset calculation

## How to use

### Option 1 — Replace hermesvm binary in your project

1. Download `hermesvm` from [Releases](https://github.com/SimonLuzardo/hermes-ios26-pac-fix/releases)
2. Replace the binary in your project:
```bash
cp hermesvm \
  ios/Pods/hermes-engine/destroot/Library/Frameworks/universal/hermesvm.xcframework/ios-arm64/hermesvm.framework/hermesvm
```
3. Build with Xcode targeting your iOS 26 device

### Option 2 — Build from source

1. Clone facebook/hermes at tag `hermes-v0.14.1`
2. Apply the changes from `PointerBase.h` in this repo
3. Build using `utils/build-ios-framework.sh iphoneos`

## Related Issues

- [facebook/hermes#1966](https://github.com/facebook/hermes/issues/1966)
- [expo/expo#44356](https://github.com/expo/expo/issues/44356)

## Support

If this fix helped you, consider:
- ⭐ Starring this repo
- 💬 Sharing with your team
- ☕ [GitHub Sponsors](https://github.com/sponsors/SimonLuzardo)

## License

Based on Hermes engine (MIT License). Fix contributed freely to the community.
