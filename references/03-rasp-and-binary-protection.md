# 03 - RASP (Runtime Application Self-Protection) and Anti-Tampering

## Purpose
Detects whether the app is executing in an unsafe or compromised environment (Rooted Android, Jailbroken iOS, active Frida/Xposed hooks, or modified APK signature).

---

## 1. Root and Jailbreak Detection

### Package: `flutter_jailbreak_detection`

#### Code Implementation:

```dart
import 'package:flutter_jailbreak_detection/flutter_jailbreak_detection.dart';
import 'dart:io';

class SecurityCheckService {
  static Future<bool> isEnvironmentSafe() async {
    bool jailbroken = await FlutterJailbreakDetection.jailbroken;
    bool developerMode = await FlutterJailbreakDetection.developerMode;

    if (jailbroken) {
      // Device is rooted or jailbroken
      return false;
    }

    // Additional custom checks for Android
    if (Platform.isAndroid && await _checkAndroidRootPaths()) {
      return false;
    }

    return true;
  }

  static Future<bool> _checkAndroidRootPaths() async {
    final paths = [
      "/system/app/Superuser.apk",
      "/sbin/su",
      "/system/bin/su",
      "/system/xbin/su",
      "/data/local/xbin/su",
      "/data/local/bin/su",
      "/system/sd/xbin/su",
      "/system/bin/failsafe/su",
      "/data/local/su"
    ];

    for (String path in paths) {
      if (File(path).existsSync()) return true;
    }
    return false;
  }
}
```

---

## 2. Frida & Dynamic Instrumentation Detection

Frida attaches to running processes to hook methods and bypass SSL pinning or authentication logic.

### Android Native Frida Check (C / Kotlin)

Check for default Frida ports (`27042`) and loaded Frida libraries (`libfrida-gadget.so` or `/data/local/tmp`).

#### File: `android/app/src/main/kotlin/com/example/app/FridaChecker.kt`

```kotlin
package com.example.app

import java.io.File
import java.net.Socket

object FridaChecker {
    fun isFridaRunning(): Boolean {
        // 1. Check default Frida server port
        try {
            val socket = Socket("127.0.0.1", 27042)
            socket.close()
            return true
        } catch (_: Exception) {}

        // 2. Check loaded maps for Frida artifacts
        try {
            val mapsFile = File("/proc/self/maps")
            if (mapsFile.exists()) {
                val content = mapsFile.readText()
                if (content.contains("frida") || content.contains("gadget")) {
                    return true
                }
            }
        } catch (_: Exception) {}

        return false
    }
}
```

---

## 3. APK / App Integrity & Signature Verification

Detects if an attacker repackaged your app with a different signing key.

### Code Implementation (Android):

```dart
import 'package:package_info_plus/package_info_plus.dart';

class SignatureVerifier {
  // SHA-256 hash of your legitimate release signing certificate
  static const String EXPECTED_SIGNATURE_HASH = "A1:B2:C3:D4:...";

  static Future<bool> verifyAppIntegrity() async {
    PackageInfo packageInfo = await PackageInfo.fromPlatform();
    
    // Ensure package name hasn't been altered
    if (packageInfo.packageName != "com.yourcompany.app") {
      return false;
    }
    
    // Further signature check via MethodChannel to native Kotlin/Swift
    return true;
  }
}
```
