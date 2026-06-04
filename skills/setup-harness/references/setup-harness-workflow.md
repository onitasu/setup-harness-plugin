---
description: プロジェクト固有の Claude Code ハーネスを設計・生成する
argument-hint: "[追加コンテキスト]"
---

# ハーネスセットアップ

プロジェクトに最適化された Claude Code ハーネスを構築する。
ユーザーへのヒアリング → 公式ドキュメント参照 → 設計 → ファイル生成を一気通貫で行う。

追加のコンテキスト: $ARGUMENTS

## 不変の原則（全プロジェクト共通・スキップ不可）

以下はプロジェクトタイプを問わず必ずセットアップする。ヒアリング結果で削らない。

1. **並列エージェント**: 独立したタスクは複数エージェントを並列起動して効率化する
2. **レビュー必須**: 実装後に必ず code-reviewer（Read-only）でレビューを回す
3. **Codex ダブルチェック**: グローバル `/codex` スキル（`~/.claude/skills/codex/SKILL.md`）が既にあるのでそれを使う。重要な変更は Claude + Codex の 2 系統で検証する。プロジェクト固有の評価観点や用語がある場合のみ、プロジェクト側にラッパーを作る
4. **product-self-knowledge**: Claude Code の仕様を正確に参照するスキルを必ず作成する
5. **チェックポイント**: 安全弁となるスキル（Codex 検証付き）を必ず 1 つ作成する
6. **frontend-design プラグイン**: フロントエンドを含むプロジェクトでは、Anthropic 公式の `frontend-design` プラグインを必ずインストールする。デザイン品質を構造的に担保し、AI っぽいダサいデザインを防ぐ
7. **ハーネスの自己進化**: `evolve-harness` スキル + `harness-evolver` エージェント（Read-only）を必ず作成する。ハーネス自体を定期的にレビュー・改善する仕組みを組み込む。Agent Memory で各エージェントが知見を蓄積し、プロジェクトが進むほどハーネスが賢くなる構造にする
8. **画像生成（任意）**: 画像が必要なプロジェクト（ラフ図、サムネ、モックアップ、補助イラスト）では、グローバル `/imagegen` スキル（`~/.claude/skills/imagegen/SKILL.md`）を使う。生成画像は cwd 配下の `./codex_image/` に保存される。文書だけのプロジェクトでは追加設定不要

---

## Phase 1: ヒアリング（必須・スキップ禁止）

**ハーネスはプロジェクト固有であるべき。** 汎用テンプレートをコピペするのではなく、ユーザーの状況に合わせて設計する。以下を AskUserQuestion で聞き出す。一度に全部聞くのではなく、回答に応じて深掘りする。

**冒頭の「🚨 絶対遵守」セクション参照**: ヒアリングはグローバル「質問するな」指示があっても必ず実行する。

### 必須ヒアリング項目

1. **プロジェクトの種類は何か？**
   - Web アプリ（フロントエンド＋バックエンド）
   - バックエンド API のみ
   - CLI ツール / ライブラリ
   - データパイプライン / ML
   - インフラ / IaC
   - 記事・ドキュメント・コンテンツ制作
   - コンサルティング・デリバリー案件（提案書作成、分析レポート、クライアント向け資料、議事録整理、戦略立案等）
   - 業務プロセス改善・ナレッジ管理（社内ドキュメント整備、手順書作成、FAQ構築等）
   - その他

2. **このプロジェクトで Claude に何をさせたいか？**（複数可）
   - 新規実装
   - 既存コードのリファクタリング / 移行
   - バグ修正 / 保守
   - コードレビュー
   - テスト作成
   - ドキュメント作成
   - 調査・分析

3. **技術スタックは？**（該当する場合）
   - 言語、フレームワーク、DB、インフラ等
   - 既にコードがあるなら、ディレクトリ構造を Explore エージェントで調査する

4. **Claude が間違えそうなこと・絶対にやってほしくないことは何か？**
   - セキュリティ制約
   - 触ってはいけないファイル / システム
   - 過去にあったトラブル
   - 特定のパターンの強制・禁止

