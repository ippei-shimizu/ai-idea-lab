# AIエージェントチームによるアプリアイデア創出 - ディープリサーチレポート

> 調査日: 2026-03-28
> 目的: Claude Codeを活用してアプリアイデアを生成・評価するエージェントチーム/Skillの構築に向けた調査

---

## 目次

1. [マルチエージェントフレームワーク](#1-マルチエージェントフレームワーク)
2. [Claude Code専用のマルチエージェント基盤](#2-claude-code専用のマルチエージェント基盤)
3. [Claude Code Skillsエコシステム](#3-claude-code-skillsエコシステム)
4. [AIアイデア生成・バリデーションツール](#4-aiアイデア生成バリデーションツール)
5. [AIアイデア創出フレームワーク](#5-aiアイデア創出フレームワーク)
6. [エージェント役割の専門化](#6-エージェント役割の専門化)
7. [AIブレインストーミング手法](#7-aiブレインストーミング手法)
8. [成功するインディーアプリのパターン](#8-成功するインディーアプリのパターン)
9. [推奨アーキテクチャ](#9-推奨アーキテクチャ)
10. [参考リンク集](#10-参考リンク集)

---

## 1. マルチエージェントフレームワーク

### 主要フレームワーク一覧

| フレームワーク | GitHub Stars | 特徴 | URL |
|---|---|---|---|
| **MetaGPT** | ~66,200 | ソフトウェア会社をシミュレート（PM, Tech Lead, Dev等）。「シミュレートされた企業」コンセプトがアイデア生成チームに転用可能 | [github.com/geekan/MetaGPT](https://github.com/geekan/MetaGPT) |
| **AutoGen** (Microsoft) | ~54,600 | マルチエージェント会話パターンの先駆者。v0.4で非同期メッセージング追加。ブレインストーミングの議論・反論パターンに最適 | [github.com/microsoft/autogen](https://github.com/microsoft/autogen) |
| **CrewAI** | ~45,900 | 役割ベースのエージェント定義（Role-Goal-Backstory）。LangChain非依存。10万人以上の認定開発者 | [github.com/crewAIInc/crewAI](https://github.com/crewAIInc/crewAI) |
| **LangGraph** | ~24,000 | グラフベースのワークフローエンジン。分岐・合流のあるアイデア生成パイプラインに最適 | [github.com/langchain-ai/langgraph](https://github.com/langchain-ai/langgraph) |
| **Pydantic AI** | ~15,100 | 型安全なPythonフレームワーク。構造化された出力（アイデアスキーマ等）に強い | - |
| **CAMEL-AI** | - | ロールプレイ会話に特化。起業家 vs 投資家 vs 顧客のシミュレーションに最適。OWL（汎用マルチエージェント）も提供 | [github.com/camel-ai/camel](https://github.com/camel-ai/camel) |
| **ChatDev** | - | v2.0でゼロコードマルチエージェントプラットフォーム。「パペティア・パラダイム」で動的にコラボレーション方式を進化 | [github.com/OpenBMB/ChatDev](https://github.com/OpenBMB/ChatDev) |
| **Agency Swarm** | ~3,900 | OpenAI Agents SDK上に構築。CEO, VA, Developer等の実世界組織構造をモデリング | [github.com/VRSEN/agency-swarm](https://github.com/VRSEN/agency-swarm) |

### 選定の指針

- **アイデア生成チームに最適**: CrewAI（Role-Goal-Backstoryが直感的）
- **会話型ブレインストーミングに最適**: AutoGen / CAMEL-AI
- **複雑なパイプラインに最適**: LangGraph
- **企業シミュレーションに最適**: MetaGPT

---

## 2. Claude Code専用のマルチエージェント基盤

### 公式機能

#### Agent Teams（実験的機能）
- 1セッションがチームリードとして機能し、タスクを割り当て結果を統合
- チームメイトは独立したコンテキストウィンドウで作業し、直接コミュニケーション
- 推奨チームサイズ: 3-5エージェント
- [公式ドキュメント](https://code.claude.com/docs/en/agent-teams)

#### Task Tool
- 一時的なサブエージェントを生成し、専用コンテキストで実行
- 最大10の同時タスク実行、インテリジェントキューイング
- アドホックな並列作業に最適

#### Custom Subagents
- カスタムシステムプロンプト、専用ツールアクセス、独立した権限
- 繰り返し可能な専門的役割に最適
- [公式ドキュメント](https://code.claude.com/docs/en/sub-agents)

### コミュニティツール

| プロジェクト | Stars | 概要 | URL |
|---|---|---|---|
| **Ruflo** (旧Claude Flow) | ~14,000 | 60以上の専門エージェント、複数のスウォームトポロジー（mesh, hierarchical, ring, star）。AgentDBで永続メモリ。トークン消費75-80%削減 | [github.com/ruvnet/ruflo](https://github.com/ruvnet/ruflo) |
| **claude_code_agent_farm** | - | 20-50+のClaude Codeエージェントを並列実行。自動バグ修正、tmuxモニタリング | [github.com/Dicklesworthstone/claude_code_agent_farm](https://github.com/Dicklesworthstone/claude_code_agent_farm) |
| **agent-flow** | - | Claude Codeエージェントのリアルタイム可視化 | [github.com/patoles/agent-flow](https://github.com/patoles/agent-flow) |
| **overstory** | - | git worktreeでワーカーエージェントを生成、SQLiteメールシステムで調整、階層的コンフリクト解決 | [github.com/jayminwest/overstory](https://github.com/jayminwest/overstory) |
| **Claude Code Workflow** | - | JSON駆動のマルチエージェント開発フレームワーク | [github.com/catlog22/Claude-Code-Workflow](https://github.com/catlog22/Claude-Code-Workflow) |

### 重要リソース

- [Claude Code Swarm Orchestration Skill (Gist)](https://gist.github.com/kieranklaassen/4f2aba89594a4aea4ad64d753984b2ea) - TeammateTool、タスクシステム、全調整パターン
- [Claude Code System Prompts](https://github.com/Piebald-AI/claude-code-system-prompts) (6,800+ stars) - 内部プロンプトの全貌
- [Claude Code Best Practices](https://github.com/shanraisshan/claude-code-best-practice) - Command -> Agent -> Skill のオーケストレーションパターン
- [Agentmaxxing](https://vibecoding.app/blog/agentmaxxing) - 並列エージェント実行の実践的上限は5-7

---

## 3. Claude Code Skillsエコシステム

### Skills の仕組み

- SKILL.md ファイル（YAMLフロントマター + Markdown本文）
- Claude が純粋なLLM推論でどのスキルを呼び出すか判断（埋め込みや分類器は不使用）
- name最大64文字、description最大1024文字、本文500行以内が最適

```yaml
---
name: my-skill
description: Brief description for Claude to decide when to load.
---

# Instructions
Step-by-step workflow...
```

### 主要スキルリポジトリ

| リポジトリ | Stars | 内容 | URL |
|---|---|---|---|
| **anthropics/skills** | 104,000+ | Anthropic公式スキル集（skill-creator, claude-api, pdf, docx, canvas-design等） | [github.com/anthropics/skills](https://github.com/anthropics/skills) |
| **awesome-agent-skills** | - | 1000+のスキル。Claude Code, Codex, Cursor等複数ツール対応 | [github.com/VoltAgent/awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills) |
| **claude-skills** | - | 192+のスキル（エンジニアリング、マーケティング、PM、コンプライアンス、C-level） | [github.com/alirezarezvani/claude-skills](https://github.com/alirezarezvani/claude-skills) |
| **awesome-claude-skills** | - | キュレーションされたスキル・リソースリスト | [github.com/travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills) |
| **awesome-claude-md** | - | 公開GitHubプロジェクトからの優れたCLAUDE.mdコレクション | [github.com/josix/awesome-claude-md](https://github.com/josix/awesome-claude-md) |

### アイデア生成に直接関連するスキル

| スキル | 内容 | URL |
|---|---|---|
| **Product-Manager-Skills** | 46のPMスキル + 再利用可能なコマンドワークフロー。PM〜VP/CPOまでのキャリア全段階をカバー | [github.com/deanpeters/Product-Manager-Skills](https://github.com/deanpeters/Product-Manager-Skills) |
| **slavingia/skills** | Gumroad創設者による「The Minimalist Entrepreneur」ベースの起業家スキル | [github.com/slavingia/skills](https://github.com/slavingia/skills) |
| **Brainstorming Skill** | 30+のリサーチ検証済みプロンプトパターン、14カテゴリ | [mcpmarket.com](https://mcpmarket.com/tools/skills/brainstorming-strategy-ideation) |
| **planning-with-files** | Manus式の永続的Markdownプランニング（task_plan.md, findings.md, progress.md） | [github.com/OthmanAdi/planning-with-files](https://github.com/OthmanAdi/planning-with-files) |

### 公式ドキュメント

- [Extend Claude with skills](https://code.claude.com/docs/en/skills)
- [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [The Complete Guide to Building Skills (PDF)](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf?hsLang=en)
- [Equipping agents for the real world](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

---

## 4. AIアイデア生成・バリデーションツール

### アイデア生成ツール

| ツール | 内容 | URL |
|---|---|---|
| **Brainstormers** | SCAMPER, Six Thinking Hats等6つのブレインストーミング手法をAIエージェント化。Next.js 15 + React 19。1セッション$0.01-0.02 | [github.com/Azzedde/brainstormers](https://github.com/Azzedde/brainstormers) |
| **Stratup.ai** | 10万以上のアイデアDB。SWOT/PESTEL分析付きレポート生成。共同創設者マッチング | [stratup.ai](https://stratup.ai/en) |
| **IdeasAI** | xAI Grokによるアイデア生成 + コミュニティ投票 | [ideasai.com](https://ideasai.com/) |

### バリデーションツール

| ツール | 内容 | URL |
|---|---|---|
| **Niches Hunter** | 毎日4万以上のiOSアプリを分析。競合スコア、収益推定、市場ギャップ分析。$9.99/月 | [nicheshunter.app](https://nicheshunter.app/) |
| **IdeaProof.io** | 30秒でTAM/SAM/SOM、競合SWOT、財務予測、実現可能性分析 | [ideaproof.io](https://ideaproof.io/) |
| **ValidatorAI** | AI価値提案、競合分析、ターゲット顧客特定、スタートアップスコア | [validatorai.com](https://validatorai.com/) |
| **Cambium AI** | GTM戦略、顧客ペルソナ、ブランドメッセージ、競合インサイト | [cambium.ai](https://blog.cambium.ai/) |

### その他の参考リポジトリ

- [500-AI-Agents-Projects](https://github.com/ashishpatel26/500-AI-Agents-Projects) - 500以上のAIエージェントユースケース集
- [a-list-of-claude-code-agents](https://github.com/hesreallyhim/a-list-of-claude-code-agents) - Claude Codeサブエージェント一覧

---

## 5. AIアイデア創出フレームワーク

### Design Thinking + AI
- AIがユーザーリサーチ合成と解決策生成を自動化
- 人間は問題定義とテストに集中
- AIエージェントが「共感」フェーズ（レビュー分析、フォーラム投稿分析）と「発想」フェーズを担当

### Jobs-to-be-Done (JTBD) + AI
1. コアプロセスをマッピング、摩擦点・ボトルネックを特定
2. 機能的ジョブ（コアタスク）と感情的ジョブ（感じ方）を理解
3. インパクト・実現可能性・整合性・コストで機会を優先順位付け
4. SMARTゴールとパイロットプロジェクトで戦略を構築
5. ジョブ成果に紐づくKPIを測定し反復

**応用**: AIエージェントがApp Storeレビュー、Reddit投稿、フォーラムを分析し、未充足の「ジョブ」を抽出、フラストレーションの頻度と強度でスコアリング

### Lean Startup + AI
- Build-Measure-Learnサイクルをアイデア生成段階から加速
- 複数のビジネスモデル仮説を生成、顧客インタビューをシミュレート、市場シグナルを評価

### Ideation 3.0（AIの5つの支援機能）
1. **Inspirer** - 創造的刺激とアナロジーを生成
2. **Stylist** - アイデアを洗練し、プレゼンテーション用にフォーマット
3. **Matchmaker** - 関連アイデアを接続し、シナジーを特定
4. **Analyst** - 実現可能性と市場ポテンシャルを評価
5. **Organizer** - アイデアを分類し優先順位付け

出典: [Journal of Product Innovation Management](https://onlinelibrary.wiley.com/doi/10.1111/jpim.12782?af=R)

---

## 6. エージェント役割の専門化

### CrewAIの Role-Goal-Backstory モデル

各エージェントを3要素で定義:
- **Role**: 専門的な機能（「テクニカルドキュメントスペシャリスト」のように具体的に）
- **Goal**: 成功基準を含むアウトカム指向
- **Backstory**: 専門性、経験、作業スタイルを確立

**重要原則**: 「エージェントは汎用的な役割より専門的な役割を与えられた方が有意に良い性能を発揮する」

**80/20ルール**: タスク設計に80%、エージェント定義に20%の労力を投資。よく設計されたタスクは完璧なエージェント定義より性能を向上させる。

### 推奨エージェントチーム構成（7エージェント）

| エージェント | 責任 | 視点 |
|---|---|---|
| **Market Analyst** | 業界トレンド、TAM/SAM/SOM、競合マッピング | データ駆動の市場機会 |
| **User Researcher** | ユーザーペインポイント、JTBD、行動パターン分析 | ユーザー共感とニーズ |
| **Tech Lead** | 技術的実現可能性、スタック推奨、ビルド複雑度 | エンジニアリングの実用主義 |
| **Business Strategist** | マネタイズモデル、ユニットエコノミクス、GTM | 収益と持続可能性 |
| **UX Designer** | ユーザビリティ、ユーザーフロー、デザインによる差別化 | ユーザー体験品質 |
| **Devil's Advocate** | 仮定への挑戦、リスク特定、盲点の発見 | 批判的思考 |
| **Indie Hacker Expert** | ソロ開発者の実現可能性、TTM、保守負担 | ソロ開発の実用性 |

### マルチエージェントコラボレーションパターン（AWS研究）

- **Peer-to-peer**: 全エージェントが対等に協力（ブレインストーミング向き）
- **Hierarchical**: マネージャーがスペシャリストに委任（構造的評価向き）
- **Sequential pipeline**: アイデアがエージェント段階を順に流れる（段階的洗練向き）

---

## 7. AIブレインストーミング手法

### Six Thinking Hats（de Bono）- AIエージェント適用版

| Hat | エージェントの焦点 | 用途 |
|---|---|---|
| White（事実） | データ駆動分析、市場統計、測定可能な証拠 | 市場リサーチエージェント |
| Red（感情） | 直感的反応、ユーザーの感情的ニーズ | ユーザー共感エージェント |
| Black（注意） | リスク特定、最悪シナリオ、失敗モード | Devil's Advocateエージェント |
| Yellow（楽観） | 機会探索、ベストケース、上振れポテンシャル | 機会探索エージェント |
| Green（創造） | 斬新なアイデア、非従来的思考、ブレークスルー | 創造的発想エージェント |
| Blue（プロセス） | 構造、方法論、順序付け、意思決定 | オーケストレーターエージェント |

ディープモードでは6エージェントが並列サブエージェントとして同時実行。

### SCAMPER - 体系的イノベーション

既存製品/アイデアに適用する7つの変換プロンプト:
1. **Substitute**（代替）: 何の要素を置き換えられるか？
2. **Combine**（結合）: 何を統合できるか？
3. **Adapt**（適応）: 他分野から何を借用できるか？
4. **Modify**（修正）: 何をスケールアップ/ダウンできるか？
5. **Put to another use**（転用）: 代替用途は？
6. **Eliminate**（排除）: 何の複雑さを除去できるか？
7. **Reverse**（逆転）: 何を反転できるか？

### その他の手法

- **Reverse Brainstorming**: 「この問題をどう悪化させるか？」→ 解決策を反転
- **Role Storming**: 異なるユーザーペルソナを採用し、ステークホルダー固有のアイデアを生成
- **Starbursting**: 解決策生成前に5W1Hで包括的な質問を生成
- **TRIZ**: ソフトウェア適応された発明的原理（セグメンテーション、抽出、予備的アクション等）

### 測定されたAIブレインストーミングの効果

- ブレインストーミング時間を25-40%短縮
- チーム創造性35%向上（Accenture調査）
- 生産性83%向上、会議41%削減（富士通調査）
- HBR (2025年12月): LLMは持続性と柔軟性を通じて創造性を解放

---

## 8. 成功するインディーアプリのパターン

### 収益データ（トップインディーアプリ）

| アプリ | 月間収益 | カテゴリ |
|---|---|---|
| ConvertKit/Kit | $3,000,000 | クリエイター向けメールマーケティング |
| Wave AI | $450,000 | AI活用アプリ |
| Formula Bot | $220,000 | スプレッドシート数式変換 |
| ShipFast | $133,000 | 開発者ボイラープレート |
| Tally Forms | $100,000 | フォームビルダー |
| SiteGPT | $95,000 | チャットボットビルダー |
| TypingMind | $50,000 | ChatGPTインターフェース |
| Bannerbear | $50,000 | 画像生成ツール |

### 共通成功パターン

1. **ゼロファンディング**: ほとんどの創設者が本業の傍らでブートストラップ
2. **ソロまたは超少人数**: 1-5人で利益率90%以上を維持
3. **ニッチフォーカス**: 「スイスアーミーナイフではなく外科用メス」
4. **マーケットタイミング**: 新技術の早期採用（AI統合が現在のトレンド）
5. **Build in Public**: SNS、Product Hunt、Reddit、Indie Hackersで集客
6. **ハイブリッドマネタイズ**: Free + Paid ティアの組み合わせ

### 高機会カテゴリ（2026年）

1. **開発者ツール & インフラ** - 最高収益ポテンシャル
2. **AI搭載ニッチソリューション** - 履歴書ビルダー、ポッドキャストアシスタント等
3. **バーティカルSaaS** - 業界特化型ワークフロー（コンプライアンス、スケジューリング等）
4. **クリエイターエコノミーツール** - コンテンツ制作、スケジューリング、マネタイズ
5. **ヘルス＆ウェルネス** - 瞑想、フィットネス、睡眠（未開拓ニッチ）
6. **Micro-SaaS** - 市場は年30%成長、2030年に$59.6B到達予測

### インディー開発15年の教訓（Lukas Petr）

- 技術への愛が内発的モチベーションの源泉
- 市場トレンドよりも情熱のある特定の問題を見つける
- コミュニティのサポートとフィードバックがマネタイズ戦略を変革
- 完璧主義を避け、ハイインパクトなビジネス活動に集中
- 成功と失敗の境界線は薄い。持続性が鍵

---

## 9. 推奨アーキテクチャ

### 全体構成

```
┌──────────────────────────────────────────────────────────┐
│              Orchestrator（チームリード）                   │
│         Claude Code Native Skills / Agent Teams           │
└────────────┬──────────────┬──────────────┬───────────────┘
             │              │              │
     ┌───────▼──────┐ ┌────▼─────────┐ ┌──▼────────────┐
     │ Phase 1      │ │ Phase 2      │ │ Phase 3       │
     │ 発散的        │ │ 構造的評価    │ │ バリデーション │
     │ アイデア生成   │ │              │ │               │
     └───────┬──────┘ └────┬─────────┘ └──┬────────────┘
             │              │              │
             ▼              ▼              ▼
     Six Thinking Hats   7専門家エージェント  市場検証
     SCAMPER             スコアリング       ランディングページ
     Reverse Brainstorm  クロスリファレンス   30日バリデーション
```

### Phase 1: 発散的アイデア生成
- Six Thinking Hats + SCAMPER + Reverse Brainstorming の並列実行
- JTBDフレームワークで実際のユーザーニーズにアンカー
- Peer-to-peer パターンで自由にアイデア生成

### Phase 2: 構造的評価
- 7つの専門エージェントがそれぞれの視点からアイデアを評価
- 複合スコアリングマトリクス: 市場機会、技術的実現可能性、ソロ開発適性、差別化、マネタイズ明確性
- 成功したインディーアプリパターンとのクロスリファレンス

### Phase 3: 高速バリデーション
- 上位アイデアのランディングページコピー自動生成
- バリデーション用コミュニティ・チャネルの特定
- 30日間バリデーションロードマップの作成

### Phase 4: ラピッドプロトタイピング（オプション）
- Claude Codeで上位アイデアのMVPを自動生成
- Build in Public 戦略の策定

### 設計原則

1. **CrewAIのRole-Goal-Backstoryパターン**でエージェント定義
2. **80/20ルール**: タスク設計に80%、エージェント定義に20%
3. **コラボレーションパターンの使い分け**: ブレスト=Peer-to-peer、評価=Hierarchical
4. **Micro-SaaS / バーティカルニッチ**をターゲット（最高成功率）
5. **ソロ開発者フィルター**を常に適用
6. **同時実行上限5-7エージェント**を考慮

---

## 10. 参考リンク集

### マルチエージェントフレームワーク
- [CrewAI GitHub](https://github.com/crewAIInc/crewAI)
- [AutoGen GitHub](https://github.com/microsoft/autogen)
- [MetaGPT GitHub](https://github.com/geekan/MetaGPT)
- [CAMEL-AI GitHub](https://github.com/camel-ai/camel)
- [ChatDev GitHub](https://github.com/OpenBMB/ChatDev)
- [Agency Swarm GitHub](https://github.com/VRSEN/agency-swarm)
- [LangGraph GitHub](https://github.com/langchain-ai/langgraph)
- [Top 10 AI Agent Frameworks 2026](https://techwithibrahim.medium.com/top-10-most-starred-ai-agent-frameworks-on-github-2026-df6e760a950b)

### Claude Code エコシステム
- [Claude Code Agent Teams 公式ドキュメント](https://code.claude.com/docs/en/agent-teams)
- [Claude Code Custom Subagents 公式](https://code.claude.com/docs/en/sub-agents)
- [Claude Code Skills 公式](https://code.claude.com/docs/en/skills)
- [Skills Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Ruflo GitHub](https://github.com/ruvnet/ruflo)
- [claude_code_agent_farm GitHub](https://github.com/Dicklesworthstone/claude_code_agent_farm)
- [Claude Code System Prompts](https://github.com/Piebald-AI/claude-code-system-prompts)
- [Claude Code Swarm Orchestration Gist](https://gist.github.com/kieranklaassen/4f2aba89594a4aea4ad64d753984b2ea)
- [Agentmaxxing Guide](https://vibecoding.app/blog/agentmaxxing)

### スキルリポジトリ
- [anthropics/skills (公式)](https://github.com/anthropics/skills)
- [awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills)
- [claude-skills (192+)](https://github.com/alirezarezvani/claude-skills)
- [awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills)
- [awesome-claude-md](https://github.com/josix/awesome-claude-md)
- [Product-Manager-Skills](https://github.com/deanpeters/Product-Manager-Skills)
- [slavingia/skills](https://github.com/slavingia/skills)
- [planning-with-files](https://github.com/OthmanAdi/planning-with-files)

### アイデア生成・バリデーション
- [Brainstormers GitHub](https://github.com/Azzedde/brainstormers)
- [Stratup.ai](https://stratup.ai/en)
- [IdeasAI](https://ideasai.com/)
- [Niches Hunter](https://nicheshunter.app/)
- [IdeaProof.io](https://ideaproof.io/)
- [ValidatorAI](https://validatorai.com/)
- [500-AI-Agents-Projects](https://github.com/ashishpatel26/500-AI-Agents-Projects)

### 方法論・知見
- [CrewAI - Crafting Effective Agents](https://docs.crewai.com/en/guides/agents/crafting-effective-agents)
- [AWS Multi-Agent Patterns](https://aws.amazon.com/blogs/machine-learning/multi-agent-collaboration-patterns-with-strands-agents-and-amazon-nova/)
- [AI Strategy with JTBD](https://medium.com/@mikeboysen/ai-strategy-a-practical-framework-using-jobs-to-be-done-jtbd-5e86f3fa7528)
- [SpecWeave Brainstorming](https://spec-weave.com/docs/guides/brainstorming/)
- [15 Lessons from 15 Years of Indie Dev](https://lukaspetr.com/15-lessons-from-15-years-of-indie-app-development/)
- [Top 15 Most Profitable Indie Apps](https://mktclarity.com/blogs/news/indie-apps-top)
- [Best Bootstrapped SaaS Niches 2026](https://entrepreneurloop.com/bootstrapped-saas-niches-solo-founders/)
- [Solo Developer Process](https://www.vooster.ai/en/blog/solo-developer-profitable-app-process)
- [Ideation 3.0 (JPIM)](https://onlinelibrary.wiley.com/doi/10.1111/jpim.12782?af=R)
