=====================================================
SpringのDebugメモ
=====================================================
本章ではSpring環境の構築およびアプリケーション開発時に詰まった内容を備忘録として残す。

Talend API Tester
==================

java.lang.IllegalArgumentException
-----------------------------------

HTTP メソッド名 [0xxxxxxxxx] に無効な文字が含まれています。
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. sourcecode:: console
   :linenos:

   java.lang.IllegalArgumentException: HTTP メソッド名[0x160x030x010x020x000x010x000x010xfc0x030x
   030x97_;&0x860xa60xeb0xdc[0xa90x040xfb0x000x130x070x84.0xd00x0e0x920x9aZ0xc00xe20xca0x050x06v
   70xb80xbe8 ] に無効な文字が含まれています。HTTP メソッド名は決められたトークンでなければなりません。

httpメソッドでの通信を開放している場合に、httpsでリクエストを投げている可能性がある。
Talend API Testerのプロトコルがhttpになっていることを確認する。

