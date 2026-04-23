# Hello PWA

最小構成の Progressive Web App (Hello World) サンプル。

## 構成

- `index.html` — 画面本体 + Service Worker 登録
- `manifest.json` — Web App Manifest
- `sw.js` — Service Worker（オフラインキャッシュ）
- `icon.svg` — アイコン

## 実行

任意の静的サーバーで配信してください（Service Worker は `file://` では動きません）。

```sh
python3 -m http.server 8000
# → http://localhost:8000 をブラウザで開く
```
