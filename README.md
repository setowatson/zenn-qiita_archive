# zenn-archive
![](https://github.com/setoatson/zenn-archive/actions/workflows/publish.yml/badge.svg)

This repository is used to manage and publish articles on [Zenn](https://zenn.dev/) and [Qiita](https://qiita.com/). Primarily, it's a place to share the code I've created and record what I've learned. This serves as a personal archive from which others can also learn. You can find my pages on Zenn [here](https://zenn.dev/setotoset) and on Qiita [here](https://qiita.com/setwatson), respectively. In addition, I use this [zenn-qiita-sync](https://github.com/C-Naoki/zenn-qiita-sync) to synchronize Qiita articles with Zenn articles.

## Directory Structure

- `.github/workflows/`: It contains GitHub Actions workflows.
- `articles/`: It contains Zenn articles written in Markdown format.
- `books/`: It contains Zenn books. The structure should follow the Zenn book guidelines.
- `images/`: It contains images used in articles and books.
- `qiita/`: It contains Qiita articles, which are generated from Zenn articles by `ztoq.sh`.

## Getting Started
Run the development server:
```bash
npx zenn preview
```
- Open [http://localhost:8000](http://localhost:8000) with your browser to see the result.

When you want to write a new article:
```bash
npx zenn new:article --slug 記事のスラッグ --title タイトル --type idea --emoji ✨
```
- The above command's options are as follows:
    - `--slug`: The slug of the article. This needs to be a combination of `a-z0-9`, `hyphen (-)`, and `underscore (_)` with 12 to 50 characters.
    - `--title`: The title of the article.
    - `--type`: The type of the article. The options should be chosen from `tech`, `idea`.
    - `--emoji`: The emoji of the article.

## Article Publishing Workflow
1. Create a new article using the Zenn CLI:
   ```bash
   npx zenn new:article --slug your-article-slug --title "Your Article Title" --type tech --emoji 🚀
   ```

2. Write your article in Markdown format in the `articles/` directory.

3. Preview your article locally:
   ```bash
   npx zenn preview
   ```

4. Commit and push your changes to the main branch:
   ```bash
   git add .
   git commit -m "Add new article: Your Article Title"
   git push origin main
   ```

5. The GitHub Actions workflow will automatically:
   - Publish the article to Zenn
   - Synchronize the article with Qiita using zenn-qiita-sync
   - Update the repository with any generated Qiita articles

---

# zenn-archive（日本語版）
![](https://github.com/setoatson/zenn-archive/actions/workflows/publish.yml/badge.svg)

このリポジトリは[Zenn](https://zenn.dev/)と[Qiita](https://qiita.com/)に同時に記事を投稿・管理するために使用しています。主に、私が作成したコードを共有し、学んだことを記録する場所として機能しています。これは個人のアーカイブとして機能し、他の人も学ぶことができます。私のZennのページは[こちら](https://zenn.dev/setotoset)、Qiitaのページは[こちら](https://qiita.com/setwatson)から確認できます。また、[zenn-qiita-sync](https://github.com/C-Naoki/zenn-qiita-sync)を使用して、Qiitaの記事をZennの記事と同期させています。

## ディレクトリ構造

- `.github/workflows/`: GitHub Actionsのワークフローが含まれています。
- `articles/`: Markdown形式で書かれたZennの記事が含まれています。
- `books/`: Zennの本が含まれています。構造はZennの本のガイドラインに従う必要があります。
- `images/`: 記事や本で使用される画像が含まれています。
- `qiita/`: `ztoq.sh`によってZennの記事から生成されたQiitaの記事が含まれています。

## 始め方
開発サーバーを実行するには：
```bash
npx zenn preview
```
- ブラウザで [http://localhost:8000](http://localhost:8000) を開いて結果を確認できます。

新しい記事を書きたい場合：
```bash
npx zenn new:article --slug 記事のスラッグ --title タイトル --type idea --emoji ✨
```
- 上記のコマンドのオプションは以下の通りです：
    - `--slug`: 記事のスラッグ。`a-z0-9`、`ハイフン (-)`、`アンダースコア (_)`の組み合わせで12〜50文字である必要があります。
    - `--title`: 記事のタイトル。
    - `--type`: 記事のタイプ。`tech`、`idea`から選択する必要があります。
    - `--emoji`: 記事の絵文字。

## 記事投稿の流れ
1. Zenn CLIを使用して新しい記事を作成します：
   ```bash
   npx zenn new:article --slug 記事のスラッグ --title "記事のタイトル" --type tech --emoji 🚀
   ```

2. `articles/`ディレクトリ内でMarkdown形式で記事を執筆します。

3. ローカルで記事をプレビューします：
   ```bash
   npx zenn preview
   ```

4. 変更をコミットしてmainブランチにプッシュします：
   ```bash
   git add .
   git commit -m "新しい記事を追加: 記事のタイトル"
   git push origin main
   ```

5. GitHub Actionsのワークフローが自動的に以下の処理を行います：
   - Zennに記事を公開
   - zenn-qiita-syncを使用してQiitaと同期
   - 生成されたQiitaの記事でリポジトリを更新
