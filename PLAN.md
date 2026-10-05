# 《小小國情島》MVP｜Implementation Plan

> **For Hermes:** plan-first；Ken 回「順序」先實作。未寫可玩 `index.html` 唔算 DONE。  
> **構思：** A 拍板（Top-down Zelda/Pokémon 式）  
> **路徑：** `~/workspace/vs code/education/value_education/national-island/`  
> **狀態：** 資料夾 ✅ · 本計劃 FIRM ✅ · `index.html` ⏳ 0%

**Goal:** 將軍澳培智中度班用 **單檔 HTML** 國民教育小遊戲：retro top-down 小島探索 + 圖選答題；PC 方向鍵/A/B + iPad 自製大 VK 同一套。

**Architecture:** 單檔 `index.html`（HTML+CSS+JS 內嵌）。畫面 state machine：身份 → 地圖行 → 撞 spot 答題 → 開區 → Boss 3 題 → 金銀銅證。無後端、無 build；GitHub Pages 可直接 deploy。

**Tech Stack:** 純 HTML/CSS/JS · Canvas 或 DOM grid 地圖 · Web Speech `zh-HK` 手動 🔊 · touch 大掣 · Pages

**Rev:** v1.0 · 2026-10-05  
**Ken lock:** 構思 **A** · 單檔 · keyboard + iPad VK · 6 關節奏 · SEN 硬規 · 金銀銅 · 身份卡

---

## 0. 構思鎖定（A 拍板）

| 選項 | 內容 | 結果 |
|------|------|------|
| **A（主推）** | Top-down《小小國情島》Zelda/Pokémon 式 | ✅ **FIRM** |
| B | 慢速橫向《升旗之路》 | 棄（可 v2 skin） |
| C | 標誌射擊皮 | 棄（hot seat 另案） |

**點解 A：** 方向鍵最自然、A/B≈SNES、內容可深可淺、可借 direction-coding／mouse-hub 經驗。

### 0.1 輸入鎖（PC + iPad 同一套）

| 鍵 | 作用 |
|----|------|
| ↑↓←→ | 行／選項游標 |
| **A** | 確認／答／對話／開始 |
| **B** | 取消／返／提示（可再按） |
| Start（可選） | 暫停／老師角 |

- **iPad** = 自製大 **VK**（十字 + A/B）；**禁** 系統鍵盤／`<input>` focus  
- **PC** = 實體鍵 + 同一 VK（可「藏 VK」）  
- 答題畫面：選項亦可用 **A/B/C/D 大圓** 點選（手勢作答）

### 0.2 Retro 適合度（已審）

| 類型 | 適否 | 備註 |
|------|------|------|
| Top-down 探索 | **最勁** | 一屏一事、A 對話／揀答 |
| 橫向跑跳 | 中 | 節奏易過難 |
| 格鬥／射擊全 game | 差／弱 | 內容難深 |
| 純 Quiz | 穩但悶 | 只做 Boss 關 |

---

## 1. 對象與硬約束（中度 SEN）

| 項 | 鎖死 |
|----|------|
| 學生 | 中度；低口語／少識字；數 1–20；看圖為主 |
| 裝置 | iPad Safari + PC Chrome；課堂橫向優先 |
| 輸入 | 大 touch ≥64px；VK + 實體鍵；禁系統鍵盤 |
| 一次一事 | 每 spot 1 題；地圖唔同時多 popup |
| 錯誤 | **無生命**、可重試；答錯 → 指錯處 + 再答；唔 Game Over |
| 選項 | 初 **2**／中 **3**／高 **4**；答案 **A/B/C/D 大圓**（渲染序） |
| 干擾項 | 只「明顯錯」；禁其他正確信條做錯項 |
| 文字 | 圖主字輔；每屏 ≤2 重點詞 highlight；學生 UI **零「SEN」字** |
| 身份 | 身份卡圖選／花名；唔強制打字 |
| 語音 | 粵 TTS **手動 🔊**；禁 auto-speak 預設開 |
| HTML 硬規 | 禁右鍵 + touch-callout + user-select none + `user-scalable=no` |
| 證書 | 金≥90%／銀≥70%／銅≥50%（可調老師角） |
| 交付 | **單檔** `index.html`；Pages only；可右鍵只老師角開 |

---

## 2. 教學內容層（MVP）

| # | 主題 | 形式 | 組別 |
|---|------|------|------|
| 1 | 國旗／國徽／區旗 | 認圖 2–4 選 1 | 全 |
| 2 | 國歌／升旗禮儀 | 正確行為圖（企定、安靜） | 全 |
| 3 | 香港是中國一部分 | 簡地圖／行去地標 | 全 |
| 4 | 傳統節日 | 圖配對（春節／中秋等） | 中高 |
| 5 | Boss 綜合 | 3 題連答（可重試每題） | 全 |
| 6 | 完節證書 | 金銀銅 + 身份名 | 全 |

