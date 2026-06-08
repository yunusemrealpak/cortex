---
term: Async Frame
category: kavram
tags:
  - software
  - debugging
  - asynchronous-programming
  - runtime
  - dart
  - javascript
summary: >-
  Asenkron bir fonksiyon askıya alınıp sonradan devam ettiğinde mantıksal çağrı
  zincirini koruyabilmek için debugger ve runtime'ların yeniden kurguladığı
  sanal yığın çerçevesidir.
relatedTerms:
  - race-condition
created: '2026-06-08T15:13:59.342Z'
updated: '2026-06-08T15:13:59.342Z'
confidence: learning
source: claude-cli
sources:
  - async_frame.png
---
# Async Frame

> Asenkron bir fonksiyon askıya alınıp sonradan devam ettiğinde mantıksal çağrı zincirini koruyabilmek için debugger ve runtime'ların yeniden kurguladığı sanal yığın çerçevesidir.

## Türkçe Karşılık

Asenkron Çerçeve (Asenkron Yığın Çerçevesi)

## Açıklama

Senkron kodda her fonksiyon çağrısı çağrı yığınına (call stack) gerçek bir çerçeve (frame) iter ve fonksiyon döndüğünde bu çerçeve yığından çıkar; böylece bir hata oluştuğunda yığın izi (stack trace) doğal olarak çağrı sırasını gösterir. Ancak `async/await`, Future, Promise veya coroutine gibi mekanizmalarda bir fonksiyon `await` noktasında askıya alınır, kontrol event loop'a geri döner ve gerçek çağrı yığını çözülür (unwind). Devam (continuation) daha sonra tamamen farklı bir yığın bağlamında, çoğu zaman event loop'tan çağrıldığı için, ona kimin sebep olduğu bilgisi fiziksel yığında artık yoktur. İşte bu boşluğu doldurmak için kavramsal olarak kurgulanan çerçeveye "async frame" denir.

## Örnekler

### Örnek 1: Dart async stack trace'inde async gap

`<asynchronous suspension>` satırı, fiziksel yığında olmayan ama runtime'ın `await` zincirini takip ederek yeniden kurguladığı async frame'i temsil eder. Bu olmadan stack trace yalnızca event loop'a kadar gider ve `main`'in `fetchUser`'ı çağırdığı bilgisi kaybolurdu.

```
Future<void> fetchUser() async {
  await Future.delayed(Duration(milliseconds: 10));
  throw StateError('not found'); // hata burada
}

Future<void> main() async {
  await fetchUser();
}

// Stack trace:
// #0  fetchUser (file.dart:3)
// <asynchronous suspension>   <-- async frame / async gap
// #1  main (file.dart:8)
```

### Örnek 2: Tarayıcı DevTools'ta Async stack

Chrome DevTools, 'Async' etiketli çerçeveler ekleyerek `setTimeout` çağrısının nereden zamanlandığını gösterir; bunlar gerçek yığın çerçeveleri değil, planlama anında yakalanıp birleştirilen async frame'lerdir.

```
setTimeout(() => {
  doWork(); // burada patlarsa
}, 100);

function doWork() {
  throw new Error('boom');
}
```

## Sık Yapılan Hatalar

- Async frame'i gerçek bir çağrı yığını çerçevesi sanmak; o fiziksel olarak yığında bulunmaz, runtime tarafından mantıksal olarak yeniden kurgulanır.
- Async stack trace toplamanın bedava olduğunu varsaymak; her askıya alma noktasında bağlam yakalandığı için production'da performans/bellek maliyeti olabilir (Dart'ta lazy/eager async stack trace ayarları gibi).
- `await` kullanmadan Future/Promise'i ateşleyip bırakmak (fire-and-forget); bu durumda async frame zinciri kopar ve hata oluştuğunda izi kaybolur.
- Async frame'i bir UI render frame'i (örn. requestAnimationFrame / Flutter frame) ile karıştırmak; bunlar tamamen farklı kavramlardır.

## İlişkili Terimler

- race-condition

## Kaynaklar

- Dart documentation — Asynchronous programming & stack traces (dart.dev)
- Chrome DevTools documentation — Asynchronous stack traces (developer.chrome.com)
- async_frame.png
