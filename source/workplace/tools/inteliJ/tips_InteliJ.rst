======================
InteliJのTips
======================
InteliJのTipsについて記述する。

コマンドラインからInteliJのプロジェクトを開く
===============================================
コマンドラインからInteliJのプロジェクトを開くための手順を記す。

手順
----
直近で開いていたプロジェクトを開く。

.. sourcecode:: bash
   :linenos:
   
   $ idea64.exe

現在のカレントディレクトリのプロジェクトを開く。

.. sourcecode:: bash
   :linenos:
   
   $ idea64.exe .

ディレクトリを指定して、プロジェクトを開く。

.. sourcecode:: bash
   :linenos:
   
   $ idea64.exe /home/project

実行構成にtomcatを追加する
===============================================
作成したプロジェクトをtomcatにデプロイ、実行するため手順を記す。

手順
----

#. `公式サイト <https://tomcat.apache.org/download-90.cgi>`_ より任意のバージョンのtomcatをインストールする。

#. インストールしたtomcatを任意のフォルダに展開する。

#. ログ出力時の文字コードを変更する。インストール時の設定のままだと文字コードがUTF-8であり、コマンドラインへのログの出力時に漢字かなが文字化けするため修正する。

   apache-tomcat-9.0.62>conf>logging.properties

   .. sourcecode:: properties
      :linenos:
   
      java.util.logging.ConsoleHandler.formatter = org.apache.juli.OneLineFormatter
      java.util.logging.ConsoleHandler.encoding = UTF-8 → SJIS

#. InteliJで以下の作業を実施する。

 * アプリケーションを実行する（Alt+Shift+F10）

 * 実行構成の編集をクリックする

 * 新規構成の追加でtomcat（ローカル）を選択する

 * 「アプリケーションサーバ」の「構成」をクリックし、インストールしたtomcatを選択する

 * そのほかの項目も入力する

 * 「実行」をクリックして、デプロイと実行が始まる

System.outを省略して打つ
===========================
「sout」と打てばSystem.outが出力できる。
