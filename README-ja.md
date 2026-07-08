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

## 参考文献

fablish は公開ドキュメントに記述された挙動の手続き的な転写です。設計と
改訂の根拠にしたソースを挙げます。

Anthropic による Fable 5 の仕事の進め方に関する文書:

- [Introducing Claude Fable 5 and Claude Mythos 5](https://www.anthropic.com/news/claude-fable-5-mythos-5)
  — 長期タスクの自律性。「自分自身のノートを使って出力を改善する」、
  Slay the Spire の評価では永続的なファイルベースのメモリによる性能向上
  幅が Opus 4.8 の3倍だったとの記述。
- [Claude Fable 5 & Claude Mythos 5 System Card](https://anthropic.com/claude-fable-5-mythos-5-system-card)
  — モデルの挙動と評価に関する一次ドキュメント。
- [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5)
  — 最も設計根拠として重いソース。進捗主張をツール結果と突き合わせて
  監査する（Phase 2 のチェックポイント）、「独立したフレッシュコンテキ
  ストの検証サブエージェントは自己批評を上回る傾向がある」（Phase 3）、
  非同期監督つきの並行サブエージェント（Phase 1）、十分な情報が揃った
  ら行動し約束でターンを終えない、作業を見ていない読者に向けた再接地
  型の最終報告（Phase 4）、実行から得た教訓の記録（Lessons の経路設計）
  の各根拠。

Anthropic による長期エージェント工学に関する文書:

- [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
  — コンパクション、構造化ノートテイキング、サブエージェント構成という
  コンテキストウィンドウを超えて働くための技法。`.task/state.md` の
  チェックポイント設計の根拠。
- [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
  — エージェントは進捗ノートを読み、状態を再検証してから作業を再開する
  というハーネス設計。state.md からの再開ルールの根拠。

後続の改訂の比較対象にしたソース:

- [toffyui/ccteams](https://github.com/toffyui/ccteams) と付随する
  [解説記事](https://zenn.dev/yui/articles/e4f8268ab5c6c1)
  — 三段階エビデンスラベル（VERIFIED / REASONED / ASSUMED）と最リスク
  仮定の先行検証の出典。v0.2.0 で採用。
- [Claude Code memory](https://code.claude.com/docs/en/memory.md) と
  [memory tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool.md)
  — v0.3.0 で「ローカルのタスク横断メモリプールを持たない（無審査の
  記憶は陳腐化しバイアスになる）」と判断した際の調査対象。恒久的な
  知見はレビューを通る経路（issue 起票等）に流す設計の背景。

## ライセンス

MIT
