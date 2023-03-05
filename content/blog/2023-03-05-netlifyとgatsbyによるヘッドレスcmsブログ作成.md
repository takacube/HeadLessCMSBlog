---
templateKey: post
date: 2023-03-05T02:16:03.408Z
title: NetlifyとGatsbyによるヘッドレスCMSブログ作成
---
Wordpressを使わずにGatsbyとNetlifyを使って5分でブログサイトを構築する.
超簡単にできてヘッドレスCMSとSSGによってセキュリティー面とサーバー費用の面で圧倒的に便利
https://github.com/takacube/HeadLessCMSBlog

1. Gatsbyとは
   GatsbyはReactをベースに作られたSSGである. 同様にReactをベースに作られたフレームワークであるNext.jsもある. Next.jsはSSRとSSGを機能として持っている.
   今回Next.jsではなく, Gatsbyを用いたのには理由が2つあり,
   一つは今回はHeadlessCMSとともに用いるのでGraphqlをより効果的に使えるという点,
   二つ目はGatsbyにはいくつかブログ用のテンプレートがあるため, コーディングの時間を削減することができる点.
2. Netlifyとは
   ヘッドレスCMSの一つで, Githubなどのリポジトリと連携させてウェブサイトホスティングできるサービスである.
3. 全体構成
   <img src="https://storage.googleapis.com/zenn-user-upload/3b049e4b86d9-20230304.png" width="100%">

   以上のような形でNetlify CMS上からコンテンツを編集・更新するとリポジトリのデータが書き換わり, Netlify上でStatic Siteがbuildされた後に, Netlify上にデプロイされ自動でホスティングまでしてくれる.
4. Gatsby設定
   今回はブログ用のテンプレートを使用するので, 以下のコマンドを実行して大枠を作成する.

```bash
npx gatsby new gatsby-starter-blog https://github.com/gatsbyjs/gatsby-starter-blog
```

そしたらこれをgitに追加してgithubのリポジトリに上げる. 一旦これでGatsby側の設定は終わり.

5. Netlify設定
   Netlifyのホームページを開きcreate new Teamでteamを作成したあと,以下の画面から上で作成したgithubリポジトリを選択する.
<img src="https://storage.googleapis.com/zenn-user-upload/d98598f40d20-20230304.png" width="100%">

   このimportが終わるとbuildの設定をおこなうことができるので, gatsby buildをコマンドとして設定する. Gatsbyがnode.jsの18.00以上出ないと上手く動かないので, ディレクトリ直下に.nvmrcファイルを作成して

```
18.0.0
```

と記述しておく方が無難
また依存関係を解決できないエラーが発生するようなら
.npmrcファイルを作成して

```
legacy-peer-deps=true

```

を記述しておくことで, peerDependenciesの依存解決をv6以前と同じようにやってくれる.

ここまでで, 実際にNetlifyにホスティングされたサイトを閲覧できるところまでできた.

<img src="https://storage.googleapis.com/zenn-user-upload/d9a91fc3edcd-20230304.png" width="100%">
この時点では, content/blog配下のmarkdownファイルを追加することで自動でbuildが走り, 本番環境で更新することができる. しかしこれではまだCMSと言えるような段階ではないので管理画面からコンテンツの追加ができるようにする

6. 管理画面からコンテンツを追加・更新できるようにする．

6.1 Gatsbyプロジェクト上でNetlifyのプラグインをインストール

```
npm install netlify-cms-app gatsby-plugin-netlify-cms
```

6.2 
gatsby-config.js内のpulugin: \[]の中に`gatsby-plugin-netlify-cms`を追加.

6.3 static/adminディレクトリ内にvonfig.yamlファイルを作成して設定

```yaml
backend:
  name: github
  repo: {githubユーザー名}/{リポジトリ名}
  accept_roles:
    - admin
    - editor
  branch: main
media_folder: static/assets
public_folder: /assets

collections:
  - name: blog
    label: Blog
    folder: content/blog
    create: true
    slug: "{{year}}-{{month}}-{{day}}-{{slug}}"
```

