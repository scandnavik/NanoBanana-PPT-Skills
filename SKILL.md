---
name: ppt-generator-pro
description: 使用此 Skill 產生高品質 AI PPT 簡報圖片和影片。當使用者提到「產生 PPT」、「做簡報」、「PPT 風格」、「投影片」、「presentation」、「香蕉簡報」、「Banana PPT」或提供文件要求轉換成簡報時觸發。
---

# PPT Generator Pro - Claude Code Skill

## 📋 中繼資料

- **Skill 名稱**: ppt-generator-pro
- **版本**: 2.0.0
- **描述**: 基於 AI 自動產生高品質 PPT 圖片和影片，支援智慧轉場和互動式播放
- **作者**: 歸藏
- **標籤**: ppt, presentation, video, ai, nano-banana, kling-ai, image-generation

## ✨ 功能特色

### 核心功能

- 🤖 **智慧文件分析** - 自動擷取核心要點，規劃 PPT 內容架構
- 🎨 **多風格支援** - 內建漸層毛玻璃、向量插畫、現代商務幾何三種專業風格
- 🖼️ **高品質圖片** - 使用 Nano Banana Pro 產生 16:9 高畫質 PPT
- 🎬 **AI 轉場影片** - 可靈 AI 產生流暢的頁面過場動畫
- 🎮 **互動式播放器** - 影片+圖片混合播放，支援鍵盤導覽
- 🎥 **完整影片匯出** - FFmpeg 合成包含所有轉場的完整 PPT 影片

### 新功能 (v2.0)

- 🔄 **首頁循環預覽** - 自動產生吸睛的循環動畫
- 🎞️ **智慧轉場** - 自動產生頁面間的過場影片
- 🔧 **參數統一** - 自動統一所有影片解析度和幀率

## 📦 系統需求

### 環境變數

**必要：**

- `GEMINI_API_KEY`: Google AI API 金鑰（用於產生 PPT 圖片）

**選用（用於影片功能）：**

- `KLING_ACCESS_KEY`: 可靈 AI Access Key
- `KLING_SECRET_KEY`: 可靈 AI Secret Key

### Python 相依套件

```bash
pip install google-genai pillow python-dotenv
```

### 影片功能相依套件

```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg
```

## 🚀 使用方法

### 在 Claude Code 中呼叫

```bash
/ppt-generator-pro
```

或直接告訴 Claude：

```
我想基於以下文件產生一個 5 頁的 PPT，使用漸層毛玻璃風格。

[文件內容...]
```

## 📝 Skill 執行流程

### 階段 1: 收集使用者輸入

#### 1.1 取得文件內容

**選項 A: 文件路徑**

```
使用者: 基於 my-document.md 產生 PPT
→ 使用 Read 工具讀取檔案內容
```

**選項 B: 直接文字**

```
使用者: 我想產生一個關於 AI 產品設計的 PPT
主要內容：
1. 現況分析
2. 設計原則
3. 案例研究
```

**選項 C: 主動詢問**

```
如果使用者未提供內容，詢問：
「請提供文件路徑或直接貼上文件內容」
```

#### 1.2 選擇風格（動態掃描）

**步驟 1：掃描可用風格**

使用 Glob 工具掃描 `styles/` 目錄中的所有 `.md` 檔案：

```bash
# 掃描路徑
~/.claude/skills/NanoBanana-PPT-Skills/styles/*.md
```

**步驟 2：讀取每個風格檔案的標題區塊**

每個風格檔案的開頭都有標準化的資訊，格式如下：

```markdown
# 風格名稱

1. 核心靈魂
設計風格: "風格關鍵字..."
設計理念: "設計理念描述..."

2. 視覺外觀
全域視覺規範:
  氛圍基調: "適用情境描述..."
```

從每個檔案擷取：
- **風格名稱**：第一行 `# ` 後的標題
- **適用情境**：從「氛圍基調」或「設計理念」擷取關鍵字

**步驟 3：動態產生選項**

使用 AskUserQuestion 呈現所有掃描到的風格：

```markdown
問題: 請選擇 PPT 風格
選項:
- [風格名稱 1]（[適用情境關鍵字]）
- [風格名稱 2]（[適用情境關鍵字]）
- [風格名稱 N]（[適用情境關鍵字]）
```

**範例：** 如果 `styles/` 目錄中有 3 個檔案，產生的選項可能是：
```markdown
- 漸層毛玻璃卡片風格（科技感、商務簡報）
- 向量插畫風格（溫馨、教育訓練）
- 現代商務幾何風（專業、銳利、企業權威感）
```

