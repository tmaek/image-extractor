# Image Extractor

WebページのURLを入力して、ページ内の画像を一括抽出・ダウンロードできる Streamlit アプリ。

## 機能

### 画像抽出
- URLを入力して「抽出する」を実行するとページ内の画像を最大500枚検出
- 以下の方法で画像URLを収集する
  - `<img>` タグの `src` / `data-src` / `data-lazy-src` / `data-original` 属性
  - `<img>` 以外の要素の `data-src` 属性（遅延読み込みdivなど）
  - インラインスタイルの `background-image: url(...)`
- 画像URLからクエリパラメータを除去してオリジナル解像度のURLを取得
- 通常モードで0件の場合、自動でJSレンダリングモードにフォールバック

### JSレンダリングモード（Playwright）
- トグルで切り替え可能
- Chromium ヘッドレスブラウザでページをレンダリングし、SPAや動的コンテンツにも対応
- 通常より5〜10秒かかる

### 選択・リネーム・ダウンロード
- 画像一覧でチェックボックスにより個別選択。「すべて選択」「選択解除」ボタンあり
- ファイル名をクリックするとリネームパネルが開き、ダウンロード時のファイル名を変更できる
- 1枚選択時: 直接ダウンロード
- 複数選択時: ZIPファイルにまとめてダウンロード。ZIPファイル名も変更可能

## 技術スタック

| 用途 | ライブラリ |
|---|---|
| UIフレームワーク | Streamlit |
| HTML解析 | BeautifulSoup4 |
| JSレンダリング | Playwright（Chromium） |
| 画像処理 | Pillow |
| HTTPクライアント | requests |

## セキュリティ

- **SSRF対策**: プライベートIPアドレス・ループバックへのアクセスをブロック。DNSリバインディング攻撃対策としてリダイレクト先も都度検証
- **バイト上限**: ページHTML 20MB、サムネイル 2MB でダウンロードを打ち切り
- **ファイル名サニタイズ**: パストラバーサル・制御文字を除去
- **XSS対策**: URLから生成する表示テキストはHTMLエスケープ処理

## ローカルでの実行

```bash
pip install -r requirements.txt
python -m playwright install chromium
streamlit run app.py
```

## Streamlit Cloud へのデプロイ

`packages.txt` に Playwright が必要とするシステムライブラリを記載済み。リポジトリをそのまま Streamlit Cloud に接続してデプロイ可能。

## 制限事項

- 1ページあたりの画像抽出上限: 500枚
- ページHTML取得上限: 20MB
- 画像ダウンロード上限: 1枚あたり 20MB
- `data:` URI の画像（base64埋め込み）は対象外
