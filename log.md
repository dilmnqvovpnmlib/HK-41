# Log

## docker-compose の開発環境と本番環境の切り替えの方法

### 背景

- 開発環境と本番環境で用いる Dockerfile が違うので、 docker-compose.yml の設定が違ってくる
- 開発環境と本番環境で別の Dockerfile と docker-compose.yml を書く必要がある

### 解決するための手段

- `-f` オプションを使ってそれぞれの環境毎の Dockerfile と docker-compose.yml を作成し、各環境にあったコマンドを打ち込む方法
  - ファイル数が多くて倍になる問題点があり、管理が面倒くさい
- 環境変数とシェルで制御する
- .env と docker-compose.yml を使って制御する
  - 各環境に .env を作成する必要が有る
- 公式が推奨する拡張 docker-compose.yml を作成する

- Dockerfile と docker-compose.yml が複数にまたがってしまうので、docker-compose.yml では、`context` を用いてパスを調整する。
- ベースとなる設定は `dokcer-compose.yml` に記述し、開発環境特有の設定は `dokcer-compose.dev.yml` し、 `-f` オプションと `make` コマンドで設定を上書きする。
- 本番環境に必要な設定は `dokcer-compose.override.yml` に記述し、`-f` オプションと `make` コマンドで設定を上書きする。
- 開発環境と本番環境で毎回ファイルを指定するのが煩わしいので、 Makefile と make コマンドを使用してコマンドを簡略化した。

- ポート番号を間違えていて、スタイルが崩れてた
- yarn start で react のアプリケーションが動いているポート番号と docker-compose が指定するポート番号が食い違っていた

### 参考

