# Project Context

## Purpose
個人のブログ記事管理リポジトリであり、ソフトウェア開発プロジェクトではありません。
主な目的は、技術的な知見や個人の思考を記録・発信することです。

## Tech Stack
- **Content Master**: [Emacs Org-mode](https://orgmode.org/)
    - 全ての記事は `content-org/` 配下の `.org` ファイルで執筆・管理されます。
- **Static Site Generator**: [Hugo Extended](https://gohugo.io/) (v0.101.0)
    - `ox-hugo` を使用して Org-mode ファイルから Markdown を生成し、Hugo でビルドします。
- **Theme**: [Wowchemy](https://wowchemy.com/) (v5.7.0)
- **Hosting**: Netlify
- **Language**: Go (v1.16) - Hugo の依存として

## Project Conventions

### Content Workflow
1. **執筆**: `content-org/` 内の `.org` ファイルを編集します。
2. **生成**: Org-mode のエクスポート機能 (`ox-hugo`) を使用して、`content/` 配下に Markdown ファイルを生成します。
    - **禁止事項**: `content/` 配下の Markdown ファイルを直接編集してはいけません。
3. **デプロイ**: Git にプッシュすると Netlify が自動的にビルド・デプロイします。

### Writing Style & Tone
- **文体**: 原則として「です・ます」調を使用します。
- **トーン**:
    - 専門的な内容は論理的かつ簡潔に記述します。
    - 個人の意見や感想を含む場合は、客観的な事実と区別できるよう記述します。
    - 一人称は「私」を基本としますが、文脈により省略することもあります。
- **構成**: Org-mode の構造化機能（見出し、リスト、タグなど）を積極的に活用します。

### Git Workflow
- `main` ブランチへのプッシュで本番環境へデプロイされます。
- コミットメッセージは日本語または英語で、変更内容が明確にわかるように記述します。

## Domain Context
- 著者はソフトウェアエンジニアであり、主に技術的なトピック（プログラミング、アーキテクチャ、ツールなど）を扱います。
- 時折、育児や生活に関する雑記も含まれます。

## External Dependencies
- **Netlify**: ホスティングおよび CMS 機能（現在は Org-mode 中心のため CMS は補助的利用か未使用）。
- **Disqus**: コメント機能（必要に応じて）。
