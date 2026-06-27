# skills

コーディングエージェントで利用しているスキル管理用リポジトリです。

## Installation

```bash
# Install all skills
npx skills add tukaelu/skills -a claude-code -g

# Install a specific skill
npx skills add tukaelu/skills --skill compose-commit -a claude-code -g
```

## Repository Structure

```
skills/
├── compose-commit/
│   └── SKILL.md
├── handoff/
│   └── SKILL.md
├── natural-japanese-writing/
│   └── SKILL.md
├── prune-branches/
│   └── SKILL.md
├── research-topics/
│   ├── references/
│   │   └── analysis-frameworks.md  # テーマに応じた分析フレームワーク集
│   └── SKILL.md
├── sketch-diagram/
│   ├── references/
│   │   └── ascii-examples.md       # ダイアグラムタイプ別の ASCII アートサンプル集
│   └── SKILL.md
├── write-article/
│   └── SKILL.md
└── write-skill/
    ├── references/
    │   └── BEST-PRACTICES.md       # スキル作成のベストプラクティス
    └── SKILL.md
```

## Skills

| Skill | Description |
|---|---|
| compose-commit | git の変更ファイルを解析し、論理的な単位で適切な粒度のコミットを作成する |
| handoff | コンテキストが肥大化したときに現在の作業状況を収集し、新しいセッションへの引き継ぎドキュメントを生成する |
| natural-japanese-writing | AI 特有のパターンを排除し、自然で簡潔な日本語テキストを生成・レビューする |
| prune-branches | ベースブランチにマージ済みのローカル作業ブランチを安全に削除する |
| research-topics | 定量・定性的な調査を通じて事実と意見を収集・分析し、構造化されたレポートや提言を提供する |
| sketch-diagram | 対話を通じてインタラクティブに Mermaid ダイアグラムを作成する |
| write-article | 記事の構造を分析し、明確さ・物語の流れ・読みやすさを高めるために再構成・書き直しを行う |
| write-skill | 公式のベストプラクティスに従って Claude エージェントスキルを設計・実装する |