- 公式
- [本番環境での Compose の利用](https://matsuand.github.io/docs.docker.jp.onthefly/compose/production/)

- [docker-compose で.env で開発/本番環境を切り替える](https://qiita.com/ozroro/items/4fcf8811b47755b8d3c8)
- [docker-compose.yml の設定を開発環境と本番環境で切り替える](https://zukucode.com/2020/08/docker-compose-dev.html)
  - ポート番号などは反映されないので、`docker-compose.override.yml` を設定して上書きマージするように設定する。
- [docker-compose.yml でポートを開発環境と本番環境で切り替える](https://zukucode.com/2020/08/docker-compose-ports-dev.html)

## alpine の image で Nginx を起動する方法

- 以下の `Docker + Alpine Linux + nginx導入手順` を参考にして Nginx を起動させる。
- Ubuntu とは違って `/etx/init.d/nginx start` では起動せず、`rc-status` コマンドで起動させる。

### 補足

- alpine Linux のシェルは `/bin/sh` で、`/bin/bash` が存在しない。
- パッケージマネージャーは `apk` で、 `apk add` コマンドでライブラリを追加する。
- alpine Linux はシステムの初期化に `OpenRC` を使用している。これによって OS 起動時にサービスを開始させることができる。
- Linux でファイル名などに `rc` が付くのがあるが、これは `Run Commands` の略らしい。
- Ubuntu には `rc.local` はデフォルトではなく、`systemd` が使用されている。

### 参考

- [Docker + Alpine Linux + nginx 導入手順](https://l-w-i.net/t/docker/alpine_nginx_001.txt)
- [Alpine Linux Init System](https://wiki.alpinelinux.org/wiki/Alpine_Linux_Init_System)

## React で a タグの中に target='\_blank' を書くと ESlint に怒られたので、その理由と対処法

### 生じたセキュリティリスク

- target='\_blank' を指定した a タグのリングを開くと別窓でサイトが表示される。その元の遷移元のページにフィッシングサイトが表示されて id/password を入力させる。その後、もとのページにリダイレクトさせて情報を盗む。
- 解決策としては、`rel="noopener noreferrer"` を指定する。
- `noopener` は a タグのあるページから別のベージにリダイレクトしないような設定で、`noreferrer` はリンク先にリファラを送信しない設定である。

### 参考

- [React で a タグの中に target='\_blank'を書いたら ESLint に怒られた話](https://qiita.com/togana/items/e739e1ec943862a71757)
- [リンクのへの rel=noopener 付与による Tabnabbing 対策](https://blog.jxck.io/entries/2016-06-12/noopener.html)
- [a タグに付いている rel="noopener"って何？](https://ocws.jp/blog/post1974/)

## Let’s Encrypt 発行制限について

- https-portal で短期間においてコンテナを up と down を繰り返すと、証明書の発行制限にかかってしまう。したがって、慎重にデプロイすることが求められるので、ある程度ローカルの開発環境で検証を行っておく。
- もしかしたら、他の解決策もあるかもしれないので、調査する。
- [ ] https-portal の証明書発行制限とそのデバッグ

## react-vertical-timeline-component をインストールした時に生じた型定義に関するエラー

### 生じたエラー

- `react-vertical-timeline-component` の `.d.ts` ファイルが無いので、エラーが生じている。
- これは、もともと JS のライブラリで型が定義されていないので、エラーが出てしまっている。したがって、型定義されたファイルを npm でインストールする。
- すなはち、まとめると、JS で書かれたライブラリでも、拡張子 `.d.ts` の型定義ファイルさえ用意すれば TS にインポートしても使える。

```bash
/app/src/components/Main.tsx
TypeScript error in /app/src/components/Main.tsx(15,8):
Could not find a declaration file for module 'react-vertical-timeline-component'. '/app/node_modules/react-vertical-timeline-component/dist-modules/index.js' implicitly has an 'any' type.
  Try `npm install @types/react-vertical-timeline-component` if it exists or add a new declaration (.d.ts) file containing `declare module 'react-vertical-timeline-component';`  TS7016

    13 |   VerticalTimeline,
    14 |   VerticalTimelineElement,
  > 15 | } from "react-vertical-timeline-component";
       |        ^
    16 | import "react-vertical-timeline-component/style.min.css";
    17 | import { createFromIconfontCN } from "@ant-design/icons";
```

### 参考

- [型定義ファイル](https://typescript-jp.gitbook.io/deep-dive/type-system/intro/d.ts)
- [りあくと 4-6. モジュールの型定義]()

## よく忘れる Docker コマンド集

### コマンド一覧

- 停止しているコンテナを全て削除するコマンド

```bash
docker container prune
```

- 使われていない volume を削除する

```bash
docker volume prune
```

- コンテナが使っていない image を全て削除する

```bash
docker image prune
```

- 停止コンテナ、未使用イメージ、未使用ボリュームを削除する

```bash
docker system prune
```

- 特定のイメージを削除する

```bash
docker rmi <image name>

docker rmi golang
```

### none のイメージに関して

- 以下のような <none> のイメージは、`dangling image` と言って、同じ名前のイメージ (既存のタグを再利用する際に) 生成されてしまう。

```bash
<none>                  <none>              cd0b351829d3        27 hours ago         1.07GB
```

- これは、Docker では、同一名のイメージを作成することができないからである。
- 挙動としては以下のようになり、`dangling` (宙ぶらりん) となる。

1. 最新のビルドした結果生成されたイメージが CLI などで指定されたイメージ名となる。
2. すでに存在していた同一名称のイメージが、`<none>` という名称未設定の状態に置き換わる。

- 例
- 以下のように Dockerfile を定義して、`docker build -t go .` でビルドしたとする。

```Dockerfile
FROM golang

RUN echo Hello
```

- この時生成されたイメージは、`docker images` コマンドで確認すると、次のようになる。

```bash
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
go                      latest              e09e94ae5c5b        2 minutes ago       839MB
golang                  latest              279be524ea74        5 hours ago         839MB
```

- そこで、Dockerfile を書き換えて再度、`docker build -t go .` でビルドをする。

```Dockerfile
FROM golang

RUN echo Hello!
```

- この時生成されたイメージは、`docker images` コマンドで確認すると、次のようになる。

```bash
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
go                      latest              1af21b73aa5a        4 seconds ago       839MB
<none>                  <none>              e09e94ae5c5b        2 minutes ago       839MB
golang                  latest              279be524ea74        5 hours ago         839MB
```

- この結果から確認できるように、もともと `go` のタグがついていた `IMAGE ID` は `e09e94ae5c5b` であるが、Dockerfile を修正して再ビルドすると、`IAMAGE ID` が `e09e94ae5c5b` のタグ名は `<none>` になり、新しく生成されたイメージのタグ名が `go` になっていることがわかる。

- したがって、Docker を再ビルドすると、不要な none のイメージが生成され、ホストマシンの容量を圧迫することになるので、定期的にイメージを削除する必要がある。

### fs layer とは

### 参考

- [Docker に<none>:<none>なイメージが生まれてくる理由](https://suin.io/540)
- [Docker の不要なコンテナ・イメージを一括削除する方法](https://suin.io/537)
- [docker images に表示される＜ none ＞を消す。dangling](https://codechord.com/2019/08/docker-images-none-dangling/)

---

## 表題

### 生じたエラー or 課題

### 解消方法

### 参考
