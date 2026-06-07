# Claude Code 運用メモ

## effort レベルの使い分け
- low / medium: 軽微な編集は Sonnet
- high（デフォルト）: 標準的な実装
- xhigh / max: 複雑なアーキテクチャや高難度ロジック（ベイズ計算など）は Opus

## コンテキスト管理
- `/compact` はコンテキスト 50% 付近で実行
- タスクの切れ目で `/clear`
- トークン節約のため Agent Teams は避ける

## ultrathink
- 深い思考が要るときは `ultrathink` キーワードを使う

## 参考リポジトリ
- shanraisshan/claude-code-best-practice （69 tips, Command→Agent→Skill）
- VoltAgent/awesome-agent-skills （`npx skills` で管理）
