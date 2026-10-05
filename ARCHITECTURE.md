# 《小小國情島》Architecture

**Rev:** 1.1 · 2026-10-05  
**Path:** `~/workspace/vs code/education/value_education/national-island/`  
**Live:** https://ihateusingai-beep.github.io/national-island/  
**Repo:** `ihateusingai-beep/national-island`  
**Deliverable:** 單檔 `index.html`（HTML+CSS+JS）· 無 build · Pages `main` root + `.nojekyll`

---

## 1. 一句話

中度 SEN **top-down 小島探索** + **圖選 MC**：身份 → 地圖行 → spot 答題 → Boss 3 題 → 金銀銅證。PC 方向鍵/A/B 同 iPad 自製 VK 同一 handler。

---

## 2. 畫面 State Machine

```
home ──開始──► id ──去小島／跳過──► map ⇄ quiz(modal)
                                      │
                                      ├ spot 清晒 → boss 可開
                                      └ boss 3 題完 ──► cert ──再玩──► map(reset)
```

| Screen | DOM | 職責 |
|--------|-----|------|
| `home` | `#sc-home` | 開始；長按標題→老師角 |
| `id` | `#sc-id` | 身份卡 8 格／跳過 |
| `map` | `#sc-map` | 8×6 grid · 玩家 · VK · HUD |
| `quiz` | `#quiz-modal` | 疊加答題（唔換 screen 名，`S.quiz` 非 null） |
| `cert` | `#sc-cert` | 完局證書 |
| teacher | `#teacher` | 難度 2/3/4 · VK · 跳 Boss · 重設 |

`showScreen(name)` 只切 `home|id|map|cert`。Quiz 用 modal class `.on`。

---

## 3. 核心資料

### 3.1 Map

```
TILES[y][x]: 0=path · 1=wall(裝飾) · 2=sea
MAP_W=8, MAP_H=6
```

**硬規：** 所有 `SPOTS` + `BOSS` 座標必須 `TILES==0`；boot 跑 `assertSpotsWalkable` + BFS `reachableFrom(start)`。

**現況佈局（v1.1）：**
- 內陸幾乎全 path；中央兩格 wall 只做裝飾
- Spots 四角：`(1,1)(6,1)(1,4)(6,4)`
- Boss：`(3,3)` · 起點：`(4,1)`

**坑（已修）：** spot 放喺 wall → `canWalk` false →「黃點行唔到／左右壞」。`canWalk` 對 spot/boss **永遠 true** 作保險。

### 3.2 Spots / Boss

```js
SPOTS[] = { id, x, y, emoji, title, visual, prompt, tts, options[{label,visual,ok}] }
BOSS = { id:'boss', x, y, questions: [ same shape as spot q … ] ×3 }
```

- 選項數由 `S.nOpts`（2/3/4）`pickOptions` 裁切：必留全部 `ok`，再抽 wrong
- Shuffle 後 **重標 A/B/C/D**（渲染序 = 手勢序）
- 干擾項只「明顯錯」（足球／雪糕…），唔用其他正確行為做錯項

### 3.3 Runtime state `S`

| 欄 | 用途 |
|----|------|
| `screen` | home/id/map/cert |
| `player {name,em,x,y}` | 身份 + 格座標 |
| `cleared { [spotId]: true }` | 已過 spot |
| `answered / correct` | 計分（重答可 rollback） |
| `nOpts` | 2/3/4 |
| `showVk` | 虛擬掣 |
| `quiz` | null 或答題 session |
| `bossQi` / `bossOpen` | Boss 進度 |

---

## 4. 輸入層（統一）

**唯一入口：** `handleAct(act)` · `act ∈ up|down|left|right|a|b`

| 來源 | 綁定 |
|------|------|
| 實體鍵 | `keydown` Arrow* / A·Enter·Space / B·Esc·Backspace / 1–4 答題 |
| VK | `.k[data-act]` · `pointerdown` + `click`（70ms debounce 防雙觸） |
| 點選項 | choice `click` → `answer(i)` |

| 情境 | ↑↓←→ | A | B |
|------|------|---|---|
| map | `move` | `tryInteract` | hint toast |
| quiz 未答 | 游標 `selChoice` | `answer` | 讀題／提示 |
| quiz 已答 | — | `nextAfterQuiz` | 關（boss 禁中途走） |
| home/id/cert | — | 前進 | 返 |

