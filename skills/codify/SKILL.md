---
name: codify
description: 為專案建立可長期沿用的 AI 協作制度（立法）。當使用者想初始化專案的 agent 守則、重整 CLAUDE.md、建立模型調度與派工制度、外化判斷力為 rubric/checklist、或檢查既有制度是否退化時使用。支援三種制度層級（lite 輕量／standard 標準／full 完整含 hooks）與兩種動作（init 立法、audit 體檢）。參數範例：`lite`、`standard`、`audit`、`upgrade full`。
---

# Codify — 專案制度立法

把一次高階模型 session 的判斷力，轉化為專案內可長期沿用的制度檔案，
讓後續較弱的模型也能照制度運作、逼近高階模型的表現。

制度的重量必須與專案的風險匹配：小工具背不動完整派工制度，正式產品也不該只有一頁備忘。
因此本 skill 是二維的：**層級（制度多重）× 動作（做什麼）**。

## 核心原則（所有層級統一，全程遵守）

1. **自主作業**：先在環境內自行查找（程式碼、git log、既有 CLAUDE.md、CI 設定），開場提問最多五個，之後不再打斷使用者。
2. **價值優先**：交付項按價值排序執行，每完成一項立即寫入檔案，不要累積到最後才寫。
3. **備份先行**：修改任何既有檔案前先備份為 `<原名>.bak-<日期>`；若專案在 git 內，改在新分支上進行並說明。
4. **弱模型友善**：寫下的每條規則必須具體、可執行、有判準與範例。禁止「保持程式碼品質」這類無法驗收的句子。
5. **能力下沉驗收**：所有產出以「Haiku 等級的模型能否照做」為標準。若一條規則需要品味才能執行，改寫成 checklist 或明定升級路徑。
6. **標註 harness 極限**：拆解與多樣本評審可補執行品質，但模糊需求與品味判斷無法補。遇到這類邊界必須明寫處理方式：升級模型、徵詢使用者、或坦言做不到。
7. **語言規範**：制度產出的語言跟隨專案——專案已有 CLAUDE.md 時沿用其語言；沒有時用與使用者對話的語言。中文產出需依照使用者的慣用變體，不得混用：繁體中文以繁體慣用字詞為準（如：程式碼、品質、資料、回饋、效能），簡體中文以簡體慣用字詞為準（如：代码、质量、数据、反馈、性能）。無論何種語言，專業術語（hook、rubric、commit、CI、subagent 等）一律保留英文原文，不硬譯；寫入檔案前自我校對一次，不得有缺字漏字。

## 第一步：決定層級與動作

### 解析參數

- 參數含 `lite` / `standard` / `full` → 指定層級
- 參數含 `init` / `audit` → 指定動作
- 參數含 `upgrade <層級>` → 升級動作（見下）
- 只有層級、沒有動作（如 `/codify standard`）：專案無層級標記 → 跑該層級的 init；已有標記且層級相同 → 跑 audit；已有標記但層級不同 → 向使用者確認是否要升降級，確認後走 upgrade 流程。
- 無參數：專案已有層級標記 → 跑該層級的 audit；否則跑 init，層級照下表推薦後**向使用者確認**（這算五個開場問題之一）。

### 層級推薦判準

| 訊號 | 推薦 |
|---|---|
| 個人腳本、單人小工具、prototype、無 CI | **lite** |
| 多人協作或長期維護、有測試與 CI、會頻繁派工給弱模型 | **standard** |
| 影響生產環境／金流／憑證、違規成本高、或使用者明確要求機制性強制 | **full** |

推薦時附一句理由。使用者永遠可以否決。

### 層級內容矩陣

| 交付項 | lite | standard | full |
|---|:-:|:-:|:-:|
| A 快速診斷 | 簡化（只找前 1 項，不出獨立報告） | ✓ | ✓ |
| B CLAUDE.md 重寫 | ✓（一切內嵌於此，見下） | ✓ | ✓ |
| C 模型調度守則 | — | ✓ | ✓ |
| D 判斷力 rubrics | 精選 ≤6 條內嵌 CLAUDE.md | ✓ | ✓ |
| E 派工 prompt 模板 | — | ✓ | ✓ |
| F 維護協議 | 3 行內嵌 CLAUDE.md | ✓ | ✓ |
| G 給未來 session 的信 | — | 選配（有值得寫的才寫） | ✓ |
| H 機制性閘門（hooks） | — | — | ✓ |

**lite 的形態**：只產出一個 CLAUDE.md（≤80 行），不建 `.claude/rules/` 目錄。內含：常用指令、硬規則、同步耦合、精選 rubric ≤6 條（必含「何時升級」與「何時停下」各取最關鍵者）、3 行維護說明（膨脹上限、踩坑就近記錄、何時該升級層級）。token 足跡 = 每個 session 只多付這一個檔案。