**注意：** 風格數量和內容會隨 `styles/` 目錄的檔案變動而自動更新，無需手動維護清單。

#### 1.3 選擇頁數範圍

使用 AskUserQuestion 詢問：

```markdown
問題: 希望產生多少頁 PPT？
選項:
- 5 頁（5 分鐘演講）
- 5-10 頁（10-15 分鐘演講）
- 10-15 頁（20-30 分鐘演講）
- 20-25 頁（45-60 分鐘演講）
```

#### 1.4 選擇解析度

```markdown
問題: 選擇圖片解析度
選項:
- 2K (2752x1536) - 推薦，快速產生
- 4K (5504x3072) - 高品質，適合列印
```

#### 1.5 是否產生影片（選用）

如果設定了可靈 AI 金鑰，詢問：

```markdown
問題: 是否產生轉場影片？
選項:
- 僅圖片（快速）
- 圖片 + 轉場影片（完整體驗）
```

### 階段 2: 文件分析與內容規劃

#### 2.1 內容規劃策略

根據頁數範圍，智慧規劃每一頁內容：

**5 頁版本：**

1. 封面：標題 + 核心主題
2. 要點 1：第一個核心觀點
3. 要點 2：第二個核心觀點
4. 要點 3：第三個核心觀點
5. 總結：核心結論或行動建議

**5-10 頁版本：**

1. 封面
2-3. 引言/背景
4-7. 核心內容（3-4 個關鍵觀點）
8-9. 案例或數據支持
2. 總結與行動建議

**10-15 頁版本：**

1. 封面
2-3. 引言/目錄
4-6. 第一章節（3 頁）
7-9. 第二章節（3 頁）
10-12. 第三章節/案例研究
13-14. 數據視覺化
2. 總結與下一步

**20-25 頁版本：**

1. 封面
2. 目錄
3-4. 引言和背景
5-8. 第一部分（4 頁）
9-12. 第二部分（4 頁）
13-16. 第三部分（4 頁）
17-19. 案例研究
20-22. 數據分析和洞察
23-24. 關鍵發現和建議
3. 總結與致謝

#### 2.2 產生 slides_plan.json

建立 JSON 檔案：

```json
{
  "title": "文件標題",
  "total_slides": 5,
  "slides": [
    {
      "slide_number": 1,
      "page_type": "cover",
      "content": "標題：AI 產品設計指南\n副標題：打造以使用者為中心的智慧體驗"
    },
    {
      "slide_number": 2,
      "page_type": "content",
      "content": "核心原則\n- 簡單直覺\n- 快速回應\n- 透明可控"
    },
    {
      "slide_number": 3,
      "page_type": "content",
      "content": "設計流程\n1. 使用者研究\n2. 原型設計\n3. 測試迭代"
    },
    {
      "slide_number": 4,
      "page_type": "data",
      "content": "使用者滿意度\n使用前：65%\n使用後：92%\n提升：+27%"
    },
    {
      "slide_number": 5,
      "page_type": "content",
      "content": "總結\n- 以使用者為中心\n- 持續優化迭代\n- 數據驅動決策"
    }
  ]
}
```

**重要：** 將此檔案儲存到：

- 獨立使用：`./slides_plan.json`
- Skill 模式：`.claude/skills/ppt-generator/slides_plan.json`

### 階段 3: 產生 PPT 圖片

#### 3.1 確定工作目錄

**獨立模式：**

```bash
cd /path/to/ppt-generator
```

**Skill 模式：**

```bash
cd ~/.claude/skills/ppt-generator
```

#### 3.2 執行產生指令

```bash
python generate_ppt.py \
  --plan slides_plan.json \
  --style styles/gradient-glass.md \
  --resolution 2K
```

**或使用 uv run（推薦）：**

```bash
uv run python generate_ppt.py \
  --plan slides_plan.json \
  --style styles/gradient-glass.md \
  --resolution 2K
```

**參數說明：**

- `--plan`: slides 規劃 JSON 檔案路徑
- `--style`: 風格檔案路徑
- `--resolution`: 解析度（2K 或 4K）
- `--template`: HTML 範本路徑（選用）

#### 3.3 監控產生進度

腳本會輸出進度資訊：

