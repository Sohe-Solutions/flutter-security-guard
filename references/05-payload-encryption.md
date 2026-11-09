# 05 - End-to-End Application Payload Encryption

## Purpose
Ensures that HTTP request and response bodies remain encrypted with AES-256-GCM at the application layer. Even if an attacker bypasses SSL Pinning using memory hooks (e.g., Frida scripts like `Android-SSL-TrustKiller`), the intercepted network traffic will only reveal encrypted ciphertext.

---

## 1. Encryption Scheme Architecture

```text
[ Client (Flutter) ]                                 [ Backend Server ]
 1. Generate random IV (12 bytes) 
 2. Encrypt JSON payload (AES-256-GCM) 
 3. Send Ciphertext + IV + Auth Tag ----(HTTPS)----> 4. Decrypt payload with AES Key
                                                     5. Process Request
```

---

## 2. AES-256-GCM Implementation

### Package: `encrypt`

```dart
import 'dart:convert';
import 'dart:typed_data';
import 'package:encrypt/encrypt.dart' as enc;
import 'package:crypto/crypto.dart';

class PayloadCipherService {
  final enc.Key key;

  PayloadCipherService.fromBase64Key(String base64Key)
      : key = enc.Key.fromBase64(base64Key);

  /// Encrypts raw JSON map string into encrypted string payload
  String encryptPayload(Map<String, dynamic> jsonData) {
    final plainText = jsonEncode(jsonData);
    
    // Generate secure 12-byte IV for AES-GCM
    final iv = enc.IV.fromSecureRandom(12);
    final encrypter = enc.Encrypter(enc.AES(key, mode: enc.AESMode.gcm));

    final encrypted = encrypter.encrypt(plainText, iv: iv);

    // Format: iv_base64:ciphertext_base64
    return "${iv.base64}:${encrypted.base64}";
  }

  /// Decrypts server response ciphertext back to Map
  Map<String, dynamic> decryptPayload(String encryptedBody) {
    final parts = encryptedBody.split(':');
    if (parts.length != 2) {
      throw FormatException("Invalid encrypted payload format");
    }

    final iv = enc.IV.fromBase64(parts[0]);
    final encryptedData = enc.Encrypted.fromBase64(parts[1]);

    final encrypter = enc.Encrypter(enc.AES(key, mode: enc.AESMode.gcm));
    final decryptedText = encrypter.decrypt(encryptedData, iv: iv);

    return jsonDecode(decryptedText) as Map<String, dynamic>;
  }
}
```

---

## 3. Dio Interceptor Integration

Automatically encrypt outgoing POST/PUT request bodies and decrypt incoming response bodies.

```dart
import 'package:dio/dio.dart';

class EncryptionInterceptor extends Interceptor {
  final PayloadCipherService cipherService;

  EncryptionInterceptor(this.cipherService);

  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    if (options.data != null && options.data is Map<String, dynamic>) {
      // Encrypt outgoing JSON
      final encryptedBody = cipherService.encryptPayload(options.data);
      options.data = {"payload": encryptedBody};
    }
    handler.next(options);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    if (response.data != null && response.data is Map && response.data.containsKey("payload")) {
      // Decrypt incoming payload
      final decryptedData = cipherService.decryptPayload(response.data["payload"]);
      response.data = decryptedData;
    }
    handler.next(response);
  }
}
```
