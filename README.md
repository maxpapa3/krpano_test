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

## 4) `Yowza, that’s a big file` エラーが出る場合

GitHubの**ブラウザアップロード**は 25MB 制限があるため、PLY追加時にこのエラーが出ます。

対処方法は次のいずれかです。

### A. Git LFSを使う（推奨）

```bash
git lfs install
git lfs track "models/*.ply"
git add .gitattributes models/scene.ply
git commit -m "Add large PLY with Git LFS"
git push
```

このリポジトリには `models/*.ply` をLFS管理する `.gitattributes` を追加済みです。

### B. Git CLIで直接pushする

- GitHubの通常Git pushは 100MB 未満なら受け付け可能です（ただし大きなバイナリはLFS推奨）。
- 25MB制限は主にブラウザアップロード時の制限です。

### C. PLYを外部ストレージに置く

- Cloudflare R2 / S3 / CDN 等に置いて配信し、`<splats url="..." />` を外部URLへ変更。
- リポジトリ肥大化を防げるので運用しやすいです。


## 5) Meta QuestでVRボタンが表示されない場合

最も多い原因は**HTTP配信**です。Meta Quest BrowserのWebXRは、原則として**HTTPS（またはlocalhost）**のセキュアコンテキストでのみ有効です。

チェック項目:

1. URLが `https://` になっているか（`http://` だと不可）
2. 証明書エラーが出ていないか
3. `krpano/plugins/webvr.xml` が正しく配置されているか
4. PCのローカルIPへHTTPアクセスしていないか（例: `http://192.168.x.x:8080` はNG）

回避策:

- テスト時はHTTPSサーバーを使う
- もしくは公開ホスティング（GitHub Pages / Cloudflare Pages / Netlify 等）のHTTPS URLで開く

## 6) PLYの位置・大きさが合わない場合（エディタはある？）

あります。用途別に次が使いやすいです。

- **CloudCompare**: 点群の移動/回転/スケール調整に強い（無料）
- **Blender**: 汎用3D編集。PLYを読み込んでTransform後に再出力
- **MeshLab**: 軽量な点群編集に便利

ただし、Gaussian Splat用PLYは属性が多く、編集ツールによっては属性が欠落することがあります。
まずは**PLYを直接編集せず**、`tours/splats.xml` の `<splats>` パラメータで合わせるのが安全です。

```xml
<splats
  url="../models/scene.ply"
  scale="1.0"
  rx="0.0" ry="0.0" rz="0.0"
  tx="0.0" ty="0.0" tz="0.0"
/>
```

調整のコツ:

1. まず `scale` を合わせる
2. 次に `ry`（水平回転）を合わせる
3. 最後に `tx/ty/tz` で位置を合わせる

krpano公式ドキュメントでも `scale / rx,ry,rz / tx,ty,tz` が `image.splats` の標準属性として定義されています。

## 7) 最高品質で表示したい場合

このプロジェクトでは、VR時の画質優先設定として以下を反映しました。

- `webvr_foveationlevel="0"`（固定フォビエーション無効 = 周辺画質を落とさない）
- `webvr_highrefreshrate="false"`（高リフレッシュ無効 = 画質側にGPU予算を寄せる）
- `<splats antialias="0.0" />`（シャープ寄り設定）

設定場所: `tours/splats.xml`

> 注意: これらはGPU負荷を上げるため、端末によってはフレームレートが下がる場合があります。
> 画質と滑らかさのバランスが必要な場合は `webvr_foveationlevel="1"` や `antialias="0.1~0.3"` を試してください。

また、最終画質は元の学習データ品質にも強く依存します。
表示だけで改善できない場合は、入力画像の解像度/枚数/学習設定の見直しも有効です。

## ファイル構成

- `index.html`: 埋め込みエントリポイント
- `tours/splats.xml`: splat表示とVR設定
- `models/`: 既存PLY配置場所