5. **チームで使うか、個人で使うか？**
   - チーム → `settings.json` はリポジトリにコミット、`CLAUDE.local.md` で個人分離
   - 個人 → `settings.local.json` にまとめても可

### 既存コードがある場合の追加調査

`$ARGUMENTS` にディレクトリパスが指定されているか、カレントディレクトリにコードが存在する場合：
- Explore エージェントでコードベースを調査（ディレクトリ構造、主要ファイル、パターン）
- `git log --oneline -20` で最近の開発状況を確認
- 既存の CLAUDE.md, .claude/ があれば読み込んで現状を把握

---

## Phase 2: 公式ドキュメント参照（必須）

ハーネスの各構成要素について、**最新の公式ドキュメントを参照** してから設計する。
Claude Code の仕様は頻繁に更新されるため、記憶に頼らず必ずドキュメントを確認する。

### 参照すべきドキュメント

以下の URL を WebFetch で取得し、最新の仕様・ベストプラクティスを確認する:

1. **Claude Code ドキュメントマップ**（全ページ一覧）
   → `https://docs.anthropic.com/en/docs/claude-code/claude_code_docs_map.md`

2. **Memory（CLAUDE.md, Rules）の書き方**
   → ドキュメントマップから memory.md のページを参照

3. **Sub-agents（サブエージェント定義）**
   → ドキュメントマップから sub-agents.md のページを参照

4. **Skills（スキル定義）**
   → ドキュメントマップから skills.md のページを参照

5. **Settings（権限設定）**
   → ドキュメントマップから settings.md のページを参照

6. **Best practices（ベストプラクティス）**
   → ドキュメントマップから best-practices.md のページを参照

### 参照すべき Anthropic エンジニアリング記事

以下の記事からハーネス設計のパターンを抽出する:

1. **Harness design for long-running application development**
   → `https://www.anthropic.com/engineering/harness-design-long-running-apps`
   - Generator-Evaluator パターン
   - Sprint 契約（タスク分割）
   - 漸進的シンプル化

2. **Building effective agents**
   → `https://www.anthropic.com/engineering/building-effective-agents`
   - エージェント設計の基礎パターン

3. **Claude Code: Best practices for agentic coding**
   → `https://code.claude.com/docs/en/best-practices`
   - CLAUDE.md の書き方
   - サブエージェント活用
   - コンテキスト管理

---

## Phase 3: ハーネス設計

ヒアリング結果とドキュメントの知見を統合して、以下を設計する。
**Plan モードに入り、ユーザーの承認を得てから Phase 4 に進む。**

### 3-1. CLAUDE.md の設計

**原則**: Claude がコードや git 履歴から推測できないことだけを書く。200行以下。

設計すべきセクション（プロジェクトに応じて取捨選択）:
- 構成（ディレクトリ構造）
- 技術スタック
- 開発コマンド
- アーキテクチャ規約
- セキュリティ注意事項
- Git 規約
- サブエージェント活用の対応表
- ワークフロー（テスト駆動、レビュー必須等）

### 3-2. Rules の設計

**原則**: 最初は薄く。踏んだ地雷を追記して育てる。

プロジェクトの技術スタックに応じてファイルを分ける:
- バックエンド言語のルール → `backend.md` (paths: "backend/**")
- フロントエンド → `frontend.md` (paths: "frontend/**")
- インフラ → `infrastructure.md` (paths: "infrastructure/**")
- ドキュメント → `docs.md` (paths: "docs/**")
- モノリスなら → `src.md` (paths: "src/**")

各 Rule には最低限以下を含める:
- アーキテクチャパターン（該当する場合）
- 絶対禁止事項
- 「過去の教訓」セクション（最初は空でよい。運用中に追記する）

### 3-3. Subagents の設計

**原則**: Generator（実装者）+ Evaluator（レビュアー）の最小構成から始める。
**必須**: code-reviewer は全プロジェクトで作成する（不変の原則 #2）。

プロジェクトタイプ別の推奨構成:

