# 免安裝 Android Studio！使用 GitHub Actions 雲端「一鍵點擊打包 APK」指南

如果你不想在電腦上下載龐大的 Android Studio（約數 GB），最推薦的方式是使用 **GitHub 提供的免費雲端虛擬機（GitHub Actions）**。你只需點擊網頁上的一個按鈕，GitHub 雲端就會自動調用官方 Android SDK 完成編譯，並在 2 分鐘內直接把打包好的 `.apk` 檔提供給你下載！

---

### 步驟 1：建立 GitHub 儲存庫並上傳專案

1. 前往 [GitHub 官網](https://github.com/) 並登入（若無帳號可免費註冊）。
2. 點擊右上角的 **+** 號，選擇 **New repository**：
   - **Repository name**: 輸入 `KotoRead`
   - 選擇 **Public** 或 **Private**（私有儲存庫每月有 2,000 分鐘免費編譯額度，完全足夠）。
   - 點擊 **Create repository**。
3. 將先前提供的 `KotoRead_Android_Project.zip` 解壓縮後的所有檔案（包含已放入的 `.github/workflows/build_apk.yml`）上傳至該儲存庫根目錄。

---

### 步驟 2：在網頁上一鍵點擊打包 APK

1. 開啟你的 GitHub 專案頁面，點擊上方的 **Actions** 分頁。
2. 在左側選單中點擊 **Build Android APK**。
3. 點擊右側的 **Run workflow** 下拉按鈕，然後點擊綠色的 **Run workflow** 按鈕！
4. GitHub 雲端伺服器將立即啟動：
   - 自動配置 Java 17 與 Android SDK 34
   - 自動下載 Jetpack Compose、Room 與 ML Kit 等依賴
   - 執行 Gradle 編譯並進行 APK 數位簽章

---

### 步驟 3：直接下載 APK 安裝到手機

1. 約等候 2～3 分鐘後，該任務會顯示綠色的勾勾 `✓`（代表打包成功）。
2. 點進該次執行紀錄，頁面最下方的 **Artifacts** 區塊會出現：
   📦 **`KotoRead-NovelReader-APK`**
3. 點擊它即可直接下載 `app-debug.apk`！
4. 將下載的 APK 傳至 Android 手機，點擊即可直接安裝使用！
