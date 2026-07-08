# fablish

Claude Code 向けのコントラクトドリブンな長期タスク実行プロトコル。
Claude Fable 5 の公開されている仕事の進め方を手続きとして模倣したスキルです。

[English README](./README.md)

## 概要

多数のツール呼び出しにまたがるタスク（機能実装、リファクタリング、
マイグレーション、調査、複数ファイル変更）に対して、エージェントに
次の手順を踏ませます。

1. 変更系のツール呼び出しの前にタスクコントラクト
   （GOAL / CONSTRAINTS / DONE-CRITERIA / NON-GOALS）を書く
2. 作業を独立性で分解し、並行可能なワークストリームを委譲する
3. エビデンス付きのチェックポイントを記録し、中断からの再開を可能にする
4. DONE-CRITERIA と成果物だけを渡したフレッシュコンテキストの
   レビュアーで完了を検証する（作業中の経緯は渡さない）
5. 結論を先頭に、基準ごとのエビデンスを添えて報告する
6. タスク横断の教訓を `.task/lessons/` に永続化する

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
（`state.md` に現在のタスクコントラクトとチェックポイント、`lessons/`
にタスク横断の教訓）。`.git/info/exclude` に登録されるため、コミット
されることはありません。

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