```
✅ 已載入環境變數: /path/to/.env
📊 開始產生 PPT 圖片...
   總頁數: 5
   解析度: 2K (2752x1536)
   風格: 漸層毛玻璃卡片風格

🎨 產生第 1 頁 (封面頁)...
   提示詞已產生
   呼叫 Nano Banana Pro API...
   ✅ 第 1 頁產生成功 (32.5 秒)

🎨 產生第 2 頁 (內容頁)...
   ✅ 第 2 頁產生成功 (28.3 秒)

...

✅ 所有頁面產生完成！
📁 輸出目錄: outputs/20260112_143022/
```

### 階段 4: 產生轉場提示詞（影片模式需要）

**這是 Skill 的核心優勢**：我（Claude Code）會分析產生的 PPT 圖片，為每個轉場產生精準的影片提示詞。

#### 4.1 讀取並分析 PPT 圖片

我會讀取所有產生的圖片：

```python
# 自動讀取輸出目錄中的所有圖片
slides = ['slide-01.png', 'slide-02.png', ...]
```

#### 4.2 分析圖片差異並產生提示詞

對於每對相鄰圖片，我會：

1. **視覺分析**：理解兩張圖片的版面、元素、色彩差異
2. **產生預覽提示詞**：為首頁建立可循環的微動效描述
3. **產生轉場提示詞**：詳細描述如何從起始幀過渡到結束幀

**範例輸出：**

```json
{
  "preview": {
    "slide_path": "outputs/.../slide-01.png",
    "prompt": "畫面保持封面的靜態構圖，中心的 3D 玻璃環緩慢旋轉..."
  },
  "transitions": [
    {
      "from_slide": 1,
      "to_slide": 2,
      "prompt": "鏡頭從封面開始，玻璃環逐漸解構，分裂成透明碎片..."
    }
  ]
}
```

#### 4.3 儲存提示詞檔案

我會將產生的提示詞儲存到：

```
outputs/TIMESTAMP/transition_prompts.json
```

**關鍵優勢：**

- ✅ 不需要單獨的 Claude API 金鑰
- ✅ 提示詞針對實際圖片內容客製化
- ✅ 考慮文字穩定性，避免影片模型弄糊文字
- ✅ 符合漸層毛玻璃風格的視覺語言

### 階段 5: 產生轉場影片（選用）

如果使用者選擇產生影片，使用階段 4 產生的提示詞檔案：

```bash
python generate_ppt_video.py \
  --slides-dir outputs/20260112_143022/images \
  --output-dir outputs/20260112_143022_video \
  --prompts-file outputs/20260112_143022/transition_prompts.json
```

**產生內容：**

- 首頁循環預覽影片（`preview.mp4`）
- 頁面間轉場影片（`transition_01_to_02.mp4` 等）
- 互動式影片播放器（`video_index.html`）
- 完整影片（`full_ppt_video.mp4`）

### 階段 6: 回傳結果

#### 6.1 僅圖片模式

```
✅ PPT 產生成功！

📁 輸出目錄: outputs/20260112_143022/
🖼️ PPT 圖片: outputs/20260112_143022/images/
🎬 播放網頁: outputs/20260112_143022/index.html

開啟播放網頁:
open outputs/20260112_143022/index.html

播放器快捷鍵:
- ← → 鍵: 切換頁面
- ↑ Home: 回到首頁
- ↓ End: 跳到末頁
- 空白鍵: 暫停/繼續自動播放
- ESC: 全螢幕切換
- H: 隱藏/顯示控制項
```

#### 6.2 影片模式

```
✅ PPT 影片產生成功！

📁 輸出目錄: outputs/20260112_143022_video/
🖼️ PPT 圖片: outputs/20260112_143022/images/
🎬 轉場影片: outputs/20260112_143022_video/videos/
🎮 互動式播放器: outputs/20260112_143022_video/video_index.html
🎥 完整影片: outputs/20260112_143022_video/full_ppt_video.mp4

開啟互動式播放器:
open outputs/20260112_143022_video/video_index.html

播放邏輯:
1. 首頁: 播放循環預覽影片
2. 按右鍵 → 播放轉場影片 → 顯示目標頁圖片（2 秒）
3. 再按右鍵 → 播放下一個轉場 → 顯示下一頁圖片
4. 依此類推...

影片播放器快捷鍵:
- ← → 鍵: 上一頁/下一頁（含轉場）
- 空白鍵: 播放/暫停目前影片
- ESC: 全螢幕切換
- H: 隱藏/顯示控制項
```

## 🔧 環境變數設定

### .env 檔案位置

Skill 會按以下順序尋找 `.env` 檔案：

