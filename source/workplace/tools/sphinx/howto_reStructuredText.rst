reStructuredTextの歩き方
=========================
Sphinxの文章はreStructuredTextというマークアップ言語で記述できる。
自身は普段からmarkdownでブログ等を記載することが多いので、
markdownの書式とreStructuredTextの書式を比較しながら記す。
なお、詳細は以下のリンク先が参考になる。

`study sphinx <https://planset-study-sphinx.readthedocs.io/ja/latest/04.html>`_

見出しの書き方
---------------

reStructuredText
^^^^^^^^^^^^^^^^

.. sourcecode:: rst
   :linenos:
   
   大見出し
   =======

   中見出し
   --------

   小見出し
   ^^^^^^^^

markdown
^^^^^^^^^

.. sourcecode:: bash
   :linenos:
   
   # 大見出し
   ## 中見出し
   ### 小見出し


コードブロックの書き方
----------------------

reStructuredText
^^^^^^^^^^^^^^^^

.. sourcecode:: rst
   :linenos:
   
   .. sourcecode:: bash(対象の言語)
      :linenos:
      
      ls xxxxxxx
      cd xxxxxxx

markdown
^^^^^^^^^

.. sourcecode:: bash
   :linenos:
   
   ``` bash(対象の言語)
   ls xxxxxxx
   cd xxxxxxx
   ```



