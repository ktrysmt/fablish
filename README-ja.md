# fablish

Claude Code 向けのコントラクトドリブンな長期タスク実行プロトコル。
Claude Fable 5 の公開されている仕事の進め方を手続きとして模倣したスキルです。

[English README](./README.md)

## 概要

多数のツール呼び出しにまたがるタスク（機能実装、リファクタリング、
マイグレーション、調査、複数ファイル変更）に対して、エージェントに
次の手順を踏ませます。

1. 変更系のツール呼び出しの前にタスクコントラクト
   （GOAL / CONSTRAINTS / DONE-CRITERIA / NON-GOALS / RISKIEST-ASSUMPTION）を書く
2. 作業を独立性で分解して並行ワークストリームを委譲し、計画を無効化しうる
   最大の仮定を、検証が安いうちに最初に確かめる
3. エビデンス付きのチェックポイントを記録し、中断からの再開を可能にする。
   各主張には VERIFIED（実行して確認）/ REASONED（コードから推論）/
   ASSUMED（未確認）のラベルを付け、根拠なしの昇格を禁じる
4. DONE-CRITERIA と成果物だけを渡したフレッシュコンテキストの
   レビュアーで完了を検証する（作業中の経緯は渡さない）
5. 結論を先頭に、基準ごとのエビデンスを添えて報告する
6. 普遍的でプロトコルレベルの知見は、ローカルに溜め込まず ktrysmt/fablish
   への GitHub issue として上流に還元する（1 issue 1知見、リポジトリの
   issueテンプレートに準拠）

単一ファイル編集や単純な調べ物にはエスケープハッチが定義されており、
不要な儀式は追加されません。

## インストール

```console
claude plugin marketplace add ktrysmt/claude-plugins
claude plugin install fablish@ktrysmt
```

## 使い方

明示的に呼び出す場合:

```
/fablish <タスクの説明>
```

複数ステップの作業では自動トリガーもされます。すべての長期タスクで
既定にしたい場合は、グローバルの `~/.claude/CLAUDE.md` に次のような
ルールを追加してください。

```markdown
- Long-horizon tasks: For multi-step work expected to span many tool
  calls, invoke the `fablish` skill before starting
```

## 作業ファイル

スキルはプロジェクトルートの `.task/` ディレクトリに状態を保持します
（`state.md` に現在のタスクコントラクトとチェックポイント）。
`.git/info/exclude` に登録されるため、コミットされることはありません。
状態は意図的に単一タスクに閉じています。レビューを経ない永続メモリは
陳腐化して以後の実行にバイアスを与えるため、タスク横断のメモリプールは
持ちません。恒久的な知見はレビューを通して外に出します（普遍的なものは
このリポジトリへのissue、プロジェクト固有のものはそのプロジェクトの
CLAUDE.mdや知識ベースへ）。

## 補足

- スキルはグローバル CLAUDE.md にある前提ルール（grounded claims、
  act on sufficiency、no promissory endings、minor-choice autonomy、
  および並行作業での TeamCreate 優先ルール）を参照します。これらが
  なくても動作しますが、揃っている環境で最も効果を発揮します。
- Fable 系モデルでは各フェーズはリマインダーとして機能し、それ以外の
  モデル（Opus、Sonnet、Haiku）では各フェーズのゲートを明示的に
  順番どおりに実行します。

## ライセンス

MIT
