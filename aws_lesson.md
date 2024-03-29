## iamの資料

### aws cliのインストール

```bash
winget install Amazon.AWSCLI

winget install  Amazon.CopilotCLI
```

## iamとはなんぞや？

その前に権限管理ってなにやって事を

linux, windowsだと、権限どのように管理しているか？
これ自分で考えるのやめてください。

OSのサービスや、アプリケーション側があらかじめ
そのアプリケーションを安全に使うためのグループをちゃんと用意してあるので、そのグループに所属させる事で、ユーザーに権限を与える。

OSにはグループには所属しているがログインが許されていないユーザーがいる。
対話する権限を与えたくないから。daemonと言って、あらかじめ与えられている権限に従って、動作する。

gcpもgcp-developerみたいなグループがあらかじめ作成されていて、これは一般開発者向けの権限が既に割り当て

rhelならオンプレミスで使う事が多いから、ストレージ周りの操作の運用をするグループがあらかじめ作られている。

windows server だったら、バックアップの操作する用のグループがあらかじめ作られている。

Windowsだったら、Remote desktop usersに所属させると
リモートデスクトップでログインする権限を割り当てたことになる。

linuxだったら、dockerグループに所属させるとdockerのサービス自体のプロセスが見れるようになる。

ここら辺はOS作っている会社だったら、あらかじめ適切なグループが作られているので、それを私たち使って所属させたらええだけ。

じゃあユーザーの話したんですが、デーモンの場合はどうなるか？

oracleだったら、DBAグループとか、oracleグループとかあらかじめ作ってあるので、そのグループに所属させる事で適切な権限が勝手割り当てられる。

oracleのインストールした人とかならわかると思うですが、
oracle-preinstall.rpmみたいなパッケージが合って、あれはなにをしているのか？というと適切な権限をもった、oracleというユーザーを作った上で、カーネルパラメーターをoracle dbで動作できる形にしている。

会社の内部で完結するなら、基本的に作成済みのグループに所属させるだけで良い。

もし、会社としてSierだったりして、プライム以外はどこどこのサービスのリソースだけで良いけど、それ以外の会社には触らせたくないなら、
この権限を参考に、新しくグループを作って所属させたら良い。
所属させて、権限を付与した後に特定の権限を抜くのは難しい。

これをできるようにしたのが　linuxのaclなんですが、 redhatさんも基本的に使わないので忘れていいです。

storage managerというstorage関係の前権限を与えたやつから、権限を抜くグループや権限を追加して運用する。
みたいな事をしているところは聞いた事ないと思います。

## つまり、基本的に

権限管理を行うということは

運用時に

1. 適切な権限を付与済みのグループにユーザーやサービスを所属させていくことで権限を与えていく。
2. ユーザーに直接権限を与えない(管理が難しい)。
3.. 既存のグループだと権限が過剰な場合は新たにグループを作ってそこに所属させる。

ってのが基本
権限管理っていったらここを触るのが必要

これはgcpでも同じなんですが、

クラウドを大きなLinuxサーバーと思って適切な権限を充てる

## 具体的に使う

よくあるパターン
### 自動化

動きとしてはOSのサービスが近い。

1. lambda function
2. ec2
3. ecs
4. app runner

に適切な権限を与える

自動で動かすので、必要な権限なんですか？
って考えたら、

絶対必要なのは

1. ログを書き込む権限ですね。

linux サーバーでもapache動いてます！
ログ？書いてないです！
とかならいやいや言わなくてもやってよってなると思います。

2. ストレージオブジェクトへの書き込み、読み込み、削除権限

画像のアップロードとか、本人確認書類を一度アップロードさせて、


こいつらを変更するエンジニアに必要な権限は？って言ったら、

developerっていうグループ作って

1. ログを見る権限
2. 書くサービスの編集、作成権限


マイクロサービスとして切り離すなら、
lambda functionにapiを叩きにいく権限を与えると

運用というか監視だけなら

operator

1. ログを見る権限
2. サービスの参照権限

だけで良い。

ec2に適切な権限を与える。

ログインしないけど、そのサービスから何かを変更するわけだから、

## とりあえず

とりあえず、rootユーザーでログイン

あらかじめ、iamでユーザーで


ネットワークACL

基本は許可

なんでそういう作りになっているかっていうと、

ルーターの作りを真似ているからで、

ルーターはネットワーク同士を繋ぐ事が仕事なので、
繋げたら動くというのが基本それに合わせている。

ネットワーク同士でやり取りしたらここの部分が通信量を多いとかで
で動きが遅くなるとかある。

パケットが入ってくるのはOKだけど、出ていくのは同じ道にしない。
これには理由があって、

セキュリティのために、攻撃されたら出て行かないようにする。
リバースシェルって言って、

攻撃して、sshで外部に接続できるユーザーを乗っ取ったら、
攻撃サーバーに繋ぎに行くように設定するというのがある。

これ以上データが出ていなかないように、
とりあえず、出口を塞ぐ。

セキュリティグループはさっきいった、クラウドを巨大なLinuxサーバーと見立てたものなので、基本は拒否で権限を追加していく形になる。


