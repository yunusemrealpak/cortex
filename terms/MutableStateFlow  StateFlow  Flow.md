---
term: MutableStateFlow / StateFlow / Flow
category: kavram
tags:
  - software
  - kotlin
  - coroutines
  - reactive-programming
  - android
  - state-management
summary: >-
  Kotlin Coroutines'te reaktif veri akışını temsil eden üç soyutlama seviyesi;
  Flow en genel soğuk akış, StateFlow her zaman bir değere sahip sıcak akış,
  MutableStateFlow ise dışarıdan değeri değiştirilebilen sıcak akıştır.
relatedTerms:
  - race-condition
created: '2026-03-19T11:21:53.786Z'
updated: '2026-03-19T11:21:53.786Z'
confidence: learning
source: claude-cli
---
# MutableStateFlow / StateFlow / Flow

> Kotlin Coroutines'te reaktif veri akışını temsil eden üç soyutlama seviyesi; Flow en genel soğuk akış, StateFlow her zaman bir değere sahip sıcak akış, MutableStateFlow ise dışarıdan değeri değiştirilebilen sıcak akıştır.

## Türkçe Karşılık

Değiştirilebilir Durum Akışı / Durum Akışı / Akış

## Açıklama

Kotlin Coroutines ekosisteminde Flow, asenkron olarak birden fazla değer üreten soğuk (cold) bir akış tipidir. 'Soğuk' olması, bir toplayıcı (collector) bağlanana kadar hiçbir işlem yapmaması anlamına gelir; her yeni collector kendi bağımsız akışını başlatır. Veritabanı sorguları, ağ istekleri veya hesaplama zincirleri gibi senaryolarda kullanılır. map, filter, combine gibi operatörlerle dönüştürülebilir.

StateFlow, Flow'un özelleşmiş bir alt tipidir ve SharedFlow'dan türer. İki temel farkı vardır: her zaman bir mevcut değere (value) sahiptir ve sıcak (hot) bir akıştır — yani collector olsun olmasın değerini tutar. Ayrıca conflated yapıdadır; art arda aynı değer emit edilirse tekrar bildirmez (distinctUntilChanged davranışı). Bu özellikler onu UI state tutmak için ideal kılar. Ancak StateFlow salt okunurdur (read-only); dışarıdan değeri değiştirilemez.

MutableStateFlow, StateFlow'un yazılabilir (mutable) versiyonudur. value property'si üzerinden veya emit() fonksiyonuyla yeni değer atanabilir. Tipik kullanım deseni şudur: ViewModel veya iş katmanı içinde MutableStateFlow oluşturulur, dışarıya ise StateFlow (salt okunur) olarak sunulur. Bu, enkapsülasyonu korur — sadece sahibi değeri değiştirebilir, tüketiciler yalnızca gözlemler. Android'de LiveData'nın modern Coroutines karşılığı olarak kabul edilir ve Compose ile doğal uyum sağlar.

## Örnekler

### Örnek 1: ViewModel'de Temel Kullanım Deseni

MutableStateFlow dahili olarak tutulur, dışarıya StateFlow olarak açılır. Bu pattern enkapsülasyonu korur ve tek yönlü veri akışını garanti eder.

```
class UserViewModel : ViewModel() {
    // Dahili: değiştirilebilir
    private val _uiState = MutableStateFlow(UserUiState())
    // Dışarıya: salt okunur
    val uiState: StateFlow<UserUiState> = _uiState.asStateFlow()

    fun loadUser(id: String) {
        viewModelScope.launch {
            _uiState.value = _uiState.value.copy(isLoading = true)
            val user = repository.getUser(id)
            _uiState.value = _uiState.value.copy(
                isLoading = false,
                user = user
            )
        }
    }
}
```

### Örnek 2: Cold Flow → StateFlow Dönüşümü

Repository soğuk Flow döner; ViewModel bunu stateIn() ile sıcak StateFlow'a dönüştürür. WhileSubscribed(5000) sayesinde son collector ayrıldıktan 5 saniye sonra upstream iptal edilir (konfigürasyon değişikliklerinde gereksiz yeniden sorguyu önler).

```
class UserRepository(private val dao: UserDao) {
    // Cold Flow — her collector veritabanını yeniden sorgular
    fun observeUsers(): Flow<List<User>> = dao.getAllUsers()
}

class UserViewModel(private val repo: UserRepository) : ViewModel() {
    // Cold Flow'u hot StateFlow'a dönüştür
    val users: StateFlow<List<User>> = repo.observeUsers()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = emptyList()
        )
}
```

### Örnek 3: Compose ile StateFlow Tüketimi

collectAsStateWithLifecycle(), StateFlow'u yaşam döngüsüne duyarlı şekilde toplar; uygulama arka plana geçtiğinde toplayıcıyı otomatik durdurur.

```
@Composable
fun UserScreen(viewModel: UserViewModel) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    when {
        uiState.isLoading -> CircularProgressIndicator()
        uiState.error != null -> ErrorView(uiState.error!!)
        else -> UserContent(uiState.user)
    }
}
```

## Sık Yapılan Hatalar

- MutableStateFlow'u doğrudan dışarıya açmak — tüketicilerin state'i keyfi değiştirmesine izin verir, tek yönlü veri akışını bozar. Her zaman asStateFlow() ile salt okunur StateFlow olarak sunulmalıdır.
- StateFlow'un distinctUntilChanged davranışını göz ardı etmek — aynı data class instance'ını yeniden emit etmek UI güncellemesini tetiklemez. State değişikliği için copy() ile yeni nesne oluşturulmalıdır.
- Cold Flow yerine her yerde StateFlow kullanmak — tek seferlik işlemler (API çağrısı, dosya okuma) için StateFlow gereksiz karmaşıklık ekler; bu senaryolarda düz Flow veya suspend fonksiyon yeterlidir.
- stateIn() kullanırken SharingStarted.Eagerly tercih etmek — collector olmasa bile upstream aktif kalır ve kaynak israfına yol açar. Çoğu UI senaryosunda WhileSubscribed tercih edilmelidir.
- MutableStateFlow üzerinde value ataması yaparken thread-safety'yi ihmal etmek — value property'si atomik olsa da read-modify-write işlemleri (value = value.copy(...)) race condition'a açıktır. update {} lambda fonksiyonu kullanılmalıdır.

## İlişkili Terimler

- race-condition

## Kaynaklar

- https://kotlinlang.org/docs/flow.html
- https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/
- https://developer.android.com/kotlin/flow/stateflow-and-sharedflow
