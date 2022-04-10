======================
GitのTips
======================
GitのTipsについて記述する。

git cloneしたリポジトリを別リポジトリにpushする
===============================================
git cloneしたリポジトリを別リポジトリとして管理するための方法を記す。
commit履歴は前のリポジトリから引き継がれる。
今回はTERASOLUNAチュートリアル用にtodo-RestAPIリポジトリを作成するため、
todo-MyBatis3をclone元とした。

手順
----
1. プロジェクト用のディレクトリを作成する。

.. sourcecode:: bash
   :linenos:
   
   $ mkdir todo-RestAPI
   $ ls
   drwxr-xr-x 1  todo-MyBatis3/
   drwxr-xr-x 1  todo-RestAPI/

2. 取得元のリポジトリからgit cloneを実行する。

.. sourcecode:: bash
   :linenos:
   
   $ cd todo-RestAPI
   $ git clone https://github.com/tm-hack/todo-MyBatis3

3. 新しいリポジトリを作成する。

Githubよりtodo-RestAPIリポジトリを作成する。

4. cloneしたリポジトリのremoteの接続先を新しいリポジトリに変更する。

.. sourcecode:: bash
   :linenos:
   
   $ cd todo
   # remote urlの確認
   $ git config remote.origin.url
   https://github.com/tm-hack/todo-MyBatis3
   $ remote urlの変更
   git remote set-url origin https://github.com/tm-hack/todo-RestAPI
   https://github.com/tm-hack/todo-RestAPI
   # first push
   $ git push origin main

変更を戻したい
================

addを取り消して、ファイルを編集前に戻す
----------------------------------------

「修正してaddをしたが、不具合を発見したため最後にcommitした状態まで戻したい」という場合の手順を以下に示す。

手順
^^^^
.. sourcecode:: bash
   :linenos:
   
   $ git long  #commitログを確認する
   $ git rm --cached -r .  #addを取り消す
   $ git checkout HEAD .  #最後にcommitした状態に戻す