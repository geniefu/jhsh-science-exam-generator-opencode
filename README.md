# 國高中自然科命題規範與自動化產卷技能 (jhsh-science-exam-generator)

> **OpenCode 專用版**：本倉庫 (`jhsh-science-exam-generator-opencode`) 存放已改寫為 OpenCode 規格的版本。
> 原始 Google Antigravity 版請見 [geniefu/jhsh-science-exam-generator](https://github.com/geniefu/jhsh-science-exam-generator)。

本技能 (Skill) 專門用於指導 **OpenCode** 等支援 `SKILL.md` 的 AI 代理人進行國高中自然科（生物、理化、地球科學、物理、化學）段考自動化命題，生成符合「新北市立錦和高級中學」教務處規範、素養導向評量與 PISA 2025 標準之試題卷與答案解析卷（雙份 Word 檔）。

---

## 🏛️ 專案模組架構 (Architecture)

本技能遵循漸進揭示 (Progressive Disclosure) 設計，採精簡化三層架構；所有路徑皆相對於技能目錄：

```text
jhsh-science-exam-generator/
├── SKILL.md                          # 核心引導文檔：規範、提詞模板、排版守則與檢查清單
├── README.md                         # 專案介紹與安裝說明
├── requirements.txt                  # Python 相依套件 (python-docx, matplotlib, numpy, sympy)
├── LICENSE                           # MIT 開源授權條款
├── scripts/                          # 自動化輔助程式碼庫
│   ├── docx_science_builder.py       # Word 試卷建構器：1cm 邊界、全卷 11Pt、浮動文繞圖、動態頁碼
│   ├── science_chem_math_omml.py     # Word 原生可編輯 OMML：化學式、物態、同位素、單位與方程式
│   ├── science_plotter.py            # 300 DPI 繪圖引擎：+8Pt 特大字級、黑白灰階與斜線填充
│   └── science_verifier.py           # 試卷品質校驗器：總分 100/連續題號/選項平衡/防超綱過濾
└── references/                       # 課綱與素養評量參考庫
    ├── curriculum_science_inquiry.md # 108 自然課綱指標、探究六歷程 (I1~I6) 與 Bloom 認知層次
    └── science_anti_out_of_bounds.md # 嚴格防超綱紅線清單 (嚴禁用「斜率」、禁複雜合金密度計算等)
```

---

## 📦 安裝（OpenCode）

OpenCode 會自動掃描以下位置，放入後即生效，**不需要重啟**：

| 範圍 | 路徑 |
| --- | --- |
| 全域（所有專案可用，推薦） | `~/.config/opencode/skills/jhsh-science-exam-generator/`（Windows：`C:\Users\<使用者>\.config\opencode\skills\jhsh-science-exam-generator\`） |
| 單一專案 | `<專案根目錄>/.opencode/skills/jhsh-science-exam-generator/` |
| 相容位置（其他代理） | `~/.claude/skills/`、`~/.agents/skills/` |

### 方式 A：Git 安裝（未來更新只需 `git pull`）

**Windows (PowerShell)**

```powershell
git clone https://github.com/geniefu/jhsh-science-exam-generator-opencode.git "$HOME\.config\opencode\skills\jhsh-science-exam-generator"
```

**macOS / Linux**

```bash
git clone https://github.com/geniefu/jhsh-science-exam-generator-opencode.git ~/.config/opencode/skills/jhsh-science-exam-generator
```

### 方式 B：手動安裝

將本倉庫資料夾複製到上表任一位置，確認 `SKILL.md` 位於 `jhsh-science-exam-generator/SKILL.md` 即可。

> 技能 ID 由資料夾名稱決定，必須是 `jhsh-science-exam-generator`；`SKILL.md` 的 frontmatter `name` 只是顯示名稱。

### 安裝 Python 依賴

```bash
pip install -r requirements.txt
```

或：

```bash
pip install python-docx matplotlib numpy sympy
```

---

## 🚀 使用方式

OpenCode 會在對話中依 `description` 自動判斷是否載入本技能；也可用 `@` 明確指定：

```text
@jhsh-science-exam-generator
```

> 注意：`/jhsh-science-exam-generator` 這種斜線寫法在 OpenCode 不會觸發技能（`/` 保留給自訂 command），請使用 `@技能ID` 或直接描述產卷需求。

### 提詞範例

```text
請使用 @jhsh-science-exam-generator 技能為我生成一份自然科段考試卷，規格參數如下：
1、標題：新北市立錦和高級中學115學年度第一學期 國中部八年級 自然科第一次段考試題卷
2、範圍：緒論到 3-1
3、題型：單一選擇題，每題有四個選項 (A)(B)(C)(D)
4、題數：35 題（含單題與素養閱讀題組）
5、配分：總分 100 分（第 1～25 題每題 3 分共 75 分；第 26～30 題題組每題 3 分共 15 分；第 31～35 題題組每題 2 分共 10 分）
6、難易程度：中等偏難（預估平均通過率約 55%～65%）
7、其它：以「皮克敏的故事世界」之虛擬情境串連試卷題組
```

---

## 🛠️ 環境依賴需求 (Dependencies)

```bash
pip install -r requirements.txt
```

- Python 3.10+（已於 Python 3.14 測試）
- `python-docx` 1.1+
- `matplotlib` 3.7+
- `numpy` 1.24+
- `sympy` 1.12+

---

## ✨ 核心特色

1. **嚴格符合錦和高中格式**：完整試卷抬頭、必放警語（未寫座號扣 5 分等）、全卷 11Pt 標楷體/Times New Roman、**A4 直式版面**、1cm 窄邊界、題號凸排、右側浮動文繞圖與動態頁碼（第 X 頁，共 Y 頁）。
2. **雙份獨立 Word 文件自動產出（免 PDF 與檢核表）**：同步產出正式試題卷 (`.docx`) 與含題題詳解的答案卷 (`.docx`)，直接開啟即可送印。
3. **Word 原生 OMML 數學與化學方程式**：零外部編譯器依賴，化學式上下標、反應箭頭與科學單位在 Word 中雙擊即可編輯。
4. **高品質向量繪圖與 +8 Pt 特大字級**：透過 Matplotlib 自動繪製 300 DPI 考卷插圖，特大字級（16.5~21 Pt）、高對比黑白灰階，避免油印模糊與文字壓線。
5. **素養導向與認知階層**：融合 Bloom 六大認知階層、探究六步驟 (I1~I6)、原創素養長文題組與生活化情境。
6. **全自動防錯驗證**：自動校驗總分 100 分、連續題號、選項配比平衡與嚴格防超綱禁忌詞清單。
7. **100% 零 LaTeX `$` 符號殘留**：內建變數與符號全域自動清洗機制，物理變數（V、M、f、λ）與化學式（H₂O、CO₂、→）以中學考卷最乾淨的純文字與 Unicode 呈現。

---

## 📝 版本紀錄

### v1.1.1 — A4 版面

- `set_base_style()` 明確設定 **A4 直式（21.0 × 29.7 cm）**，不再沿用 python-docx 預設的 Letter 版面。
- `SKILL.md` 的版面規格與技能描述同步改為 A4。

### v1.1.0 — OpenCode 相容與繪圖修正

- **修正 `add_question()` 致命錯誤**：原本漏建 `p = doc.add_paragraph()`，呼叫即 `NameError`，導致無法產卷。
- **SKILL.md 改為 OpenCode 用法**：改用 `@jhsh-science-exam-generator`、所有檔案連結改為相對路徑，移除 Antigravity 專屬絕對路徑。
- **README 改為 OpenCode 安裝說明**：全域／專案／相容路徑與自動重載說明。
- **繪圖重疊修正**：橫波圖「波峰／波長 λ」與振幅標籤不再壓線；量筒截距標籤移至直線下方空白區；天平衡「刻度盤 (0)」不再與指針、支柱重疊。
- **天平衡指針方向修正**：`pointer_angle > 0` 正確偏右，與 docstring 一致。

### v1.0.0 — 初版

- 三層模組化架構、雙份 Word 產出、OMML 化學式、300 DPI 繪圖與自動檢核。

---

## 📄 授權

MIT License，詳見 [LICENSE](LICENSE)。
