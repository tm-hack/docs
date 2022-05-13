=====================================================
CNA（basic）
=====================================================
本章では「クラウドネイティブアプリケーションの基本」を実施時の作業メモを残す。
参考サイトは川畑さんが執筆された
`こちらのサイト <https://news.mynavi.jp/techplus/series/AWS>`_
である。
昨年度の先輩の成島さんの
`メモ <https://github.com/narushimas/doc/blob/main/ECS_Spring.md#%E7%AC%AC6%E5%9B%9E-springboot%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E4%BD%9C%E6%88%90>`_
がかなり参考になりそう。

構成イメージは次の通り。

.. image:: https://news.mynavi.jp/techplus/article/techp4354/images/0311AWS04_001.jpg

以下の7ステップで進めていく。

#. VPC(Virtual Private Cloud)環境の構築
#. アプリケーションロードバランサ(ALB)の作成
#. Springを使用したコンテナアプリケーションの実装方法
#. Dockerコンテナの作成・DockerHubへのプッシュ
#. ECSクラスタの作成
#. ECSタスクの定義
#. ECSサービスの実行

ステップ1: VPC(Virtual Private Cloud)環境の構築
========================================================

用語整理
----------

.. csv-table::
  :header-rows: 1

  "用語","意味","参考URL"
  "リージョン","インスタンスを配置するための地域や国の単位","https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-availability-zones"
  "アベイラビリティゾーン","リージョンに内包される。物理的なデータセンターの単位と同義だと思われる。","https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-availability-zones"
  "NATゲートウェイ","NAT ゲートウェイは、ネットワークアドレス変換 (NAT) サービスです。NAT ゲートウェイを使用すると、プライベートサブネット内のインスタンスは VPC 外のサービスに接続できますが、外部サービスはそれらのインスタンスとの接続を開始できません。",https://docs.aws.amazon.com/ja_jp/vpc/latest/userguide/vpc-nat-gateway.html

作業内容
----------
* 自分に割り当てられたサブネットは10.2.34.0/24

1. VPCおよびサブネットのCDIRブロックの割り当て

構成に従い、インターネットアクセス用のサブネットと内部アクセス用のサブネットの組み合わせを2つ作成する。
自分に割り当てられたサブネットを踏まえ、第4オクテットの上位2bitで4ネットワークを分割する。

.. csv-table::
  :header-rows: 1

  "区分","Name","CIDRブロック"
  "VPC", "ma-masuda-vpc", 10.2.34.0/24
  "サブネット","ma-masuda-subnet-public1-ap-northeast-1a", 10.2.34.0/26
  "サブネット","ma-masuda-subnet-private1-ap-northeast-1a", 10.2.34.64/26
  "サブネット","ma-masuda-subnet-public2-ap-northeast-1c", 10.2.34.128/26
  "サブネット","ma-masuda-subnet-private2-ap-northeast-1c", 10.2.34.192/26


2. VPC、サブネット、カスタムルートテーブルの作成

`VPCのダッシュボード <https://ap-northeast-1.console.aws.amazon.com/vpc/home?region=ap-northeast-1#vpcs:>`_
より「VPCを作成」ボタンを押下する。
「1. VPCおよびサブネットのCDIRブロックの割り当て」で作成した内容を元に各項目を設定し、作成ボタンを押下する。
なお、サブネット等も同時に設定するため、画面上部の「VPC、サブネットなど」を選択して作業を進める必要があるため注意が必要。

インターネットゲートウェイは1hあたりで料金がかかるようなので、使わない時は停止する。

.. csv-table::
  :header-rows: 1

  "項目","設定内容", "留意事項"
  "名前タグの自動生成", "ma-masuda","各nameの接頭語にma-masudaがつく"
  "IPv4 CIDR ブロック", 10.2.34.0/24
  "IPv6 CIDR ブロック",  CIDR ブロックなし
  "テナンシー", "デフォルト"
  "アベイラビリティゾーン", "2","AZはap-northeast-1a、ap-northeast-1cとした"
  "パブリックサブネットの数","2","CIDRブロックについては上記で設計した内容を設定した"
  "プライベートサブネットの数","2","CIDRブロックについては上記で設計した内容を設定した"
  "NATゲートウェイ","1AZ内","構成図に含まれているため設定した"
  "VPCエンドポイント","なし","構成図に含まれていないためなしとした"
  "DNSオプション","どちらもチェック","外部アクセスのDNSではなく内部アクセス時に使用するDNSであるため、有効が必須のよう"

メモ
-----
* サブネットはパブリックとプライベートに分割する構成がよく用いられる。
* 分割する理由は以下の２点である。

 * パブリックサブネットは外部アクセス（インターネット）、プライベートサブネットは内部アクセス（VPC内のAPサーバ等）とアクセス制御を行う。
 * プライベートサブネットではDBへの接続やオンプレミスとVPNアクセスするサーバを配置し、セキュリティを向上する。


ステップ2: アプリケーションロードバランサ(ALB)の作成
=====================================================

用語整理
-----------