**Non-goals MVP：** 基本法長文、歷史年表、帳號／排行、多結局、真實政治爭議題、外部圖 CDN 依賴（emoji/SVG 先）。

---

## 3. 核心 Loop

```
身份卡 → 小島地圖
  → 行到 spot（國旗亭／升旗台／地圖碑／節日園／Boss 門）
  → 圖 + 1 句題 → A/B/C/D 揀
  → 啱：開下一區 + 記分；錯：指錯 + 再答（可回滾本題分）
  → 全 spot 清 → Boss 3 題
  → 證書（可再玩）
```

**動態分：** `answered` / `correct`；重答本題先扣回再重計。

---

## 4. 關卡／地圖（6 節拍 MVP）

| 節 | Spot id | 地圖位（示意） | 題數 | 通關條件 |
|----|---------|----------------|------|----------|
| 1 | `flag_school` | 學堂／國旗亭 | 1 | 認國旗 |
| 2 | `anthem_etiquette` | 升旗台 | 1 | 正確禮儀行為 |
| 3 | `map_china` | 地圖碑 | 1 | 香港↔祖國簡關係／地標 |
| 4 | `festival` | 節日園 | 1 | 節日配對 |
| 5 | `boss` | 大門 | 3 | 連答（每題可重試） |
| 6 | `cert` | 證書屏 | — | 顯示等級 |

**地圖技術（選一，FIRM 實作 A）：**

| 方案 | 說明 | 選 |
|------|------|-----|
| **A DOM grid** | `display:grid` 瓦片 + 角色 absolute；碰撞用格座標 | ✅ MVP |
| B Canvas | 更「game」但 debug 重 | v1.1 |
| C 純關卡清單無自由行 | 似 ebook-pledge | 備援若碰撞拖太長 |

**格：** 約 8×6 可行格；牆／水不可行；spot 用大 emoji／色塊（後可換 SVG）。

**移動：** 每按方向 1 格；撞牆停；撞 spot 且未清 → 開對話／題。

---

## 5. 畫面架構

```
┌─────────────────────────────────────────────┐
│  小小國情島   ⭐3/5   🔊   [老師⚙]          │
├───────────────────────────┬─────────────────┤
│                           │  （可選）迷你地圖 │
│      🏝️  top-down 地圖     │                 │
│         👤 玩家            │                 │
│      🚩 未清 spot          │                 │
│                           │                 │
├───────────────────────────┴─────────────────┤
│           ┌───┐                             │
│       ┌───┤ ↑ ├───┐      ┌───┐  ┌───┐      │
│       │ ← │   │ → │      │ A │  │ B │      │
│       └───┤ ↓ ├───┘      └───┘  └───┘      │
│           └───┘           VK（iPad 常顯）    │
└─────────────────────────────────────────────┘
```

**答題 modal（蓋地圖）：**
- 大圖／行為圖
- 題幹 1 句 + highlight
- 2–4 選：大圓 A/B/C/D + 圖或短 label
- 🔊 讀題；答完可 🔊 讀正確答案
- 「再答一次」+「下一題／返地圖」

**證書屏：** 大 medal、身份花名、正確率、再玩／返主頁。

---

## 6. 資料結構（內嵌 JS）

```js
// 示意 — 實作時寫死喺 index.html
const CONFIG = {
  title: '小小國情島',
  difficulty: 'mid', // easy=2 mid=3 high=4 選項（老師角可改）
  gold: 0.9, silver: 0.7, bronze: 0.5,
  tile: 64,
  mapW: 8, mapH: 6,
};

const SPOTS = [
  {
    id: 'flag_school',
    x: 2, y: 2, emoji: '🚩',
    title: '認國旗',
    q: {
      prompt: '邊面係國旗？',
      tts: '邊面係國旗',
      // options 渲染後標 A/B/C…
      options: [
        { id: 'cn', label: '國旗', ok: true, visual: '🇨🇳' /* 或 inline SVG */ },
        { id: 'wrong1', label: '亂色旗', ok: false, visual: '🏳️' },
        { id: 'wrong2', label: '足球', ok: false, visual: '⚽' },
      ],
    },
  },
  // … anthem / map / festival / boss×3
];
```

**Boss：** `questions: [q1,q2,q3]` 連續；清晒先開 cert。

**身份卡：** 6–8 格（色／動物），`playerName` 花名；可跳過→「同學」。

---

## 7. 檔案清單

| 檔 | 用途 |
|----|------|
| `index.html` | **唯一可玩交付**（CSS+JS 全內嵌） |
| `PLAN.md` | 本計劃（FIRM） |
| `README.md` | 短 QA：鍵位、Pages、老師角 |
| `.nojekyll` | Pages |
| （可選）`tests/smoke.js` | regex／結構 smoke（後期） |

**Repo 建議：** `ihateusingai-beep/national-island`（或 `little-national-island`）· Pages `/`  
**本地：** `value_education/national-island/`

---