**VK CSS：** `grid-template-areas` 明確 `up/left/right/down`（唔好用「全部 .k 先鎖同一格」）。

---

## 5. 玩法 Loop

```
map: move → 踩 spot
  → openSpotQuiz → answer
      啱: celebrate(ok) → 繼續 → cleared + celebrate(spot) + 地圖 just-clear
      錯: 標正確 + 再答（rollback 分）
  → 4 spot 清: bossOpen · toast 大門 · boss just-open
  → openBoss → 3 題（每題要啱先下一題）
  → showCert: fanfare + celebrate(win) + 金/銀/銅
```

**計分：** 每次 `answer` 首次 `scored` 先 +answered/+correct；`retryQuiz` 回滾 snapshot。

**通關門檻（cert）：** 金≥90% · 銀≥70% · 銅≥50% · 其餘「完成」。

---

## 6. 回饋系統（SEN 成功感）

| 時機 | 視覺 | 聲 | 文 |
|------|------|----|----|
| 答對 | `#fx-layer` burst+星+ring · choice pulse · modal 綠邊 | `beep(true)` 上升音列 | 好叻／太棒 + TTS |
| 清 spot | cell `.just-clear` · toast ok big | beep | 🎉 完成一個地點 |
| 開 boss | `.just-open` | — | 🚪 大門開咗 |
| 完局 | cert stamp/medal/stars/praise · win burst | `fanfare()` | 金銀銅讚語 + TTS |

`prefers-reduced-motion: reduce` → 關動畫，保留靜態結果。

---

## 7. SEN 硬規（學生面）

| 規 | 實作 |
|----|------|
| 禁右鍵 | `contextmenu` prevent |
| 禁選字／callout | CSS user-select / touch-callout |
| 無 `<input>` 彈系統鍵盤 | 身份用圖卡 |
| 答案 A/B/C/D 大圓 | 渲染序 letter |
| 無生命 | 可重試 |
| 手動 🔊 | 頂欄／題目；答對短鼓勵 TTS 例外可接受 |
| 學生 UI 零「SEN」字 | 老師角長按 |
| 單檔 Pages | `.nojekyll` |

---

## 8. 檔案

| 檔 | 角色 |
|----|------|
| `index.html` | **唯一 runtime** |
| `ARCHITECTURE.md` | 本檔（架構真相） |
| `PLAN.md` | 構思 FIRM / 任務史 |
| `README.md` | 鍵位 + 驗收一句 |
| `.nojekyll` | Pages |

**Skill：** `education-game-dev` → `references/national-island-topdown-sen-mvp.md`

---

## 9. 擴展點（日後）

| 想加 | 落點 |
|------|------|
| 新地點 | `SPOTS` + 座標 path 檢查 |
| 新 Boss 題 | `BOSS.questions` |
| 真圖 | option/spot `visual` 改 `<img>` 或 background |
| Canvas 地圖 | 換 `renderMap`；保留 grid 座標 API |
| 多島／關卡包 | `S.world` + spots 陣列切換 |
| 老師 PIN | `#teacher` 加閘（而家長按即開） |

**Non-goals：** 帳號、排行、生命、系統鍵盤、Vite build、政治長文。

---

## 10. 驗收／回歸

```bash
# parse
python3 -c "import re,pathlib; h=pathlib.Path('index.html').read_text(); s=max(re.findall(r'<script(?![^>]*src)[^>]*>([\s\S]*?)</script>',h),key=len); open('/tmp/ni.js','w').write(s)"
node --check /tmp/ni.js

# 邏輯（browser console）
NI.startGame();
// 所有 spot + boss 在 NI.reachableFrom(4,1)
// 左右：NI.move(-1,0); NI.move(1,0)
```

| 手動 | 期望 |
|------|------|
| 四角黃點 | 全部行到、A 開題 |
| 左右 VK／鍵 | 一格一格行 |
| 答對 | 星爆 + 音 + 綠 |
| Boss 完 | 證書 fanfare |
| Live | 200 + `小小國情島` + `celebrate` |

---

## 11. 已踩坑

1. **Spot on wall** → 行唔到；BFS boot 必做  
2. **D-pad CSS 全 `.k` 鎖同一 grid 格** → 左右「壞」；改 `grid-area`  
3. **撞牆 toast spam** → 以為掣壞；改靜音  
4. **成功感太弱** → 中度要 burst + fanfare + 讚語  
5. **寫 HTML 卡死** → 一次寫完整單檔；plan 先 FIRM  