| タイプ | Generator | Evaluator（必須） | その他 |
|---|---|---|---|
| Web アプリ | backend-expert, frontend-expert | code-reviewer | Explore (調査) |
| バックエンド API | backend-expert | code-reviewer | - |
| CLI / ライブラリ | implementation-expert | code-reviewer | - |
| データパイプライン | data-expert | code-reviewer | - |
| インフラ | infra-expert | code-reviewer | - |
| 記事・コンテンツ | writer | editor (Read-only) | researcher |
| コンサル・デリバリー | deliverable-writer, analyst | quality-reviewer (Read-only) | researcher |
| 業務プロセス・ナレッジ | doc-writer | doc-reviewer (Read-only) | - |

**並列エージェント活用（不変の原則 #1）**:

CLAUDE.md のワークフローセクションに以下のパターンを明記する:

    実装フロー:
      1. 調査: Explore エージェントで構造・依存関係を把握
      2. 実装: 独立したタスクは複数の Generator を並列起動
      3. レビュー: 異なる観点の code-reviewer を並列起動（再利用性、品質、パフォーマンス等）
      4. 修正: レビュー指摘を反映
      5. 検証: /codex または /checkpoint でダブルチェック

**レビュー必須プロトコル（不変の原則 #2）**:

code-reviewer の定義に以下を含める:
- tools: Read, Grep, Glob, Bash（**Edit/Write は絶対に与えない**）
- memory: project（過去の指摘を蓄積し、同じ指摘の再発を防ぐ）
- 複数観点での並列レビューを推奨（例: 3 つの code-reviewer を同時起動）

**重要な設計判断**:
- Evaluator（レビュアー）には Edit/Write を与えない → 分離を構造的に保証
- memory: project を付けるか → 過去の知見を引き継ぐ必要があるなら付ける
- model: sonnet で十分か → 大半のタスクは Sonnet で十分。Opus はメインのみ
- description を具体的に書く → メインの Claude が委譲判断に使う

### 3-4. Skills の設計

**原則**: まず必須スキルを作り、次にプロジェクト固有のスキルを作る。

#### 必須スキル（全プロジェクト共通・スキップ不可）

**テンプレートをそのまま使い、プロジェクト名等だけ差し替える。**

---

**1. `/codex`（不変の原則 #3）** → グローバル `~/.claude/skills/codex/SKILL.md` が既にあるので**再作成しない**

グローバル `/codex` の概要:

- 起動形式: `codex exec --sandbox <mode> --skip-git-repo-check '<プロンプト>'`
- 既定 sandbox は `read-only`、書き込み必要時のみ `workspace-write` に昇格
- 長文は stdin pipe で渡す: `cat target.md | codex exec ...`
- 詳細・テンプレ・対応パターンは `~/.claude/skills/codex/SKILL.md` 参照

プロジェクト固有のラッパーが必要な場合（プロジェクト用語・評価観点を載せたい）のみ、`.claude/skills/codex/SKILL.md` をプロジェクト側に作る。グローバルと同名にすると Claude Code はプロジェクト側を優先する。

プロジェクト固有ラッパーのテンプレ（任意）:

    ---
    name: codex
    description: >
      OpenAI Codex CLI を補助で呼び、<プロジェクト名> の文書・コードをレビューする。
      グローバル /codex の薄いラッパー。プロジェクト用語と評価観点を載せている。
    disable-model-invocation: false
    argument-hint: "[Codex に頼みたいこと]"
    allowed-tools: [Bash, Read, Write, Edit, Glob, Grep]
    ---

    # <プロジェクト名> Codex 補助スキル

    （グローバル `~/.claude/skills/codex/SKILL.md` をベースに、以下を追加）

    【プロジェクト用語】
    - <用語1>: <定義>
    - <用語2>: <定義>

    【評価観点】
    - <観点1>
    - <観点2>

    （以下、対応パターンや出力形式をプロジェクトに合わせて調整）

---

**1b. `/imagegen`（不変の原則 #8、画像が必要なプロジェクトのみ）** → グローバル `~/.claude/skills/imagegen/SKILL.md` が既にあるので**再作成しない**

