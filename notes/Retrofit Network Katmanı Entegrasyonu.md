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
summary: >-
  Android projede Retrofit + OkHttp + Gson ile network altyapısı kurulumu ve
  Hilt ile DI entegrasyonu
relatedTerms:
  - Hilt Dependency Injection Entegrasyonu
created: '2026-03-17T22:16:04.912Z'
updated: '2026-03-17T22:16:04.912Z'
---
# Retrofit Network Katmanı Entegrasyonu

> Android projede Retrofit + OkHttp + Gson ile network altyapısı kurulumu ve Hilt ile DI entegrasyonu

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

## Hilt ile NetworkModule

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
    fun provideOkHttpClient(loggingInterceptor: HttpLoggingInterceptor): OkHttpClient {
        return OkHttpClient.Builder()
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

- Retrofit interface tabanlı çalışır - interface yaz, implementasyonu Retrofit üretir
- converter-gson ile JSON otomatik parse edilir
- LoggingInterceptor debug için çok değerli, production'da kapatılmalı
- Hilt @Module ile Retrofit singleton olarak sağlanır
- OkHttpClient'a auth interceptor gibi ek katmanlar eklenebilir

## İlişkili Terimler

- [[Hilt Dependency Injection Entegrasyonu|Hilt Dependency Injection Entegrasyonu]]
