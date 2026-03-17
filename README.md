# Compose Multiplatform Migration

**Organization**: MetaBrainz
**Date**: March 2026

---

## Introduction

### Contact Information

| Field | Detail |
|----|----|
| **Name** | Anuj |
| **IRC** | anuj\_ |
| **Email** | aanuj8619@gmail.com |
| **GitHub** | [anuj990](https://github.com/anuj990) |
| **LinkedIn** | [anuj990](https://linkedin.com/in/anuj990) |

Hi, I am **Anuj**, a sophomore at the Indian Institute of Information Technology Jabalpur (IIITDM Jabalpur), pursuing B.Tech in **Electronics and Communication Engineering**.

I started my coding journey in my first semester with C++ for competitive programming. Through my college’s open source programme BSOC (BitByte Summer of Code) I discovered open source and Android development. Since then my primary stack has been Kotlin, Jetpack Compose, and MVVM  and over the past few months I have been working directly with Kotlin Multiplatform and Compose Multiplatform.

I am particularly drawn to this project because it is not about adding new features  it is about making the existing codebase structurally sound so that a completely new group of users (iOS users) can access ListenBrainz.  That kind of work is what excites me most.

---

## **Project Overview**

[ListenBrainz](https://listenbrainz.org) is an open source music tracking service built by the [MetaBrainz Foundation](https://metabrainz.org) (the same foundation behind [MusicBrainz](https://musicbrainz.org)). The Android app is the mobile client for this ecosystem. Currently, the app is only available for Android which means iOS users lack access to this service.

This proposal outlines a seamless migration of the current Android codebase to [Kotlin Multiplatform](https://kotlinlang.org/docs/multiplatform.html) and [Compose Multiplatform](https://www.jetbrains.com/lp/compose-multiplatform/)

### KMP Architecture

<img width="500" height="600" alt="image" src="https://github.com/user-attachments/assets/fec52940-ef6b-4442-8a42-55fab3638817" />

Figure 1: KMP Architecture

<img width="500" height="977" alt="image" src="https://github.com/user-attachments/assets/df9d9ba3-0bd9-4e54-85f4-538ed593d83c" />


---

## Current State of Migration

The migration is actively in progress on the `cmp` branch, tracked under [Epic Issue #614](https://github.com/metabrainz/listenbrainz-android/issues/614)

### Done

| Task |
|----|
| KMP project structure + shared module |
| Configure KMP Gradle plugin |
| Source sets (commonMain, androidMain, iosMain) |
| coil/glide → coil3 |
| androidsvg → coil SVG decoder |
| Dagger/Hilt → Koin |
| Gson → Kotlin Serialization |
| Retrofit → Ktor |
| JUnit → kotlin-test |
| turbine (already KMP compatible) |
| androidx.datastore → DataStore KMP |
| compose-ratingbar → custom CMP component |
| threetenabp → kotlinx-datetime |

### Partial

| Task | Notes |
|----|----|
| socket.io-client → ktor-client-websockets | PR in Review |
| androidx.preference → DataStore KMP | Minor cleanup remaining  `androidx.datastore.preferences` still in app module, `androidx.preference-ktx` untouched |
| Room → Room KMP | Infrastructure done (Room 2.7.2 + KSP for all targets), DAOs/Entities/Database still in androidMain |
| androidx.lifecycle → KMP ViewModel | PR #732 attempted SocialViewModel but used plain `CoroutineScope` needs JetBrains lifecycle-kmp dependency + proper `ViewModel()` extension |

### Remaining

| Task |
|----|
| Logger-Android → Kermit |
| jsoup → Ksoup |
| ktor-client-okhttp → platform engines (darwin/okhttp) |
| Remove chucker |
| Androidx.paging → paging-common |
| koin-android → platform-specific setup |
| koin-androidx-compose → koin-compose |
| koin-androidx-workmanager → remove (WorkManager is Android only) |
| lottie/lottie-compose → Compottie |
| androidx.lifecycle → KMP ViewModel (full migration) |
| ExoPlayer → interface (AVPlayer for iOS) |
| spotify-app-remote → stub for iOS |
| accompanist-permissions → platform-specific |
| accompanist-systemuicontroller → platform-specific |
| share-android → platform-specific |
| androidx.browser → platform-specific |
| androidx.core.splashscreen → platform-specific |
| app-update → Android-only |
| androidx.palette → Custom implementation |
| androidx.compose.\* → JetBrains CMP |
| androidx.navigation → Voyager |
| compose-shimmer → KMP shimmer |

## Ultimate Goal

 1. Migrating Models to `shared/commonMain` (kotlinx.serialization is already in place this covers moving actual model classes to commonMain)

 2. Networking: Ktor client + API Interfaces in shared, `expect/actual` implementation   for Android/iOS

 3. Replace `socket.io-client` with `ktor-client-websockets`

 4. Replacing `lottie/lottie-compose` with [Compottie](https://github.com/alexzhirkevich/compottie)

 5. Replace `Logger-Android` with Kermit (KMP logger)

 6. Update Room to Room KMP

 7. Lifecycle to KMP-Compatible ViewModel

 8. Replace `ktor-client-okhttp` with platform-specific engines:

    * Android: `ktor-client-okhttp` or `ktor-client-android`
    * iOS: `ktor-client-darwin`

 9. Platform-Specific (`expect/actual`):

    * **Media playback:** Android → ExoPlayer, iOS → AVPlayer
    * **Background sync:** Android → WorkManager, iOS → BGTaskScheduler
    * **App update (Play Core):** Android-only
    * **Permissions:** Android → Accompanist, iOS → Native permissions
    * **Sharing:** Android → Android ShareSheet, iOS → UIActivityViewController
    * **Splash Screen:** Android → AndroidXCoreSplashScreen, iOS → iOS Launch Screen

10. Testing: `kotlin-test` is already in place. Remaining work is to move
    platform-independent tests to `commonTest` source set, and keep
    Android-specific UI tests (espresso, compose-ui-test-junit4) in
    `androidTest` only.

> **Note:** Turbine is already KMP compatible and requires no changes.

11. Navigation: Replace `androidx.navigation.compose` and `androidx.navigation3`
    with Voyager

12. Parsing/scraping: Jsoup → Ksoup

13. Remove Chucker

14. Koin migration to platform-specific setup:

    * Move `koin-core` to `commonMain`
    * Move `koin-android` to `androidMain` only
    * Replace `koin-androidx-compose` with `koin-compose` (KMP version)
    * Remove `koin-androidx-workmanager` (WorkManager is Android-only)

15. Migrate `Androidx.paging` → `paging-common` in `commonMain`,
    keep `paging-runtime` and `paging-compose` in `androidMain` only

16. Replace `compose-shimmer` with KMP-compatible shimmer in `commonMain`

17. Replace `androidx.preference-ktx` with DataStore KMP
    (foundation already done — minor cleanup remaining)

In the end we will have both Android and iOS apps running seamlessly.

---

## Step By Step Plan Of Migration

<img width="600" height="700" alt="svgviewer-png-output" src="https://github.com/user-attachments/assets/6c330b2e-f252-4e6c-9c46-554f09101be9" />


---

## Phase 1: Reconfiguring Build Infrastructure

This phase establishes the final build configuration that the entire migration
targets. By the end of GSoC, all build files must look like this.

### Final `libs.versions.toml`

```toml
[versions]
kotlin = "2.3.10"
androidGradlePlugin = "8.13.2"
ksp = "2.3.0"
compose = "1.10.2"
koin = "4.1.1"
room = "2.7.2"
sqlite = "2.6.2"
ktor-client = "3.4.1"
lifecycle-kmp = "2.8.0"
paging = "3.3.5"
kotlinxCoroutines = "1.9.0"
kotlinxSerializationJson = "1.10.0"
coil = "3.4.0"
voyager = "1.1.0"
compottie = "2.0.0"
kermit = "2.0.3"
ksoup = "0.1.2"
shimmer-kmp = "1.3.3"
turbine = "1.1.0"
compileSdk = "36"
targetSdk = "36"
minSdk = "23"

# Android-only
accompanist = "0.36.0"
work = "2.11.1"
exoplayer = "2.19.1"
appUpdate = "2.1.0"
coreSplashscreen = "1.2.0"
browser = "1.8.0"
paletteKtx = "1.0.0"
appcompat = "1.7.1"
composeBom = "2026.03.00"

[libraries]
kotlin-stdlib = { group = "org.jetbrains.kotlin", name = "kotlin-stdlib" }

# KMP Coroutines
kotlinx-coroutines-core = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-core", version.ref = "kotlinxCoroutines" }

# KMP Serialization
kotlinx-serialization-json = { module = "org.jetbrains.kotlinx:kotlinx-serialization-json", version.ref = "kotlinxSerializationJson" }

# KMP DateTime
kotlinx-datetime = { module = "org.jetbrains.kotlinx:kotlinx-datetime", version = "0.7.1" }

# DataStore KMP
datastore-preferences-core = { module = "androidx.datastore:datastore-preferences-core", version = "1.2.1" }

# Room KMP
androidx-room-runtime = { module = "androidx.room:room-runtime", version.ref = "room" }
androidx-room-compiler = { module = "androidx.room:room-compiler", version.ref = "room" }
androidx-sqlite-bundled = { module = "androidx.sqlite:sqlite-bundled", version.ref = "sqlite" }

# Ktor KMP
ktor-client-core = { module = "io.ktor:ktor-client-core", version.ref = "ktor-client" }
ktor-client-content-negotiation = { module = "io.ktor:ktor-client-content-negotiation", version.ref = "ktor-client" }
ktor-serialization-kotlinx-json = { module = "io.ktor:ktor-serialization-kotlinx-json", version.ref = "ktor-client" }
ktor-client-logging = { module = "io.ktor:ktor-client-logging", version.ref = "ktor-client" }
ktor-client-websockets = { module = "io.ktor:ktor-client-websockets", version.ref = "ktor-client" }

# Ktor Platform-specific
ktor-client-okhttp = { module = "io.ktor:ktor-client-okhttp", version.ref = "ktor-client" }
ktor-client-darwin = { module = "io.ktor:ktor-client-darwin", version.ref = "ktor-client" }

# Koin KMP
koin-core = { module = "io.insert-koin:koin-core", version.ref = "koin" }
koin-compose = { module = "io.insert-koin:koin-compose", version.ref = "koin" }
koin-android = { module = "io.insert-koin:koin-android", version.ref = "koin" }

# Lifecycle KMP ViewModel
lifecycle-viewmodel-kmp = { module = "org.jetbrains.androidx.lifecycle:lifecycle-viewmodel", version.ref = "lifecycle-kmp" }
lifecycle-viewmodel-compose-kmp = { module = "org.jetbrains.androidx.lifecycle:lifecycle-viewmodel-compose", version.ref = "lifecycle-kmp" }

# Paging KMP
androidx-paging-common = { module = "androidx.paging:paging-common", version.ref = "paging" }

# Coil KMP
coil-compose = { module = "io.coil-kt.coil3:coil-compose", version.ref = "coil" }
coil-ktor = { module = "io.coil-kt.coil3:coil-network-ktor3", version.ref = "coil" }
coil-svg = { module = "io.coil-kt.coil3:coil-svg", version.ref = "coil" }

# Navigation KMP (Voyager)
voyager-navigator = { module = "cafe.adriel.voyager:voyager-navigator", version.ref = "voyager" }
voyager-screenmodel = { module = "cafe.adriel.voyager:voyager-screenmodel", version.ref = "voyager" }
voyager-koin = { module = "cafe.adriel.voyager:voyager-koin", version.ref = "voyager" }
voyager-transitions = { module = "cafe.adriel.voyager:voyager-transitions", version.ref = "voyager" }

# Logging KMP
kermit = { module = "co.touchlab:kermit", version.ref = "kermit" }

# Animations KMP
compottie = { module = "io.github.alexzhirkevich:compottie", version.ref = "compottie" }

# Parsing KMP
ksoup = { module = "com.fleeksoft.ksoup:ksoup", version.ref = "ksoup" }

# Shimmer KMP
shimmer-kmp = { module = "com.valentinilk.shimmer:compose-shimmer", version.ref = "shimmer-kmp" }

# Testing KMP
kotlin-test = { module = "org.jetbrains.kotlin:kotlin-test" }
turbine = { module = "app.cash.turbine:turbine", version.ref = "turbine" }
kotlinx-coroutines-test = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-test", version = "1.8.1" }

# Android-only
androidx-appcompat = { module = "androidx.appcompat:appcompat", version.ref = "appcompat" }
androidx-compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "composeBom" }
androidx-activity-compose = { group = "androidx.activity", name = "activity-compose" }
androidx-paging-runtime = { module = "androidx.paging:paging-runtime-ktx", version.ref = "paging" }
androidx-paging-compose = { module = "androidx.paging:paging-compose", version.ref = "paging" }
androidx-work-runtime-ktx = { module = "androidx.work:work-runtime-ktx", version.ref = "work" }
androidx-datastore-preferences = { module = "androidx.datastore:datastore-preferences", version = "1.2.1" }
google-accompanist-permissions = { module = "com.google.accompanist:accompanist-permissions", version.ref = "accompanist" }
androidx-core-splashscreen = { module = "androidx.core:core-splashscreen", version.ref = "coreSplashscreen" }
androidx-browser = { module = "androidx.browser:browser", version.ref = "browser" }
androidx-palette-ktx = { group = "androidx.palette", name = "palette-ktx", version.ref = "paletteKtx" }
google-exoplayer-core = { module = "com.google.android.exoplayer:exoplayer-core", version.ref = "exoplayer" }
google-exoplayer-ui = { module = "com.google.android.exoplayer:exoplayer-ui", version.ref = "exoplayer" }
google-exoplayer-mediasession = { module = "com.google.android.exoplayer:extension-mediasession", version.ref = "exoplayer" }
app-update = { module = "com.google.android.play:app-update", version.ref = "appUpdate" }
app-update-ktx = { module = "com.google.android.play:app-update-ktx", version.ref = "appUpdate" }

[plugins]
android-application = { id = "com.android.application", version.ref = "androidGradlePlugin" }
android-library = { id = "com.android.library", version.ref = "androidGradlePlugin" }
android-lint = { id = "com.android.lint", version.ref = "androidGradlePlugin" }
android-kotlin-multiplatform-library = { id = "com.android.kotlin.multiplatform.library", version.ref = "androidGradlePlugin" }
kotlin-multiplatform = { id = "org.jetbrains.kotlin.multiplatform", version.ref = "kotlin" }
kotlin-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
compose-compiler = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
jetbrains-compose = { id = "org.jetbrains.compose", version.ref = "compose" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
androidx-room = { id = "androidx.room", version.ref = "room" }
```

### Final `shared/build.gradle.kts`

```kotlin
plugins {
    alias(libs.plugins.kotlin.multiplatform)
    alias(libs.plugins.android.kotlin.multiplatform.library)
    alias(libs.plugins.jetbrains.compose)
    alias(libs.plugins.compose.compiler)
    alias(libs.plugins.kotlin.serialization)
    alias(libs.plugins.android.lint)
    alias(libs.plugins.ksp)
    alias(libs.plugins.androidx.room)
}

kotlin {
    androidLibrary {
        namespace = "org.listenbrainz.shared"
        compileSdk = libs.versions.compileSdk.get().toInt()
        minSdk = libs.versions.minSdk.get().toInt()
    }

    iosX64()
    iosArm64()
    iosSimulatorArm64()

    sourceSets {
        commonMain {
            dependencies {
                implementation(libs.kotlin.stdlib)

                // Coroutines
                api(libs.kotlinx.coroutines.core)

                // Serialization
                implementation(libs.kotlinx.serialization.json)

                // DateTime
                implementation(libs.kotlinx.datetime)

                // DataStore KMP
                implementation(libs.datastore.preferences.core)

                // Room KMP
                implementation(libs.androidx.room.runtime)
                implementation(libs.androidx.sqlite.bundled)

                // Ktor KMP
                implementation(libs.ktor.client.core)
                implementation(libs.ktor.client.content.negotiation)
                implementation(libs.ktor.serialization.kotlinx.json)
                implementation(libs.ktor.client.logging)
                implementation(libs.ktor.client.websockets)

                // Koin KMP
                implementation(libs.koin.core)
                implementation(libs.koin.compose)

                // Lifecycle ViewModel KMP
                implementation(libs.lifecycle.viewmodel.kmp)
                implementation(libs.lifecycle.viewmodel.compose.kmp)

                // Paging KMP
                implementation(libs.androidx.paging.common)

                // Coil KMP
                implementation(libs.coil.compose)
                implementation(libs.coil.ktor)
                implementation(libs.coil.svg)

                // Navigation KMP
                implementation(libs.voyager.navigator)
                implementation(libs.voyager.screenmodel)
                implementation(libs.voyager.koin)
                implementation(libs.voyager.transitions)

                // Logging KMP
                implementation(libs.kermit)

                // Animations KMP
                implementation(libs.compottie)

                // Parsing KMP
                implementation(libs.ksoup)

                // Shimmer KMP
                implementation(libs.shimmer.kmp)

                // Compose KMP
                implementation(compose.runtime)
                implementation(compose.foundation)
                implementation(compose.material3)
                implementation(compose.ui)
                implementation(compose.components.resources)
                implementation(compose.components.uiToolingPreview)
            }
        }

        commonTest {
            dependencies {
                implementation(libs.kotlin.test)
                implementation(libs.turbine)
                implementation(libs.kotlinx.coroutines.test)
            }
        }

        androidMain {
            dependencies {
                // Ktor Android engine
                implementation(libs.ktor.client.okhttp)

                // Koin Android
                implementation(libs.koin.android)

                // DataStore Android
                implementation(libs.androidx.datastore.preferences)

                // Paging Android
                implementation(libs.androidx.paging.runtime)
                implementation(libs.androidx.paging.compose)

                // ExoPlayer
                implementation(libs.google.exoplayer.core)
                implementation(libs.google.exoplayer.ui)
                implementation(libs.google.exoplayer.mediasession)

                // WorkManager
                implementation(libs.androidx.work.runtime.ktx)

                // Permissions
                implementation(libs.google.accompanist.permissions)

                // Splash Screen
                implementation(libs.androidx.core.splashscreen)

                // Browser
                implementation(libs.androidx.browser)

                // Palette
                implementation(libs.androidx.palette.ktx)

                // App Update
                implementation(libs.app.update)
                implementation(libs.app.update.ktx)
            }
        }

        iosMain {
            dependencies {
                // Ktor iOS engine
                implementation(libs.ktor.client.darwin)
            }
        }
    }
}

```

### Final `app/build.gradle.kts` (androidApp — thin shell only)

```kotlin
dependencies {
    // Shared KMP module — all logic lives here
    implementation(project(":shared"))

    // Android shell only
    implementation(libs.androidx.appcompat)
    implementation(libs.androidx.activity.compose)

    // Spotify SDK (Android-only stub)
    api(project(":spotify-app-remote"))

    // Testing
    testImplementation(libs.kotlin.test)
    androidTestImplementation(libs.kotlin.test)
    androidTestImplementation(libs.androidx.compose.bom)
}
```

### File Structure at End of GSoC

```
listenbrainz-kmp/
├── shared/
│   ├── commonMain/
│   │   ├── model/        ← All data classes (100% shareable)
│   │   ├── repository/   ← interfaces + implementations
│   │   ├── viewmodel/    ← shared ViewModels
│   │   ├── di/           ← Koin modules
│   │   ├── util/         ← shared utilities
│   │   └── network/      ← Ktor APIs
│   ├── androidMain/      ← Android specific actuals
│   └── iosMain/          ← iOS specific actuals
├── androidApp/           ← thin Android shell
└── iosApp/               ← thin iOS shell (Swift)
```

---

## Phase 2: Relocating the Domain Models

Data classes representing the application’s state and business entities can be moved
directly from `androidMain` into `commonMain`. Since `kotlinx.serialization` and
`kotlinx-datetime` are already in place on the `cmp` branch, this phase focuses purely
on moving the actual model classes to `commonMain` and annotating them
with `@Serializable` where not already done.

### Key Changes:

* Package changes from `org.listenbrainz.android.model` → `org.listenbrainz.shared.model`
* All Android-specific imports removed from model classes
* `kotlinx-datetime` types used instead of Java date types where applicable

---



## Phase 3: Network Layer

### Platform-Specific Ktor Engines

```toml
# libs.versions.toml
[versions]
ktor-client = "3.4.1"

[libraries]
# commonMain
ktor-client-core = { module = "io.ktor:ktor-client-core", version.ref = "ktor-client" }
ktor-client-content-negotiation = { module = "io.ktor:ktor-client-content-negotiation", version.ref = "ktor-client" }
ktor-serialization-kotlinx-json = { module = "io.ktor:ktor-serialization-kotlinx-json", version.ref = "ktor-client" }
ktor-client-logging = { module = "io.ktor:ktor-client-logging", version.ref = "ktor-client" }
ktor-client-websockets = { module = "io.ktor:ktor-client-websockets", version.ref = "ktor-client" }
ktor-client-encoding = { module = "io.ktor:ktor-client-encoding", version.ref = "ktor-client" }

# androidMain only
ktor-client-okhttp = { module = "io.ktor:ktor-client-okhttp", version.ref = "ktor-client" }

# iosMain only
ktor-client-darwin = { module = "io.ktor:ktor-client-darwin", version.ref = "ktor-client" }
```

```kotlin
// shared/build.gradle.kts
commonMain {
    dependencies {
        implementation(libs.ktor.client.core)
        implementation(libs.ktor.client.content.negotiation)
        implementation(libs.ktor.serialization.kotlinx.json)
        implementation(libs.ktor.client.logging)
        implementation(libs.ktor.client.websockets)
        implementation(libs.ktor.client.encoding)
    }
}
androidMain {
    dependencies {
        implementation(libs.ktor.client.okhttp)
    }
}
iosMain {
    dependencies {
        implementation(libs.ktor.client.darwin)
    }
}
```

### Migrating createBaseHttpClient to commonMain

Currently `createBaseHttpClient` in `KoinModules.kt` is Android-only because
it uses `HttpClient(OkHttp)` and `ChuckerInterceptor`. We abstract the engine
via `expect/actual` and remove Chucker entirely.

#### Before (androidMain — KoinModules.kt):

```kotlin
//  Android-only: OkHttp engine + ChuckerInterceptor + androidContext()
private fun createBaseHttpClient(
    context: Context,
    appPreferences: AppPreferences? = null,
    baseUrl: String = LISTENBRAINZ_API_BASE_URL,
): HttpClient {
    return HttpClient(OkHttp) {  //  Android only engine
        install(ContentNegotiation) {
            json(jsonConfig)
        }
        if (BuildConfig.DEBUG) {
            engine {
                config {
                    addInterceptor(ChuckerInterceptor(context)) //  Android only
                }
            }
        }
        defaultRequest {
            url(baseUrl)
            appPreferences?.let { prefs ->
                runBlocking {
                    val accessToken = prefs.lbAccessToken.get()
                    if (accessToken.isNotEmpty()) {
                        header("Authorization", "Token $accessToken")
                    }
                }
            }
        }
    }
}
```

#### After — expect/actual for platform engine:

```kotlin
// commonMain
expect fun createPlatformHttpEngine(): HttpClientEngine
```

```kotlin
// androidMain
actual fun createPlatformHttpEngine(): HttpClientEngine = OkHttp.create()
```

```kotlin
// iosMain
actual fun createPlatformHttpEngine(): HttpClientEngine = Darwin.create()
```

#### After  createBaseHttpClient in commonMain:

```kotlin
//  commonMain no Android imports, no Context, no Chucker
private val jsonConfig = Json {
    ignoreUnknownKeys = true
    coerceInputValues = true
    isLenient = true
    encodeDefaults = true
    explicitNulls = false
}

fun createBaseHttpClient(
    appPreferences: AppPreferences? = null,
    baseUrl: String = LISTENBRAINZ_API_BASE_URL,
): HttpClient {
    return HttpClient(createPlatformHttpEngine()) {
        expectSuccess = true

        install(ContentNegotiation) {
            json(jsonConfig)
        }

        install(HttpRedirect) {
            checkHttpMethod = false
        }

        //  WebSockets support for SocketRepository
        install(WebSockets)

        install(Logging) {
            logger = object : Logger {
                override fun log(message: String) {
                    co.touchlab.kermit.Logger.d("Ktor") { message } //  Kermit
                }
            }
            level = LogLevel.ALL
        }

        defaultRequest {
            url(baseUrl)
            contentType(ContentType.Application.Json)
            appPreferences?.let { prefs ->
                runBlocking {
                    val accessToken = prefs.lbAccessToken.get()
                    if (accessToken.isNotEmpty()) {
                        header("Authorization", "Token $accessToken")
                    }
                }
            }
        }
    }
}
```

### Migrating Koin network Module to commonMain

#### Before (androidMain KoinModules.kt):

```kotlin
//  All services use androidContext() Android-only
val networkModule = module {
    single<HttpClient> {
        createBaseHttpClient(androidContext(), get<AppPreferences>())
    }
    single<UserService> {
        val httpClient = createBaseHttpClient(androidContext(), get<AppPreferences>())
        Ktorfit.Builder()
            .baseUrl(LISTENBRAINZ_API_BASE_URL)
            .httpClient(httpClient)
            .build()
            .createUserService()
    }
    single<FeedServiceKtor> { FeedServiceKtorImpl(get()) }
    // all other services same pattern
}
```

#### After (commonMain):

```kotlin
// No androidContext() pure KMP
val networkModule = module {
    single<HttpClient> {
        createBaseHttpClient(get<AppPreferences>())
    }
    single<UserService> {
        Ktorfit.Builder()
            .baseUrl(LISTENBRAINZ_API_BASE_URL)
            .httpClient(get<HttpClient>())
            .build()
            .createUserService()
    }
    single<FeedServiceKtor> { FeedServiceKtorImpl(get()) }
    // all other services same pattern
}
```

### Migrating UserService to commonMain

#### Before (androidMain):

```kotlin
// org.listenbrainz.android.service 
import org.listenbrainz.android.model.Listens
import org.listenbrainz.android.model.user.TopArtists

interface UserService {
    @GET("user/{user_name}/listen-count")
    suspend fun getListenCount(
        @Path("user_name") username: String
    ): Listens

    @GET("stats/user/{user_name}/artists")
    suspend fun getTopArtistsOfUser(
        @Path("user_name") username: String,
        @Query("range") rangeString: String?,
        @Query("count") count: Int = 25
    ): TopArtists
    // ... rest of methods
}
```

#### After (commonMain):

```kotlin
// org.listenbrainz.shared.service 
import org.listenbrainz.shared.model.Listens
import org.listenbrainz.shared.model.user.TopArtists

// Code is IDENTICAL — only package and imports change
interface UserService {
    @GET("user/{user_name}/listen-count")
    suspend fun getListenCount(
        @Path("user_name") username: String
    ): Listens

    @GET("stats/user/{user_name}/artists")
    suspend fun getTopArtistsOfUser(
        @Path("user_name") username: String,
        @Query("range") rangeString: String?,
        @Query("count") count: Int = 25
    ): TopArtists
    // ... rest of methods unchanged
}
```

### Migrating FeedServiceKtorImpl to commonMain

#### Before (androidMain):

```kotlin
// org.listenbrainz.android.service 
import org.listenbrainz.android.model.feed.FeedData
import org.listenbrainz.android.model.feed.FeedEventDeletionData

class FeedServiceKtorImpl(
    private val httpClient: HttpClient
) : FeedServiceKtor {
    override suspend fun getFeedEvents(
        username: String,
        count: Int,
        maxTs: Long?,
        minTs: Long?
    ): FeedData {
        return httpClient.get("user/$username/feed/events") {
            url {
                parameters.append("count", count.toString())
                maxTs?.let { parameters.append("max_ts", it.toString()) }
                minTs?.let { parameters.append("min_ts", it.toString()) }
            }
        }.body()
    }
    // ... rest unchanged
}
```

#### After (commonMain):

```kotlin
// org.listenbrainz.shared.service 
import org.listenbrainz.shared.model.feed.FeedData
import org.listenbrainz.shared.model.feed.FeedEventDeletionData

// Code body is IDENTICAL only package and imports change
class FeedServiceKtorImpl(
    private val httpClient: HttpClient
) : FeedServiceKtor {
    override suspend fun getFeedEvents(
        username: String,
        count: Int,
        maxTs: Long?,
        minTs: Long?
    ): FeedData {
        return httpClient.get("user/$username/feed/events") {
            url {
                parameters.append("count", count.toString())
                maxTs?.let { parameters.append("max_ts", it.toString()) }
                minTs?.let { parameters.append("min_ts", it.toString()) }
            }
        }.body()
    }
    // ... rest unchanged
}
```

### Replace socket.io-client with ktor-client-websockets

#### Before (androidMain — SocketRepositoryImpl.kt):

```kotlin
//  Android only socket.io
class SocketRepositoryImpl : SocketRepository {

    private val json = Json { ignoreUnknownKeys = true }

    private val socket: Socket = IO.socket(
        "https://listenbrainz.org/",
        IO.Options.builder().setPath("/socket.io/").build()
    )

    override fun listen(usernameProvider: suspend () -> String) = callbackFlow {
        val username = usernameProvider()
        socket
            .on(EVENT_CONNECT) {
                Log.d("Listen socket connected.")
                socket.emit("json", JSONObject().put("user", username))
            }
            .on("playing_now") {
                val listen = json.decodeFromString<Listen>(it[0] as String)
                trySendBlocking(listen)
                    .onFailure { throwable -> throwable?.printStackTrace() }
            }
            .on("listen") {
                val listen = json.decodeFromString<Listen>(it[0] as String)
                trySendBlocking(listen)
                    .onFailure { throwable -> throwable?.printStackTrace() }
            }
        socket.connect()
        awaitClose {
            socket.disconnect()
            socket.off()
        }
    }
}
```

#### After (commonMain — SocketRepositoryImpl.kt):

```kotlin
//  KMP ktor client-websockets
class SocketRepositoryImpl(
    private val httpClient: HttpClient  // injected via Koin
) : SocketRepository {

    private val json = Json { ignoreUnknownKeys = true }
    private val log = Logger.withTag("Socket")

    override fun listen(usernameProvider: suspend () -> String) = callbackFlow {
        val username = usernameProvider()

        // use callbackFlow's own scope directly
        launch {
            while (isActive) {
                try {
                    val session = httpClient.webSocketSession(
                        urlString = "wss://listenbrainz.org/socket.io/?EIO=4&transport=websocket"
                    )
                    log.d { "Connected" }

                    for (frame in session.incoming) {
                        if (frame !is Frame.Text) continue
                        val data = frame.readText()

                        when {
                            // Engine.IO handshake
                            data.startsWith("0") -> {
                                session.send(Frame.Text("40"))
                                val subscribe = """42["json",{"user":"$username"}]"""
                                session.send(Frame.Text(subscribe))
                            }
                            data == "2" -> {
                                session.send(Frame.Text("3"))
                            }
                            // listen/playing_now events
                            data.startsWith("42") -> {
                                val array = json.parseToJsonElement(
                                    data.removePrefix("42")
                                ) as? JsonArray ?: continue

                                val event = array.getOrNull(0)
                                    ?.jsonPrimitive?.content ?: continue
                                val payload = array.getOrNull(1) ?: continue

                                if (event == "listen" || event == "playing_now") {
                                    val listen = runCatching {
                                        json.decodeFromJsonElement<Listen>(payload)
                                    }.getOrNull() ?: continue
                                    trySend(listen)
                                }
                            }
                        }
                    }
                    session.close()
                    log.d { "Disconnected" }

                } catch (e: Exception) {
                    log.e(e) { "Socket error: ${e.message}" }
                    delay(2000) // retry after 2 seconds
                }
            }
        }

        // callbackFlow handles cancellation
        awaitClose {
            log.d { "Socket flow closed" }
        }
    }
}
```

#### Key Changes:

- `IO.socket()` → `httpClient.webSocketSession()` (KMP)

- `JSONObject` (Android-only) → `kotlinx.serialization` (KMP)

* `Log.d` (Android-only) → `Kermit Logger` (KMP)

* HttpClient injected via Koin

### Replace Logger-Android with Kermit

#### Before (androidMain — KoinModules.kt):

```kotlin
//  Android-only logger
install(Logging) {
    logger = object : Logger {
        override fun log(message: String) {
            Log.d("Ktor: $message")  //  Android Log
        }
    }
    level = LogLevel.ALL
}
```

#### After (commonMain):

```kotlin
install(Logging) {
    logger = object : Logger {
        override fun log(message: String) {
            KermitLogger.d(tag = "Ktor") { message }
        }
    }
    level = LogLevel.ALL
}
```

### Remove Chucker

Chucker is an Android-only HTTP inspector used only in debug builds.
It has no KMP equivalent and must be removed entirely


---

## Phase 4: Local Data Storage (Room KMP Migration)

ListenBrainz uses two Room databases:
- `BrainzPlayerDatabase` — caches local media (songs, albums, artists, playlists)
- `ListensSubmissionDatabase` — offline scrobble queue (pending listens)

Both databases currently live in `androidMain`. This phase migrates all
entities, DAOs, and database classes to `commonMain`.

### Step 1: Dependency Update
```toml
# libs.versions.toml
[versions]
room = "2.7.2"
sqlite = "2.6.2"
ksp = "2.3.0"

[libraries]
androidx-room-runtime = { module = "androidx.room:room-runtime", version.ref = "room" }
androidx-room-compiler = { module = "androidx.room:room-compiler", version.ref = "room" }
androidx-sqlite-bundled = { module = "androidx.sqlite:sqlite-bundled", version.ref = "sqlite" }

[plugins]
androidx-room = { id = "androidx.room", version.ref = "room" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
```
```kotlin
// shared/build.gradle.kts
commonMain {
    dependencies {
        implementation(libs.androidx.room.runtime)
        implementation(libs.androidx.sqlite.bundled)
    }
}

room {
    schemaDirectory("$projectDir/schemas")
}

dependencies {
    add("kspAndroid", libs.androidx.room.compiler)
    add("kspIosX64", libs.androidx.room.compiler)
    add("kspIosArm64", libs.androidx.room.compiler)
    add("kspIosSimulatorArm64", libs.androidx.room.compiler)
}
```

### Step 2: Migrate Entities to commonMain

#### Before (app module androidMain):
```kotlin
// org.listenbrainz.android.model 
package org.listenbrainz.android.model

import androidx.room.Entity
import androidx.room.PrimaryKey
import kotlinx.serialization.Serializable

@Serializable
@Entity(tableName = "ALBUMS")
data class AlbumEntity(
    @PrimaryKey
    val albumId: Long = 0L,
    val title: String = "",
    val artist: String = "",
    val albumArt: String = ""
)
```

#### After (shared/src/commonMain):
```kotlin
// org.listenbrainz.shared.model 
package org.listenbrainz.shared.model

import androidx.room.Entity
import androidx.room.PrimaryKey
import kotlinx.serialization.Serializable

// Code is IDENTICAL only package changes
@Serializable
@Entity(tableName = "ALBUMS")
data class AlbumEntity(
    @PrimaryKey
    val albumId: Long = 0L,
    val title: String = "",
    val artist: String = "",
    val albumArt: String = ""
)
```

Same migration applies to `SongEntity`, `ArtistEntity`, `PlaylistEntity`
and `ListenSubmitBody.Payload`.

### Step 3: Migrate DAOs to commonMain

#### Before (app module — androidMain):
```kotlin
// org.listenbrainz.android.model.dao 
package org.listenbrainz.android.model.dao

@Dao
interface AlbumDao {

    @Query("SELECT * FROM ALBUMS WHERE albumId = :albumId")
    fun getAlbumEntity(albumId: Long): Flow<AlbumEntity>

    @Query("SELECT * FROM ALBUMS ORDER BY `title`")
    fun getAlbumEntities(): Flow<List<AlbumEntity>>

    @Query("SELECT * FROM ALBUMS ORDER BY `title`")
    fun getAlbumEntitiesAsList(): List<AlbumEntity>

    @Insert(onConflict = OnConflictStrategy.IGNORE)
    suspend fun addAlbums(albumEntities: List<AlbumEntity>)

    @Delete
    suspend fun deleteAlbum(albumEntity: AlbumEntity)
}
```

#### After (shared/src/commonMain):
```kotlin
// org.listenbrainz.shared.model.dao 
package org.listenbrainz.shared.model.dao

import androidx.room.*
import kotlinx.coroutines.flow.Flow
import org.listenbrainz.shared.model.AlbumEntity

// Code is IDENTICAL only package changes
@Dao
interface AlbumDao {

    @Query("SELECT * FROM ALBUMS WHERE albumId = :albumId")
    fun getAlbumEntity(albumId: Long): Flow<AlbumEntity>

    @Query("SELECT * FROM ALBUMS ORDER BY `title`")
    fun getAlbumEntities(): Flow<List<AlbumEntity>>

    @Query("SELECT * FROM ALBUMS ORDER BY `title`")
    fun getAlbumEntitiesAsList(): List<AlbumEntity>

    @Insert(onConflict = OnConflictStrategy.IGNORE)
    suspend fun addAlbums(albumEntities: List<AlbumEntity>)

    @Delete
    suspend fun deleteAlbum(albumEntity: AlbumEntity)
}
```

Same migration applies to `SongDao`, `ArtistDao`, `PlaylistDao`
and `PendingListensDao`.

### Step 4: Migrate Database Classes to commonMain

#### Before (app module  androidMain):
```kotlin
// org.listenbrainz.android.di.brainzplayer 
@Database(
    entities = [
        SongEntity::class,
        AlbumEntity::class,
        ArtistEntity::class,
        PlaylistEntity::class
    ],
    version = 2,
    exportSchema = false
)
@TypeConverters(TypeConverter::class)
abstract class BrainzPlayerDatabase : RoomDatabase() {
    abstract fun songDao(): SongDao
    abstract fun albumDao(): AlbumDao
    abstract fun artistDao(): ArtistDao
    abstract fun playlistDao(): PlaylistDao
}
```

#### After (shared/src/commonMain):
```kotlin
// org.listenbrainz.shared.database 
@Database(
    entities = [
        SongEntity::class,
        AlbumEntity::class,
        ArtistEntity::class,
        PlaylistEntity::class
    ],
    version = 2,
    exportSchema = true  //  required for Room KMP
)
@TypeConverters(TypeConverter::class)
abstract class BrainzPlayerDatabase : RoomDatabase() {
    abstract fun songDao(): SongDao
    abstract fun albumDao(): AlbumDao
    abstract fun artistDao(): ArtistDao
    abstract fun playlistDao(): PlaylistDao
}
```

Same migration applies to `ListensSubmissionDatabase`.

### Step 5: Database Builder via expect/actual

The database file path is platform-specific this is the only part
that cannot live in `commonMain`.

#### commonMain:
```kotlin
expect fun getBrainzPlayerDatabaseBuilder(): RoomDatabase.Builder<BrainzPlayerDatabase>
expect fun getListensSubmissionDatabaseBuilder(): RoomDatabase.Builder<ListensSubmissionDatabase>
```

#### androidMain:
```kotlin
actual fun getBrainzPlayerDatabaseBuilder(): RoomDatabase.Builder<BrainzPlayerDatabase> {
    val dbFilePath = NSFileManager.defaultManager.URLForDirectory(
        directory = NSDocumentDirectory,
        inDomain = NSUserDomainMask,
        appropriateForURL = null,
        create = false,
        error = null
    )!!.URLByAppendingPathComponent("brainzplayer_database.db")!!.path!!

    return Room.databaseBuilder<BrainzPlayerDatabase>(
        name = dbFilePath,
        factory = { BrainzPlayerDatabase::class.instantiateImpl() }
    )
}
actual fun getListensSubmissionDatabaseBuilder(): RoomDatabase.Builder<ListensSubmissionDatabase> {
    val context: Context = KoinPlatformTools.defaultContext().get()

    return Room.databaseBuilder<ListensSubmissionDatabase>(
        context = context,
        name = "listens_scrobble_database.db"
    )
}
```

#### iosMain:
```kotlin
actual fun getBrainzPlayerDatabaseBuilder(): RoomDatabase.Builder<BrainzPlayerDatabase> {
    val dbFilePath = NSFileManager.defaultManager.URLForDirectory(
        directory = NSDocumentDirectory,
        inDomain = NSUserDomainMask,
        appropriateForURL = null,
        create = false,
        error = null
    )!!.URLByAppendingPathComponent("brainzplayer_database")!!.path!!
    return Room.databaseBuilder<BrainzPlayerDatabase>(
        name = dbFilePath,
        factory = { BrainzPlayerDatabase::class.instantiateImpl() }
    )
}

actual fun getListensSubmissionDatabaseBuilder(): RoomDatabase.Builder<ListensSubmissionDatabase> {
    val dbFilePath = NSFileManager.defaultManager.URLForDirectory(
        directory = NSDocumentDirectory,
        inDomain = NSUserDomainMask,
        appropriateForURL = null,
        create = false,
        error = null
    )!!.URLByAppendingPathComponent("listens_scrobble_database")!!.path!!
    return Room.databaseBuilder<ListensSubmissionDatabase>(
        name = dbFilePath,
        factory = { ListensSubmissionDatabase::class.instantiateImpl() }
    )
}
```

### Step 6: Update Koin databaseModule

#### Before (androidMain KoinModules.kt):
```kotlin
//  Android only Room.databaseBuilder with androidContext()
val databaseModule = module {
    single<BrainzPlayerDatabase> {
        Room.databaseBuilder(
            androidContext(),
            BrainzPlayerDatabase::class.java,
            "brainzplayer_database"
        )
            .addMigrations(Migrations.MIGRATION_1_2)
            .build()
    }
    single<ListensSubmissionDatabase> {
        Room.databaseBuilder(
            androidContext(),
            ListensSubmissionDatabase::class.java,
            "listens_scrobble_database"
        ).build()
    }
}
```

#### After (commonMain):
```kotlin
//  No androidContext()  uses expect/actual builders
val databaseModule = module {
    single<BrainzPlayerDatabase> {
        getBrainzPlayerDatabaseBuilder().build()
    }
    single<ListensSubmissionDatabase> {
        getListensSubmissionDatabaseBuilder().build()
    }
}
```
---

## Phase 5: Migration of Repository Layer to commonMain

All repositories must be migrated to `commonMain` before ViewModel migration.
The key change is removing Android specific imports and replacing them with
KMP equivalents. Here are the key migration examples:

### Paging Migration (commonMain)
```toml
# libs.versions.toml
[versions]
paging = "3.3.5"

[libraries]
# commonMain
androidx-paging-common = { module = "androidx.paging:paging-common", version.ref = "paging" }

# androidMain only
androidx-paging-runtime = { module = "androidx.paging:paging-runtime-ktx", version.ref = "paging" }
androidx-paging-compose = { module = "androidx.paging:paging-compose", version.ref = "paging" }
```
```kotlin
// shared/build.gradle.kts
commonMain {
    dependencies {
        implementation(libs.androidx.paging.common)
    }
}
androidMain {
    dependencies {
        implementation(libs.androidx.paging.runtime)
        implementation(libs.androidx.paging.compose)
    }
}
```

#### Before (androidMain  `CollabPlaylistPagingSource.kt`):
```kotlin
class CollabPlaylistPagingSource(
    private val username: String?,
    private val onError: (error: ResponseError?) -> Unit,
    private val playlistDataRepository: PlaylistDataRepository,
    private val ioDispatcher: CoroutineDispatcher
) : PagingSource<Int, UserPlaylist>() {

    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, UserPlaylist> {
        val result = withContext(ioDispatcher) {
            playlistDataRepository.getUserCollabPlaylists(
                username = username,
                offset = params.key ?: 0,
                count = params.loadSize
            )
        }
        return when (result.status) {
            Resource.Status.SUCCESS -> {
                val data = (result.data?.playlists ?: emptyList()).map { it.playlist }
                val nextKey = if (data.isEmpty()) null
                              else params.key?.plus(params.loadSize)
                LoadResult.Page(
                    data = data,
                    prevKey = null,
                    nextKey = nextKey
                )
            }
            else -> {
                onError(result.error)
                LoadResult.Error(Exception(result.error?.toast))
            }
        }
    }
}
```

#### After (commonMain):
```kotlin
// Code body is IDENTICAL only package and imports change
class CollabPlaylistPagingSource(
    private val username: String?,
    private val onError: (error: ResponseError?) -> Unit,
    private val playlistDataRepository: PlaylistDataRepository,
    private val ioDispatcher: CoroutineDispatcher
) : PagingSource<Int, UserPlaylist>() {

    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, UserPlaylist> {
        val result = withContext(ioDispatcher) {
            playlistDataRepository.getUserCollabPlaylists(
                username = username,
                offset = params.key ?: 0,
                count = params.loadSize
            )
        }
        return when (result.status) {
            Resource.Status.SUCCESS -> {
                val data = (result.data?.playlists ?: emptyList()).map { it.playlist }
                val nextKey = if (data.isEmpty()) null
                              else params.key?.plus(params.loadSize)
                LoadResult.Page(
                    data = data,
                    prevKey = null,
                    nextKey = nextKey
                )
            }
            else -> {
                onError(result.error)
                LoadResult.Error(Exception(result.error?.toast))
            }
        }
    }
}
```

### SocialRepository Interface Migration

#### Before (androidMain):
```kotlin
import java.text.SimpleDateFormat  // Java only
import java.util.Calendar          // Java only
import java.util.Locale            // Java only

interface SocialRepository {
    suspend fun getFollowers(username: String?): Resource<SocialData>
    suspend fun followUser(username: String): Resource<SocialResponse>
    // ...

    companion object {
        fun getPinDateString(): String {
            val formatter = SimpleDateFormat("MMM dd, hh:mm aaa", Locale.getDefault()) 
            val calendar = Calendar.getInstance(Locale.ENGLISH)
                .apply { timeInMillis = getPinTimeMs() }
            return formatter.format(calendar.time)
        }
        fun getPinTimeMs(): Long = System.currentTimeMillis() + 604800000
    }
}
```

#### After (commonMain):
```kotlin
import kotlinx.datetime.Clock                    //  KMP
import kotlinx.datetime.TimeZone                 //  KMP
import kotlinx.datetime.toLocalDateTime          //  KMP

interface SocialRepository {
    suspend fun getFollowers(username: String?): Resource<SocialData>
    suspend fun followUser(username: String): Resource<SocialResponse>
    // ...

    companion object {
        fun getPinDateString(): String {
            val instant = kotlinx.datetime.Instant.fromEpochMilliseconds(getPinTimeMs())
            val local = instant.toLocalDateTime(TimeZone.currentSystemDefault())
            return "${local.month.name.take(3)} ${local.dayOfMonth}, " +
                   "${local.hour}:${local.minute.toString().padStart(2, '0')}"
        }
        fun getPinTimeMs(): Long = Clock.System.now().toEpochMilliseconds() + 604800000
    }
}
```

### SocialRepositoryImpl Migration

#### Before (androidMain):
```kotlin
import org.listenbrainz.android.model.SocialData      //  android package
import org.listenbrainz.android.service.SocialService  //  android package

class SocialRepositoryImpl(
    private val service: SocialService,
    private val appPreferences: AppPreferences
) : SocialRepository {

    override suspend fun getFollowers(username: String?): Resource<SocialData> = parseResponse {
        failIf(username == null) { ResponseError.AuthHeaderNotFound() }
        service.getFollowersData(username = username!!)
    }

    override suspend fun followUser(username: String): Resource<SocialResponse> = parseResponse {
        service.followUser(username = username)
    }
    // ... rest of methods
}
```

#### After (commonMain):
```kotli
import org.listenbrainz.shared.model.SocialData      //  shared package
import org.listenbrainz.shared.service.SocialService  //  shared package

// Code body is IDENTICAL only package and imports change
class SocialRepositoryImpl(
    private val service: SocialService,
    private val appPreferences: AppPreferences
) : SocialRepository {

    override suspend fun getFollowers(username: String?): Resource<SocialData> = parseResponse {
        failIf(username == null) { ResponseError.AuthHeaderNotFound() }
        service.getFollowersData(username = username!!)
    }

    override suspend fun followUser(username: String): Resource<SocialResponse> = parseResponse {
        service.followUser(username = username)
    }
    // ... rest of methods
}
```

### Key Changes Across All Repositories:
- Package changes
- `java.text.SimpleDateFormat` / `java.util.Calendar` → `kotlinx-datetime`
- `System.currentTimeMillis()` → `Clock.System.now().toEpochMilliseconds()`
- `androidx.paging.PagingSource` → `paging-common` (same API, KMP compatible)
- `AppPreferences` already KMP interface  no changes needed
- All business logic remains **identical** only imports and packages change

---

## Phase 6: Migration of ViewModel to shared

### Step 1: Add JetBrains Lifecycle KMP Dependency
```toml
# libs.versions.toml
[versions]
lifecycle-kmp = "2.8.0"

[libraries]
lifecycle-viewmodel-kmp = {
    module = "org.jetbrains.androidx.lifecycle:lifecycle-viewmodel",
    version.ref = "lifecycle-kmp"
}
lifecycle-viewmodel-compose-kmp = {
    module = "org.jetbrains.androidx.lifecycle:lifecycle-viewmodel-compose",
    version.ref = "lifecycle-kmp"
}
```
```kotlin
// shared/build.gradle.kts
commonMain {
    dependencies {
        implementation(libs.lifecycle.viewmodel.kmp)
        implementation(libs.lifecycle.viewmodel.compose.kmp)
    }
}
```

>  ViewModels must extend KMP `ViewModel()` NOT plain class with
> `CoroutineScope`. `viewModelScope` auto cancels on both platforms 
> My PR #732 (SocialViewModel) used plain `CoroutineScope`  i will update soon

---

### Simple ViewModel Migration AlbumViewModel

#### Before (androidMain):
```kotlin
import androidx.lifecycle.viewModelScope  //  Android only
import org.listenbrainz.android.repository.album.AlbumRepository  //  android package
import org.listenbrainz.android.ui.screens.album.AlbumUiState     //  android package

class AlbumViewModel(
    private val repository: AlbumRepository,
    private val ioDispatcher: CoroutineDispatcher
) : BaseViewModel<AlbumUiState>() {

    private val albumUIStateFlow = MutableStateFlow(AlbumUiState())

    suspend fun fetchAlbumData(albumMbid: String?) {
        val albumInfo = repository.fetchAlbumInfo(albumMbid).data
        val albumData = repository.fetchAlbum(albumMbid).data
        val albumReviews = repository.fetchAlbumReviews(albumMbid).data

        albumUIStateFlow.emit(AlbumUiState(
            isLoading = false,
            name = albumInfo?.title,
            // ... rest of state
        ))
    }

    override val uiState: StateFlow<AlbumUiState> = createUiStateFlow()

    override fun createUiStateFlow(): StateFlow<AlbumUiState> {
        return combine(albumUIStateFlow) { it[0] }
            .stateIn(
                scope = viewModelScope,  //  Android viewModelScope
                started = SharingStarted.Eagerly,
                AlbumUiState()
            )
    }
}
```

#### After (commonMain):
```kotlin
import androidx.lifecycle.ViewModel          //  KMP ViewModel
import androidx.lifecycle.viewModelScope     //  KMP viewModelScope
import org.listenbrainz.shared.repository.album.AlbumRepository  //  shared package
import org.listenbrainz.shared.ui.screens.album.AlbumUiState     //  shared package

// Code body is IDENTICAL  only package, imports and ViewModel base change
class AlbumViewModel(
    private val repository: AlbumRepository,
    private val ioDispatcher: CoroutineDispatcher
) : ViewModel() {  //  KMP ViewModel

    private val albumUIStateFlow = MutableStateFlow(AlbumUiState())

    suspend fun fetchAlbumData(albumMbid: String?) {
        val albumInfo = repository.fetchAlbumInfo(albumMbid).data
        val albumData = repository.fetchAlbum(albumMbid).data
        val albumReviews = repository.fetchAlbumReviews(albumMbid).data

        albumUIStateFlow.emit(AlbumUiState(
            isLoading = false,
            name = albumInfo?.title,
            // ... rest of state
        ))
    }

    override val uiState: StateFlow<AlbumUiState> = createUiStateFlow()

    override fun createUiStateFlow(): StateFlow<AlbumUiState> {
        return combine(albumUIStateFlow) { it[0] }
            .stateIn(
                scope = viewModelScope,  //  KMP viewModelScope auto-cancels both platforms
                started = SharingStarted.Eagerly,
                AlbumUiState()
            )
    }
}
```

---

### Complex ViewModel SocialViewModel (Platform Boundaries)

`SocialViewModel` has two Android specific dependencies that cannot move
to `commonMain`:

- `android.net.Uri` 
- `RemotePlaybackHandler` 
- `R.string.*`

The solution is to split into a **shared ViewModel** in `commonMain`
and a **thin Android wrapper** in `androidMain`.

#### Before (androidMain):
```kotlin
import android.net.Uri                              //  Android only
import org.listenbrainz.android.R                  //  Android only

class SocialViewModel(
    private val repository: SocialRepository,
    private val listensRepository: ListensRepository,
    private val appPreferences: AppPreferences,
    private val remotePlaybackHandler: RemotePlaybackHandler, 
    private val ioDispatcher: CoroutineDispatcher,
    private val defaultDispatcher: CoroutineDispatcher,
) : FollowUnfollowModel<SocialUiState>(repository, ioDispatcher) {

    init {
        viewModelScope.launch(defaultDispatcher) {
            searchFollowerQuery.collectLatest { query ->
                if (query.isEmpty()) return@collectLatest
                val result = repository.getFollowers(appPreferences.username.get())
                if (result.status == Resource.Status.SUCCESS) {
                    searchFollowerResult.emit(
                        result.data?.followers?.filter {
                            it.startsWith(query, ignoreCase = true)
                        } ?: emptyList()
                    )
                }
            }
        }
    }

    fun recommend(metadata: Metadata) {
        viewModelScope.launch(ioDispatcher) {
            val result = repository.postRecommendationToAll(
                username = appPreferences.username.get(),
                data = RecommendationData(...)
            )
            if (result.status == Resource.Status.SUCCESS) {
                emitMsg(R.string.recommendation_greeting) 
            }
        }
    }

    fun playListen(trackMetadata: TrackMetadata) {
        val spotifyId = trackMetadata.additionalInfo?.spotifyId
        if (spotifyId != null) {
            Uri.parse(spotifyId).lastPathSegment?.let { trackId -> 
                remotePlaybackHandler.playUri(trackId = trackId)    
            }
        }
    }
}
```

#### After  Shared ViewModel (commonMain):
```kotlin
// Android-specific: RemotePlaybackHandler, Uri, R.string moved out
class SocialViewModel(
    private val repository: SocialRepository,
    private val listensRepository: ListensRepository,
    private val appPreferences: AppPreferences,
    private val ioDispatcher: CoroutineDispatcher,
    private val defaultDispatcher: CoroutineDispatcher,
) : ViewModel() {  //  KMP ViewModel no RemotePlaybackHandler

    private val inputSearchFollowerQuery = MutableStateFlow("")

    @OptIn(FlowPreview::class)
    private val searchFollowerQuery =
        inputSearchFollowerQuery.asStateFlow().debounce(500).distinctUntilChanged()
    private val searchFollowerResult = MutableStateFlow<List<String>>(emptyList())

    val uiState: StateFlow<SocialUiState> = combine(
        searchFollowerResult,
        errorFlow,
        successMsgFlow
    ) { searchResult, error, message ->
        SocialUiState(searchResult, error, message)
    }.stateIn(viewModelScope, SharingStarted.Eagerly, SocialUiState())

    init {
        viewModelScope.launch(defaultDispatcher) {
            searchFollowerQuery.collectLatest { query ->
                if (query.isEmpty()) return@collectLatest
                val result = repository.getFollowers(appPreferences.username.get())
                if (result.status == Resource.Status.SUCCESS) {
                    searchFollowerResult.emit(
                        result.data?.followers?.filter {
                            it.startsWith(query, ignoreCase = true) ||
                            it.contains(query, ignoreCase = true)
                        } ?: emptyList()
                    )
                } else {
                    emitError(error = result.error)
                }
            }
        }
    }

    fun searchUser(query: String) {
        viewModelScope.launch {
            inputSearchFollowerQuery.emit(query)
        }
    }

    fun recommend(metadata: Metadata) {
        viewModelScope.launch(ioDispatcher) {
            val trackMetadata = metadata.trackMetadata ?: return@launch
            val result = repository.postRecommendationToAll(
                username = appPreferences.username.get(),
                data = RecommendationData(
                    metadata = RecommendationMetadata(
                        trackName = trackMetadata.trackName ?: return@launch,
                        artistName = trackMetadata.artistName.orEmpty(),
                        recordingMbid = trackMetadata.mbidMapping?.recordingMbid,
                        recordingMsid = trackMetadata.additionalInfo?.recordingMsid
                    )
                )
            )
            if (result.status == Resource.Status.FAILED) emitError(result.error)
            else if (result.status == Resource.Status.SUCCESS) emitSuccess() //  no R.string
        }
    }

    fun pin(metadata: Metadata, blurbContent: String?) {
        viewModelScope.launch(ioDispatcher) {
            val result = repository.pin(
                recordingMsid = metadata.trackMetadata?.additionalInfo?.recordingMsid,
                recordingMbid = metadata.trackMetadata?.mbidMapping?.recordingMbid,
                blurbContent = blurbContent
            )
            if (result.status == Resource.Status.FAILED) emitError(result.error)
            else if (result.status == Resource.Status.SUCCESS) emitSuccess()
        }
    }

    suspend fun getFollowers(): Resource<SocialData> {
        return repository.getFollowers(appPreferences.username.get()).also {
            if (it.status == Resource.Status.FAILED) emitError(it.error)
        }
    }
}
```

#### After  Android thin wrapper (androidMain):
```kotlin
// androidMain  handles Android specific: Uri, R.string, RemotePlaybackHandler
class AndroidSocialViewModel(
    private val sharedViewModel: SocialViewModel,         
    private val remotePlaybackHandler: RemotePlaybackHandler
) : ViewModel() {

    val uiState = sharedViewModel.uiState

    fun searchUser(query: String) = sharedViewModel.searchUser(query)
    fun recommend(metadata: Metadata) = sharedViewModel.recommend(metadata)
    fun pin(metadata: Metadata, blurbContent: String?) = sharedViewModel.pin(metadata, blurbContent)

    //  Android specific playback stays in androidMain
    fun playListen(trackMetadata: TrackMetadata) {
        val spotifyId = trackMetadata.additionalInfo?.spotifyId
        if (spotifyId != null) {
            Uri.parse(spotifyId).lastPathSegment?.let { trackId ->
                remotePlaybackHandler.playUri(
                    trackId = trackId,
                    onFailure = { playFromYoutubeMusic(trackMetadata) }
                )
            }
        } else {
            playFromYoutubeMusic(trackMetadata)
        }
    }

    private fun playFromYoutubeMusic(trackMetadata: TrackMetadata) {
        viewModelScope.launch {
            remotePlaybackHandler.playOnYoutube {
                sharedViewModel.searchYoutubeMusicVideoId(trackMetadata)
            }
        }
    }
}
```

---

### Update Koin viewModelModule

#### Before (androidMain  KoinModules.kt):
```kotlin
//  All viewModels registered with androidContext dependencies
val viewModelModule = module {
    viewModel { SocialViewModel(
        get(), get(), get(), get(),
        get(named(IO_DISPATCHER)),
        get(named(DEFAULT_DISPATCHER))
    )}
    viewModel { AlbumViewModel(get(), get(named(IO_DISPATCHER))) }
    // ... all others
}
```

#### After shared viewModelModule (commonMain):
```kotlin
//  Shared ViewModels registered in commonMain
val viewModelModule = module {
    viewModel { SocialViewModel(
        get(), get(), get(),
        get(named(IO_DISPATCHER)),
        get(named(DEFAULT_DISPATCHER))
    )}
    viewModel { AlbumViewModel(get(), get(named(IO_DISPATCHER))) }
    // ... all others
}
```

#### After  Android specific additions (androidMain):
```kotlin
//  Android-only wrappers registered in androidMain
val androidViewModelModule = module {
    viewModel { AndroidSocialViewModel(get(), get()) }
}
```

---

### Key Changes:
- All ViewModels extend KMP `ViewModel()`  `viewModelScope` auto cancels on both platforms
- `android.net.Uri` → stays in `androidMain` thin wrapper
- `R.string.*` → replaced with string keys or moved to androidMain wrapper
- `RemotePlaybackHandler` → stays in `androidMain` thin wrapper
- Pure business logic ViewModels (like `AlbumViewModel`) move directly to `commonMain` unchanged
- Complex ViewModels (like `SocialViewModel`) split into shared + thin Android wrapper

---

## Phase 7: The User Interface Transition (Compose Multiplatform)

The final layer of the shared code migration involves the visual presentation.
The ListenBrainz application currently uses Jetpack Compose organizing its UI
within `ui/screens/`, `ui/components/`, and `ui/themes/` directories.
The majority of composable functions can be moved directly into `commonMain`.
However, several UI adjacent libraries must be substituted to ensure
multiplatform compatibility.

### Step 1: Dependency Update
```toml
# libs.versions.toml
[versions]
kotlin = "2.3.10"
compose = "1.10.2"
voyager = "1.1.0"
compottie = "2.0.0"
shimmer-kmp = "1.3.3"

[plugins]
jetbrains-compose = { id = "org.jetbrains.compose", version.ref = "compose" }
compose-compiler = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
```
```kotlin
// shared/build.gradle.kts
plugins {
    alias(libs.plugins.jetbrains.compose)
    alias(libs.plugins.compose.compiler)
}

commonMain {
    dependencies {
        implementation(compose.runtime)
        implementation(compose.foundation)
        implementation(compose.material3)
        implementation(compose.ui)
        implementation(compose.components.resources)
        implementation(compose.components.uiToolingPreview)
        // Navigation
        implementation(libs.voyager.navigator)
        implementation(libs.voyager.screenmodel)
        implementation(libs.voyager.koin)
        implementation(libs.voyager.transitions)
        // Animations
        implementation(libs.compottie)
        // Shimmer
        implementation(libs.shimmer.kmp)
    }
}
```

### Step 2: Resource Migration

Android's `res/` directory system is incompatible with iOS.
All UI resources must be migrated to the shared `composeResources/` directory.
```
# Before (Android only)
app/src/main/res/
├── drawable/
│   ├── ic_play.xml        
│   └── globe.png          
└── raw/
    └── login.json          Lottie file

# After (shared — both platforms)
shared/src/commonMain/composeResources/
├── drawable/
│   ├── ic_play.xml        
│   └── globe.png          
└── files/
    └── login.json          Compottie file
```

Refactor usage throughout the codebase:
```kotlin
//  Before
painterResource(R.drawable.globe)

//  After
painterResource(Res.drawable.globe)
```

### Step 3: Replace lottie-compose with Compottie

#### Before (androidMain — LoginScreen.kt):
```kotlin
//  Android only lottie
import com.airbnb.lottie.compose.LottieAnimation
import com.airbnb.lottie.compose.LottieCompositionSpec
import com.airbnb.lottie.compose.LottieConstants
import com.airbnb.lottie.compose.rememberLottieComposition

@Composable
fun LoginScreen(...) {
    val comp by rememberLottieComposition(
        spec = LottieCompositionSpec.RawRes(R.raw.login)  //  Android R.raw
    )
    LottieAnimation(
        modifier = Modifier
            .fillMaxWidth(0.7f)
            .aspectRatio(1f),
        composition = comp,
        iterations = LottieConstants.IterateForever,
    )
}
```

#### After (commonMain):
```kotlin
//  KMP Compottie
import io.github.alexzhirkevich.compottie.LottieAnimation
import io.github.alexzhirkevich.compottie.animateLottieCompositionAsState
import io.github.alexzhirkevich.compottie.rememberLottieComposition
import io.github.alexzhirkevich.compottie.LottieCompositionSpec
import org.jetbrains.compose.resources.ExperimentalResourceApi
import listenbrainz.shared.generated.resources.Res

@OptIn(ExperimentalResourceApi::class)
@Composable
fun LoginScreen(...) {
    val composition by rememberLottieComposition {
        LottieCompositionSpec.JsonString(
            Res.readBytes("files/login.json").decodeToString()  //  shared resources
        )
    }
    val progress by animateLottieCompositionAsState(
        composition = composition,
        iterations = Int.MAX_VALUE
    )
    LottieAnimation(
        modifier = Modifier
            .fillMaxWidth(0.7f)
            .aspectRatio(1f),
        composition = composition,
        progress = { progress }
    )
}
```

### Step 4: Replace compose-shimmer with KMP shimmer

#### Before (androidMain — StatsScreen.kt):
```kotlin
//  Android only shimmer
import com.valentinilk.shimmer.Shimmer
import com.valentinilk.shimmer.ShimmerBounds
import com.valentinilk.shimmer.rememberShimmer
import com.valentinilk.shimmer.shimmer

val shimmerInstance = rememberShimmer(shimmerBounds = ShimmerBounds.View)

@Composable
fun ShimmerTopArtistItem(shimmer: Shimmer) {
    Box(
        modifier = Modifier
            .width(140.dp)
            .height(18.dp)
            .shimmer(shimmer)  //  Android only
            .background(Color.Gray.copy(alpha = 0.8f), RoundedCornerShape(2.dp))
    )
}
```

#### After (commonMain):
```kotlin
//  KMP shimmer  same API, different import
import com.valentinilk.shimmer.Shimmer
import com.valentinilk.shimmer.ShimmerBounds
import com.valentinilk.shimmer.rememberShimmer
import com.valentinilk.shimmer.shimmer

// Code is IDENTICAL only dependency changes from
// compose shimmer (Android) → shimmer kmp (KMP)
val shimmerInstance = rememberShimmer(shimmerBounds = ShimmerBounds.View)

@Composable
fun ShimmerTopArtistItem(shimmer: Shimmer) {
    Box(
        modifier = Modifier
            .width(140.dp)
            .height(18.dp)
            .shimmer(shimmer)  //  same API
            .background(Color.Gray.copy(alpha = 0.8f), RoundedCornerShape(2.dp))
    )
}
```

### Step 5: Replace androidx.navigation with Voyager

#### Before (androidMain — AppNavigation.kt):
```kotlin
//  Android-only navigation
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import androidx.navigation.NavHostController
import androidx.navigation.NavType
import androidx.navigation.navArgument

@Composable
fun AppNavigation(navController: NavController = rememberNavController(), ...) {
    NavHost(
        navController = navController as NavHostController,
        startDestination = startRoute
    ) {
        appComposable(
            route = "${AppNavigationItem.Profile.route}?${ARG_USERNAME}={$ARG_USERNAME}",
            arguments = listOf(
                navArgument(ARG_USERNAME) {
                    type = NavType.StringType
                    nullable = true
                    defaultValue = null
                }
            )
        ) {
            val username = it.arguments?.getString(ARG_USERNAME)
            ProfileScreen(username = username, ...)
        }

        appComposable(
            route = "${AppNavigationItem.Artist.route}/{mbid}",
            arguments = listOf(navArgument("mbid") { type = NavType.StringType })
        ) {
            val artistMbid = it.arguments?.getString("mbid")
            ArtistScreen(artistMbid = artistMbid, ...)
        }
    }
}
```

#### After (commonMain  Voyager):
```kotlin
//  KMP Voyager navigation
import cafe.adriel.voyager.navigator.Navigator
import cafe.adriel.voyager.core.screen.Screen
import cafe.adriel.voyager.navigator.LocalNavigator
import cafe.adriel.voyager.navigator.currentOrThrow
import cafe.adriel.voyager.transitions.FadeTransition

// Each screen is a data class — type-safe, no string routes
data class ProfileScreen(val username: String?) : Screen {
    @Composable
    override fun Content() {
        val navigator = LocalNavigator.currentOrThrow
        ProfileScreenContent(
            username = username,
            goToUserProfile = { uname -> navigator.push(ProfileScreen(uname)) },
            goToArtistPage = { mbid -> navigator.push(ArtistScreen(mbid)) },
            goToPlaylist = { mbid -> navigator.push(PlaylistDetailScreen(mbid)) }
        )
    }
}

data class ArtistScreen(val mbid: String) : Screen {
    @Composable
    override fun Content() {
        val navigator = LocalNavigator.currentOrThrow
        ArtistScreenContent(
            artistMbid = mbid,
            goToArtistPage = { navigator.push(ArtistScreen(it)) },
            goToAlbumPage = { navigator.push(AlbumScreen(it)) }
        )
    }
}

// Root App entry point
@Composable
fun App() {
    ListenBrainzTheme {
        Navigator(screen = FeedScreen()) { navigator ->
            FadeTransition(navigator)
        }
    }
}
```

### Step 6: Migrate App Theme to commonMain

#### Before (androidMain Color.kt):
```kotlin
// org.listenbrainz.android.ui.theme 
import androidx.compose.ui.graphics.Color  //  Android Compose

val lb_orange = Color(0xFFEA743B)
val lb_purple = Color(0xFF353070)
val app_bg_day = Color(0xFFFFFFFF)
val app_bg_dark = Color(0xFF292929)
```

#### After (commonMain):
```kotlin
// org.listenbrainz.shared.ui.theme 
import androidx.compose.ui.graphics.Color  //  JetBrains CMP same API

// Code is IDENTICAL  only package changes
val lb_orange = Color(0xFFEA743B)
val lb_purple = Color(0xFF353070)
val app_bg_day = Color(0xFFFFFFFF)
val app_bg_dark = Color(0xFF292929)
```

### Step 7: Platform Entry Points

#### In shared (commonMain):
```kotlin
// Both MainActivity (Android) and
// ComposeUIViewController (iOS) simply call App()
@Composable
fun App() {
    ListenBrainzTheme {
        Navigator(screen = FeedScreen())
    }
}
```

#### Android (androidMain MainActivity.kt):
```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            App()  //  calls shared App()
        }
    }
}
```

#### iOS (iosMain):
```kotlin
fun MainViewController() = ComposeUIViewController {
    App()  //  calls shared App()
}
```
```swift
// Swift  iOSApp.swift
struct ComposeView: UIViewControllerRepresentable {
    func makeUIViewController(context: Context) -> UIViewController {
        MainViewController()
    }
    func updateUIViewController(_ uiViewController: UIViewController, context: Context) {}
}

@main
struct iOSApp: App {
    var body: some Scene {
        WindowGroup {
            ComposeView()
                .ignoresSafeArea(.all, edges: .bottom)
        }
    }
}
```
---

## Phase 8: Platform-Specific Boundaries (Expect/Actual)

While the objective of KMP is to maximize code reuse, certain features require
direct interaction with the host operating system. These cannot be implemented
in pure Kotlin and must use `expect/actual`. This phase covers the four major
platform boundaries in ListenBrainz.

---

### 1. NotificationListenerService  ScrobblerManager

The scrobbling engine is ListenBrainz's core feature. It monitors music
playback across all apps via `NotificationListenerService` on Android.
iOS has no equivalent  it cannot read other app's notifications by design

#### commonMain:
```kotlin
expect class ScrobblerManager {
    val isListening: StateFlow<Boolean>
    fun startListening()
    fun stopListening()
}
```

#### androidMain:
```kotlin
actual class ScrobblerManager(private val context: Context) {

    private val _isListening = MutableStateFlow(false)
    actual val isListening: StateFlow<Boolean> = _isListening

    actual fun startListening() {
        val intent = Intent(context, ListenSubmissionService::class.java)
        if (!context.isServiceRunning(ListenSubmissionService::class.java)) {
            context.startService(intent)
        }
        _isListening.value = true
    }

    actual fun stopListening() {
        context.stopService(Intent(context, ListenSubmissionService::class.java))
        _isListening.value = false
    }
}
```

#### iosMain:
```kotlin
// iOS cannot read other app's notifications permanent stub
actual class ScrobblerManager {
    private val _isListening = MutableStateFlow(false)
    actual val isListening: StateFlow<Boolean> = _isListening

    // iOS does not support NotificationListenerService
    actual fun startListening() { }
    actual fun stopListening() { }
}
```



---

### 2. BrainzPlayer   ExoPlayer vs AVPlayer

`BrainzPlayerService` is deeply coupled to ExoPlayer, `MediaBrowserServiceCompat`,
and `MediaSessionCompat`  all Android only. The audio engine must be abstracted
via `expect/actual`.

#### commonMain:
```kotlin
expect class AudioPlayer {
    val isPlaying: StateFlow<Boolean>
    val currentSongDuration: StateFlow<Long>
    fun play()
    fun pause()
    fun seekTo(index: Int, position: Long)
    fun setMediaItems(songs: List<SongEntity>)
    fun release()
}
```

#### androidMain wraps ExoPlayer:
```kotlin
actual class AudioPlayer(private val exoPlayer: ExoPlayer) {

    private val _isPlaying = MutableStateFlow(false)
    actual val isPlaying: StateFlow<Boolean> = _isPlaying

    private val _currentSongDuration = MutableStateFlow(0L)
    actual val currentSongDuration: StateFlow<Long> = _currentSongDuration

    actual fun play() {
        exoPlayer.playWhenReady = true
        _isPlaying.value = true
    }

    actual fun pause() {
        exoPlayer.playWhenReady = false
        _isPlaying.value = false
    }

    actual fun seekTo(index: Int, position: Long) {
        exoPlayer.seekTo(index, position)
    }

    actual fun setMediaItems(songs: List<SongEntity>) {
        exoPlayer.setMediaItems(songs.map { it.toMediaSource() })
        exoPlayer.prepare()
        _currentSongDuration.value = exoPlayer.duration
    }

    actual fun release() {
        exoPlayer.release()
    }
}
```

#### iosMain  wraps AVPlayer:
```kotlin
actual class AudioPlayer {

    private val _isPlaying = MutableStateFlow(false)
    actual val isPlaying: StateFlow<Boolean> = _isPlaying

    private val _currentSongDuration = MutableStateFlow(0L)
    actual val currentSongDuration: StateFlow<Long> = _currentSongDuration

    private var avPlayer: AVPlayer? = null

    actual fun play() {
        avPlayer?.play()
        _isPlaying.value = true
    }

    actual fun pause() {
        avPlayer?.pause()
        _isPlaying.value = false
    }

    actual fun seekTo(index: Int, position: Long) {
        avPlayer?.seekToTime(CMTimeMakeWithSeconds(position / 1000.0, 600))
    }

    actual fun setMediaItems(songs: List<SongEntity>) {
        val items = songs.map {
            AVPlayerItem(uRL = NSURL.fileURLWithPath(it.uri))
        }
        avPlayer = AVQueuePlayer(items = items)
        _currentSongDuration.value = avPlayer?.currentItem?.duration
            ?.let { (it.value / it.timescale * 1000) } ?: 0L
    }

    actual fun release() {
        avPlayer?.pause()
        avPlayer = null
    }
}
```


---

### 3. Background Sync  WorkManager vs BGTaskScheduler

`ListenSubmissionState` currently uses `WorkManager` to enqueue
`ListenSubmissionWorker` for both `PLAYING_NOW` and `SINGLE` listen submissions.
WorkManager is Android only  iOS uses `BGTaskScheduler`.

#### commonMain:
```kotlin
expect fun scheduleListenSubmission(
    playingTrack: PlayingTrack,
    listenType: ListenType
)
```

#### androidMain uses existing WorkManager setup:
```kotlin
actual fun scheduleListenSubmission(
    playingTrack: PlayingTrack,
    listenType: ListenType
) {
    val workManager = KoinPlatformTools.defaultContext().get<WorkManager>()
    workManager.enqueue(
        ListenSubmissionWorker.buildWorkRequest(playingTrack, listenType)
    )
}
```

#### iosMain uses BGTaskScheduler:
```kotlin
actual fun scheduleListenSubmission(
    playingTrack: PlayingTrack,
    listenType: ListenType
) {
    val request = BGProcessingTaskRequest("org.listenbrainz.listen_sync").apply {
        requiresNetworkConnectivity = true
    }
    BGTaskScheduler.sharedScheduler.submitTaskRequest(request, error = null)
}
```

---

### 4. Permissions Accompanist vs Native iOS

The `ListeningAppSelectionScreen` currently uses Android's permission system
via `PermissionEnum.requestPermission()` and checks like
`PermissionStatus.GRANTED`. These need `expect/actual`.

#### commonMain:
```kotlin
expect class PermissionChecker {
    fun isNotificationListenerGranted(): Boolean
    fun isBatteryOptimizationIgnored(): Boolean
    val criticalPermissionsGranted: StateFlow<Boolean>
}

enum class PermissionStatus {
    GRANTED, NOT_REQUESTED, DENIED, DENIED_TWICE
}
```

#### androidMain:
```kotlin
actual class PermissionChecker(private val context: Context) {

    private val _criticalPermissionsGranted =
        MutableStateFlow(checkCritical())
    actual val criticalPermissionsGranted: StateFlow<Boolean> =
        _criticalPermissionsGranted

    actual fun isNotificationListenerGranted(): Boolean =
        Settings.Secure.getString(
            context.contentResolver,
            "enabled_notification_listeners"
        )?.contains(context.packageName) == true

    actual fun isBatteryOptimizationIgnored(): Boolean {
        val powerManager = context
            .getSystemService(Context.POWER_SERVICE) as PowerManager
        return powerManager
            .isIgnoringBatteryOptimizations(context.packageName)
    }

    private fun checkCritical(): Boolean =
        isNotificationListenerGranted() && isBatteryOptimizationIgnored()
}
```

#### iosMain:
```kotlin
// iOS has no NotificationListenerService or battery optimization 
// these are permanently true on iOS
actual class PermissionChecker {

    private val _criticalPermissionsGranted = MutableStateFlow(true)
    actual val criticalPermissionsGranted: StateFlow<Boolean> =
        _criticalPermissionsGranted

    // iOS cannot listen to other apps' notifications
    actual fun isNotificationListenerGranted(): Boolean = false

    // iOS has no battery optimization concept
    actual fun isBatteryOptimizationIgnored(): Boolean = true
}
```
---

## Timeline

| Week | Task | Priority | Technical Details |
|------|------|----------|-------------------|
| **May 1 – May 24** | Community Bonding | Medium | Study cmp branch deeply, read Epic #614, confirm Voyager as navigation library, discuss Phase priorities with mentor |
| **May 25 – May 31** | Phase 1: Final Build Setup | High | Update `libs.versions.toml` to final target state add `lifecycle-kmp`, `voyager`, `kermit`, `compottie`, `ksoup`, `shimmer-kmp`, `jetbrains-compose` plugins and all KMP-compatible dependency versions |
| **Jun 1 – Jun 7** | Phase 2: Domain Models | High | Move all model classes from `androidMain` → `commonMain`, annotate missing `@Serializable` |
| **Jun 8 –Jun 14** | Phase 3: Network Layer (Part 1) | Critical | Migrate `HttpClient` setup to commonMain via `expect/actual` platform engine, remove `ChuckerInterceptor`, replace `Logger-Android` with Kermit, migrate all Ktorfit service interfaces to shared |
| **Jun 15 – Jun 21** | Phase 3: Network Layer (Part 2) | Critical | Finalize `socket.io-client` → `ktor-client-websockets`, replace `jsoup` with `Ksoup`, split ktor platform engines to androidMain/iosMain, migrate Koin `networkModule` to commonMain |
| **Jun 22 – Jun 28** | Phase 4: Room KMP | High | Move all entities, DAOs and database classes to commonMain, add `expect/actual` database builders for Android + iOS platform specific file paths |
| **Jun 29 – Jul 5** | Phase 5: Repository + Paging | High | Migrate all repositories to commonMain,migrate all paging sources to `paging-common`,  |

---

### Midterm Evaluation  July 6–10

By midterm the following must be complete:
-  All models in `commonMain`
- Network layer fully KMP  socket.io removed, Ktor websockets in place
-  Logger-Android replaced with Kermit
-  Jsoup replaced with Ksoup
-  Room KMP entities, DAOs, databases all in commonMain
-  All repositories in commonMain
-  Paging migrated to `paging-common`
-  Koin `networkModule` + `repositoryModule` in commonMain

---

| Week | Task | Priority | Technical Details |
|------|------|----------|-------------------|
| **Jul 11 – Jul 17** | Tests + Midterm Bug Fixes | High | Apply mentor feedback, move platform independent tests to `commonTest`, keep Android UI tests in `androidTest`,add tests for migrated repositories |
| **Jul 18 – Jul 24** | Phase 6: ViewModel Migration | Critical | Add `lifecycle-kmp` to commonMain, migrate all ViewModels to extend KMP `ViewModel()`, keep thin Android wrappers for Android specific dependencies|
| **Jul 25 – Jul 31** | Phase 8: Platform Boundaries | Critical | Implement `expect/actual` for `ScrobblerManager`, `AudioPlayer` (ExoPlayer/AVPlayer), `scheduleListenSubmission` (WorkManager/BGTaskScheduler), `PermissionChecker` (Accompanist/Native iOS) |
| **Aug 1 – Aug 7** | Phase 7: Compose UI (Part 1) | High | Replace `androidx.compose.*` with JetBrains CMP, migrate app theme + colors to commonMain, migrate all resources from Android `res/` to shared `composeResources/`, replace `lottie-compose` with Compottie |
| **Aug 8 – Aug 14** | Phase 7: Compose UI (Part 2) | High | Replace `androidx.navigation.compose` + `androidx.navigation3` with Voyager, replace `compose-shimmer` with KMP shimmer, migrate all screen composables to commonMain, wire platform entry points (`MainActivity` / `ComposeUIViewController`) |
| **Aug 15 – Aug 24** | Testing, Polish + Final Submission | Critical | Full Android + iOS build testing, remove all unused Android only dependencies from app module, verify all `expect/actual` implementations, submit final work product and evaluation |

---

## Post GSoC Plan

- Complete any Epic #614 checklist items that could not be finished during the summer
- Investigate `spotify-app-remote`  this is the hardest open item in the entire migration since there is no public iOS equivalent. It will need either a creative solution or a clear stub strategy
- Stay involved in MetaBrainz discussions and contribute to future features built on top of the shared multiplatform architecture

MetaBrainz has a strong open source community and I plan to stay involved long term

---

## Why Me?

I have been contributing to ListenBrainz Android since December 2025 across
both the `main` branch and the active `cmp` (Compose Multiplatform) migration
branch. I have read the code, received review feedback from jasje, acted on it,
and gotten PRs merged.

Beyond the code, I am drawn to this project because it is not about adding new
features  it is about making the existing codebase  so that a completely new group of users (iOS users) can access ListenBrainz for the first time. That kind of foundational work is what excites me most.

---

## My Contributions

| PR | Branch | Description |
|----|--------|-------------|
| [#639](https://github.com/metabrainz/listenbrainz-android/pull/639) | main | Fixed search state mismatch between user and BrainzPlayer search |
| [#648](https://github.com/metabrainz/listenbrainz-android/pull/648) | main | Fixed broken overflow menu on BrainzPlayer Albums screen |
| [#727](https://github.com/metabrainz/listenbrainz-android/pull/727) | main | Implemented Delete Listen feature |
| [#732](https://github.com/metabrainz/listenbrainz-android/pull/732) | cmp | Migrating SocialViewModel to shared/commonMain for KMP(currently in review) |

My PRs till date: [PR list](https://github.com/metabrainz/listenbrainz-android/pulls/anuj990)
My commits till date: [Commits](https://github.com/metabrainz/listenbrainz-android/commits?author=anuj990)

---

## About Me

**Tell us about the computer(s) you have available for working on your GSoC project?**

I am using an Asus TUF Gaming A15 with AMD Ryzen 7 7445HS 3.20 GHz, 16GB RAM,
running Fedora KDE OS.

**When did you first start programming?**

I started my programming journey in my first semester with C++ for competitive
programming. Through my college's open source programme BSOC (BitByte Summer
of Code) I discovered open source and Android development.

**What type of music do you listen to?**

I mostly listen to Sufi and Ghazal music. Some of my favorite artists are
Nusrat Fateh Ali Khan and Jagjit Singh.

Some of my MBIDs are:
- [a39a2243-3f86-4c02-82f0-1200fa611d3e](https://musicbrainz.org/artist/a39a2243-3f86-4c02-82f0-1200fa611d3e)
- [a78b6b9a-d7ea-4f59-81df-20ddc3046f9e](https://musicbrainz.org/artist/a78b6b9a-d7ea-4f59-81df-20ddc3046f9e)
- [64d905e4-7d08-4dc7-976a-7bbcfe653f3c](https://musicbrainz.org/artist/64d905e4-7d08-4dc7-976a-7bbcfe653f3c)
- [19f55b19-2ec3-41c3-9a8c-b55e89774ca3](https://musicbrainz.org/artist/19f55b19-2ec3-41c3-9a8c-b55e89774ca3)

**What aspects of the project are you most interested in?**

I am applying for ListenBrainz Android and the most interesting aspect is the
evolving music community and the recommendation system built through sharing,
reviewing, and connecting. The Android app makes this ecosystem more accessible
and extending it to iOS makes it available to an entirely new users.

**Have you contributed to other open source projects?**

Yes, I contributed to open source earlier through my college's open source
programme BSOC (BitByte Summer of Code).

**What sorts of programming projects have you done on your own time?**

**MyAssistant** : A voice recording Android app built with Kotlin + Jetpack Compose
that records audio in the background, splits it into 30-second chunks, transcribes
each chunk via Gemini API, and generates a structured meeting summary with Title,
Summary, Action Items and Key Points. Key technical highlights include a foreground
service for background recording, phone call detection with automatic pause/resume,
audio focus loss handling, low storage checks, silence detection, and retry logic
for failed transcription chunks. Built with MVVM, Hilt, Room, Retrofit, WorkManager,
and Coroutines + Flow. [GitHub](https://github.com/anuj990/MyAssistant)

Beyond this, I have developed several other apps including a QR Code scanner with
CameraX and Google's ML Kit, a Notes App with Room DB and MVVM Architecture, and
MatchMates  :  a hackathon project with Firebase and real-time chat integration.

**How much time do you have available, and how would you plan to use it?**

I can dedicate 30-40 hours per week. By the end of GSoC I will deliver a
fully working Android and iOS app with complete migration of the codebase
to KMP, following the phase by phase plan outlined in this proposal
