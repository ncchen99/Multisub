# MultiSub 跨裝置開發與簽署指南 (Developer Signing & Collaboration Guide)

為了方便多位開發者在不同 Mac 裝置上共同協作、發送 PR，本專案採用了**「變數化 Bundle ID」**（Option B）的設計。

本設計解決了 iOS/macOS 開發中常見的**個人簽署設定、Team ID 與自訂 Bundle ID 導致專案設定檔（`project.pbxproj`、`.entitlements`）產生 Git 衝突**的問題。現在，所有與個人帳號相關的簽署設定都存放在本地的設定檔中，不會被提交。

---

## 1. 架構設計與運作原理

1. **`Local.xcconfig`（已被寫入 `.gitignore`，不提交）**：
   * 用於存放本機開發者的私密資訊：**`DEVELOPMENT_TEAM`**（開發團隊 ID）以及 **`MULTISUB_BUNDLE_PREFIX`**（套件識別碼前綴）。
   * 範本檔為 `Local.xcconfig.example`。
2. **專案與權限設定檔（`project.pbxproj`、`.entitlements`）**：
   * 專案內原本寫死的 Bundle ID（如 `com.seanhuang.multisub`）已改為變數：**`$(MULTISUB_BUNDLE_PREFIX).multisub`**。
   * App Group 與 Keychain Access Group 亦改為相對應的動態變數。建置時 Xcode 會根據您在 `Local.xcconfig` 設定的前綴自動展開。
   * 移除了需要付費帳號才能在官網註冊的靜態 App Group，改為免註冊的 Team-ID 格式，支援**免費的個人 Apple ID 帳號**直接編譯。
3. **動態 Swift 程式碼**：
   * 程式碼中所有對應的 Keychain 服務名稱、App Group 目錄路徑，皆改為在執行期動態讀取 `Bundle.main.bundleIdentifier` 自動推導，不硬編碼任何字串。

---

## 2. 協作者（包含原作者）如何設定本機環境？

當您 Clone 此專案或拉取更新後，只需在您本機進行一次性的簡單設定：

### 步驟 1：複製並建立 `Local.xcconfig`
在專案根目錄下，複製範本檔建立您專屬的設定檔：
```bash
cd MultiSub
cp Local.xcconfig.example Local.xcconfig
```

### 步驟 2：填入您的 Apple 帳號資訊
開啟您本機新建的 `Local.xcconfig`，將其內容修改為對應您的帳號：

* **如果是原作者 (Sean Huang)**：
  ```text
  DEVELOPMENT_TEAM = <您的10碼AppleTeamID>
  MULTISUB_BUNDLE_PREFIX = com.seanhuang
  ```
* **如果是協作者 (以 Nian-Cheng Chen 為例)**：
  ```text
  DEVELOPMENT_TEAM = 47TQLDUVUD
  MULTISUB_BUNDLE_PREFIX = com.ncchen
  ```

*(註：您可以在 Xcode ➜ Settings ➜ Accounts 選取您的 Apple ID，在右下角點擊團隊名稱，即可在括號中找到您的 10 碼 Team ID；或使用命令列執行 `security find-identity -p codesigning -v` 查詢。)*

### 步驟 3：在 Xcode 圖形介面載入帳號（首次建置）
由於新 Bundle ID 需要在您的帳號下註冊設定檔：
1. 雙擊開啟 `MultiSub.xcodeproj`。
2. 在左側點選藍色 **MultiSub** 專案，在中間 **TARGETS** 列表中，分別選取 **MultiSub** 和 **MultiSub Extension**。
3. 在 **Signing & Capabilities** 中，將 **Team** 選取為您自己的帳號，等候 Xcode 的紅字錯誤消失。
4. 設定完成後即可在終端機直接執行 `./deploy.sh` 安裝！

---

## 3. 開發優勢

* **零 Git 衝突**：所有簽署、Team ID 的個人化設定全部留在 `Local.xcconfig`，提交 PR 時，專案檔與 entitlements 的變數設定保持不變，不用擔心覆蓋他人的簽署設定。
* **安全無虞**：敏感的開發團隊識別碼與私密金鑰資訊永遠不會不小心被 Git 追蹤或公開上傳至 GitHub。
* **低門檻**：任何想參與此開源專案的開發者，只需準備一個免費的 Apple ID 即可在自己的 Mac 上一鍵編譯與測試，不需向原作者申請任何特許權限。
