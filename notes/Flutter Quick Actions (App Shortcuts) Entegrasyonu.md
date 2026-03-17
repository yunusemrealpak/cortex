---
title: Flutter Quick Actions (App Shortcuts) Entegrasyonu
type: note
tags:
  - flutter
  - quick_actions
  - app-shortcuts
  - android
  - ios
  - emergency
  - biz1iz
  - F06
summary: >-
  quick_actions paketi ile uygulama ikonuna uzun basınca OS-level shortcut
  menüsü ekleme — biz1iz_flutter'da F06 kapsamında acil durum bildirimi
  shortcut'ı olarak implement edildi.
relatedTerms: []
created: '2026-03-11T23:16:23.040Z'
updated: '2026-03-11T23:16:23.040Z'
---
# Flutter Quick Actions (App Shortcuts) Entegrasyonu

> quick_actions paketi ile uygulama ikonuna uzun basınca OS-level shortcut menüsü ekleme — biz1iz_flutter'da F06 kapsamında acil durum bildirimi shortcut'ı olarak implement edildi.

## İçerik

## Paket

```yaml
quick_actions: ^1.0.1  # Flutter team official
```

## Nasıl Çalışır

`quick_actions` paketi, iOS (Haptic Touch / 3D Touch) ve Android (long-press app icon) üzerindeki OS-level shortcut menüsünü yönetir.

### Temel API

```dart
const QuickActions quickActions = QuickActions();

// 1. Handler'ı kaydet + mevcut pending shortcut'ı yakala
quickActions.initialize((String shortcutType) {
  // shortcut'a tıklandığında çağrılır
});

// 2. Shortcut item'larını tanımla
quickActions.setShortcutItems([
  const ShortcutItem(
    type: 'emergency_notification',   // unique identifier
    localizedTitle: 'Acil Durum Bildirimi',
    icon: 'ic_emergency_shortcut',    // platform icon adı
  ),
]);

// 3. Temizleme (gerekirse)
quickActions.clearShortcutItems();
```

### Platform Icon Kuralları

| Platform | Icon Formatı | Konum |
|----------|-------------|-------|
| Android | Drawable resource adı (uzantısız) | `android/app/src/main/res/drawable/` |
| iOS | Asset catalog image adı veya SF Symbol | `ios/Runner/Assets.xcassets/` |

## biz1iz_flutter Implementasyonu

### Mimari

```
lib/src/core/services/app_shortcuts_service.dart   ← Facade / wrapper
lib/src/features/app/presentation/pages/app_page.dart  ← Handler'ı register eden yer
android/app/src/main/res/drawable/ic_emergency_shortcut.xml  ← Android icon
```

### AppShortcutsService

```dart
class AppShortcutsService {
  static const String emergencyShortcutType = 'emergency_notification';
  static const QuickActions _quickActions = QuickActions();

  static void initialize(void Function(String) onShortcutTriggered) {
    try {
      _quickActions.initialize(onShortcutTriggered);
      _quickActions.setShortcutItems([
        const ShortcutItem(
          type: emergencyShortcutType,
          localizedTitle: 'Acil Durum Bildirimi',
          icon: 'ic_emergency_shortcut',
        ),
      ]);
    } catch (e) {
      // Sessizce geç — shortcut desteklenmeyen platformda crash olmaz
    }
  }
}
```

### AppPage.initState'e Entegrasyon

```dart
@override
void initState() {
  super.initState();

  // Shortcut handler'ı register et
  // initState'te çağrılır çünkü:
  //   - Auto-login sonrası AppPage'e navigate edilmiş olur
  //   - Cold-start shortcut'ı burada yakalanır (initialize callback'i pending shortcut'ı tetikler)
  AppShortcutsService.initialize(_handleAppShortcut);

  // ... diğer initState işlemleri
}

void _handleAppShortcut(String shortcutType) {
  if (shortcutType == AppShortcutsService.emergencyShortcutType) {
    // Dialog göstermeden önce ilk frame'in render edilmesini bekle
    WidgetsBinding.instance.addPostFrameCallback((_) {
      if (mounted) _onEmergencyLongPress();
    });
  }
}
```

### Android Vector Drawable (`ic_emergency_shortcut.xml`)

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp" android:height="24dp"
    android:viewportWidth="24" android:viewportHeight="24">
  <path android:fillColor="#E2312E"
        android:pathData="M12,2L1,21h22L12,2zM13,18h-2v-2h2v2zM13,14h-2L11,9h2v5z"/>
</vector>
```

## Kritik Detaylar

### initialize() nerede çağrılmalı?

`initialize()` hem handler'ı register eder hem de varsa **pending shortcut**'ı tetikler.
Bu yüzden **navigation tamamlandıktan sonra** çağrılmalı:

- ✅ `AppPage.initState` — auto-login sonrası buraya gelinmiş olur
- ❌ `main()` / `ApplicationInit` — kullanıcı henüz login olmamış, context yok
- ❌ `SplashPage` — navigate edilmeden dialog gösterilemez

### Cold Start vs Hot Start

| Senaryo | Davranış |
|---------|---------|
| **Cold start** (uygulama kapalıyken shortcut) | `initialize()` çağrıldığı an callback tetiklenir |
| **Hot start** (uygulama açıkken shortcut) | `initialize()` callback'i saklar, shortcut tıklanınca tetiklenir |

### addPostFrameCallback Neden Gerekli?

`initialize()` `initState` içinde çağrılır. Eğer cold-start shortcut callback'i anında tetiklenirse, widget henüz build edilmemiştir — `showDialog` çağrılamaz.
`addPostFrameCallback` ilk frame'den sonra çalıştığı için güvenlidir.

### Shortcut UX Tasarımı

Acil durum shortcut'ı, SOS butonuna uzun basmayla birebir aynı dialog'u açar (`_EmergencyCountdownDialog`):
- 5 saniyelik geri sayım
- Heartbeat ses efekti
- Konum bilgisi alınması
- `EmergencyStuffBloc` üzerinden bildirim gönderimi (`status: 'isOther'`, note: `'SOS - SİSTEM UYARISI'`)

## Yeni Shortcut Eklemek İçin

1. `AppShortcutsService.emergencyShortcutType` benzeri yeni bir `static const String` ekle
2. `setShortcutItems` listesine yeni `ShortcutItem` ekle
3. Android için drawable ekle
4. `_handleAppShortcut` switch/if bloğuna yeni tipi handle et
5. Hedef route'a navigate et veya ilgili dialog'u göster

## Bilinen Kısıtlamalar

- Android'de shortcut sayısı OS'a göre değişir (genellikle max 4–5)
- Subcontractor kullanıcılar `SubcontractorHomeRoute`'a gider, `AppPage`'e gelmez → shortcut handler çalışmaz (beklenen davranış)
- iOS'ta icon için `Assets.xcassets`'e image eklenmesi gerekir; yoksa icon görünmez ama shortcut çalışır


## Önemli Noktalar

- initialize() hem handler'ı register eder hem de pending cold-start shortcut'ı tetikler — AppPage.initState'te çağrılmalı
- addPostFrameCallback kullan: initState'te dialog göstermek için ilk frame'in bitmesi gerekir
- Android icon: drawable resource adı (uzantısız), iOS icon: asset catalog veya SF Symbol adı
- Mevcut emergency dialog (_EmergencyCountdownDialog) aynen yeniden kullanılır — tutarlı UX
- Exception try/catch ile sarıla — desteklenmeyen platformda crash olmaması için
