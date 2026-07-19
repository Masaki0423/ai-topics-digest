# ai-topics-digest

週次AIトピックのSlack投稿パイプライン。「頭脳（生成）」と「配送（投稿）」を分離し、
webhook URL の存在場所を GitHub Actions Secrets の1箇所に限定する構成。

```
[Anthropicクラウド routine（週1）]          [このリポジトリ]
 トレンド収集 → digest整形                  on: push (digest/**, main)
 → digest/YYYY-MM-DD.md を push  ────→    → curl で Secrets の webhook に投稿
 ※秘密情報ゼロ                             ※ SLACK_WEBHOOK_URL は Secrets のみ
```

- 生成手順は [PIPELINE.md](PIPELINE.md)（クラウドroutineがこれを読んで実行する）
- 配送は [.github/workflows/post-digest.yml](.github/workflows/post-digest.yml)
- `digest/` は投稿アーカイブを兼ねる

## セットアップ

1. Slack Workflow Builder で Webhook トリガーのワークフローを作成（変数 `text`・送信先チャンネル固定）
2. このリポジトリの Settings → Secrets and variables → Actions → New repository secret で
   `SLACK_WEBHOOK_URL` を登録（`https://hooks.slack.com/triggers/…`）
3. claude.ai でこのリポジトリを接続し、週次routineを作成（プロンプト例:
   「このリポジトリの PIPELINE.md に従って今週のdigestを生成し、mainにpushして」）

## セキュリティ運用

- webhook URL は認証を持たない capability URL。**URLの秘匿＝認証**
- このリポジトリ・コミット・ログ・routineプロンプトに URL を書かない
- 漏洩を疑ったら Slack 側でワークフローのトリガーを作り直す（旧URLは即失効）→ Secrets を更新
- 漏洩時の影響範囲は「対象チャンネルへのテキスト投稿」のみ（読み取り・他チャンネル・なりすまし不可）

## 手動実行（フォールバック）

ローカルの ai-community プロジェクトの `/trending-ai-topics` スキルでも同じ投稿ができる
（`--clipboard` で手貼りも可）。routineが落ちた週はそちらで代替する。
