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

ALBの作成
=========
パブリックサブネットを構築してよいのか不明だったので、いったん飛ばす。

メモ
----
* AWSのロードバランサーはELB(Elasstic Load Balancer)というサービス名で提供されている。
* ELBには4種類の製品が提供されている。(https://aws.amazon.com/jp/elasticloadbalancing/features/#compare)
* ALB(Application Load Balancer)はHTTP, HTTPS, gRPCベースのルーティングを行える。L7スイッチが備える機能に相当する。
* NLB(Netowork Load Balancer)はTCP, UDP, TLSベースのルーティングを行える。L4スイッチが備える機能に相当する。
* GLB(Gateway Load Balancer)はIPベースのルーティングを行える。
* CLB(Classic Load Balancer)は１世代前のロードバランサーサービスになる。具備するサービスが劣る？（精査が必要）


Springを使用したコンテナアプリケーションの実装方法
====================================================

メモ
-----
* ECSを用いると単にDockerをEC2上で実行する場合に比べてクラスタのポート管理やコンテナ実行がAWSのマネージドサービスになるため、コンテナアプリケーション間のサービス連携はALBを介して行う方がよい（詳しく知りたい）
