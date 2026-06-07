# Netlify デプロイメモ

## Functions
- `netlify/functions/*.js` が自動でエンドポイント化される
- 呼び出しパス: `/.netlify/functions/<ファイル名>`
- 環境変数は Site settings → Environment variables で設定
- ローカル確認は `netlify dev`（`npm i -g netlify-cli`）

## 単一HTMLアプリの配布
- リポジトリ直下に index.html を置けばそのまま公開される
- 再デプロイは git push で自動

## 環境変数の例
- ANTHROPIC_API_KEY : Claude API キー（ブラウザに出さない）