1. **腳本所在目錄** - `./ppt-generator/.env`
2. **向上尋找專案根目錄** - 直到找到包含 `.git` 或 `.env` 的目錄
3. **Claude Skill 標準位置** - `~/.claude/skills/ppt-generator/.env`
4. **系統環境變數** - 如果以上都未找到

### .env 檔案範例

```bash
# Google AI API 金鑰（必要）
GEMINI_API_KEY=your_gemini_api_key_here

# 可靈 AI API 金鑰（選用，用於影片功能）
KLING_ACCESS_KEY=your_kling_access_key_here
KLING_SECRET_KEY=your_kling_secret_key_here
```

## ⚠️ 錯誤處理

### 常見錯誤及解決方案

**1. API 金鑰未設定**

```
錯誤: ⚠️ 未找到 .env 檔案，嘗試使用系統環境變數
      未設定 GEMINI_API_KEY 環境變數

解決:
1. 建立 .env 檔案
2. 新增 GEMINI_API_KEY=your_key_here
```

**2. Python 相依套件缺失**

```
錯誤: ModuleNotFoundError: No module named 'google.genai'

解決: pip install google-genai pillow python-dotenv
```

**3. FFmpeg 未安裝**

```
錯誤: ❌ FFmpeg 不可用！

解決: brew install ffmpeg  # macOS
      sudo apt-get install ffmpeg  # Ubuntu
```

**4. API 呼叫失敗**

```
錯誤: API 呼叫逾時或失敗

解決:
1. 檢查網路連線
2. 確認 API 金鑰有效
3. 稍後重試
```

**5. 影片產生失敗**

```
錯誤: 可靈 AI 金鑰未設定

解決:
1. 如果只需要圖片，跳過影片產生步驟
2. 如果需要影片，設定 KLING_ACCESS_KEY 和 KLING_SECRET_KEY
```

## 🎨 風格系統（動態擴充）

### 風格檔案位置

所有風格檔案存放於：

```
~/.claude/skills/NanoBanana-PPT-Skills/styles/
```

### 風格檔案標準格式

每個風格 `.md` 檔案必須遵循以下結構，Claude 會自動解析：

```markdown
# 風格名稱

1. 核心靈魂
設計風格: "關鍵字1, 關鍵字2, ..."
設計理念: "一段描述這個風格的設計哲學..."

2. 視覺外觀
全域視覺規範:
  氛圍基調: "適用情境的關鍵字描述"
  色彩計畫:
    背景色: "色彩描述"
    主色: "色彩描述"
    強調色: "色彩描述"
  材質與紋理: "材質描述"

3. 空間邏輯
通用佈局準則:
  構圖邏輯: "版面配置描述"
  空間留白: "留白策略"
  層次關係: "視覺層次描述"

4. 執行細節
細部設計執行規範:
  線條處理: "線條風格"
  裝飾元素: "裝飾說明"
  圖片處理規則: "圖片處理方式"

5. 進階延伸
圖像生成指引:
  渲染風格: "給 AI 圖像模型的正面提示詞"
  負面提示: "給 AI 圖像模型的負面提示詞"
```

### Claude 如何使用風格檔案

1. **掃描**：執行 Skill 時，使用 Glob 掃描 `styles/*.md`
2. **解析**：讀取每個檔案的標題和「氛圍基調」作為選項描述
3. **呈現**：動態產生 AskUserQuestion 選項讓使用者選擇
4. **套用**：將選中的風格檔案完整內容傳給圖像產生模型

### 新增自訂風格

1. 在 `styles/` 目錄建立新的 `.md` 檔案（檔名即為風格識別碼）
2. 按照上述標準格式撰寫風格定義
3. 儲存後，下次執行 Skill 時會自動出現在選項中

### 風格命名建議

| 檔名格式 | 範例 |
|---------|------|
| 英文小寫 + 連字號 | `gradient-glass.md` |
| 中文描述性名稱 | `現代商務幾何風.md` |
| 主題 + 風格 | `tech-futuristic.md` |

### 目前已有風格

執行以下指令查看所有可用風格：

```bash
ls ~/.claude/skills/NanoBanana-PPT-Skills/styles/
```

Claude 會在執行時自動讀取這些檔案並擷取關鍵資訊呈現給使用者。

## 📊 技術細節

### API 設定

**Nano Banana Pro（圖片產生）：**

- 模型：`gemini-3-pro-image-preview`
- 比例：`16:9`
- 回應模式：`IMAGE`
- 解析度：2K (2752x1536) 或 4K (5504x3072)

**可靈 AI（影片產生）：**

