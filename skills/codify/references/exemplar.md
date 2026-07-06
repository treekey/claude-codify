# 立法成品範例（exemplar）

一份完整 init 成品的濃縮示範，由 Fable 5 撰寫留存。目的：讓執行 init 的模型**用模仿代替創造**——
先看合格成品長什麼樣，再回去填自己專案的值。
虛構專案：`orderhub`，Python FastAPI 訂單服務，兩人維護、有 CI、常派工給 Sonnet → 層級 standard。
**警告：這裡的一切都是範例。你的產出不得出現 orderhub 字樣（judgment.md 失敗模式 1）。**

## 開場提問（實際只問了 3 個——問滿 5 個不是目標）

1. 「`legacy/` 目錄還有人維護嗎？」→ 沒有，等 Q4 移除 → 立為禁改區
2. 「出貨壓力大時，測試和時程哪個讓步？」→ 時程優先但要留記錄 → 寫進信
3. 「上次讓你最火大的 AI 失誤是什麼？」→ 擅自升級了 pydantic 版本 → 立為硬規則

注意問的都是「只有使用者知道的事」；專案用什麼框架、測試怎麼跑，模型自己查，不浪費提問額度。

## 診斷報告節選（前三項之一）

```markdown
## 問題一：測試指令在 README 與 CI 中不一致
- 證據：README.md:12 寫 `pytest`；.github/workflows/ci.yml:31 實際用
  `pytest -m "not slow" --cov`。session 照 README 跑會漏掉 coverage 門檻，本地綠 CI 紅。
- 影響：git log 近 30 筆中有 4 筆「fix ci」型 commit。
- 修法：CLAUDE.md 常用指令段以 CI 的指令為準（已寫入）。
```

好診斷的樣子：證據帶行號、影響有數字、修法本次已完成。

## 產出的 CLAUDE.md（完整，58 行）

```markdown
# orderhub

FastAPI 訂單服務，對內供 ERP 呼叫。兩人維護，正式環境在 AWS ECS。

## 常用指令
- 測試（與 CI 一致）：`pytest -m "not slow" --cov`（根目錄執行）
- 慢測試（改動 `services/settlement/` 時必跑）：`pytest -m slow`
- Lint：`ruff check . && ruff format --check .`
- 本地起服務：`uvicorn app.main:app --reload`（需先 `docker compose up -d db`）

## 硬規則
1. 不得修改 `legacy/` 下任何檔案（Q4 廢棄；需要它的行為時複製到新模組再改）。
2. 不得變更 `pyproject.toml` 中任何既有依賴的版本；新增依賴前先問使用者。
3. 不得修改 `alembic/versions/` 中已存在的 migration 檔（只能新增）。
4. `app/schemas/generated/` 為產生碼，改上游 `openapi.yaml` 後執行 `make gen`。

## 同步耦合
- 改 `app/models/*.py` → 必同時新增 alembic migration（`alembic revision --autogenerate`）。
- 改 `openapi.yaml` → 必跑 `make gen` 並將產生碼一起 commit。
- 新增環境變數 → 必同步 `app/config.py`、`.env.example`、`deploy/task-def.json` 三處。

## 細則索引
- 模型調度與派工：.claude/rules/dispatch.md
- 品質與完成判準：.claude/rules/rubrics.md
- 派工模板：.claude/rules/prompt-templates.md
- 制度維護：.claude/rules/maintenance.md

<!-- codify: tier=standard date=2026-07-06 -->
```

值得模仿的點：每條硬規則都有出處（規則 2 來自使用者的踩坑回答）；同步耦合寫「動作 → 動作」
而非「注意保持一致」；指令附執行目錄與前置條件。

## 各細則檔的專案特定段落（節選）

模板裡通用的部分照 references 產出即可，價值在填進去的**專案值**：

**dispatch.md 模型分級表的「例」欄**：
> 低（Haiku 級）：補 `.env.example` 缺漏、按模板產生新 endpoint 的 schema 三件組
> 中（Sonnet 級）：新增 endpoint 全套（route + service + 測試）、settlement 以外的 bug 修復
> 高：settlement 金額計算相關的任何改動、資料模型變更的設計

**rubrics.md「何時算完成」的專案化條目**：
> - [ ] `pytest -m "not slow" --cov` 全綠且 coverage 未低於 main 分支（CI 會擋，先在本地確認）
> - [ ] 涉及 `services/settlement/`：另跑 `pytest -m slow` 並附輸出

**maintenance.md 體檢節奏**：
> 每次 sprint 結束（隔週五）或換用新模型時執行 audit。

**letter.md「三件事」之一**：
> ### 出貨壓力下的取捨
> 情境：deadline 前使用者說「先能動就好」。
> 該怎麼做：可以略過補測試，但必須在 PR 描述留「未測項目」清單——這是使用者明確要的記錄方式，
> 不留清單的「先能動」事後會被當成正常品質標準。

## 這份成品符合的自檢（對照 judgment.md 第五節）

- 無佔位符殘留、無範例專案名（你的產出要搜的是「orderhub」以外你抄進去的任何虛構名）
- 每條硬規則有出處：規則 1、2 來自提問，3、4 來自目錄結構與 git log
- 沒有「寫好測試」「保持整潔」這類通用充數條目
- 規則總量克制：硬規則 4 條、耦合 3 條——不是 20 條
