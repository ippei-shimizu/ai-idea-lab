# AI Idea Lab

Claude Codeを活用して個人開発で「1発当てる」ためのアプリアイデアを考え、戦略を立てるプロジェクト。

## プロジェクト概要

- **目的**: ソロ開発者が現実的に作れて、収益化できるアプリアイデアの発見と検証
- **前提**: 開発者1人、外部資金なし、Claude Code Maxプランのみ
- **言語**: 日本語で応答すること

## ディレクトリ構成

```
.claude/
├── agents/                    # カスタムサブエージェント定義
│   ├── market-killer.md       # 市場観点からアイデアを破壊的検証
│   ├── tech-killer.md         # 技術観点からアイデアを破壊的検証
│   ├── idea-synthesizer.md    # Killer結果を統合し最終判定
│   ├── trend-hunter.md        # 海外トレンドアプリの発見
│   ├── idea-extractor.md      # トレンドから本質的コンセプトを抽出
│   └── scope-distiller.md     # 独自アイデアの具体化・スコープ縮小
├── skills/
│   ├── idea-killer/SKILL.md   # /idea-killer: アイデア破壊的検証スキル
│   ├── trend-scanner/SKILL.md # /trend-scanner: 海外トレンドからインスピレーションを得て独自アイデア生成
│   └── auto-commit-push/      # /auto-commit-push: 自動コミット&プッシュ
research/
├── validations/               # アイデア検証結果（idea-killerの出力先）
│   └── [アイデア名]/
│       ├── market-killer-report.md
│       ├── tech-killer-report.md
│       └── synthesis-report.md
├── trend-scans/               # トレンドスキャン結果（trend-scannerの出力先）
│   └── [YYYY-MM-DD]/
│       ├── trend-hunter-report.md
│       ├── idea-extractor-report.md
│       └── scope-distiller-report.md
├── deep-research-report.md    # マルチエージェント/スキルの調査レポート
└── agent-teams-ideation-research-ja.md  # Agent Teams調査レポート（日本語）
```

## 核心的な設計思想

1. **AIにアイデアを生成させるのではなく、AIにアイデアを殺させる** — 生き残ったアイデアだけが価値がある
2. **肯定より破壊** — エージェントの仕事はリスクと問題の発見であり、励ましではない
3. **ソロ開発者フィルター** — 全てのアイデアは「1人で4週間以内にMVPを作れるか」で判断

## スキルの使い方

### /trend-scanner [カテゴリ（任意）]
海外で急成長中のアプリからインスピレーションを得て、独自のアプリアイデアを生成する。3つのエージェント（Trend Hunter → Idea Extractor → Scope Distiller）が順次実行。クローンやローカライズではなく、コンセプトの転用。

**例**: `/trend-scanner AI tools` or `/trend-scanner`（全カテゴリ）

**出力**:
- 海外トレンド発見 → コンセプト抽出 → 独自アイデア具体化
- 各候補の `/idea-killer` コピペ用テキスト付き
- 詳細レポートは `research/trend-scans/[日付]/` に保存

### /idea-killer [アイデアの説明]
アプリアイデアを3つの専門エージェント（Market Killer, Tech Killer, Synthesizer）で破壊的に検証する。

**例**: `/idea-killer フリーランス向けの請求書自動生成SaaS`

**出力**:
- 💀 KILL（処刑）/ 🔄 PIVOT（方向転換）/ 🚀 GO（実行）の判定
- 総合スコア（/25）
- 詳細レポートは `research/validations/` に保存

### 推奨ワークフロー

```
/trend-scanner → 候補を発見 → /idea-killer [候補] → 生き残ったアイデアを実行
```
