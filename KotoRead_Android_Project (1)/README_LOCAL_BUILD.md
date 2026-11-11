# 言ノ葉 Reader (KotoRead) — Android 專案建置與 APK 打包指南

本專案採用 **Kotlin + Jetpack Compose + Room + WorkManager + Google ML Kit** 開發。由於 Android APK 是經過 Google Android SDK / D8 編譯器將 JVM 位元組碼打包為 Dalvik Executable (`.dex`)、壓縮資源並使用 Android 金鑰簽章（APK Signature v2/v3）的二進位安裝檔，因此無法直接透過單一文本腳本轉出，需要透過 **Android Studio** 或命令列的 **Gradle Wrapper** 來編譯產生。

以下為最快速將程式碼轉換為手機可直接安裝之 `.apk` 的完整步驟：

### 第一步：在 Android Studio 中建立專案結構

1. 開啟 **Android Studio**（建議使用 Hedgehog 2023.1.1 或更新版本），點擊 **New Project**。

2. 範本選擇 **Empty Activity (Compose)**，設定如下：

   * **Name**: `KotoRead`

   * **Package name**: `com.example.kotoread`

   * **Language**: `Kotlin`

   * **Minimum SDK**: `API 26: Android 8.0 (Oreo)` 或以上

   * **Build configuration language**: `Kotlin DSL (build.gradle.kts)`

3. 建立後，將本對話中的以下兩個檔案放入對應目錄：

   * `KotoReadNovelApp.kt` 放至：`app/src/main/java/com/example/kotoread/`

   * `NovelExportManager.kt` 放至：`app/src/main/java/com/example/kotoread/`

### 第二步：設定 `app/build.gradle.kts`

在 `app/build.gradle.kts` 頂部啟用 KSP 外掛，並加入專案所需的所有依賴庫：

```
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
    id("com.google.devtools.ksp") version "2.0.20-1.0.24"
}

android {
    namespace = "com.example.kotoread"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.example.kotoread"
        minSdk = 26
        targetSdk = 34
        versionCode = 1
        versionName = "1.0.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    kotlinOptions {
        jvmTarget = "17"
    }
    buildFeatures {
        compose = true
    }
}

dependencies {
    implementation("androidx.core:core-ktx:1.13.1")
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.8.5")
    implementation("androidx.activity:activity-compose:1.9.2")
    implementation(platform("androidx.compose:compose-bom:2024.09.00"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-graphics")
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.compose.material3:material3")
    implementation("androidx.compose.material:material-icons-extended")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.5")

    // 1. Android Room 資料庫（書架與離線快取）
    val roomVersion = "2.6.1"
    implementation("androidx.room:room-runtime:$roomVersion")
    implementation("androidx.room:room-ktx:$roomVersion")
    ksp("androidx.room:room-compiler:$roomVersion")

    // 2. Android WorkManager 背景定期連載巡檢
    implementation("androidx.work:work-runtime-ktx:2.9.1")

    // 3. Google ML Kit 端側離線日翻中模型（免連網直譯）
    implementation("com.google.mlkit:translate:17.0.3")

    // 4. Jsoup 網頁爬蟲與 OkHttp 網路連線
    implementation("com.squareup.okhttp3:okhttp:4.12.0")
    implementation("org.jsoup:jsoup:1.18.1")
}

```

### 第三步：設定 `AndroidManifest.xml` 與 FileProvider

在 `app/src/main/AndroidManifest.xml` 中宣告網路權限、通知權限、以及提供 EPUB/TXT 電子書安全分享的 FileProvider：

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- 網際網路存取權限 -->
    <uses-permission android:name="android.permission.INTERNET" />
    <!-- Android 13 (API 33)+ 連載更新推播通知權限 -->
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="言ノ葉 Reader"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@android:style/Theme.Material.NoActionBar">

        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>

            <!-- 支援從 Chrome 瀏覽器直接按「分享至本 App」開啟小說網址 -->
            <intent-filter>
                <action android:name="android.intent.action.SEND" />
                <category android:name="android.intent.category.DEFAULT" />
                <data android:mimeType="text/plain" />
            </intent-filter>
        </activity>

        <!-- 電子書匯出安全分享 Provider -->
        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="${applicationId}.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>

    </application>

</manifest>

```

在 `app/src/main/res/xml/` 資料夾下新建 `file_paths.xml`：

```
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <!-- 指向 cacheDir/exported_novels/ 供外部閱讀器安全讀取產出的 EPUB/TXT 檔案 -->
    <cache-path name="exported_novels" path="exported_novels/" />
</paths>

```

### 第四步：一鍵打包產出 APK

在 Android Studio 中，你可以透過以下兩種方式之一立即產出 APK：

#### 方式 A：介面一鍵產生（最簡單）

1. 點擊頂部功能表：**Build** → **Build Bundle(s) / APK(s)** → **Build APK(s)**。

2. 編譯完成後，右下角會跳出通知提示 `APK(s) generated successfully.`。

3. 點擊旁邊的 **locate** 連結，即可開啟檔案夾，裡面即是產出的測試安裝檔：
   `app-debug.apk`

#### 方式 B：終端機命令列打包

在專案根目錄下開啟 Terminal，執行：

```
./gradlew assembleDebug

```

產出的 APK 檔案路徑位於：
`app/build/outputs/apk/debug/app-debug.apk`

### 第五步：安裝至 Android 手機

1. **USB 傳輸或通訊軟體傳送**：將 `app-debug.apk` 透過 USB 線、Google 雲端硬碟、LINE 或 Telegram 傳送到你的 Android 手機。

2. **開啟未知來源安裝**：在手機檔案管理員點擊該 `.apk` 檔，若系統提示安全警告，選擇「允許來自此來源的應用程式安裝」。

3. **完成安裝與體驗**：安裝後即可開啟「言ノ葉 Reader」，在瀏覽器逛小說網時按「分享」直接開啟章節，並使用一鍵日翻中與 EPUB 電子書匯出功能！