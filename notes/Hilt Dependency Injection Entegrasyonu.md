---
title: Hilt Dependency Injection Entegrasyonu
type: note
tags:
  - android
  - hilt
  - dependency-injection
  - dagger
  - ksp
  - gradle
summary: >-
  Android projede Hilt DI kurulumu - Gradle, Application, Activity
  konfigürasyonu
relatedTerms:
  - Retrofit Network Katmanı Entegrasyonu
created: '2026-03-17T22:08:33.225Z'
updated: '2026-03-17T22:16:04.916Z'
---
# Hilt Dependency Injection Entegrasyonu

> Android projede Hilt DI kurulumu - Gradle, Application, Activity konfigürasyonu

## İçerik


## Hilt Nedir?
Google'ın Android için resmi Dependency Injection çözümü. Dagger üzerine kurulu, annotation tabanlı çalışır.

## Gradle Kurulumu

### libs.versions.toml
```toml
[versions]
hilt = "2.56.2"
hiltNavigationCompose = "1.2.0"
ksp = "2.0.21-1.0.28"  # Kotlin versiyonuyla eşleşmeli

[libraries]
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-android-compiler = { group = "com.google.dagger", name = "hilt-android-compiler", version.ref = "hilt" }
androidx-hilt-navigation-compose = { group = "androidx.hilt", name = "hilt-navigation-compose", version.ref = "hiltNavigationCompose" }

[plugins]
hilt-android = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
```

### Root build.gradle.kts
```kotlin
plugins {
    alias(libs.plugins.hilt.android) apply false
    alias(libs.plugins.ksp) apply false
}
```

### app/build.gradle.kts
```kotlin
plugins {
    alias(libs.plugins.hilt.android)
    alias(libs.plugins.ksp)
}

dependencies {
    implementation(libs.hilt.android)
    ksp(libs.hilt.android.compiler)
    implementation(libs.androidx.hilt.navigation.compose)
}
```

## Kod Kurulumu

### Application sınıfı
```kotlin
@HiltAndroidApp
class LingustaApp : Application()
```
AndroidManifest.xml'de `android:name=".LingustaApp"` eklenmeli.

### Activity
```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity()
```

## Önemli Annotation'lar
- `@HiltAndroidApp` → Application sınıfına, DI başlangıç noktası
- `@AndroidEntryPoint` → Activity/Fragment'e, DI enjeksiyon noktası
- `@Module` → Nesne oluşturma tariflerini içeren sınıf
- `@InstallIn(SingletonComponent::class)` → Modülün yaşam kapsamı
- `@Provides` → Nesne üretme fonksiyonu
- `@Singleton` → Tek instance, uygulama boyunca aynı
- `@Inject` → Constructor injection
- `@HiltViewModel` → ViewModel'e DI desteği

## KSP Versiyon Notu
KSP versiyonunun ilk kısmı Kotlin versiyonuyla birebir eşleşmeli:
- Kotlin `2.0.21` → KSP `2.0.21-x.x.x`


## Önemli Noktalar

- KSP versiyonu Kotlin versiyonuyla eşleşmeli
- Application sınıfına @HiltAndroidApp, Activity'ye @AndroidEntryPoint şart
- @Module + @InstallIn ile nesne tarifleri tanımlanır
- Root build.gradle.kts'te plugin'ler apply false olarak eklenmeli


## İlişkili Terimler

- [[Retrofit Network Katmanı Entegrasyonu]]
