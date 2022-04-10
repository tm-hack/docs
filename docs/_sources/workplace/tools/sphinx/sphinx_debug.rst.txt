=====================================================
SphinxのDebugメモ
=====================================================
本章ではSphinxの更新時に詰まった内容を備忘録として残す。

サイドバーをクリックすると子のツリーが閉じる
=================================================

原因
-----
親のindex.rstと子のindex.rstが1対1になっていると生じるようである。
問題が生じる場合のtree構造を以下に示す。

.. sourcecode:: bash
   :linenos:

   └── java
        ├── index.rst
        └── spring
            ├── article
            │   ├── debug.rst
            │   └── spring_tutorial.rst
            └── index.rst

解決方法
---------
この場合はspringフォルダに直接index.rstを格納し、もう一つ上の世代（祖父母）から子のindex.rstを参照する。
具体的な構造を以下に示す。javaフォルダ配下のindex.rstを削除している。

.. sourcecode:: bash
   :linenos:

   └── java
        └── spring
            ├── article
            │   ├── debug.rst
            │   └── spring_tutorial.rst
            └── index.rst