GitのTips
======================
GitのTipsについて記述する。

git cloneしたリポジトリを別リポジトリにpushする
-------------------------------------------------

概要
^^^^
git cloneしたリポジトリを別リポジトリとして管理するための方法を記す。
commit履歴は前のリポジトリから引き継がれる。
今回はTERASOLUNAチュートリアル用にtodo-RestAPIリポジトリを作成するため、
todo-MyBatis3をclone元とした。

手順
^^^^
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

5. commit履歴の確認

履歴が引き継がれていることが分かる。

.. sourcecode:: bash
   :linenos:
   
   $ git log
   commit c7562fb28a4ee9edc8c16c70991ab3431e30d688 (HEAD -> main, origin/main)
   Author: takumi <xxxxxxxxxx>
   Date:   Fri Apr 8 20:45:20 2022 +0900

    merge aplication&fix mybatis property file

   ~~~~~~~~~~

