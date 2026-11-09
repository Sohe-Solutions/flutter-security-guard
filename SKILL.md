---
name: flutter-security-guard
description: Complete security guide for Flutter mobile applications to prevent reverse engineering, binary decompilation, secret extraction, dynamic analysis (Frida), MITM proxy inspection, and payload tampering. Use when asked to secure a Flutter app or implement mobile hardening.
---
# Flutter Security Guard

This skill provides step-by-step instructions and code implementations for AI agents (Claude Code, OpenCode, Gemini, etc.) to harden Flutter applications against reverse engineering, binary inspection, dynamic tampering, network sniffing, and payload interception.

## Security Architecture Overview

| Layer | Security Objective | Primary Technique | Detailed Reference |
| :--- | :--- | :--- | :--- |
| **1. Code & Binaries** | Prevent code decompilation & symbol extraction | Dart AOT Obfuscation, R8/ProGuard, Symbols splitting | `references/01-obfuscation-and-symbols.md` |
| **2. Secrets & Storage** | Hide API keys & sensitive data | Secure Storage, Envied code obfuscation, FFI | `references/02-secure-storage-and-secrets.md` |
| **3. Application Integrity** | Detect compromised runtime environments | RASP, Root/Jailbreak detection, Frida/Xposed blocking | `references/03-rasp-and-binary-protection.md` |
| **4. Transport Layer** | Prevent MitM proxy sniffing (Burp / Charles) | SSL/TLS SPKI Public Key Pinning, Native NetSec Config | `references/04-ssl-pinning.md` |
| **5. Application Layer** | E2EE payload protection | AES-256-GCM + RSA Hybrid Application Encryption | `references/05-payload-encryption.md` |

## Execution Checklist for Agents

When requested to audit, harden, or implement security in a Flutter application, execute the following steps in order:

1. **Obfuscate Code & Native Binaries**
   - Configure Dart AOT obfuscation parameters in build pipelines.
   - Configure ProGuard/R8 in `android/app/build.gradle` and symbols stripping in Xcode.
   - See `references/01-obfuscation-and-symbols.md`.

2. **Protect Static & Dynamic Secrets**
   - Migrate hardcoded secrets out of Dart files into `envied` with obfuscation.
   - Replace `SharedPreferences` / `UserDefaults` storing sensitive data with `flutter_secure_storage`.
   - See `references/02-secure-storage-and-secrets.md`.

3. **Implement RASP (Runtime Protection)**
   - Add Root and Jailbreak checks at app startup.
   - Add Frida default port and loaded library checks.
   - See `references/03-rasp-and-binary-protection.md`.

4. **Implement SSL Pinning**
   - Extract public key SHA-256 (SPKI) hashes from server certificates.
   - Configure `Dio` / `HttpClient` pinning interceptor with fallback pin support.
   - Configure Android `network_security_config.xml` and iOS `Info.plist` domain keys.
   - See `references/04-ssl-pinning.md`.

5. **Apply End-to-End Payload Encryption (If Required for High Security)**
   - Wrap sensitive HTTP requests in AES-256-GCM payload encryption interceptor.
   - See `references/05-payload-encryption.md`.

## Reference Directory Index
- `references/01-obfuscation-and-symbols.md`
- `references/02-secure-storage-and-secrets.md`
- `references/03-rasp-and-binary-protection.md`
- `references/04-ssl-pinning.md`
- `references/05-payload-encryption.md`