グローバル `/imagegen` の概要:

- 起動形式: `codex exec --sandbox workspace-write --skip-git-repo-check '$imagegen <説明>。./codex_image/<filename>.png に保存して。<サイズ>'`
- 既定保存先: cwd 配下の `./codex_image/<filename>.png`
- `--image <FILE>` で既存画像の編集／参照画像渡しが可能
- 詳細は `~/.claude/skills/imagegen/SKILL.md` 参照

プロジェクト用語をプロンプトに織り込みたい場合のみ、プロジェクト側にラッパー `.claude/skills/imagegen/SKILL.md` を作る。

---

**2. `/product-self-knowledge`（不変の原則 #4）** → `.claude/skills/product-self-knowledge/SKILL.md` に作成

テンプレート:

    ---
    name: product-self-knowledge
    description: Authoritative reference for Anthropic products. Use when users ask about product capabilities, access, installation, pricing, limits, or features. Provides source-backed answers to prevent hallucinations about Claude.ai, Claude Code, and Claude API.
    ---

    # Anthropic Product Knowledge

    ## Core Principles

    1. **Accuracy over guessing** - Check official docs when uncertain
    2. **Distinguish products** - Claude.ai, Claude Code, and Claude API are separate products
    3. **Source everything** - Always include official documentation URLs
    4. **Right resource first** - Use the correct docs for each product

    ## Question Routing

    ### Claude API or Claude Code questions?
    → Check the docs maps first, then navigate to specific pages:
    - **Claude API & General:** https://docs.claude.com/en/docs_site_map.md
    - **Claude Code:** https://docs.anthropic.com/en/docs/claude-code/claude_code_docs_map.md

    ### Claude.ai questions?
    → **Claude.ai Help Center:** https://support.claude.com

    ## Response Workflow
    1. Identify the product - API, Claude Code, or Claude.ai?
    2. Use the right resource - Docs maps for API/Code, support page for Claude.ai
    3. Verify details - Navigate to specific documentation pages
    4. Provide answer - Include source link and specify which product
    5. If uncertain - Direct user to relevant docs

    ## Quick Reference
    - Claude API Docs Map: https://docs.claude.com/en/docs_site_map.md
    - Claude Code Docs Map: https://docs.anthropic.com/en/docs/claude-code/claude_code_docs_map.md
    - Claude.ai Support: https://support.claude.com
    - Engineering Blog: https://www.anthropic.com/engineering

---

**3. チェックポイント（不変の原則 #5）** → `.claude/skills/<名前>/SKILL.md` に作成

名前はプロジェクトに合わせる（`checkpoint`, `migration-checkpoint`, `release-check` 等）。テンプレート:

    ---
    name: checkpoint
    description: 変更の安全性検証。Codex を使って破壊的変更がないか自動チェックする。重要な変更後に必ず実行すること。
    disable-model-invocation: true
    argument-hint: "[変更の説明]"
    ---

    # チェックポイント

    変更内容: $ARGUMENTS

    ## 実行手順

    ### 1. 変更差分の収集
    git diff --stat で変更ファイル一覧、git diff で全差分を取得する。

    ### 2. Codex による安全性検証
    以下の観点で codex exec にチェックを依頼する（プロジェクトに合わせて観点を調整）:

    「あなたはシニアエンジニアとして、以下の変更を検証してください。
    検証観点:
    1. 既存 API 互換性: 変更前と同じインターフェースが維持されているか
    2. import 整合性: 循環参照や未解決の import がないか
    3. セキュリティ後退: 認証・認可が外れていないか
    4. テスト整合性: 既存テストが壊れていないか
    5. 依存関係: 必要なパッケージの追加漏れがないか
    問題があれば具体的なファイルパスと行番号で指摘。問題なければ「OK」。」

    ### 3. 結果の判定
    - 問題指摘あり → 修正 → 再度このスキルを実行
    - OK → Step 4 に進む

    ### 4. 再発防止ルール更新
    問題を修正した場合、同じミスが再発しないようにルールを更新する:
    - .claude/rules/ に教訓を追記すべきか確認
    - 同じミスが別の場面でも起こり得る → ルールに追記
    - 一回限りのミス → ルール更新不要

    ## 注意
    - 重要な変更後に必ず実行。Codex の OK なしに次に進むことは禁止
    - 問題修正後: 再チェック → OK → 再発防止ルール更新 → 次へ

