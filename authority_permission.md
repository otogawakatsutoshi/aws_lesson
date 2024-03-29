
# 権限管理

## なぜやるか？

1. オペミスを防ぐため
2. 個人情報などの機密性
3. 内部犯対策(攻撃したり、データ盗んだり)

## 基本

アプリケーションの安全性、機密性のために、
ユーザーごとに
編集権限、新規作成権限、削除権限、管理者権限
を与えること。

ただ、自分で考えるとそれなりにダウンロード数があるライブラリ作成者の
ツヨツヨエンジニアでも適切な権限を与えるのはかなり難しい。

## じゃあどうするか？

どのアプリケーションやサービスでも、 作成者側が特に何も考えなくても、
安全に使える手段を提供しているのでそれを使うというのが基本。

## osで例えると

### linux

postgresql dbを扱うために
公式がpostgresqlユーザーを作成しました。
postgresqlの設定やデータを直接いじるならこれを使ってください。
となっている。

dockerも同じようにdocker全体のサービスの停止、起動はdockerユーザー
で行ってください。そのための権限は設定されています。
dockerの各々のインスタンスはdockerグループに所属したユーザーがその
インスタンスの立ち上げと停止ができるように権限が設定されています。

これrらはlinuxファイルのパーミッションをpostgresqlユーザーに適切な
権限を与えるように設定してあるからだが、これは一般ユーザーは知らなくても良い。

### windows

remote desktop
公式がRemote desktopのグループに所属したら、外部からそのマシンのそのユーザーに
対してログインできる権限が与えられる。

linuxよりも隠蔽されているが、公式が用意しているものを使わしてもらっているのは一緒。

## 我々が行う権限管理は

我々サービスやアプリケーションを使う開発者は
公式が作った権限が与えられているグループに所属させることによって、
適切な権限を設定している。

## awsでは？

### iamとRoleのみの時代の場合

awsの場合はポリシーが一つ以上のまとめられたオブジェクトがある。
これをポリシーと言い、これを複数個まとめてRoleを作る。
このRoleを自分が作るグループに割り当ててユーザーに権限をあてていた。

lambdaがS3バケットの情報が欲しいなら、S3にIAM Roleを割り当てる

ただ、これだと一つのルートユーザーの作ったプラットフォームの中に、

複数のグループが剥き出しの状態で存在している状態

1. 部署ごとにいくら使ったかわかならい。
2. 部署ごとにグループがあって、部署ごとに絶対使わないサービスがある
3. 常に自分が所属しているグループの権限で実行することになる。(閲覧だけしたくても常にそのサービスに対して管理者権限で触っているのと一緒の状態。)
4. 自分で作成したRoleとAWSが作成したRoleとごちゃ混ぜ。 -> 特定のAWSサービスを有効にしたときに謎のRoleが作成されているという不思議な現象が起きる。

### aws organizations

awsユーザーごとに独立した環境を提供している。

部署ごとに使わない権限があるなら、それをorganizations unitsごとに設定すると良い。
請求はいく。

1. rootユーザー単位でなくて、AWSユーザー単位に環境とサービスをまとめれる。
2. AWSユーザーごとに環境を隠蔽できる
3. awsユーザーを削除したら、環境ごとまるまる削除。
4. rootユーザーの環境に請求がいく。いくら使ったかはCost Explorerから確認できる。

ただ、AWSユーザーごとに独立した環境といっても、
一つのAWSユーザーを複数人で作業するから、 問題が発生する。
誰がやったのかわからない。

### aws identity center

aws organizations で作成したawsユーザーに

aws identity centerで作成したユーザー(ユーザーというよりidに近い)を割り当てることで
一つのAWSユーザーを複数の人間で使えるようにする。

例えば、社内エンジニアグループとして
Aさん、Bさん、Cさんと所属させる。

社内のユーザーなので、PowerUserというiamユーザーの作成と、
新たにサービスを作成できない権限を与えておく。

これはPermission setsという
AWS側がええ感じのポリシーを組み合わせて、安全に運用できるものを用意しているのでそれを使う。

このPmisession setsを割り当てたグループを社内の掲示板サーバー環境のAWSユーザーに割り当る。

これはidentity userにグループを割り当てる方法なので当然、

identity centerのユーザーでログインする場合は、identity center用のポータルサイト
が作成されているので、そのユーザーでログインした後に
与えられた*permission setsを切り替えて*必要なpermissionだけでコンソールを操作するようにする。


つまり、最新のやり方でやると*IAMロールさえできるだけ触らないような設計*になっている。

#### よく使うパーミッション

##### AdministratorAccess

開発初期メンバーとマネージャー用。

AdministratorAccessとあって重そうな権利に見えるが、
これがないとlambdaやapp runner作成時に適切なポリシーを持ったロールが
作れない、S3バケットが作れないため、マネージャーだけにしかあげない権利とかにはできない。
よって結構使う。

ガワの部分だけ作って、他の人に任せるとか。

##### SystemAdministrator

AdministratorAccessからIAMRoleの権限を除いたもの。
基本的にPowerUserAccessで権限が足りなかったらこれを使う。

##### PowerUserAccess

PowerUserAccessというと強権限に見えるが、
一般開発者向けの権限。
簡単な設定の編集や作成など。
S3Bucketの設定触ったりとか。

##### ReadonlyAccess

編集権限が無いがコンテンツは見れるもの。
今の状態を見たいときに使う。

##### ViewOnlyAccess

ReadOnlyからコンテンツを見れる権限を除外したもの。
外部の人間にシステム説明するときに使ったり、
AWS初学習者確認向けに使う。
