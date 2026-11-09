[![skills.sh](https://skills.sh/b/Sohe-Solutions/flutter-security-guard)](https://skills.sh/Sohe-Solutions/flutter-security-guard)
# 🛡️ Flutter Security Guard (Agent Skill)

**Flutter Security Guard** is a modular skill built under the [skills.sh](https://skills.sh) specification. It is designed to integrate into AI coding agents (such as Claude Code, OpenCode, Gemini, or Cursor) to guide them step-by-step through implementing advanced mobile hardening, anti-decompilation techniques, and network traffic protection in Flutter (Android & iOS).

## 📌 Security Coverage

- 🔩 **Code & Binary Protection:** AOT build flags (`--obfuscate`), debug symbol splitting, and native hardening via ProGuard/R8 and Xcode symbol stripping.
- 🔑 **Secure Secret Storage:** Plaintext elimination using `flutter_secure_storage` (KeyStore/Keychain) and static key obfuscation via byte-shifting with `envied`.
- 🛡️ **RASP (Runtime Self-Protection):** Root/Jailbreak detection, APK signature integrity verification, and dynamic instrumentation blocking (Frida/Xposed).
- 🌐 **Network Security (SSL Pinning):** SPKI Public Key Pinning configuration using `Dio` interceptors alongside native Android `network_security_config.xml` and iOS `Info.plist` policies.
- 🔐 **Application-Layer Encryption (E2E):** AES-256-GCM symmetric payload encryption for HTTP/HTTPS requests to keep sensitive data encrypted even if TLS is bypassed.