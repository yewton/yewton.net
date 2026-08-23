## ADDED Requirements

### Requirement: Markdown を正とする記事執筆

新規記事は Markdown を唯一のソースとして執筆・編集しなければならない (SHALL)。具体的には Hugo のページバンドル `content/post/<YYYY-MM-DD_slug>/index.md` を直接編集し、Emacs Org-mode / `ox-hugo` を経由してはならない。

既存記事および `content-org/*.org` は当面そのまま温存され、参照専用として扱われる。

#### Scenario: 新規記事を Markdown で書く
- **WHEN** 執筆者またはエージェントが新規記事を作成するとき
- **THEN** `content/post/<YYYY-MM-DD_slug>/index.md` を直接作成・編集し、同ディレクトリに画像などのアセットを配置する
- **AND** Org ファイルの編集や Emacs での再エクスポートは不要である

#### Scenario: Emacs を持たないエージェントが編集・レビューする
- **GIVEN** Emacs を持たないエージェントが記事のレビュー・修正を依頼されたとき
- **WHEN** そのエージェントが `content/post/<bundle>/index.md` を編集するとき
- **THEN** Org からの再エクスポートを介さずに変更が反映される

### Requirement: 更新日は意図的な内容更新時のみ手動付与

「最終更新」日は、内容を意図的に更新したときにのみ front matter の `lastmod:`（Hugo 標準フィールド、ISO 8601）で手動付与しなければならない (SHALL)。機械的・軽微な修正では更新日を変更してはならず、git コミット日時からの自動導出（`enableGitInfo`）も用いない。

#### Scenario: 未更新記事に最終更新を表示しない
- **GIVEN** `lastmod:` を持たない（公開後に内容更新していない）記事
- **WHEN** その記事ページを表示したとき
- **THEN** 「最終更新」行は表示されない（公開日を最終更新として重複表示しない）

#### Scenario: 意図的更新時に最終更新を表示する
- **GIVEN** 執筆者が記事内容を意図的に更新し、`lastmod:` を公開日より後の日時に設定したとき
- **WHEN** その記事ページを表示したとき
- **THEN** 設定した日時が「最終更新」として表示される

#### Scenario: 機械的修正で更新日が動かない
- **WHEN** 移行に伴う整形・タイポ修正など、内容を伴わない機械的修正を行ったとき
- **THEN** `lastmod:` は変更されず、「最終更新」表示も変化しない

### Requirement: 目次はテーマの自動生成に委ねる

新規記事は目次（TOC）を手書きしてはならない (SHALL NOT)。目次は HugoBlox テーマが見出し構造から自動生成する仕組みに委ねる。

#### Scenario: 自動生成される目次
- **GIVEN** 見出しを持つ新規記事
- **WHEN** その記事ページを表示したとき
- **THEN** テーマが見出しから自動生成した目次が表示され、記事本文に静的な目次ブロックは含まれない

### Requirement: 素の Markdown と Hugo ショートコードで記述する

新規記事の本文は素の Markdown 記法で書かなければならない (SHALL)。装飾・埋め込みには Hugo ショートコードおよび Markdown alert 構文を直接用い、Org マクロ（`{{{…}}}`）や `{{% callout %}}` は用いない。

#### Scenario: 注意喚起を Markdown alert で書く
- **WHEN** 記事に注意書きや警告を入れるとき
- **THEN** `> [!NOTE]` / `> [!WARNING]` の Markdown alert 構文を用いる

#### Scenario: 画像クレジットをショートコードで書く
- **WHEN** Unsplash / Pexels / Pixabay 由来の画像クレジットを入れるとき
- **THEN** 対応する Hugo ショートコード（`{{< unsplash … >}}` 等）を直接記述する