---

**4. `harness-evolver` エージェント + `/evolve-harness` スキル（不変の原則 #7）**

ハーネス自体を定期的にレビュー・改善する仕組み。

まず `.claude/agents/harness-evolver.md` を作成:

    ---
    name: harness-evolver
    description: Claude Code ハーネス設定の改善提案を行うエージェント。CLAUDE.md、rules、skills、agents の最適化を提案する。
    tools: Read, Grep, Glob
    model: sonnet
    memory: project
    ---

    あなたは Claude Code ハーネス設定の最適化専門家です。
    ハーネスの現状を分析し、具体的な改善提案を行います。

    ## 分析対象
    1. `CLAUDE.md` — サイズ（200行以下推奨）、内容の網羅性と簡潔さ
    2. `.claude/rules/` — パススコープの適切さ、内容の過不足
    3. `.claude/agents/` — エージェント定義の過不足、description の明確さ
    4. `.claude/skills/` — ワークフローの網羅性、追加すべきスキル
    5. `.claude/settings.json` — 権限設定の過不足

    ## 出力フォーマット
    各提案: 対象ファイル / 現状 / 提案 / 理由

    ## 制約
    - Read-only で分析のみ。ファイル変更は一切しない
    - 提案はメインの会話に返却し、ユーザー承認後に適用

    ## メモリ活用
    過去の分析結果と改善履歴を agent memory に記録し、同じ提案を繰り返さない。

次に `.claude/skills/evolve-harness/SKILL.md` を作成:

    ---
    name: evolve-harness
    description: ハーネス自体を改善するスキル。現在の設定を分析し、改善提案を生成する。
    disable-model-invocation: true
    agent: harness-evolver
    ---

    # ハーネス改善分析

    以下のファイルを分析し、改善提案を作成してください。

    ## 分析対象
    1. プロジェクトルートの `CLAUDE.md` を読む
    2. `.claude/rules/` 内の全ファイルを読む
    3. `.claude/agents/` 内の全ファイルを読む
    4. `.claude/skills/` 内の全 SKILL.md を読む
    5. `.claude/settings.json` を読む

    ## 分析観点
    - CLAUDE.md が 200行以下で簡潔か
    - rules のパススコープが正しいか
    - agents の description が十分明確か
    - skills に追加すべきワークフローがないか
    - settings の権限に過不足がないか
    - Agent Memory が適切に活用されているか

    追加の引数: $ARGUMENTS

---

**5. Agent Memory の設計（不変の原則 #7 の一部）**

Generator と Evaluator の両方に `memory: project` を付け、知見を蓄積させる:
- **Generator（実装者）** → 踏んだ地雷、設計判断の理由、残存課題を記録
- **Evaluator（レビュアー）** → 過去の指摘パターン、修正されたか未修正かの追跡

Agent Memory は手動管理しない。エージェント定義の中で「作業中に発見した知見をメモリに記録すること」と指示するだけで、エージェントが自律的に書き込む。

---

#### プロジェクト固有スキル（必要に応じて追加）

| タイプ | 推奨スキル |
|---|---|
| Web アプリ | refactor, ux-improve, ux-evaluate |
| 移行プロジェクト | analyze-system |
| ライブラリ | test-and-release |
| インフラ | plan-review |
| 記事 | review-draft, fact-check |
| コンサル・デリバリー | deliverable-review, meeting-prep, analysis-framework |
| 業務プロセス・ナレッジ | doc-review, process-audit |

### 3-5. Settings の設計

