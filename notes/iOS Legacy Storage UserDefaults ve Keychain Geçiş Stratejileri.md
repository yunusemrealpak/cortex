---
title: 'iOS Legacy Storage: UserDefaults ve Keychain Geçiş Stratejileri'
type: note
tags:
  - ios
  - flutter
  - migration
  - userdefaults
  - keychain
  - legacy-storage
  - native-interop
summary: >-
  Native iOS uygulamasından Flutter'a geçişte UserDefaults (standard ve App
  Group) ile Keychain verilerinin nasıl taşınacağını ve her yöntemin davranış
  farklarını açıklayan diyagram.
relatedTerms: []
created: '2026-03-11T10:18:12.027Z'
updated: '2026-03-11T10:18:12.027Z'
imageFiles:
  - ios_legacy_store.png
---
# iOS Legacy Storage: UserDefaults ve Keychain Geçiş Stratejileri

![[ios_legacy_store.png]]

> Native iOS uygulamasından Flutter'a geçişte UserDefaults (standard ve App Group) ile Keychain verilerinin nasıl taşınacağını ve her yöntemin davranış farklarını açıklayan diyagram.

## İçerik

## iOS Legacy Storage

Native iOS uygulamasından Flutter'a geçiş senaryolarında eski depolama mekanizmalarındaki verilerin aktarılması kritik bir adımdır. Üç farklı yöntem mevcuttur:

---

### 1. UserDefaults — Standard

```swift
UserDefaults.standard
```

- Uygulama silinince veri **silinir**.
- Native kod ile okunup Flutter'a taşınabilir; ancak bu senaryo (TestFlight ile mağazadaki uygulamanın üzerine güncelleme) **test edilemez**.

---

### 2. UserDefaults — App Group

```swift
UserDefaults(suiteName: "group.xxx")
```

- Uygulama silinince veri **kaybolmaz**.
- App Group'u kullanan **son cihazdan uygulama kaldırılana kadar** UserDefaults silinmez; Apple Sandbox'ta tutulur.
- Native kod ile veri okunup Flutter'a aktarılır. Flutter tarafında kayıt altına alındıktan sonra **native taraftaki key silinir**.

---

### 3. Keychain

```swift
SecItemCopyMatching(...)
```

- Uygulama silinse bile veri **silinmez**.
- **Team ID ve Bundle ID'nin aynı olduğu** senaryoda Flutter uygulamasından erişim sağlanır.
- Native kod ile veri okunup Flutter'a aktarılır. Flutter tarafında kayıt altına alındıktan sonra **native taraftaki key silinir**.

---

### Geçiş Akışı (Genel)

```
Native Storage
    └── Native Plugin (MethodChannel)
            └── Flutter → Yeni Storage'a Yaz → Native Key'i Sil
```

> **Not:** Standard UserDefaults ile yapılan geçiş, TestFlight ortamında doğrulanamaz. Güvenli geçiş için App Group veya Keychain tercih edilmelidir.

## Önemli Noktalar

- Standard UserDefaults uygulama silinince kaybolur; geçiş testi TestFlight ile yapılamaz.
- App Group UserDefaults, son cihazdan kaldırılana kadar Apple Sandbox'ta korunur.
- Keychain verisi uygulama silinse dahi cihazda kalır; aynı Team ID + Bundle ID ile erişilebilir.
- Veri Flutter'a aktarıldıktan sonra native taraftaki key temizlenmelidir.
- Geçiş mekanizması için MethodChannel aracılığıyla native plugin yazılması gerekir.
