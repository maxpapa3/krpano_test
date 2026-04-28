# krpano Gaussian Splat Viewer (VR対応)

krpano sample（https://krpano.com/examples/?splats）を参考に、既存 `.ply` ファイルを使って表示する最小構成のビューワーです。
PICO G3 / Meta Quest のヘッドセットブラウザからアクセスし、VRモードで閲覧できます。

## 1) 事前準備

1. krpano配布物を `krpano/` フォルダに配置
   - `krpano.js`
   - `krpano.swf`（任意/互換用）
   - `plugins/webvr.xml`
   - `plugins/controls3d.xml`
   - `plugins/cursor3d.xml`
   - `plugins/wakelock.xml`
2. 既存のGaussian Splat PLYを `models/scene.ply` として配置
   - 別名を使う場合は `tours/splats.xml` の `<splats url="..." />` を変更

## 2) 実行

任意の静的サーバーで配信してください（ローカル `file://` はWebXRで制約あり）。

例:

```bash
python3 -m http.server 8080
```

ブラウザで `http://localhost:8080` を開くと表示されます。

## 3) VR利用（PICO G3 / Meta Quest）

1. ヘッドセットのブラウザで同じURLへアクセス
2. 画面上のVRボタンからVRモードへ入る
3. コントローラーで移動・注視

## ファイル構成

- `index.html`: 埋め込みエントリポイント
- `tours/splats.xml`: splat表示とVR設定
- `models/`: 既存PLY配置場所
