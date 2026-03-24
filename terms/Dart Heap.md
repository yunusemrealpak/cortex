---
term: Dart Heap
category: Flutter / Dart Runtime
tags:
  - dart
  - flutter
  - memory
  - performance
  - devtools
  - runtime
summary: >-
  Dart VM'in runtime sırasında oluşturulan tüm Dart objelerini sakladığı dinamik
  bellek bölgesi.
relatedTerms:
  - Garbage Collection
  - RSS (Resident Set Size)
  - Native Memory
  - Memory Leak
  - New Space
  - Old Space
created: '2026-03-24T12:56:02.523Z'
updated: '2026-03-24T12:56:02.523Z'
confidence: learning
source: claude-cli
---
# Dart Heap

> Dart VM'in runtime sırasında oluşturulan tüm Dart objelerini sakladığı dinamik bellek bölgesi.

## Türkçe Karşılık

Dart Yığın Belleği

## Açıklama

## Dart Heap Nedir?\n\nDart Heap, Dart VM'in çalışma zamanında yarattığı tüm Dart objelerinin (String, List, Map, widget instance, BLoC state, entity vb.) tutulduğu bellek alanıdır. `new` keyword'ü veya literal syntax ile yaratılan her obje bu heap'te yer alır.\n\nDart VM heap'i iki nesil (generation) olarak yönetir:\n- **New Space (Young Generation)**: Kısa ömürlü objeler burada yaratılır. Sık ve hızlı GC yapılır (minor GC).\n- **Old Space (Old Generation)**: Birkaç GC döngüsü boyunca hayatta kalan objeler buraya taşınır. Daha seyrek ama maliyetli GC yapılır (major GC).\n\n## Neden Önemli?\n\n1. **Memory Leak Tespiti**: Heap sürekli büyüyorsa ve GC sonrası düşmüyorsa, objeler referans tutuluyor ve serbest bırakılmıyor demektir (dispose edilmemiş listener, cancel edilmemiş stream vb.).\n2. **GC Baskısı**: Çok fazla kısa ömürlü obje yaratılırsa Garbage Collector sık tetiklenir — bu micro-jank'lara neden olabilir.\n3. **Dart Heap ≠ Toplam Bellek**: Dart Heap sadece Dart objelerini tutar. Decode edilmiş görseller, video buffer'ları, SQLite verileri **native memory**'de yaşar ve Dart Heap metriklerinde görünmez.\n\n## Formül\n\n```\nRSS (toplam process belleği) = Dart Heap + Native Memory (images, video, platform)\n```\n\nBu yüzden DevTools'ta Dart Heap düşük görünürken RSS çok yüksek olabilir — aradaki fark native taraftadır.

## Örnekler

### Örnek 1: DevTools'ta Dart Heap izleme

Flutter DevTools → Memory sekmesinde turuncu çizgi Dart Heap kullanımını gösterir. Sabit kalması sağlıklı, sürekli yükselmesi leak işareti.

### Örnek 2: Memory pressure handling

didHaveMemoryPressure() callback'inde PaintingBinding.instance.imageCache.clear() çağrısı native image cache'i temizler ama Dart Heap'i doğrudan etkilemez — çünkü image pixel verileri native memory'dedir.

### Örnek 3: GC baskısı örneği

Scroll sırasında her frame'de yeni List<FeedPost> kopyası yaratmak (immutable state pattern) New Space'te obje baskısı oluşturur. GC sık tetiklenir, micro-jank riski artar.

## Sık Yapılan Hatalar

- Dart Heap düşük diye bellek sorunu yok sanmak — native memory (görseller, video) RSS'te ayrı görünür
- Dart Heap'teki büyümeyi her zaman leak olarak yorumlamak — bazen sadece cache dolması veya beklenen veri birikimi olabilir
- GC'nin otomatik olduğunu düşünüp dispose/cancel yapmayı ihmal etmek — GC sadece referanssız objeleri temizler, referans tutuluyorsa temizleyemez

## İlişkili Terimler

- Garbage Collection
- RSS (Resident Set Size)
- Native Memory
- Memory Leak
- New Space
- Old Space

## Kaynaklar

- https://dart.dev/tools/dart-devtools/memory
