# fablish

Claude Code 向けのコントラクトドリブンな長期タスク実行プロトコル。
Claude Fable 5 の公開されている仕事の進め方を手続きとして模倣したスキルです。

[English README](./README.md)

## 概要

多数のツール呼び出しにまたがるタスク（機能実装、リファクタリング、
マイグレーション、調査、複数ファイル変更）に対して、エージェントに
次の手順を踏ませます。

1. そもそも変更が依頼されたのかを最初に判定する。問題の記述に対する
   成果物は修正ではなく評価である
2. 変更系のツール呼び出しの前にタスクコントラクト（GOAL / CONSTRAINTS /
   DONE-CRITERIA / CHECK / NON-GOALS / RISKIEST-ASSUMPTION）を書く
3. 作業を独立性で分解して並行ワークストリームを委譲し（TeamCreate が
   ない環境では並行 Agent 呼び出しの一括起動にフォールバック）、計画を
   無効化しうる最大の仮定を、検証が安いうちに最初に確かめる
4. エビデンス付きのチェックポイントを記録し、中断からの再開を可能にする。
   各主張には VERIFIED（実行して確認）/ REASONED（コードから推論）/
   ASSUMED（未確認）のラベルを付け、根拠なしの昇格を禁じる。長期実行では
   一定間隔の中間検証で失敗を安いうちに表面化させる
5. DONE-CRITERIA と成果物だけを渡したフレッシュコンテキストの
   レビュアーで完了を検証する（作業中の経緯は渡さない）。徹底監査級の
   依頼では複数の独立レビュアーによる多数決へ強度を引き上げる
6. 結論を先頭に、基準ごとのエビデンスを添えて報告する
7. 教訓はキュレーション付きでコミットされる `.claude/lessons/` に保持し
   （重複より更新、誤りは削除）、普遍的でプロトコルレベルの知見は
   ユーザーの明示的な確認を得たうえで ktrysmt/fablish への GitHub issue
   として上流に還元する

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

スキルは状態をセッション単位の揮発ファイル
`/tmp/fablish/$CLAUDE_CODE_SESSION_ID/state.md` に保持します
（現在のタスクコントラクトとチェックポイント）。セッションIDで分離する
ため、同一リポジトリで並行するセッション同士が衝突せず、ワーキング
ツリーにも一切入りません。したがって `.gitignore` や
`.git/info/exclude` の登録は不要で、作業用の状態を誤ってコミットする
余地もありません。アプリ再起動をまたぐ再開は意図的に対象外で、
永続化したい記憶は別の場所に置きます。タスク横断の教訓は別途
`.claude/lessons/` に置きます。通常の変更と同様に
コミットされレビューされ、書き込みのたびにキュレーションされます
（重複より更新、誤った教訓は削除）。レビューを経ない永続メモリは陳腐化
して以後の実行にバイアスを与えるためです。普遍的でプロトコルレベルの
知見はユーザーの明示的な確認を経てこのリポジトリへの issue として外に
出し、プロジェクト固有の慣習はそのプロジェクトの CLAUDE.md や知識ベース
へ送ります。

## 補足

- スキルはグローバル CLAUDE.md にある前提ルール（grounded claims、
  act on sufficiency、no promissory endings、minor-choice autonomy、
  および並行作業での TeamCreate 優先ルール）を参照します。これらが
  なくても縮退しつつ動作します。TeamCreate がセッションのツールセットに
  ない場合、委譲は並行 Agent 呼び出しの一括起動にフォールバックします。
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
  型の最終報告（Phase 4）、実行から得た教訓のキュレーション付きメモリ
  構築（`.claude/lessons/`）の各根拠。issue による上流化は本スキル独自
  の機構であり、文書化された Fable の挙動ではない。

Anthropic による長期エージェント工学に関する文書:

- [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
  — コンパクション、構造化ノートテイキング、サブエージェント構成という
  コンテキストウィンドウを超えて働くための技法。`state.md` の
  チェックポイント設計の根拠。
- [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
  — エージェントは進捗ノートを読み、状態を再検証してから作業を再開する
  というハーネス設計。state.md からの再開ルールの根拠。
- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
  — orchestrator-workers と evaluator-optimizer パターン（Phase 1 の
  委譲、Phase 3 の検証・修正ループ）。「可能な限り単純な解を探し、
  必要な場合にのみ複雑さを足す」はエスケープハッチの根拠。
- [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
  — 「各サブエージェントには目的、出力形式、使うツールとソースの指針、
  明確なタスク境界が必要」。委譲プロンプトテンプレート
  （CONTEXT / SCOPE / BOUNDARIES / TOOLS / RETURN）の根拠。並行サブエージェント
  が幅優先に分解できる作業で優位になる理由も。
- [Claude Code best practices](https://code.claude.com/docs/en/best-practices)
  — エージェントには実行可能なチェックを与え、成功を主張させるのでは
  なくエビデンスを示させる（DONE-CRITERIA とチェックポイントの根拠）。
  フレッシュなサブエージェントによる敵対的レビューは「diff と与えられた
  基準だけを見て、変更を生んだ推論は見ない」（Phase 3 の根拠）。
  「diff を一文で説明できるなら計画は省け」（エスケープハッチの根拠）。

裏付けとなる研究:

- Huang et al., [Large Language Models Cannot Self-Correct Reasoning Yet](https://arxiv.org/abs/2310.01798)
  (ICLR 2024) — LLM は外部フィードバックなしには自身の推論を確実に
  修正できない。自己批評ではなく外部のフレッシュコンテキストレビュアー
  に検証を委ねる設計の前提。

後続の改訂の比較対象にしたソース:

- [toffyui/ccteams](https://github.com/toffyui/ccteams)
  — 三段階エビデンスラベル（VERIFIED / REASONED / ASSUMED）と最リスク
  仮定の先行検証の出典。v0.2.0 で採用。
- [Claude Code memory](https://code.claude.com/docs/en/memory.md) と
  [memory tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool.md)
  — v0.3.0 で「ローカルのタスク横断メモリプールを持たない」と判断した
  際の調査対象。v0.4.0 で再検討し、prompting ガイドのメモリシステム
  推奨に沿って、キュレーション付きでコミットされる `.claude/lessons/`
  としてローカルストアを復活させた（レビューを通すことで陳腐化を防ぐ）。

## ライセンス

MIT
