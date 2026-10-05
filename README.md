# 小小國情島

中度班國民教育 top-down 小遊戲（單檔 HTML）。

## 開玩

- 本地：雙擊 `index.html` 或用瀏覽器開
- 鍵：`↑↓←→` 行 · `A`/`Enter` 確認 · `B`/`Esc` 提示／返 · 答題可用 `1–4`
- iPad：底部虛擬十字 + A/B（老師可關）

## 流程

身份卡 → 地圖 4 地點答題 → 大門 Boss 3 題 → 金／銀／銅證

## 老師

長按標題／地圖標題（或雙撃地圖標題）→ 選項數 2/3/4、VK 開關、跳 Boss、重設

## 驗

```bash
# 抽主 script
python3 -c "import re,pathlib; h=pathlib.Path('index.html').read_text(); s=max(re.findall(r'<script(?![^>]*src)[^>]*>([\s\S]*?)</script>',h),key=len); open('/tmp/ni.js','w').write(s)"
node --check /tmp/ni.js
```

## 計劃

見 `PLAN.md`
