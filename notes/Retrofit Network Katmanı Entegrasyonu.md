---
title: Retrofit Network Katmanı Entegrasyonu
type: note
tags:
  - android
  - retrofit
  - okhttp
  - gson
  - network
  - hilt
  - gradle
  - interceptor
summary: >-
  Android projede Retrofit + OkHttp + Gson ile network altyapısı kurulumu,
  custom interceptor yapısı ve Hilt ile DI entegrasyonu
relatedTerms:
  - Hilt Dependency Injection Entegrasyonu
created: '2026-03-17T22:33:50.785Z'
updated: '2026-03-17T22:33:50.785Z'
---
# Retrofit Network Katmanı Entegrasyonu

> Android projede Retrofit + OkHttp + Gson ile network altyapısı kurulumu, custom interceptor yapısı ve Hilt ile DI entegrasyonu

## İçerik


## Retrofit Nedir?
Android'de HTTP istekleri için standart kütüphane. Interface tanımlarsın, Retrofit otomatik olarak çalışan implementasyon üretir.

## Gradle Kurulumu

### libs.versions.toml
```toml
[versions]
retrofit = "2.11.0"
okhttp = "4.12.0"
gson = "2.11.0"

[libraries]
retrofit-core = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
retrofit-converter-gson = { group = "com.squareup.retrofit2", name = "converter-gson", version.ref = "retrofit" }
okhttp-logging-interceptor = { group = "com.squareup.okhttp3", name = "logging-interceptor", version.ref = "okhttp" }
gson = { group = "com.google.code.gson", name = "gson", version.ref = "gson" }
```

### app/build.gradle.kts
```kotlin
// Network
implementation(libs.retrofit.core)
implementation(libs.retrofit.converter.gson)
implementation(libs.okhttp.logging.interceptor)
implementation(libs.gson)
```

## Custom Interceptor Yapısı

### Interceptor Nedir?
Her HTTP isteği bir interceptor zincirinden geçer. Interceptor, istek sunucuya gitmeden araya girip isteği değiştirebilen veya cevabı kontrol edebilen bir katman.

```
Uygulama → [Interceptor 1] → [Interceptor 2] → [Interceptor N] → Sunucu
Sunucu  → [Interceptor N] → [Interceptor 2] → [Interceptor 1] → Uygulama
```

### Custom Interceptor Oluşturma
`Interceptor` interface'ini implement et, `intercept()` metodunu override et:

```kotlin
@Singleton
class AuthInterceptor @Inject constructor() : Interceptor {

    private var token: String? = null

    fun setToken(newToken: String?) {
        token = newToken
    }

    override fun intercept(chain: Interceptor.Chain): Response {
        val originalRequest = chain.request()

        // Token yoksa isteği olduğu gibi gönder
        if (token.isNullOrEmpty()) {
            return chain.proceed(originalRequest)
        }

        // Token varsa header ekle
        val authenticatedRequest = originalRequest.newBuilder()
            .header("Authorization", "Bearer $token")
            .build()

        return chain.proceed(authenticatedRequest)
    }
}
```

### Temel Kavramlar
- `chain.request()` → Orijinal isteği al
- `chain.proceed(request)` → İsteği bir sonraki halkaya veya sunucuya gönder
- `originalRequest.newBuilder()` → Request immutable, yeni kopya oluşturup değiştirirsin
- `@Inject constructor()` → Hilt otomatik oluşturur, @Module'e gerek yok

### OkHttpClient'a Bağlama
```kotlin
@Provides
@Singleton
fun provideOkHttpClient(
    loggingInterceptor: HttpLoggingInterceptor,
    authInterceptor: AuthInterceptor
): OkHttpClient {
    return OkHttpClient.Builder()
        .addInterceptor(authInterceptor)
        .addInterceptor(loggingInterceptor)
        .build()
}
```

### Interceptor Sırası Önemli!
1. Önce `authInterceptor` → token'ı ekler
2. Sonra `loggingInterceptor` → isteğin son halini (token dahil) loglar

Sıra ters olursa log'da token header'ı görünmez.

## Hilt ile NetworkModule (Güncel)

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    private const val BASE_URL = "https://api.lingusta.com/"

    @Provides
    @Singleton
    fun provideLoggingInterceptor(): HttpLoggingInterceptor {
        return HttpLoggingInterceptor().apply {
            level = HttpLoggingInterceptor.Level.BODY
        }
    }

    @Provides
    @Singleton
    fun provideOkHttpClient(
        loggingInterceptor: HttpLoggingInterceptor,
        authInterceptor: AuthInterceptor
    ): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor(authInterceptor)
            .addInterceptor(loggingInterceptor)
            .build()
    }

    @Provides
    @Singleton
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }
}
```

## Kütüphane Rolleri
- **Retrofit** → HTTP client, interface'den API çağrı sınıfı üretir
- **converter-gson** → JSON ↔ Kotlin sınıfı otomatik dönüşümü
- **OkHttp logging-interceptor** → Debug sırasında API istek/cevaplarını loglar
- **Gson** → JSON parse kütüphanesi

## Mimari Notlar
- NetworkModule `core/network/` altında yaşar
- `@Singleton` ile uygulama boyunca tek Retrofit instance'ı kullanılır
- OkHttpClient'a interceptor zinciri eklenebilir (auth token, error handling vb.)
- LoggingInterceptor production'da kapatılmalı veya seviyesi düşürülmeli


## Önemli Noktalar

- Interceptor Interceptor interface'ini implement ederek oluşturulur
- chain.request() ile orijinal istek alınır, chain.proceed() ile gönderilir
- Request immutable - newBuilder() ile kopya oluşturup değiştirilir
- Interceptor sırası önemli: auth önce, logging sonra
- @Inject constructor ile Hilt otomatik oluşturur, @Module gerekmez
- Retrofit interface tabanlı çalışır - interface yaz, implementasyonu Retrofit üretir

## İlişkili Terimler

- [[Hilt Dependency Injection Entegrasyonu|Hilt Dependency Injection Entegrasyonu]]
