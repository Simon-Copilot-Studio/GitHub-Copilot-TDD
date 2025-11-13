一頁心智圖（高階結構）
根：AI 編程專案 Workflow（Context-first）
├─ 啟動：商業目標、KPI、利害關係人
├─ 需求拆解（模組化）
│   ├─ 模組A（登入）
│   ├─ 模組B（付款）
│   └─ ...
├─ 上下文管理（Context Engineering）
│   ├─ 模組級計劃（Markdown 存檔）
│   ├─ 外部知識快取（官方文件、API 範例）
│   └─ 全域規則/記憶（Project Memory）
├─ 設計與實驗（TDD + 實驗追蹤）
├─ 開發（雙 Agent 工作流 + Code Review Agent）
├─ 測試（單元 / 合約 / e2e / 偏差）
├─ CI/CD（模型與應用變更）
├─ 部署（金絲雀 / 藍綠）
├─ 監控（模型效能、資料漂移、logs）
├─ 除錯（Bug Report → Debug Logs → 回滾/總結）
└─ 回顧與文件化（post-mortem、教訓）

核心原則
上下文優先：每個模組、任務都要有高品質計劃與對應 Markdown 文件供 AI/開發讀取。
提供最新外部知識：把官方文件、API 範例或檢索工具（Conscious Search 類）整合進開發流程與 agent。
全域規則/記憶：設定 Project Memory 規則（例如：遇到 API 相關問題，必先檢索官方文件）。
預防為主：採用 TDD 與獨立 Code Review Agent。
除錯流程化：清楚 Bug 報告模板、加 Log、讓 AI 閱讀 Log、必要時回退並另開新對話改版。
任務拆解（可直接當 Sprint / Issue 模板）
每個模組（例如：User Login）遵循以下子任務序列：
模組計劃（Module Plan） — deliverable: module-name/plan.md
內容：功能需求、API 介面、技術棧、相依性、接受標準（Acceptance Criteria）、安全/隱私注意事項、測試案例雛形、交付時間。
規則：以 Markdown 儲存，版本化（git）並指派到該模組的 Agent/開發者。
Gate：PM 與技術負責人簽核。
外部知識收集 — deliverable: module-name/external-resources.md
內容：官方 API 文件連結（或全文貼入）、範例程式片段、已知限制、版本號。
規則：每次開發前由 Agent 執行「官方文件檢索」流程並附上檢索紀錄（timestamp + source）。
設計與 TDD (Test-First) — deliverable: 測試套件 + 設計文件
先編寫單元 + 合約測試（包含資料合約與邊界案例），測試檔置於 tests/。
寫出 mock 資料與輸入樣本（point-in-time snapshot）。
Gate：所有測試都跑通（除非是故意標註為 flaky 且有 ticket）。

實作（由 Agent 或開發者）
依 plan.md 與 external-resources.md 開發。
實作中每個步驟寫入 commit 並 reference issue/plan。
若使用 AI 協助，確保 AI 每次變更都有相對應的「變更說明」與「預期測試」。
Code Review（採用 Code Review Sub-Agent）
Agent A 寫碼 → Agent B（Code Review Agent）僅做閱讀/審核（不允許改寫工具），檢核點列明（安全、邏輯、測試覆蓋、可讀性）。

若 Code Review 回饋，Agent A 修正並重新提交 review。
整合測試 & 性能測試
包括資料流測試、延遲/吞吐測試、偏差檢查（若有 ML 推論）。
打包/發佈 (CI/CD)
Build → Test → Package → Canary/Blue-Green → Promote/rollback。
監控 & 維運
建立指標板（accuracy, latency p95, input distribution, drift metrics）。
設重訓觸發條件（threshold 或定期）。
Post-mortem & 文檔更新
每次發佈或事故後完成回顧，更新 module-name/retrospective.md。
模組級 Markdown 規範（強調上下文）

每個模組資料夾結構（範例：user-login/）：

user-login/
├─ plan.md                # 模組計劃（功能、Acceptance Criteria）
├─ external-resources.md  # 官方文件、版本、重要程式碼片段
├─ tests/
│   ├─ unit/
│   └─ integration/
├─ src/
├─ logs/                  # Debug logs（開發環境）
└─ retrospective.md


plan.md 範例要含：

需求描述

input/output schema（資料合約）

edge cases

acceptance tests（具體步驟）

安全/權限需求

使用範例與 mock data

全域規則 / Project Memory 範例（settings）

規則 1：每次涉及外部 API 或第三方套件時，啟動「Conscious Search」檢索最新官方文件，並把檢索結果寫入 module/external-resources.md（包含時間戳與版本號）。

規則 2：所有程式變更必附對應測試，未通過測試不得合併。

規則 3：Code Review Agent 僅能讀取 repo 與文檔，不可執行修改或使用外部工具。

規則 4：每次 CI failing 時，CI 系統自動建立 debugging issue，並收集失敗 logs 到 ci-logs/。

除錯（Debug）標準化流程與範本
Bug 報告模板（Issue 範本）
標題： [模組] 簡短描述（例：User Login - 無法登入）
描述：
1. 重現步驟（環境、頁面、按鈕、資料）
2. 實際行為（Console Error / API 回應 / UI 表現）
3. 預期行為
4. 環境資訊（Browser/版本、OS、後端版本、資料 snapshot id）
5. 相關 log 檔位置（ci-logs/ 或 logs/)
6. 初步猜測的 root cause（若有）

