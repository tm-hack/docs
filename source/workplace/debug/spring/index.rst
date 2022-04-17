=====================================================
SpringのDebugメモ
=====================================================
本章ではSpring環境の構築およびアプリケーション開発時に詰まった内容を備忘録として残す。

Talend API Tester
==================

HTTP メソッド名 [0xxxxxxxxx] に無効な文字が含まれています、とエラーを出力する
------------------------------------------------------------------------------

事象および原因
^^^^^^^^^^^^^^

.. sourcecode:: console
   :linenos:

   java.lang.IllegalArgumentException: HTTP メソッド名[0x160x030x010x020x000x010x000x010xfc0x030x
   030x97_;&0x860xa60xeb0xdc[0xa90x040xfb0x000x130x070x84.0xd00x0e0x920x9aZ0xc00xe20xca0x050x06v
   70xb80xbe8 ] に無効な文字が含まれています。HTTP メソッド名は決められたトークンでなければなりません。

httpメソッドでの通信を開放している場合に、httpsでリクエストを投げている可能性がある。

対処内容
^^^^^^^^^^^^^^

Talend API Testerのプロトコルがhttpになっていることを確認する。
httpsになっていればhttpに変更する。

InteliJ
==================

Mapper XMLファイルが読み込まれず、「Could not autowire.」とエラーを出力する
--------------------------------------------------------------------------------

事象および原因
^^^^^^^^^^^^^^
.. sourcecode:: console
   :linenos:

   Could not autowire. No beans of 'AccountRepository' type found.

resources配下にMapper XMLファイルを作成して、MyBatisよりRepositoryを自動出力している場合に「Could not autowire.」とエラーを出力する。
ただ、アプリケーションを実行すると正常に動作する。


原因は、MyBatisを用いてRepositoryを作成する場合、IDEの静的解析では実体クラスが作成されておらずIDEが実体クラスを検知できないため。

対処内容
^^^^^^^^
（2022/04/17 検証中）
対象のRepositoryインタフェースに@Repositoryアノテーションを付与することで、エラーを回避できる。