**原則**:
- リバーシブルな操作は allow → 流れを止めない
- 外部に影響する操作は allow しない → 確認を挟む
- 破壊的操作は deny → 構造的に封印

    allow の候補:
    - テスト実行、ビルド、Lint/Format
    - git status, diff, log, add, commit
    - Docker 操作（ローカル開発用）

    deny の候補:
    - sudo, git push --force, git reset --hard

    allow しない（確認を挟む）:
    - git push, deploy 系, terraform apply

### 3-6. MCP の設計（該当する場合）

| ニーズ | MCP |
|---|---|
| ブラウザテスト / UI 評価 | `@playwright/mcp` |
| GitHub 連携 | `gh` CLI で十分 |
| データベース操作 | DB 固有の MCP |
| 外部 API 連携 | 個別の MCP サーバー |

---

## Phase 4: ファイル生成

Plan モードでユーザー承認を得た後、以下の順序でファイルを生成する。
**独立したファイルは並列で作成して効率化する。**

### Step 1: CLAUDE.md を作成（200行以下）
### Step 2: Rules + Settings を並列作成
### Step 3: Subagents を作成（Generator + Evaluator + harness-evolver）
### Step 4: 必須スキルを作成（並列可）
- `.claude/skills/product-self-knowledge/SKILL.md`
- `.claude/skills/<checkpoint>/SKILL.md`
- `.claude/agents/harness-evolver.md` + `.claude/skills/evolve-harness/SKILL.md`

**作成しないもの（グローバルに既存）**:
- `/codex` → `~/.claude/skills/codex/SKILL.md` を流用
- `/imagegen` → `~/.claude/skills/imagegen/SKILL.md` を流用（画像が必要なプロジェクトのみ）

プロジェクト固有の用語・評価観点を載せたい場合のみ、`.claude/skills/codex/SKILL.md` や `.claude/skills/imagegen/SKILL.md` にラッパーを作る（同名でプロジェクト側を優先）。

### Step 5: プロジェクト固有スキルを作成
### Step 6: プラグインインストール（不変の原則 #6）
フロントエンドを含むプロジェクトの場合、Anthropic 公式 `frontend-design` プラグインをインストールする。
### Step 7: MCP 設定（該当する場合）
### Step 8: .gitignore 確認（CLAUDE.local.md, settings.local.json を追加）

---

## Phase 5: 検証

1. CLAUDE.md のサイズ確認（`wc -l CLAUDE.md` → 200行以下）
2. Rules のパススコープ確認
3. Settings の allow/deny が意図通りか確認
4. `/evolve-harness` で自己診断

---

## 設計原則チェックリスト（Phase 3 で毎回確認）

### 不変の原則（スキップ不可）
- [ ] 並列エージェント活用がワークフローに明記されているか
- [ ] code-reviewer（Read-only）が定義されているか
- [ ] グローバル `/codex` スキルが利用可能か（`~/.claude/skills/codex/SKILL.md` 存在確認）。プロジェクト固有ラッパーが必要なら作成済みか
- [ ] `/product-self-knowledge` スキルが作成されているか
- [ ] Codex 検証付きチェックポイントスキルが作成されているか
- [ ] `frontend-design` プラグインがインストールされているか（フロントエンド有の場合）
- [ ] `harness-evolver` エージェント（Read-only）が定義されているか
- [ ] `/evolve-harness` スキルが作成されているか
- [ ] Generator と Evaluator の両方に `memory: project` が付いているか
- [ ] 画像が必要なプロジェクトで、グローバル `/imagegen` が利用可能か（`~/.claude/skills/imagegen/SKILL.md` 存在確認）

### 構造設計
- [ ] CLAUDE.md は 200行以下か
- [ ] Rules は最初は薄く、育てる前提か
- [ ] Generator と Evaluator が分離されているか
- [ ] Evaluator に Edit/Write 権限を与えていないか
- [ ] 破壊的操作が deny されているか
- [ ] git push が allow されていないか（確認を挟む設計か）
- [ ] コードから推測できることを CLAUDE.md に書いていないか
- [ ] 人間がゲートキーパーになるポイントが設計されているか
