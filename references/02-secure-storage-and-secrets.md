# 02 - Secure Secret Storage and Dynamic Key Protection

## Purpose
Prevents API keys, JWT tokens, endpoints, and encryption keys from being discovered using static analysis (e.g. running `strings libapp.so`) or stored in plain-text storage (such as `SharedPreferences` or `UserDefaults`).

---

## 1. Dynamic Secret Storage (Runtime)

Use hardware-backed secure storage (`Android KeyStore` and `iOS Keychain`).

### Package: `flutter_secure_storage`

#### Implementation snippet:

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class SecureStorageService {
  static const _storage = FlutterSecureStorage(
    aOptions: AndroidOptions(
      encryptedSharedPreferences: true,
      resetOnError: true,
    ),
    iOptions: IOSOptions(
      accessibility: KeychainAccessibility.first_unlock,
    ),
  );

  static Future<void> saveToken(String token) async {
    await _storage.write(key: 'auth_token', value: token);
  }

  static Future<String?> getToken() async {
    return await _storage.read(key: 'auth_token');
  }

  static Future<void> deleteToken() async {
    await _storage.delete(key: 'auth_token');
  }
}
```

---

## 2. Static Compile-Time Secrets Obfuscation

Never define static keys as plain Dart strings:
```dart
// BAD - Will appear verbatim in binary
const String apiKey = "AIzaSyD-EXAMPLE_KEY_12345";
```

### Using `envied` with Code Obfuscation

#### File: `pubspec.yaml`
```yaml
dependencies:
  envied: ^0.5.0

dev_dependencies:
  build_runner: ^2.4.0
  envied_generator: ^0.5.0
```

#### File: `.env`
```env
API_KEY=AIzaSyD-EXAMPLE_KEY_12345
BASE_URL=https://api.securebackend.com
```

#### File: `lib/env/env.dart`
```dart
import 'package:envied/envied.dart';

part 'env.g.dart';

@Envied(path: '.env', obfuscate: true)
abstract class Env {
  @EnviedField(varName: 'API_KEY', obfuscate: true)
  static final String apiKey = _Env.apiKey;

  @EnviedField(varName: 'BASE_URL', obfuscate: true)
  static final String baseUrl = _Env.baseUrl;
}
```

#### Generate Obfuscated Code:
```bash
dart run build_runner build --delete-conflicting-outputs
```
*How it works:* `envied` encrypts the string into an array of byte operations in `env.g.dart`, preventing simple `strings` extraction from the binary.

---

## 3. C/C++ FFI Secret Hiding (Advanced)

For maximum isolation, embed secrets in a C file compiled into a native `.so` / `.dylib` binary and access them via `dart:ffi`.

### File: `src/secrets.c`
```c
#include <stdint.h>

// XOR encrypted bytes of secret
static const uint8_t enc_key[] = { 0x5F, 0x1A, 0x7B, 0x3C }; 
static const uint8_t xor_mask = 0xAA;

void get_decrypted_key(char* output, int len) {
    for(int i = 0; i < len; i++) {
        output[i] = enc_key[i] ^ xor_mask;
    }
}
```