Debug 流程（每個 Bug）

複現：工程師/Agent 複現 Bug，把重現流程寫入 Issue。

收集 Logs：在關鍵位置加入詳盡 Debug log（transaction id、inputs、stack trace、外部 API 回應）。

建議：Log format JSON 並包含 trace_id 和 module 欄位。

AI 協助讀 Logs：把相關 logs 與 plan.md、external-resources.md 提供給 AI（或 Agent）要求：「根據以下 log 與模組計劃，請分析可能的失敗點及修復建議」。

撰寫修正並執行 TDD：先加/修測試，再改碼。

若 AI 反覆使碼品質下降：用影片建議的做法：修好後讓 AI 撰寫「問題根源總結」與「修改步驟」，再回退至最初良好 commit，然後另開新對話（或新 branch）一次性實作正確修正。

上線前回歸測試：確認所有 acceptance tests 與監控指標正常。

回顧 & 文件化：寫入 retrospective.md。

.NET / ASP.NET Core 實作 & CI/CD 範例（精要）
Dockerfile（範例）
# 使用 .NET 7 SDK 進行 build
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY . .
RUN dotnet restore
RUN dotnet publish -c Release -o /app/publish

# 執行階段
FROM mcr.microsoft.com/dotnet/aspnet:7.0
WORKDIR /app
COPY --from=build /app/publish .
ENV ASPNETCORE_ENVIRONMENT=Production
ENTRYPOINT ["dotnet", "YourApp.dll"]

GitHub Actions（簡化）— 重點：TDD → Build → Test → Package → Canary
name: CI

on:
  push:
    branches: [ main, 'releases/*' ]
  pull_request:

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '7.0.x'
      - name: Restore
        run: dotnet restore
      - name: Run Unit Tests
        run: dotnet test --no-build --verbosity normal
      - name: Build
        run: dotnet publish -c Release -o out

  package-and-deploy:
    needs: build-and-test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker image
        run: |
          docker build -t ghcr.io/${{ github.repository }}/app:${{ github.sha }} .
          docker push ghcr.io/${{ github.repository }}/app:${{ github.sha }}
      - name: Deploy Canary
        run: |
          # 呼叫你們的部署腳本 / terraform / kubectl 將新版本打到 canary
          ./deploy/deploy_canary.sh ghcr.io/${{ github.repository }}/app:${{ github.sha }}


建議把「模型 artifact」(ONNX / saved model) 上傳到 artifact registry，並把 artifact id 寫入 release metadata。

Code Review Sub-Agent 設定建議（流程化）

限制能力：只讀 repo + 只讀文檔，不給修改權、也不給網路外部工具（避免 noise）。

審核檢查清單：

是否符合 plan.md 與 external-resources.md？

是否有對應測試（含邊界）？覆蓋率是否足夠？

是否有安全/敏感資料處理檢查？

代碼是否含過多複雜度／可讀性問題？

輸出格式：產生具體 review comments（file + line + 建議），並把 summary 寫入 Issue comment。

若 Review Agent 找到設計違反：建立 blocker issue 並阻擋 merge。

監控指標（最少要有）

功能面：Acceptance test pass rate、SLO 滿足率

ML 面：validation vs production performance delta、drift score（KL divergence 等）

服務面：p95 latency、error rate、throughput

運營面：部署次數、回滾次數、MTTR（平均修復時間）

Release Gate Checklist（可放入 CI）

plan.md 與 external-resources.md 已存在且為最新。

Unit + Integration tests 全數通過（包括資料合約測試）。

Code Review Agent 無 blocker comments。

Canary 在 24–72 小時監控內無顯著退化。

監控告警、回滾 playbook、owner on-call 名單就緒。

模型 artifact 與資料 snapshot 已記錄（含 ID/URL）。

範例任務拆解（4 週落地計畫，能直接套用）

Week 0（準備）

建立 repo 與模組目錄結構

定義 Project Memory 規則（檔案 + agent 策略）

撰寫 module/*/plan.md 範例

Week 1（基礎建置）

實作模組骨架（API contract）

撰寫單元測試與資料合約測試

設定 GitHub Actions 基本 CI（test）

Week 2（AI/模型整合）

上傳模型 artifact（ONNX）並示範推論流程

實作 Canary 部署腳本

設定基本監控（Prometheus + Grafana）

Week 3（強化 QA & Debug）

建立 Code Review Agent 流程並完成第一次 review

模擬 Bug，跑除錯流程並寫 retrospective.md

導入偏差檢測與重訓策略

Week 4（上線與回顧）

Production 推廣（藍綠或金絲雀）

30 天監控排程與回顧會議

更新文件與 Training material

常見落地挑戰與快速解法

AI 提供錯誤外部資訊（幻覺） → 強制 Agent 先檢索外部文件並附上來源；CI gate 檢查 external-resources.md 是否有 timestamp 與 source。

AI 修復反覆導致品質下降 → 回退到最後穩定 commit，另開 branch，並採用 Code Review Agent 做「僅讀」第二層審查。

資料漂移未被偵測 → 導入每日自動資料分布檢測，超過閾值則自動建立 ticket 並觸發重訓流程。