.. csv-table::
  :header-rows: 1

  "用語","意味","参考URL"
  "ELB(Elasstic Load Balancer)","AWSが提供するロードバランサーのサービス名","https://aws.amazon.com/jp/elasticloadbalancing/features/#compare"
  "ALB(Application Load Balancer)", "HTTP, HTTPS, gRPCベースのルーティングを行える。L7スイッチが備える機能に相当する。"
  "NATゲートウェイ","NAT ゲートウェイは、ネットワークアドレス変換 (NAT) サービスです。NAT ゲートウェイを使用すると、プライベートサブネット内のインスタンスは VPC 外のサービスに接続できますが、外部サービスはそれらのインスタンスとの接続を開始できません。",https://docs.aws.amazon.com/ja_jp/vpc/latest/userguide/vpc-nat-gateway.html
  "NLB(Netowork Load Balancer)", "TCP, UDP, TLSベースのルーティングを行える。L4スイッチが備える機能に相当する。"
  "GLB(Gateway Load Balancer)", "IPベースのルーティングを行える。"
  "CLB(Classic Load Balancer)", "1世代前のロードバランサーサービスになる。具備するサービスが劣る？（精査が必要）"
  "セキュリティグループ", "セキュリティグループは、インスタンスの仮想ファイアウォールとして機能し、インバウンドトラフィックとアウトバウンドトラフィックをコントロールします。",https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-security-groups.html
  "インバウンドルール", "インスタンスへの受信トラフィックをコントロールする"
  "アウトバウンドルール", "インスタンスからの送信トラフィックをコントロールする"
  "HTTP1", "1997年に策定され、現在まで広く用いられている歴史の長いプロトコル。多重リクエスト/レスポンス時のTCPの多重化など幾つかの欠点を抱えている。"
  "HTTP2", "2015年5月に標準化され、現在では多くのWebサイトやWebアプリケーションがHTTP/2を利用している。1つのTCP接続で多重リクエスト/レスポンスを行えるなど、HTTP1の課題を解決したプロトコルになっている", "https://qiita.com/mogamin3/items/7698ee3336c70a482843#%E3%83%90%E3%82%A4%E3%83%8A%E3%83%AA%E3%83%99%E3%83%BC%E3%82%B9"

作業内容
-----------
`EC2のダッシュボード <https://ap-northeast-1.console.aws.amazon.com/ec2/v2/home?region=ap-northeast-1#LoadBalancers:sort=loadBalancerName>`_
からロードバランサーを指定して作成画面に入る。パブリック用とプライベート用の2種類のロードバランサーを作成する。

.. csv-table::
  :header-rows: 1

  "区分","Name"
  "パブリックロードバランサー","ma-masuda-public-alb"
  "プライベートロードバランサー","ma-masuda-private-alb"


セキュリティグループには以下を作成した。

.. csv-table::
  :header-rows: 1

  "区分","Name", "設定内容"
  "パブリックセキュリティグループ","ma-masuda-public-alb-sg", "外部からのインターネットアクセスを可能とするため、インバウンドルールとしてIP4,IP6共にHTTPの全IPを許可"
  "プライベートセキュリティグループ","ma-masuda-private-alb-sg", "内部のアクセスのみを可能とするため、インバウンドルールとしてVPC(10.2.34.0/24)からのアクセスのみを許可する"

ALBのターゲットグループには以下を作成した。
protocol versionにHTTP2を指定して作成したところ、ALBの候補に表示されなかったため、HTTP1で作り直した。

また、ヘルスチェック用のhtmlの名前はパブリックとプライベート両者ともに/backend-for-frontend/index.htmlとした。

.. csv-table::
  :header-rows: 1

  "区分","Name"
  "パブリックターゲットグループ","ma-masuda-public-alb-tg"
  "プライベートターゲットグループ","ma-masuda-private-alb-tg"

メモ
----
* コンテナアプリケーションの識別にはALBのパスルーティング機能を使ってコンテナアプリケーションを識別するため、ロードバランサーにはALBを指定する


ステップ3：Springを使用したコンテナアプリケーションの実装方法
=========================================================================

SpringBootアプリケーションをECSに配置するにあたり、

1. SpringBootを使ってExcutableJar実行形式のJavaアプリケーションを作成する
2. コンテナにアプリケーションを配置する

という流れをとる。こうすることで、簡易にコンテナ上でアプリケーションを実行できる。

作業内容
---------

記事の内容のみだと実装を進めていくことが難しいので、
必要に応じて
`川畑さんのリポジトリ <https://github.com/debugroom/mynavi-sample-aws-ecs>`_
を参照して進めていく。

プロジェクトのセットアップ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

以下の2つのプロジェクトをセットアップした。
セットアップについてはSpringInitializerのホームぺージに行かなくてもInteliJの新規プロジェクトから選択可能。

* 単純なAPIを持つアプリケーション(projectName: backend)
* HTMLを返すWebアプリケーション(projectName: bff)

記事と上記のリポジトリを参照して、backendとbffにそれぞれ以下のdependencyを追加した。

a. backend

* spring web
* Lombok
* spring-boot-configuration-processor

b. bff

* Lombok
* spring-boot-configuration-processor
* Thymeleaf
* spring-boot-maven-plugin

main配下のディレクトリ構成については上記のリポジトリを参考にそれぞれ以下の構成とした。


メモ
------

ステップ4：Dockerコンテナの作成・DockerHubへのプッシュ
=========================================================================

用語整理
----------

作業内容
---------

メモ
------

ステップ5：ECSクラスタの作成
=========================================================================

用語整理
----------

作業内容
---------

メモ
------


ステップ6：ECSタスクの定義
=========================================================================

用語整理
----------

作業内容
---------

メモ
------

ステップ7：ECSサービスの実行
=========================================================================

用語整理
----------

作業内容
---------

メモ
------


