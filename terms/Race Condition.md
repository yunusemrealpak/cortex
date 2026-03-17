---
term: Race Condition
category: kavram
tags:
  - software
  - concurrency
  - multithreading
  - distributed-systems
  - bug
summary: >-
  İki veya daha fazla işlemin paylaşılan bir kaynağa eş zamanlı erişiminde,
  sonucun işlemlerin zamanlamasına bağlı olarak öngörülemez biçimde değişmesi
  durumu.
relatedTerms:
  - CAP Teoremi
  - pitfall
created: '2026-03-17T11:33:32.835Z'
updated: '2026-03-17T11:33:32.835Z'
confidence: learning
source: claude-cli
---
# Race Condition

> İki veya daha fazla işlemin paylaşılan bir kaynağa eş zamanlı erişiminde, sonucun işlemlerin zamanlamasına bağlı olarak öngörülemez biçimde değişmesi durumu.

## Türkçe Karşılık

Yarış Durumu

## Açıklama

Race condition, birden fazla iş parçacığının (thread), sürecin (process) veya sistemin aynı kaynağa (değişken, dosya, veritabanı satırı vb.) eş zamanlı olarak erişip değiştirmeye çalıştığında ortaya çıkan bir hata türüdür. Sorunun kökeni, işlemlerin yürütülme sırasının deterministik olmamasıdır; işletim sistemi zamanlayıcısı, ağ gecikmeleri veya donanım farklılıkları gibi faktörler hangi işlemin önce tamamlanacağını belirler. Bu da aynı kodun farklı çalıştırmalarda farklı sonuçlar üretmesine yol açar.

Race condition'lar yazılımda en zor tespit edilen hata türlerinden biridir. Geliştirme ve test ortamlarında ortaya çıkmayıp yalnızca üretim ortamında, yüksek yük altında veya belirli zamanlama koşullarında kendini gösterebilir. Klasik örneği 'check-then-act' (kontrol et, sonra işlem yap) kalıbıdır: bir koşulu kontrol etme ile o koşula göre işlem yapma arasındaki zaman aralığında başka bir iş parçacığı durumu değiştirebilir. Çözüm yöntemleri arasında mutex, semafor, atomik işlemler, veritabanı kilitlri (lock) ve mesaj kuyruğu tabanlı sıralama gibi senkronizasyon mekanizmaları yer alır.

Dağıtık sistemlerde race condition'lar daha da karmaşık hale gelir. Birden fazla sunucunun aynı veritabanı kaydını güncellemeye çalışması, iki kullanıcının aynı ürünün son stoğunu satın almaya çalışması gibi senaryolarda ortaya çıkar. Bu bağlamda optimistic locking, distributed lock, idempotent tasarım ve event sourcing gibi mimari düzeyde çözümler gerekir.

## Örnekler

### Örnek 1: Klasik Sayaç Problemi

İki thread aynı anda counter değişkenini okuyup artırmaya çalışır. Her ikisi de 0 değerini okur, birer artırır ve 1 yazar. Sonuç 2 olması gerekirken 1 olur.

```
// Thread A ve Thread B aynı anda çalışıyor
int counter = 0;

// Thread A          // Thread B
temp = counter; // 0  temp = counter; // 0
temp = temp + 1;     temp = temp + 1;
counter = temp; // 1  counter = temp; // 1

// Beklenen: 2, Gerçek: 1
```

### Örnek 2: Check-Then-Act (TOCTOU)

Time-of-check to time-of-use (TOCTOU) hatası. Dosyanın varlığını kontrol etme ile silme işlemi arasında başka bir thread dosyayı siler. İkinci silme girişimi başarısız olur veya beklenmedik davranışa yol açar.

```
// Thread A
if (file.exists()) {    // 1. Kontrol: dosya var
    file.delete();      // 3. Sil → BAŞARILI
}

// Thread B
if (file.exists()) {    // 2. Kontrol: dosya hâlâ var
    file.delete();      // 4. Sil → HATA! Dosya artık yok
}
```

### Örnek 3: Banka Hesabı Çift Harcama

İki eş zamanlı para çekme işlemi bakiyeyi aynı anda okur. Her ikisi de yeterli bakiye olduğunu görür ve işlemi onaylar. Toplam 150 TL çekilir ancak hesapta yalnızca 100 TL vardır.

```
// Bakiye: 100 TL
// İşlem A: 80 TL çek     İşlem B: 70 TL çek
balance = getBalance();  // 100
                         balance = getBalance();  // 100
if (balance >= 80)       if (balance >= 70)
  withdraw(80);            withdraw(70);
// Sonuç: -50 TL (eksi bakiye!)
```

## Sık Yapılan Hatalar

- Race condition'ı yalnızca multithreading problemi sanmak; tek thread'li event loop'larda (Node.js) bile async işlemler arasında oluşabilir
- Test ortamında hata görülmediği için race condition olmadığını varsaymak; düşük yük altında zamanlama çakışması olasılığı düşüktür
- Tüm paylaşılan kaynakları kilitleyerek çözmek yerine granüler senkronizasyon kullanmamak, bu da deadlock'a yol açabilir
- Veritabanı işlemlerinde transaction isolation level'ı düşünmeden varsayılan seviyeye güvenmek
- Atomik olmayan read-modify-write işlemlerini atomik sanmak (örn: i++ ifadesi tek işlem gibi görünür ama değildir)

## İlişkili Terimler

- [[CAP Teoremi]]
- [[pitfall]]

## Kaynaklar

- Tanenbaum, A. S. – Modern Operating Systems (Chapter 2: Processes and Threads)
- Goetz, B. – Java Concurrency in Practice (Addison-Wesley, 2006)
- Kleppmann, M. – Designing Data-Intensive Applications (O'Reilly, 2017)