- 模式：專業模式（professional）
- 時長：5 秒
- 解析度：1920x1080
- 幀率：24fps

**FFmpeg（影片合成）：**

- 編碼：H.264
- 品質：CRF 23
- 幀率：24fps（統一）
- 解析度：1920x1080（統一）

### 效能指標

**產生速度：**

- PPT 圖片：~30 秒/頁（2K）| ~60 秒/頁（4K）
- 轉場影片：~30-60 秒/段
- 影片合成：~5-10 秒

**檔案大小：**

- PPT 圖片：~2.5MB/頁（2K）| ~8MB/頁（4K）
- 轉場影片：~3-5MB/段（1080p，5 秒）
- 完整影片：~12-20MB（5 頁 PPT + 轉場）

## 📁 檔案組織

### 輸出目錄結構

**僅圖片模式：**

```
outputs/20260112_143022/
├── images/
│   ├── slide-01.png
│   ├── slide-02.png
│   └── ...
├── index.html          # 圖片播放器
└── prompts.json        # 提示詞記錄
```

**影片模式：**

```
outputs/20260112_143022_video/
├── videos/
│   ├── preview.mp4              # 首頁循環預覽
│   ├── transition_01_to_02.mp4
│   ├── transition_02_to_03.mp4
│   └── ...
├── video_index.html             # 互動式播放器
└── full_ppt_video.mp4           # 完整影片
```

## 🎯 最佳實務

1. **文件品質**：輸入文件內容越清晰結構化，產生的 PPT 品質越高
2. **頁數選擇**：根據文件長度和簡報情境合理選擇頁數
3. **解析度選擇**：日常使用推薦 2K，重要展示場合可選 4K
4. **影片功能**：首次使用建議先嘗試僅圖片模式，熟悉後再使用影片功能
5. **提示詞調整**：查看 `prompts.json` 了解產生邏輯，可手動調整後重新產生

## 📝 使用範例

### 範例 1: 快速產生

**使用者輸入：**

```
我需要基於這份會議紀錄產生一個 5 頁的 PPT，使用向量插畫風格。

會議主題：Q1 產品路線圖規劃
參與人：產品團隊

討論內容：
1. 使用者回饋彙整
2. 新功能優先順序
3. 技術可行性評估
4. Q1 里程碑
5. 下一步行動項目
```

**Skill 執行：**

1. 收集輸入（已提供內容）
2. 確認風格（向量插畫）
3. 確認頁數（5 頁）
4. 確認解析度（詢問使用者）
5. 產生 slides_plan.json
6. 執行產生指令
7. 回傳結果

### 範例 2: 完整流程

**使用者輸入：**

```
基於 AI-Product-Design.md 文件，產生一個 15 頁的 PPT，使用漸層毛玻璃風格，需要轉場影片。
```

**Skill 執行：**

1. 讀取文件內容
2. 確認風格（漸層毛玻璃）
3. 確認頁數（15 頁）
4. 確認解析度（詢問使用者）
5. 確認產生影片（是）
6. 分析文件，規劃 15 頁內容
7. 產生 slides_plan.json
8. 產生 PPT 圖片
9. 產生轉場影片
10. 合成完整影片
11. 回傳所有結果

## 🔄 更新日誌

### v2.0.0 (2026-01-12)

- 🎬 **新增影片功能**
  - 可靈 AI 轉場影片產生
  - 互動式影片播放器
  - FFmpeg 完整影片合成
  - 首頁循環預覽影片
- 🔧 **最佳化影片合成**
  - 自動統一解析度和幀率
  - 修復影片拼接相容性問題
  - 靜態圖片展示時間改為 2 秒
- 🔑 **改進環境變數**
  - 智慧尋找 .env 檔案
  - 支援多種部署模式
  - 自動向上尋找專案根目錄
- 📚 **文件完善**
  - 重新命名為 SKILL.md（符合官方規範）
  - 更新所有路徑和指令
  - 新增影片功能使用指南

### v1.0.0 (2026-01-09)

- ✨ 首次發布
- 🎨 內建 2 種專業風格
- 🖼️ 支援 2K/4K 解析度
- 🎬 HTML5 圖片播放器
- 📊 智慧文件分析

## 📄 授權條款

MIT License

## 📞 技術支援

- 專案架構：參見 `ARCHITECTURE.md`
- API 管理：參見 `API_MANAGEMENT.md`
- 環境設定：參見 `ENV_SETUP.md`
- 安全說明：參見 `SECURITY.md`
- 完整文件：參見 `README.md`
