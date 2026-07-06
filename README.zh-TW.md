# claude-codify

**用一次高階模型的 session 為專案立憲，讓之後每個較便宜的 session 都能照制度自治。**

`codify` 是一個 [Claude Code](https://claude.com/claude-code) skill，把強模型的判斷力轉化為可長期沿用、專案特定的制度檔案——精簡的 `CLAUDE.md`、模型調度守則、判斷 rubric、派工模板、維護協議，以及（選配的）hook 強制閘門。之後較弱的模型照著成文制度運作，而不是即興發揮，表現可明顯逼近高階模型。

[English README → README.md](README.md)

## 為什麼

高階模型的 session 拿來「立法」比拿來做任務更值得：把它的判斷力外化成檔案，只做一次，之後所有 session 都能繼承。但純條文在較弱模型手上會漂移，而處處機制強制又太重——**制度的重量必須與專案的風險匹配。** 個人小腳本背不動派工制度和驗證閘門；正式產品也不該只靠條文自律。所以這個 skill 是分層的。

## 三個層級

| | 產出 | Token 足跡 | 適用 |
|---|---|---|---|
| **lite** | 單一 `CLAUDE.md`（≤80 行）、內嵌 6 條精選 rubric、無 hook | 每個 session 只多付一個小檔案 | 個人腳本、prototype |
| **standard** | 精簡 `CLAUDE.md` + `.claude/rules/` 細則（調度、rubric、模板、維護、信） | 細則按階段載入，用到才付 | 多人協作、有 CI、常派工給較弱模型 |
| **full** | standard + 1–3 個 hook 強制閘門（保護路徑、驗證閘門）+ 重大結論抗辯 | 同上 + 每輪 hook 延遲 | 生產環境、金流、憑證——違規代價高的地方 |

三個層級共用同一套核心原則，差別只在承載形式與強制力。升級是增量的（`lite → standard → full`），不重新立法。

## 使用方式

**第一次在專案使用**——在專案目錄開啟 Claude Code，輸入：

```text
/codify
```

接下來 skill 會自己動：檢視專案（程式碼結構、git 歷史、既有 `CLAUDE.md`、CI 設定），附一句理由推薦一個層級，最多問你五個問題。你確認後，它會寫出下一節列出的制度檔案，對自己的產出做對抗審查，最後給你一頁總結——改了什麼、為什麼、之後怎麼用。

這一步**不需要最強的模型**。skill 內建了從高階模型蒸餾下來的判斷快照與完整成品範例（`references/judgment.md`、`references/exemplar.md`），中階模型執行 `init` 也能立出扎實的制度——立法變成「模仿＋填空」，不是從零發明。有更強的模型當然更好，但不是前提。

已經知道要哪個層級的話，可以跳過推薦直接指定：

```text
/codify lite
/codify standard
/codify full
```

**每隔幾週，或換用不同模型之後**——輸入：

```text
/codify audit
```

它會檢查制度是否退化——`CLAUDE.md` 膨脹、規則指向已不存在的檔案、踩坑記錄堆著沒人裁決——並且只做增量修正。任何模型都能執行。

**專案長大、超出現行層級時**（個人腳本變成團隊專案、服務要上生產環境）：

```text
/codify upgrade standard
/codify upgrade full
```

只補上缺少的交付項，已經寫好的東西一律不動。

## 會寫進專案的東西

skill 的所有產出都是純 Markdown（full 層級多兩個小 Python hook 腳本），而且**歸你的專案所有**：像一般檔案一樣 commit、修改、刪除都行。這些檔案不綁定特定模型、也不依賴本 skill——就算解除安裝 `codify`，制度照樣運作，只是少了方便執行 `audit`／`upgrade` 的入口。

`init` 跑完後，專案會長這樣：

```text
lite                      standard                        full = standard 再加上
─────────────             ─────────────────────────       ─────────────────────────
CLAUDE.md   (≤80 行，     CLAUDE.md   (精簡本體 +          .claude/hooks/
             全部內嵌)                 一行式索引)           protect_paths.py
                          .claude/rules/                    verify_gate.py
                            diagnosis.md                  .claude/settings.json
                            dispatch.md                     (hook 註冊)
                            rubrics.md
                            prompt-templates.md
                            maintenance.md
                            letter.md
```

每個檔案的用途：

| 交付項 | 內容 |
|---|---|
| `CLAUDE.md` | 唯一每個 session 都會載入的檔案：常用指令速查（立法時實際執行驗證過）、硬規則、同步耦合清單（「改 A 必改 B」）、指向其餘細則的一行式索引。上限 150 行。 |
| `diagnosis.md` | init 時找出的 token／失焦／出錯前三大熱點——每項附證據（檔案:行號或 git 記錄）與當場完成的修法。 |
| `dispatch.md` | 哪個等級的模型接哪種工作、派工三要素（目標／驗收／回報格式）、升降級觸發條件、執行與驗證分離、哪些結論必須通過重大結論抗辯。 |
| `rubrics.md` | 判斷力寫成的 checklist：何時升級、何時算完成（含修 bug 的 fail-then-pass 證據要求）、何時停下、驗證者逐項走的品質關卡。 |
| `prompt-templates.md` | 五種常見任務形態（搜尋、實作、重構、研究、審查）的填空派工模板。 |
| `maintenance.md` | 誰可以改哪個制度檔案、踩坑記錄（session 只能追加提案，由 `audit` 裁決）、制度退化的警訊。 |
| `letter.md` | 沒人問但每個未來 session 都該知道的事：真實優先序、環境的隱性事實、這套制度最可能的死法。 |
| `.claude/hooks/`（僅 full） | 最多 3 個 fail-safe 閘門腳本，只留給最硬的規則——例如擋掉對產生碼的編輯、擋掉改了邏輯卻沒有測試證據就收工。每個 hook 裝好後都會實際觸發一次證明擋得住。 |

執行前值得知道的兩件事：

- **既有的 `CLAUDE.md` 絕不會被無聲丟棄。** 改寫前先備份（專案在 git 內則改在分支上進行），要刪除的內容會先整理成清單給你過目。
- **產出語言跟隨專案**：既有 `CLAUDE.md` 沿用其語言；否則用你對話的語言撰寫。無論哪種語言，專業術語一律保留英文。

每條規則都要通過同一個測試：**Haiku 等級的模型只看條文，能否無歧義地執行？** 不能，就改寫成 checklist 或明定升級路徑。

## 值得知道的設計決定

- **強模型的判斷力保存在 skill 裡。** 診斷的取捨方式、把判斷改寫成可執行規則的手法、各種專案形態的立法要點、弱模型立法的已知失敗模式，全部寫在 `judgment.md`；一份完整成品放在 `exemplar.md`。高階模型下線之後，它的判斷還在。
- **漸進揭露**——skill 每個階段只載入該階段的 reference，從不一次載入整本規則，也沒有每輪注入的協議。
- **執行中的 session 不得修改約束自己的規則。** 踩坑只能記錄提案，由 `audit` 裁決——這是防止制度腐化的核心機制。
- **Hook 是掙來的，不是預設的。** 一條規則要落成 hook，必須同時滿足「違規成本高」與「判定可機械化」。Hook 必須 fail-safe（腳本故障一律放行），阻擋時必須說明原因與正確做法。
- **驗證閘門檢查的是測試「通過」，不是測試指令「出現過」**——只用 regex 比對「跑過測試指令」的閘門，跑一個無關測試就能矇混過關。
- **重大結論抗辯**——三個反方鏡頭各有弱模型也能執行的通過條件（構造具體反例／證據強度對比最壞代價／勝過更簡單的替代方案），過半存活才算 confirmed。只保留給重大結論——它是整套制度中最貴的機制。

## 安裝

直接告訴 Claude Code：

> 依照 https://github.com/treekey/claude-codify 的 INSTALL.md 安裝 codify skill

或見 [INSTALL.md](INSTALL.md) 的兩步驟手動安裝。安裝不會裝任何 hook，也不會動到 `~/.claude/skills/codify/` 以外的任何東西。

## 授權

[MIT](LICENSE)
