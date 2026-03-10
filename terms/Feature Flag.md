---
term: Feature Flag
category: pattern
tags:
  - software
  - devops
  - deployment
  - continuous-delivery
  - configuration
summary: >-
  Yeni kod deploy etmeden, çalışma zamanında belirli özellikleri açıp kapatmayı
  sağlayan yapılandırma mekanizması.
relatedTerms:
  - Pitfall
created: '2026-03-10T20:19:48.868Z'
updated: '2026-03-10T20:19:48.868Z'
confidence: learning
source: claude-cli
---
# Feature Flag

> Yeni kod deploy etmeden, çalışma zamanında belirli özellikleri açıp kapatmayı sağlayan yapılandırma mekanizması.

## Türkçe Karşılık

Özellik Bayrağı / Özellik Anahtarı

## Açıklama

Feature Flag (aynı zamanda Feature Toggle veya Feature Switch olarak da bilinir), bir uygulamadaki belirli davranışları veya özellikleri kod değişikliği yapmadan, yalnızca yapılandırma değeri değiştirerek etkinleştirip devre dışı bırakmayı sağlayan bir yazılım geliştirme patternidir. Temel fikir, yeni bir özelliğin kodunu üretime göndermek ancak onu bir 'bayrak' (genellikle boolean bir değer) kontrolünün arkasına saklamaktır; böylece özellik hazır olduğunda veya belirli koşullar sağlandığında açılabilir.

Bu pattern, Continuous Delivery pratikleriyle birlikte özellikle güçlüdür. Trunk-based development yapan ekipler, henüz tamamlanmamış özellikleri flag arkasına alarak ana branch'e merge edebilir; bu sayede uzun süren feature branch'lerden ve merge cehenneminden kaçınılır. Ayrıca A/B testi, canary release ve kullanıcı segmentine göre kademeli açılım (rollout) gibi senaryolar da feature flag'ler aracılığıyla kolayca uygulanabilir.

Feature flag'ler kullanım amacına göre farklı türlere ayrılır: Release toggles (yeni özelliği kademelendirmek için), Experiment toggles (A/B testi için), Ops toggles (anlık devre dışı bırakma için) ve Permission toggles (belirli kullanıcı gruplarına özel özellikler için). Ancak uzun süre aktif kalan ve temizlenmeyen flag'ler, kod tabanında 'toggle debt' adı verilen bir teknik borç yaratabilir.

## Örnekler

### Örnek 1: Basit boolean flag kontrolü

Ortam değişkeni ile kontrol edilen basit bir release toggle örneği. Flag false iken eski sayfa, true iken yeni sayfa render edilir.

```
const NEW_CHECKOUT_ENABLED = process.env.FEATURE_NEW_CHECKOUT === 'true';

function renderCheckout() {
  if (NEW_CHECKOUT_ENABLED) {
    return <NewCheckoutPage />;
  }
  return <LegacyCheckoutPage />;
}
```

### Örnek 2: Kullanıcı segmentine göre kademeli açılım

Kullanıcıların yalnızca %10'una yeni özelliği açmak gibi kademeli rollout senaryoları için kullanılan segment bazlı flag kontrolü.

```
function isFeatureEnabled(userId: string, flagName: string): boolean {
  const flag = featureFlagService.getFlag(flagName);
  if (!flag.enabled) return false;
  // Belirli yüzdelik dilime dahil mi?
  return hashUserId(userId) % 100 < flag.rolloutPercentage;
}
```

### Örnek 3: Ops toggle — anlık devre dışı bırakma

Yoğun trafik anında performans sorunu yaratan yeni bir öneri algoritması, yeniden deploy gerekmeksizin flag kapatılarak anında devre dışı bırakılır; bu bir 'kill switch' işlevi görür.

## Sık Yapılan Hatalar

- Flag'leri temizlememek: Artık gerekmeyen flag'lerin kod tabanında birikmesi 'toggle debt' yaratır ve kod okunabilirliğini düşürür.
- Çok fazla iç içe flag: Birden fazla flag'in birbiriyle etkileşimi test edilmesi güç, kombinatoryal patlama yaratan karmaşık koşul ağaçlarına yol açar.
- Flag durumunu test etmemek: Hem flag açık hem de kapalı senaryoların test edilmemesi, eski kod yolunun bozulmasına neden olabilir.
- Flag'leri doğrudan veritabanından okumak: Her istek için DB sorgusu yapılması cidli performans sorunlarına yol açar; flag değerleri cache'lenmeli ve async güncellenmelidir.
- Flag isimlerinin belirsiz olması: 'newFeature' gibi bağlamsız isimler yerine 'checkout_v2_enabled' gibi açıklayıcı isimler kullanılmalıdır.

## İlişkili Terimler

- [[Pitfall]]

## Kaynaklar

- Martin Fowler — Feature Toggles (Feature Flags): https://martinfowler.com/articles/feature-toggles.html
- Pete Hodgson — Feature Flags kategorileri ve yönetimi üzerine makale serisi
- LaunchDarkly — Feature Flag best practices rehberi
