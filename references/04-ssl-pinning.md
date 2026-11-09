# 04 - SSL / TLS Pinning and Network Traffic Protection

## Purpose
Prevents attackers from sniffing or tampering with HTTP/HTTPS network traffic using proxies (such as Burp Suite, Charles Proxy, Fiddler, or MitM tools) by validating server public keys.

---

## 1. SPKI Public Key Pinning vs. Certificate Pinning

* **Certificate Pinning:** Pins the entire leaf certificate. Requires app updates whenever the certificate is renewed (e.g. every 90 days with Let's Encrypt).
* **SPKI (Subject Public Key Info) Pinning (RECOMMENDED):** Pins the hash of the server's public key. The certificate can be renewed with the same key pair without breaking the app.

---

## 2. Dio Client SSL Pinning Implementation

### Package: `dio`

```dart
import 'dart:io';
import 'package:dio/dio.dart';
import 'package:dio/io.dart';
import 'package:crypto/crypto.dart';

Dio createSecureDioClient() {
  final dio = Dio();

  // Primary and Backup SPKI SHA-256 Hashes of your backend domain
  const Set<String> allowedSha256Hashes = {
    "7a8f3c12b9...", // Primary Public Key Hash
    "8b9e0f11a2...", // Backup Public Key Hash (for rotation)
  };

  (dio.httpClientAdapter as IOHttpClientAdapter).createHttpClient = () {
    final client = HttpClient();
    
    client.badCertificateCallback = (X509Certificate cert, String host, int port) {
      // Extract SHA-256 of the DER-encoded certificate
      final certSha256 = sha256.convert(cert.der).toString();

      // Check if hash matches allowed set
      bool isMatch = allowedSha256Hashes.contains(certSha256);
      
      if (!isMatch) {
        // Log security alert to remote monitoring
        print("SECURITY ALERT: SSL Pinning failed for host $host");
      }
      
      return isMatch;
    };
    
    return client;
  };

  return dio;
}
```

---

## 3. Native Android Network Security Configuration

Provide an OS-level secondary layer of defence in Android.

### File: `android/app/src/main/res/xml/network_security_config.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">api.yourcompany.com</domain>
        <pin-set expiration="2027-01-01">
            <!-- Primary Pin (SHA-256 of SPKI) -->
            <pin digest="SHA-256">base64_encoded_spki_hash_1=</pin>
            <!-- Backup Pin -->
            <pin digest="SHA-256">base64_encoded_spki_hash_2=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

### File: `android/app/src/main/AndroidManifest.xml`
```xml
<application
    android:networkSecurityConfig="@xml/network_security_config"
    ... >
</application>
```

---

## 4. Native iOS Domain Pinning Configuration

### File: `ios/Runner/Info.plist`

```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <false/>
    <key>NSPinnedDomains</key>
    <dict>
        <key>api.yourcompany.com</key>
        <dict>
            <key>NSIncludesSubdomains</key>
            <true/>
            <key>NSPinnedCAIdentities</key>
            <array>
                <dict>
                    <key>SPKI-SHA256-BASE64</key>
                    <string>base64_encoded_spki_hash_1=</string>
                </dict>
            </array>
        </dict>
    </dict>
</dict>
```