## 8. 實作任務（bite-size · 「順序」執行）

### Task 1 — Scaffold 單檔殼
- Create: `index.html`（viewport、SEN CSS 硬規、screen 切換：home／id／map／quiz／cert）
- 空 state：`screen`, `player`, `stats`, `cleared[]`

### Task 2 — 身份卡
- 6–8 圖選；寫入 `player.emoji` + `player.name`
- 跳過 OK

### Task 3 — 地圖 grid + 玩家移動
- 8×6 瓦片；牆；角色格移動
- keydown ↑↓←→ + VK 同 handler
- 撞牆停

### Task 4 — Spot 碰撞 → Quiz modal
- 未清 spot 觸發；已清只短「完成」toast
- 選項 shuffle 後 **重標 A/B/C/D**
- 啱／錯 feedback；再答回滾分

### Task 5 — 內容填 4 spot + Boss 3 題
- 國旗、禮儀、地圖、節日；干擾項明顯錯
- difficulty 控選項數（slice）

### Task 6 — 計分 + 金銀銅證
- `correct/answered`；門檻金銀銅
- 證書大字 + 身份；再玩 reset map 進度

### Task 7 — TTS + SFX 輕量
- `speechSynthesis` zh-HK 手動 🔊
- 可選 WebAudio beep 啱／錯（無外部 mp3 亦可 MVP）

### Task 8 — 老師角
- 長按標題或 Start：難度 2/3/4 選、顯示 VK、跳 Boss、reset
- 學生面零「SEN」標籤

### Task 9 — 驗收
```bash
# 抽最大 inline script
node --check /tmp/ni_main.js
# 人手：PC 鍵行圖→4 spot→Boss→證；iPad VK 點 A
```

### Task 10 — README + Push（Ken 講 Push 先）
- README 鍵位／課堂 5 分流程
- `git init`（若未）→ commit → gh repo → Pages

---

## 9. UI／視覺

| 項 | 鎖 |
|----|-----|
| 風格 | Retro 暖色小島；8-bit 感用 CSS 圓角色塊即可（唔強制 pixel font） |
| 主色 | 綠島 + 紅金點綴（國慶感，唔刺眼） |
| 選項 | 大卡 + 字母大圓（ebook-pledge 同規） |
| 圖 | MVP emoji／純 CSS／inline SVG；v1.1 可 Ghibli 生成圖 |
| 動畫 | 短 pulse／綠框；`prefers-reduced-motion` 關 |

---

## 10. 驗收標準（Definition of Done）

- [ ] 單檔雙擊／Pages 可開，無 build
- [ ] PC：方向鍵行、A 確認、B 返／提示
- [ ] iPad：VK 可完整打完一局（唔靠系統鍵盤）
- [ ] 4 spot + Boss 3 題內容齊、可重試
- [ ] 答錯有指位；無生命死亡
- [ ] A/B/C/D 大圓跟渲染序
- [ ] 身份卡可跳過；證書有名＋金銀銅
- [ ] 手動 🔊；預設唔自動朗讀
- [ ] SEN HTML 硬規齊
- [ ] `node --check` 主 script 過

---

## 11. 風險與備援

| 風險 | 備援 |
|------|------|
| 地圖碰撞耗時 | 改「關卡清單」線性 6 屏（仍 A/B 鍵） |
| 國旗 emoji 顯示唔一 | inline SVG 簡旗 |
| iPad 多指誤觸 | VK `touch-action` + 單點閘 |
| 內容敏感／過深 | 只認圖＋禮貌行為；老師可關 festival／boss 難題 |
| 寫太長未完成 | 先 2 spot + cert 可玩再加 |

---

## 12. 執行開關

| 指令 | 動作 |
|------|------|
| **順序** | 按 Task 1→10 一次做完可玩 MVP（Push 仍等口令，除非再講 push） |
| **寫** / **做** | 同順序（寫 index） |
| **1** | 只 Task 1 scaffold |
| **NT-D** | 對本計劃審計缺口 |
| **Push** | commit + Pages |
| **改主題名** | 改 `CONFIG.title`／README |

---

## 13. 進度（實時）

| 項 | 狀態 |
|----|------|
| 構思 A | ✅ |
| FIRM PLAN.md | ✅（本檔） |
| 資料夾 | ✅ |
| index.html | ✅ 可玩 MVP（node --check OK） |
| README | ✅ |
| Live Pages | ⏳ 等 `Push` |

**下一步：** 本地開 `index.html` 試玩；要上線講 **Push**。

---

## 14. 參考

- 構思 session：2026-10-04 國民教育 keyboard/retro  
- 同校 SEN 單檔：`ebook-pledge-game`、`direction-coding`、`mouse-master-hub`  
- Skill：`education-game-dev`（SEN 硬規、手勢 A/B/C、身份卡、金銀銅）  
- 記憶：中度 2 選 1／中 3／高 4；干擾項只明顯錯；禁右鍵  
