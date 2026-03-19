---
term: Unidirectional Data Flow
category: pattern
tags:
  - software
  - architecture
  - frontend
  - state-management
  - reactive-programming
summary: >-
  Uygulama içindeki verinin yalnızca tek bir yönde — genellikle Action → State →
  View döngüsünde — akmasını zorunlu kılan mimari desen.
relatedTerms:
  - MutableStateFlow / StateFlow / Flow
created: '2026-03-19T15:17:10.225Z'
updated: '2026-03-19T15:17:10.225Z'
confidence: learning
source: claude-cli
---
# Unidirectional Data Flow

> Uygulama içindeki verinin yalnızca tek bir yönde — genellikle Action → State → View döngüsünde — akmasını zorunlu kılan mimari desen.

## Türkçe Karşılık

Tek Yönlü Veri Akışı

## Açıklama

Unidirectional Data Flow, uygulama durumunun (state) ve kullanıcı etkileşimlerinin önceden belirlenmiş tek bir yönde ilerlemesini öngören bir yazılım tasarım desenidir. Klasik modelde kullanıcı bir eylem (action/event) tetikler, bu eylem bir işleyici (reducer, store, cubit vb.) tarafından işlenir, sonuç olarak yeni bir state üretilir ve bu state arayüze (view) yansıtılır. View tekrar bir eylem tetiklediğinde döngü baştan başlar. Bu sayede veri hiçbir zaman geriye doğru veya çapraz akmaz; her değişikliğin kaynağı ve etkisi net biçimde izlenebilir.

Bu desenin temel motivasyonu, karmaşık uygulamalarda verinin nereden geldiğini ve nereye gittiğini anlaşılır kılmaktır. İki yönlü veri bağlama (two-way data binding) yaklaşımlarında bir bileşenin hem state'i okuması hem de doğrudan değiştirmesi, büyük projelerde zincirleme güncellemelere ve hata ayıklaması güç 'hayalet' değişikliklere yol açabilir. Tek yönlü akış bu sorunu ortadan kaldırır: state'i yalnızca belirli bir mekanizma (dispatch, emit, send) aracılığıyla değiştirebilirsiniz; dolayısıyla her state geçişi açık, tekrarlanabilir ve test edilebilirdir.

Facebook'un Flux mimarisiyle popülerleşen bu yaklaşım, bugün Redux, MVI (Model-View-Intent), BLoC/Cubit, Elm Architecture, Jetpack Compose'un state hoisting'i ve SwiftUI'ın @State/@Binding modeli gibi birçok framework ve pattern tarafından benimsenmiştir. Ortak payda her zaman aynıdır: veri tek yönde akar, state immutable veya kontrollü biçimde değişir ve yan etkiler (side effects) açıkça yönetilir.

## Örnekler

### Örnek 1: Redux Döngüsü (React)

React/Redux'ta kullanıcı bir butona tıklar → dispatch(action) çağrılır → reducer yeni state üretir → component yeniden renderlanır. Hiçbir bileşen state'i doğrudan mutasyona uğratmaz.

```
// Action → Reducer → Store → View
const increment = () => ({ type: 'INCREMENT' });

function counterReducer(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT': return state + 1;
    default: return state;
  }
}

// View dispatch ile action gönderir, store güncellenir,
// view yeni state'i okur — döngü tek yönlüdür.
```

### Örnek 2: BLoC / Cubit (Flutter)

Flutter'da widget, Cubit'e bir metot çağrısıyla event gönderir; Cubit yeni state'i emit eder; BlocBuilder yeni state'i alıp arayüzü yeniden çizer. Veri her zaman Widget → Cubit → State → Widget yönünde akar.

```
// Event → Bloc → State → Widget
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);
  void increment() => emit(state + 1);
}

// Widget:
// BlocBuilder<CounterCubit, int>(
//   builder: (context, count) => Text('$count'),
// )
```

### Örnek 3: Elm Architecture

Elm dilinin temel mimari modeli olan Model-Update-View üçlüsü, tek yönlü veri akışının en saf halidir. Kullanıcı etkileşimi bir Msg (mesaj) üretir, update fonksiyonu mevcut Model ve Msg'yi alarak yeni Model döndürür, view fonksiyonu Model'den HTML üretir. Yan etkiler bile Cmd tipi ile açıkça modellenir.

## Sık Yapılan Hatalar

- State'i view katmanından doğrudan mutasyona uğratmak (örn. setState içinde iş mantığı yazmak) ve böylece akışın tek yönlülüğünü kırmak.
- Tek yönlü akışı yalnızca state management kütüphanesi kullanmakla eşdeğer sanmak; kütüphane kullansanız bile view'dan state'e geri yazma kanalları açarsanız desen bozulur.
- Her küçük bileşen için global store üzerinden akış zorlamak — tek yönlü akış, lokal state'in kullanılmaması anlamına gelmez; aşırı merkezileştirme gereksiz karmaşıklık yaratır.
- Immutability'yi ihmal etmek — referans türlerini doğrudan değiştirip emit etmek, framework'ün değişikliği algılayamamasına ve sessiz hatalara yol açar.

## İlişkili Terimler

- [[MutableStateFlow / StateFlow / Flow]]

## Kaynaklar

- https://facebook.github.io/flux/docs/in-depth-overview
- https://redux.js.org/tutorials/fundamentals/part-2-concepts-data-flow
- https://guide.elm-lang.org/architecture/
- https://bloclibrary.dev/architecture/