**standard 的形態**：CLAUDE.md 精簡本體 + `.claude/rules/` 細則，全部是條文，不裝任何 hook。

**full 的形態**：standard 的一切 + H（把最關鍵的 1–3 條硬規則落成 hook 強制）。條文會漂移，hook 不會；但 hook 有安裝與維護成本，所以只留給違規成本高的專案。

### 層級標記（必做）

init 完成時，在 CLAUDE.md 檔尾寫入一行：

```
<!-- codify: tier=<lite|standard|full> date=<YYYY-MM-DD> -->
```

audit 與 upgrade 靠這行判斷現況。**audit 不得把低層級「本來就沒有」的檔案判為退化。**

---

## init（立法）

init 是把判斷力寫成制度的一次性投資。本 skill 已內建高階模型（Fable 5）留存的判斷快照與成品範例，因此**任何等級的模型都能執行 init**：照快照的判準做、照範例的樣子寫。用更強的模型與更高的 reasoning effort 執行仍會更好，但不是前提。

**步驟 0（必做，所有層級）**：動手前先讀 `references/judgment.md`（怎麼判斷）與 `references/exemplar.md`（合格成品長什麼樣）。遇到 judgment.md 未涵蓋的模糊情境，問使用者或明寫「無法確定」，不要用自己的直覺補——這正是快照存在的原因。

之後依層級矩陣執行對應交付項，每階段開始時才讀對應 reference（不要一次全讀）：

| 交付 | reference | 落點（standard/full） |
|---|---|---|
| A 診斷 | `references/diagnose.md` | `.claude/rules/diagnosis.md` |
| B CLAUDE.md | `references/claude-md-style.md` | 專案根目錄 `CLAUDE.md` |
| C 調度守則 | `references/dispatch-protocol.md` | `.claude/rules/dispatch.md` |
| D rubrics | `references/rubrics.md` | `.claude/rules/rubrics.md` |
| E 派工模板 | `references/prompt-templates.md` | `.claude/rules/prompt-templates.md` |
| F 維護協議 | `references/maintenance.md` | `.claude/rules/maintenance.md` |
| G 信 | `references/letter-template.md` | `.claude/rules/letter.md` |
| H 閘門 | `references/gates.md` | `.claude/hooks/` + `.claude/settings.json` |

lite 只讀 B、D 兩份 reference，並依「lite 的形態」內嵌產出。

### 收尾四步驟（所有層級不可省略）

1. **對抗審查**：以「我要證明這些規則弱模型做不到」的立場重讀全部產出，並對照 `references/judgment.md` 第五節（失敗模式）與第六節（攻擊角度）逐條自檢，修掉需要品味才能執行的條文。full 另加：以「我要繞過這些 hook」的立場檢查 H 的漏洞並記錄已知繞法。
2. **read-back 驗證**：重新開檔確認每個落點存在、索引連結正確；full 另需實際觸發一次每個 hook 證明會擋（見 gates.md 的驗證段）。
3. **一頁總結**：改了什麼、為什麼、之後怎麼用（含 audit 的觸發時機與升級層級的方法）。
4. **交接**：context 將盡而有未完項目時，寫入 letter.md 交接段（lite 寫入 CLAUDE.md 檔尾註解）。

---

## audit（體檢，增量修法）

1. 讀 CLAUDE.md 檔尾的層級標記，確定體檢範圍。
2. standard/full：照專案 `.claude/rules/maintenance.md` 的體檢 checklist 執行。lite：只檢查 CLAUDE.md——行數是否超標、指令是否仍有效、rubric 是否被遵守、踩坑記錄是否該裁決。
3. full 另加：驗證每個 hook 仍能觸發；檢查是否有 session 反覆撞牆的記錄（撞牆多＝規則或 hook 設計有問題，提案修正）。
4. 只做增量修正。大改需使用者同意，改跑 init。
5. 若專案複雜度已明顯超過現行層級（lite 專案出現多人協作、頻繁派工），在差異摘要中建議升級，不擅自升級。
6. 產出差異摘要：每項修改一行，附原因。

---

## upgrade（升級層級）

`upgrade standard` / `upgrade full`。原則：**增量補齊，不重新立法**。

1. 讀層級標記確認現況；降級（full→standard 等）需使用者明確確認，動作是移除 hook 但保留條文。
2. lite→standard：把 CLAUDE.md 內嵌的 rubric 與維護段外移到 `.claude/rules/` 完整版，再補 C、E、F、G。既有規則全數保留，只擴充不改寫。
3. standard→full：只補 H（讀 `references/gates.md`），從既有硬規則中挑違規成本最高的 1–3 條落成 hook。
4. 更新層級標記，跑收尾四步驟的 2 與 3。
