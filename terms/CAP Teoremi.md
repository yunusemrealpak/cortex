---
term: CAP Teoremi
category: teori
tags:
  - distributed-systems
  - software
  - database
  - consistency
  - availability
  - partition-tolerance
summary: >-
  Dağıtık bir sistemin Tutarlılık (Consistency), Erişilebilirlik (Availability)
  ve Bölünme Toleransı (Partition Tolerance) özelliklerinden aynı anda yalnızca
  ikisini garanti edebileceğini öne süren teorem.
relatedTerms:
  - Race Condition
created: '2026-03-10T22:35:14.005Z'
updated: '2026-03-17T11:33:32.842Z'
confidence: learning
source: claude-cli
---
# CAP Teoremi

> Dağıtık bir sistemin Tutarlılık (Consistency), Erişilebilirlik (Availability) ve Bölünme Toleransı (Partition Tolerance) özelliklerinden aynı anda yalnızca ikisini garanti edebileceğini öne süren teorem.

## Türkçe Karşılık

CAP Teoremi

## Açıklama

CAP Teoremi, Eric Brewer tarafından 2000 yılında öne sürülen ve 2002'de Gilbert & Lynch tarafından kanıtlanan temel bir dağıtık sistemler teoremidir. Teoreme göre bir dağıtık sistem; Tutarlılık (C), Erişilebilirlik (A) ve Bölünme Toleransı (P) olmak üzere üç özelliği aynı anda sağlayamaz; bu üçünden en fazla ikisini seçmek zorundadır.

Tutarlılık, her okuma işleminin en son yazılan veriyi ya da bir hata döndürmesini ifade eder. Erişilebilirlik, her isteğin hata olmaksızın bir yanıt almasını (ancak en güncel veri olmayabilir) garanti eder. Bölünme Toleransı ise ağ üzerindeki düğümler arasında iletişim kopukluğu (partition) yaşansa dahi sistemin çalışmaya devam etmesi anlamına gelir. Gerçek dünya dağıtık sistemlerinde ağ bölünmeleri kaçınılmaz olduğundan, pratikte çoğunlukla P sabit tutulur ve sistemler CP ya da AP arasında bir tercih yapar.

CP sistemleri (örn. HBase, Zookeeper) bölünme anında erişilebilirliği feda ederek tutarlılığı korur; AP sistemleri (örn. Cassandra, CouchDB) ise tutarlılığı gevşeterek her koşulda yanıt vermeyi önceliklendirir. Bu ayrım, mimari kararlar alınırken hangi iş gereksiniminin daha kritik olduğunu belirler.

## Örnekler

### Örnek 1: CP Sistemi — Zookeeper

Zookeeper, lider seçimi ve dağıtık kilitleme gibi kritik koordinasyon görevlerinde tutarlılığı ön plana alır. Bir ağ bölünmesi yaşandığında azınlık taraftaki düğümler istek kabul etmeyi durdurur; böylece tutarsız veri okunmasının önüne geçilir.

### Örnek 2: AP Sistemi — Cassandra

Cassandra, ağ bölünmesi durumunda bile tüm düğümler üzerinden okuma/yazma işlemi kabul eder. Bu nedenle kısa süreli tutarsızlıklar (eventual consistency) oluşabilir; ancak sistem asla yanıt vermeyi kesmez. E-ticaret sepet verileri gibi yüksek erişilebilirlik gerektiren senaryolar için uygundur.

### Örnek 3: CA Sistemi — Tek Düğümlü RDBMS

PostgreSQL gibi tekil bir veritabanı sunucusu hem tutarlı hem de erişilebilirdir; ancak bu noktada gerçek anlamda bir ağ bölünmesi senaryosu yoktur. Dağıtık bir ortama geçildiği anda P seçimi zorunlu hale gelir ve CA gerçek bir seçenek olmaktan çıkar.

## Sık Yapılan Hatalar

- CAP'ı mutlak bir kural gibi yorumlayarak sistemi tamamen 'CP' veya 'AP' diye sınıflandırmak; gerçekte sistemler belirli işlemler için farklı trade-off'lar yapabilir.
- Bölünme Toleransı'nın opsiyonel olduğunu düşünmek; ağ hatası her zaman olasıdır, bu yüzden P pratikte vazgeçilemez.
- CAP'ı ACID ile karıştırmak; ACID tek bir veritabanının işlem garantilerini, CAP ise dağıtık sistemlerin ağ senaryolarını ele alır.
- Eventual Consistency'yi 'veri kaybı' olarak yorumlamak; AP sistemler veriyi kaybetmez, yalnızca kısa süreli tutarsızlığa izin verir.

## Kaynaklar

- Brewer, E. (2000). Towards Robust Distributed Systems. PODC Keynote.
- Gilbert, S. & Lynch, N. (2002). Brewer's Conjecture and the Feasibility of Consistent, Available, Partition-Tolerant Web Services. ACM SIGACT News.
- Martin Kleppmann — Designing Data-Intensive Applications (O'Reilly, 2017), Chapter 9


## İlişkili Terimler

- [[Race Condition]]
