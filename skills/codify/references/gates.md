# H. 機制性閘門（hooks）— 僅 full 層級

目標：把專案中違規成本最高的 1–3 條硬規則，從條文升級為 harness 層強制的 hook。
條文依賴模型自律、會漂移；hook 由 harness 執行，模型再弱也擋得住。
落點：腳本放 `.claude/hooks/`，註冊寫入專案 `.claude/settings.json` 的 `hooks` 段。

## 什麼規則值得落成 hook（兩個條件同時成立）

1. **違規成本高**：動到生產設定、覆蓋產生的程式碼、未驗證就收工等，錯一次的代價遠大於每輪多跑一個腳本。
2. **判定可機械化**：能用路徑比對、regex、exit code 判斷。需要語意理解的規則（「diff 不得超出任務範圍」）留在條文與審查，硬塞進 hook 只會製造誤擋。

上限 3 個。每個 hook 都是每輪 latency 與一份維護負擔；想加第 4 個時，先刪一個。

## 設計鐵則（每個 hook 都要遵守）

- **fail-safe**：腳本任何異常（解析失敗、環境缺 python）一律靜默放行。閘門故障不得癱瘓 session。
- **擋人要講理**：阻擋訊息必須說明「違反哪條規則 + 正確做法」，讓被擋的模型能自行合規，而不是反覆撞牆。
- **條文對照**：每個 hook 在 CLAUDE.md 硬規則段標註 `[hook 強制]`。條文與 hook 必須同步，audit 時互相核對。
- **標註已知繞法**：立法時以「我要繞過它」的立場找出繞法（例如 Stop gate 的二次收工放行），寫進維護協議並註明「繞過即違規，audit 會查記錄」。

## 模板一：保護路徑（PreToolUse）

擋掉對產生的程式碼、vendored 依賴、生產設定的直接編輯。判定純路徑比對，幾乎零誤擋，是 CP 值最高的 hook。

`.claude/hooks/protect_paths.py`：
```python
import json, sys
PROTECTED = ["src/generated/", "vendor/", "config/prod."]  # ← 換成專案實際路徑
try:
    data = json.load(sys.stdin)
    path = (data.get("tool_input") or {}).get("file_path", "").replace("\\", "/")
    for p in PROTECTED:
        if p in path:
            print(f"[硬規則] {p} 為保護路徑，不得直接修改。"
                  f"正確做法見 CLAUDE.md 硬規則段（如：改上游模板後重新生成）。", file=sys.stderr)
            sys.exit(2)  # exit 2 = 阻擋，stderr 回饋給模型
except Exception:
    pass  # fail-safe
sys.exit(0)
```

`.claude/settings.json` 註冊：
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Edit|Write|NotebookEdit",
      "hooks": [{ "type": "command", "command": "python .claude/hooks/protect_paths.py" }]
    }]
  }
}
```

## 模板二：驗證閘門（Stop）

改了程式碼卻沒有測試通過的證據，就擋下收工。比「檢查測試指令出現過」更嚴：要求指令**成功執行**。

邏輯（腳本依專案生成，骨架如下）：
1. 讀 stdin JSON；`stop_hook_active` 為 true 直接放行（防死鎖——此即已知繞法，需在條文標註）。
2. 掃 transcript：最後一則使用者訊息之後，是否有 Edit/Write 涉及程式碼副檔名（依專案定清單）。沒有 → 放行。
3. 有 → 檢查其後是否出現 `<專案測試指令>` 的執行記錄**且結果非失敗**（檢查 exit code 或輸出中的失敗標記，如 `failed`、`FAILED`、非零 exit）。只跑過但紅著，不放行。
4. 阻擋訊息寫明：「改了 <檔案清單> 但無測試通過證據。請執行 `<專案測試指令>`；若改動確實無邏輯影響（純註解/文件），在回報中明述後再收工。」
5. 全程 try/except，異常放行。

立法時把 `<專案測試指令>` 與副檔名清單寫死進腳本——這正是通用 harness 做不到、專案立法能做到的精度。

## 安裝後驗證（收尾步驟 2 的一部分，不可省略）

每個 hook 實際觸發一次證明會擋：
- 保護路徑：嘗試 Edit 一個保護檔案，確認被擋且訊息可讀，然後還原。
- 驗證閘門：改一個程式碼檔後直接嘗試收工，確認被擋；跑測試後確認放行，然後還原改動。

觸發結果記入 diagnosis.md 或一頁總結。裝了沒驗證的 hook 等於沒裝。
