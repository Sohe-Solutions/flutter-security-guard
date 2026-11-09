# 01 - Code Obfuscation and Symbol Management

## Purpose
Prevents attackers from extracting human-readable class names, method signatures, properties, and control flow using decompilers or disassemblers (such as Ghidra, IDA Pro, or Jadx).

---

## 1. Dart AOT Obfuscation

Dart compiles to native machine code (`libapp.so` on Android, `App.framework` on iOS). By default, metadata tables retain symbol names for reflection.

### Release Build Commands

#### Android (APK / App Bundle)
```bash
# Generate obfuscated APK and separate debug symbols
flutter build apk --release --obfuscate --split-debug-info=build/app/outputs/symbols

# Generate obfuscated App Bundle (.aab)
flutter build appbundle --release --obfuscate --split-debug-info=build/app/outputs/symbols
```

#### iOS (IPA)
```bash
# Generate obfuscated iOS package
flutter build ipa --release --obfuscate --split-debug-info=build/ios/outputs/symbols
```

### Stack Trace Symbolization
Save the contents of `build/<platform>/outputs/symbols` securely for each release build version. To translate obfuscated stack traces:

```bash
flutter symbolize -i crash_log.txt -d build/app/outputs/symbols/app.android-arm64.symbols
```

---

## 2. Android Native Wrapper Obfuscation (R8 / ProGuard)

Flutter apps contain a native Java/Kotlin Android wrapper (`MainActivity.kt`). R8 must be enabled to shrink and obfuscate native code and dependencies.

### File: `android/app/build.gradle` (or `build.gradle.kts`)

```groovy
android {
    ...
    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

### File: `android/app/proguard-rules.pro`

```proguard
# Preserve Flutter Engine entry points
-keep class io.flutter.app.** { *; }
-keep class io.flutter.plugin.** { *; }
-keep class io.flutter.util.** { *; }
-keep class io.flutter.view.** { *; }
-keep class io.flutter.embedding.** { *; }
-keep class io.flutter.provider.** { *; }
-keep class io.flutter.plugin.editing.** { *; }

# Keep native methods for JNI
-keepclasseswithmembernames class * {
    native <methods>;
}

# Preserve GeneratedPluginRegistrant
-keep class io.flutter.plugins.GeneratedPluginRegistrant { *; }
```

---

## 3. iOS Binary Stripping

In Xcode, ensure debug symbols are stripped from the final binary:

1. Open `ios/Runner.xcworkspace` in Xcode.
2. Select `Runner` target -> **Build Settings**.
3. Set **Deployment Postprocessing** = `YES`.
4. Set **Strip Linked Product** = `YES`.
5. Set **Strip Style** = `All Symbols`.