以上の設定が終わってたら一旦pushしてbuildが成功するか確認する.

成功すると
サイトのURL/adminを開くと管理画面が表示されるようになるはず.

6.4 ここで管理画面に認証をかけるために, GithubAuthを使用する.

* https://github.com/settings/developers ここからNew OAuth Appをクリック
* app名は適当につけていい
* create a new client secretをクリックしクリップボードに保存
* Netlify上のsite settingでaccess controlをクリック→OAuthの中のinstall providerをクリックして保存したclient secretとclient idを貼り付ける.

こんな感じで BlogサイトURL/adminに入るとログインが行われるようにできる.
<img src="https://storage.googleapis.com/zenn-user-upload/dbc0f93a2fda-20230305.png" width="100%">

7. Typescriptの導入
   元々とってきたテンプレートがJSになっているのでTypescriptを導入する．
   簡単に導入できるのでやってしまう. 
7.1 プラグインのインストール

```
npm install gatsby-plugin-typescript gatsby-plugin-typegen

```

7.2 gatsby-config.jsにプラグイン定義追加

```
plugins: [
`gatsby-plugin-typescript`,
`gatsby-plugin-typegen`,

```

7.3 tsconfig.jsonをベースディレクトリに追加してよしなに設定する.

```json
{
    "compilerOptions": {
        "target": "esnext",                                  /* Set the JavaScript language version for emitted JavaScript and include compatible library declarations. */
        "lib": ["dom", "esnext"],                                        /* Specify a set of bundled library declaration files that describe the target runtime environment. */
        "jsx": "react",                                /* Specify what JSX code is generated. */
        "module": "esnext",                                /* Specify what module code is generated. */
        "moduleResolution": "node",                       /* Specify how TypeScript looks up a file from a given module specifier. */
        "esModuleInterop": true,                             /* Emit additional JavaScript to ease support for importing CommonJS modules. This enables `allowSyntheticDefaultImports` for type compatibility. */
        "forceConsistentCasingInFileNames": true,            /* Ensure that casing is correct in imports. */
        "strict": true,                                      /* Enable all strict type-checking options. */
        "skipLibCheck": true                                 /* Skip type checking all .d.ts files. */
    },
    "include": ["./src/**/*", "./gatsby-node.ts", "./gatsby-config.ts", "./plugins/**/*"]
}
```

7.4 ファイル拡張子変更
もともと.jsのファイルをts, tsxに変更する.

7.5 型を暫定的につける.
pages/index.tsx内の
BlogIndex: React.FC<any>
const posts: any\[]
で一旦エラーを回避, あとで片付けはちゃんとしよう．

```tsx
const BlogIndex: React.FC<any> = ({ data, location }) => {
    const siteTitle = data.site.siteMetadata?.title || `Title`
    const posts: any[] = data.allMarkdownRemark.nodes

    if (posts.length === 0) {
        return (
            <Layout location={location} title={siteTitle}>
                <Bio />
                <p>
                No blog posts found. Add markdown posts to "content/blog" (or the
                directory you specified for the "gatsby-source-filesystem" plugin in
                gatsby-config.js).
                </p>
            </Layout>
        )
    }
```

404.tsxでも同じように変更する.
	
7.6 ビルドしなおす．

```bash
gatsby clean && gatsby build
```

ビルドが通ったら完成.

8. 独自でホスティング
   独自でホスティングさせたい場合, 自分でビルドして静的サイトファイルを作成して, S3などに保存させるようなことができる.
   流れ的には以下のようにできそう,
   <img src="https://storage.googleapis.com/zenn-user-upload/b2e211c99b01-20230305.png" width="100%">

コンテンツを更新すると, githubのリポジトリのcontents/が更新されるので更新が行われるごとにgithubactionsを走らせてS3にデプロイすれば良さそう.

ただわざわざs3でホスティングさせる意味が見出せないので今回はやらない. 他のHeadless CMSで独自でホスティングが必要な場合は
<img src="https://storage.googleapis.com/zenn-user-upload/c6fec9b041e3-20230305.png" width="100%">
こんな感じに組めば良さそうだなと.