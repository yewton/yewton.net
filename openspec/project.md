# Project Context

## Purpose
個人のブログ記事管理リポジトリであり、ソフトウェア開発プロジェクトではありません。
主な目的は、技術的な知見や個人の思考を記録・発信することです。

## Tech Stack
- **Content Format**: Markdown（Hugo のページバンドル）
    - 新規記事は `content/post/<YYYY-MM-DD_slug>/index.md` を直接執筆・編集します。
    - 旧記事の Org ソース（`content-org/all-posts.org`）は参照用として残していますが、新規執筆の正ではありません。
- **Static Site Generator**: [Hugo Extended](https://gohugo.io/)（`.tool-versions` で 0.159.1 に固定）
- **Theme**: [HugoBlox](https://hugoblox.com/)（Tailwind CSS 版）を Hugo Modules で取得
    （`github.com/HugoBlox/kit/modules/blox`）。旧 Wowchemy/Bootstrap テーマから移行済み。
- **Hosting**: Netlify
- **Language**: Go（`.tool-versions` で 1.24.3 に固定）- Hugo Modules と HugoBlox の依存として

## Project Conventions

### Content Workflow
1. **執筆**: `content/post/<YYYY-MM-DD_slug>/index.md` を直接編集します。画像などのアセットは同ディレクトリに置きます。
    - Markdown なので code 表記はバッククォート（`` `code` ``）を使います。
    - 公開状態は front matter の `draft` で制御します。`draft: false`（公開）にするのは人間が判断します。
2. **更新日**: 内容を意図的に更新したときだけ、標準の `lastmod:`（ISO 8601）を手動付与します。
   機械的・軽微な修正では更新日を動かしません（`enableGitInfo` は使いません）。
3. **目次・装飾**: 目次は HugoBlox が見出しから自動生成します（手書きしない）。注意喚起は Markdown alert
   構文（`> [!NOTE]` / `> [!WARNING]`）、画像クレジット等は Hugo ショートコード（`{{< unsplash … >}}` 等）を直接使います。
4. **デプロイ**: Git にプッシュすると Netlify が自動的にビルド・デプロイします。

### Writing Style & Tone
- **文体**: 原則として「です・ます」調を使用します。
- **トーン**:
    - 専門的な内容は論理的かつ簡潔に記述します。
    - 個人の意見や感想を含む場合は、客観的な事実と区別できるよう記述します。
    - 一人称は「私」を基本としますが、文脈により省略することもあります。
- **構成**: Markdown の構造化機能（見出し、リスト、front matter のタグ／カテゴリなど）を積極的に活用します。

### Git Workflow
- `main` ブランチへのプッシュで本番環境へデプロイされます。
- コミットメッセージは日本語または英語で、変更内容が明確にわかるように記述します。

### AI & OpenSpec Guidelines
- **Language**: OpenSpec の成果物（`proposal.md`, `tasks.md`, `spec.md` 等）は全て**日本語**で記述します。
- **Workflow**: プロセスを厳格に遵守します。
    1. **Proposal**: プロポーザルを作成・検証する。
    2. **Review**: **必ずユーザーの承認を得る。** 承認なしに実装フェーズに進んではいけません。
    3. **Apply**: 承認後に実装を行い、タスクを完了させる。
- **Tone**: ブログ記事の文体（です・ます調）を厳格に守ります。ドラフト段階から適用すること。

## Domain Context
- 著者はソフトウェアエンジニアであり、主に技術的なトピック（プログラミング、アーキテクチャ、ツールなど）を扱います。
- 時折、育児や生活に関する雑記も含まれます。

## External Dependencies
- **Netlify**: ホスティングおよびビルド・デプロイ（`hugo` ビルド後に Pagefind で検索インデックスを生成）。
- **Disqus**: コメント機能（必要に応じて）。
