---
term: Atomic Design
category: metodoloji
tags:
  - software
  - UI/UX
  - design-system
  - frontend
  - component-architecture
summary: >-
  Kullanıcı arayüzlerini kimyadan esinlenen beş hiyerarşik seviyede (Atoms →
  Molecules → Organisms → Templates → Pages) tasarlayıp inşa etmeyi öngören
  bileşen tabanlı tasarım metodolojisi.
relatedTerms:
  - unidirectional-data-flow
created: '2026-04-02T16:15:38.267Z'
updated: '2026-04-02T16:15:38.267Z'
confidence: learning
source: claude-cli
---
# Atomic Design

> Kullanıcı arayüzlerini kimyadan esinlenen beş hiyerarşik seviyede (Atoms → Molecules → Organisms → Templates → Pages) tasarlayıp inşa etmeyi öngören bileşen tabanlı tasarım metodolojisi.

## Türkçe Karşılık

Atomik Tasarım

## Açıklama

Atomic Design, Brad Frost tarafından 2013 yılında ortaya atılan ve kullanıcı arayüzlerini sistematik biçimde oluşturmayı amaçlayan bir metodolojidir. Kimyadaki madde yapısından ilham alarak, arayüz elemanlarını en küçük birimden en karmaşık bütüne doğru beş katmanda organize eder: Atoms (buton, input, label gibi bölünemez temel HTML elemanları), Molecules (bir arama çubuğu gibi birkaç atomun anlamlı bir işlev oluşturacak şekilde birleşmesi), Organisms (header, footer gibi moleküllerin ve atomların bir araya geldiği bağımsız arayüz bölgeleri), Templates (organizmaların sayfa düzeninde yerleştirildiği iskelet yapılar) ve Pages (gerçek içerikle doldurulmuş şablonların son halleri).

Bu metodolojinin temel gücü, tasarım sistemlerini tutarlı ve ölçeklenebilir kılmasıdır. Her seviye bir öncekinin üzerine inşa edildiği için, bir atomdaki değişiklik (örneğin buton rengi) tüm sisteme otomatik olarak yansır. Bu, büyük ekiplerde tasarım-geliştirme uyumunu sağlar ve 'tek seferlik bileşen' üretimini minimize eder. Atomic Design, React, Vue, Flutter gibi bileşen tabanlı framework'lerle doğal bir uyum gösterir çünkü her iki yaklaşım da küçük, yeniden kullanılabilir parçaların kompozisyonuna dayanır.

Önemli bir nokta olarak Atomic Design, bir klasör yapısı zorunluluğu değil, bir düşünce modelidir. Projenin ihtiyacına göre katmanlar birleştirilebilir veya sadeleştirilebilir. Asıl amaç, arayüz bileşenlerini izole, test edilebilir ve yeniden kullanılabilir birimler olarak ele alma disiplinini kazandırmaktır.

## Örnekler

### Örnek 1: Atoms — Temel yapı taşları

Buton, ikon, text input gibi daha fazla parçalanamayan en küçük UI birimleri. Tek başlarına anlamlı bir iş yapmazlar ama tüm sistemin temelini oluştururlar.

```
// Flutter örneği
class AppButton extends StatelessWidget {
  final String label;
  final VoidCallback onPressed;
  const AppButton({required this.label, required this.onPressed});

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(onPressed: onPressed, child: Text(label));
  }
}
```

### Örnek 2: Molecules — Atomların anlamlı birleşimi

Birkaç atomun bir araya gelerek belirli bir işlevi yerine getirdiği bileşik yapı. Arama çubuğu = TextField (atom) + Button (atom) birleşimidir.

```
// Bir arama çubuğu molekülü
class SearchBar extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Row(
      children: [
        AppTextField(hint: 'Ara...'),  // Atom
        AppButton(label: 'Ara', onPressed: _search),  // Atom
      ],
    );
  }
}
```

### Örnek 3: Organisms — Bağımsız arayüz bölgeleri

Header (logo + navigasyon menüsü + arama çubuğu), ürün kartı listesi veya footer gibi, birden fazla molekül ve atomun bir arada çalıştığı, kendi başına anlam taşıyan büyük UI blokları. Bir organizma, sayfanın belirli bir bölgesini tamamen yönetir.

### Örnek 4: Templates → Pages dönüşümü

Template, organizmaların sayfa üzerindeki yerleşim düzenini tanımlar (wireframe gibi). Page ise bu şablonun gerçek verilerle doldurulmuş halidir. Örneğin bir e-ticaret ürün detay template'i, gerçek ürün görseli ve fiyatıyla doldurulduğunda Page olur. Bu ayrım, tasarımcıların yapıyı içerikten bağımsız düşünmesini sağlar.

## Sık Yapılan Hatalar

- Atom/Molecule/Organism ayrımını katı bir klasör yapısı zorunluluğu olarak görmek — bu bir düşünce modeli, dosya organizasyonu reçetesi değil.
- Her bileşeni zorla beş katmandan birine sığdırmaya çalışmak; bazı bileşenler sınırda kalabilir ve bu normaldir.
- Atom seviyesinde aşırı genelleştirme yaparak, hiçbir varsayılan stili olmayan çıplak wrapper'lar üretmek — atomlar yeniden kullanılabilir ama aynı zamanda tasarım sisteminizin kimliğini taşımalıdır.
- Molecules ve Organisms arasındaki farkı karmaşıklığa (eleman sayısına) göre değil, bağımsızlığa göre belirlemek gerektiğini gözden kaçırmak.
- Sadece UI katmanında uygulamak; Atomic Design state yönetimi veya iş mantığı için değil, görsel bileşen hiyerarşisi için tasarlanmıştır.

## İlişkili Terimler

- unidirectional-data-flow

## Kaynaklar

- Brad Frost — Atomic Design (2013), https://bradfrost.com/blog/post/atomic-web-design/
- Brad Frost — Atomic Design (Kitap), https://atomicdesign.bradfrost.com/
