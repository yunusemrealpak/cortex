---
title: Flutter DevTools Memory Grafiği Okuması
type: note
tags:
  - flutter
  - devtools
  - memory
  - profiling
  - debugging
  - garbage-collection
  - dart-vm
  - performance
summary: >-
  Flutter DevTools'taki bellek grafiğindeki dört temel elemanın (Dart Heap, RSS,
  GC Events, Heap Capacity) renk kodları, tipik seviyeleri ve ne anlama
  geldiğini açıklayan referans tablosu.
relatedTerms: []
created: '2026-03-24T12:53:16.304Z'
updated: '2026-03-24T12:53:16.304Z'
imageFiles:
  - flutter_devtools_memory_graph.png
---
# Flutter DevTools Memory Grafiği Okuması

![[flutter_devtools_memory_graph.png]]

> Flutter DevTools'taki bellek grafiğindeki dört temel elemanın (Dart Heap, RSS, GC Events, Heap Capacity) renk kodları, tipik seviyeleri ve ne anlama geldiğini açıklayan referans tablosu.

## İçerik

## Grafik Okuması

Flutter DevTools Memory sekmesindeki grafikte dört ana eleman bulunur. Her birinin renk kodu, tipik seviyesi ve anlamı aşağıdaki tabloda özetlenmiştir:

| Eleman | Renk | Seviye | Ne Anlama Geliyor |
|---|---|---|---|
| **Dart Heap** | Turuncu kesikli çizgi | ~150–200 MB | Dart objelerinin kullandığı RAM. Sabit kalması = memory leak yok |
| **RSS** | Mavi noktalar | ~350–400 MB | Toplam process belleği (Dart + native: image buffer, video buffer, platform) |
| **GC Events** | Pembe üçgenler | Üstte | Garbage Collection tetiklenmeleri |
| **Heap Capacity** | Açık mavi çizgi | ~50–100 MB | Dart VM'in ayırdığı heap kapasitesi |

### Yorumlama İpuçları

- **Dart Heap** sürekli yükseliyorsa ve GC sonrası düşmüyorsa → **memory leak** şüphesi.
- **RSS** ile **Dart Heap** arasındaki fark büyükse → native tarafta (image cache, platform channel buffer vb.) yüksek bellek tüketimi var.
- **GC Events** çok sık tetikleniyorsa → kısa ömürlü objeler aşırı üretiliyor (object churn).
- **Heap Capacity** sabit kalırken **Dart Heap** ona yaklaşıyorsa → VM daha fazla heap ayırmak zorunda kalabilir, bu da GC baskısını artırır.

## Önemli Noktalar

- Dart Heap (turuncu kesikli) sabit kalıyorsa memory leak yoktur
- RSS (mavi noktalar) toplam process belleğini gösterir — Dart + native katman dahil
- GC Events (pembe üçgenler) Garbage Collection tetiklenmelerini işaret eder
- Heap Capacity (açık mavi) Dart VM'in ayırdığı heap kapasitesidir
- RSS ile Dart Heap arasındaki fark native bellek tüketimini gösterir
