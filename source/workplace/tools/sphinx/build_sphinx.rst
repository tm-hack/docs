Sphinxの環境構築
================
本章では、Sphinxの環境構築について記述する。

前提事項
---------
構築に使用した環境は以下の通り。

* OS：Ubuntu 20.04.3 LTS（WSL2）
* python 3.8.10
* pip 20.0.2

Sphinxの導入
--------------
1. pip3を利用してSphinxのインストールを行う。

.. sourcecode:: bash
   :linenos:

   $ sudo pip install sphinx

2. Sphinxの拡張テーマである（sphinx_rtd_theme）をインストールする。

.. sourcecode:: bash
   :linenos:

   $ sudo pip install sphinx_rtd_theme

Sphinxプロジェクトの作成
--------------------------
1. sphinx-quickstartコマンドでsphinxのプロジェクトを作成する。

.. sourcecode:: bash
   :linenos:

   $ sphinx-quickstart

2. 拡張テーマ（sphinx_rtd_theme）の適用のため、conf.pyを修正する。

.. sourcecode:: python3
   :linenos:

   # -- Project information -----
   import sphinx_rtd_theme

   # -- Options for HTML output ---
   html_theme = 'sphinx_rtd_theme'
   html_static_path = [sphinx_rtd_theme.get_html_theme_path()]

3. Sphinxプロジェクトのビルドを行う。

.. sourcecode:: bash
   :linenos:

   $ make html

ビルドが正常終了すれば、build配下にindex.htmlおよびhtml等のstaticファイルが作成される。

ホットリロードの導入
--------------------
1. ホットリロードを実現するためにsphinx-autobuildをインストールする。

.. sourcecode:: bash
   :linenos:

   $ pip install sphinx-autobuild

2. sphinx-autobuildを起動する。

.. sourcecode:: bash
   :linenos:

   $ sphinx-autobuild source build

source配下のファイルが変更されると自動でビルドが動き、build配下にhtmlファイル等が格納される。
また、sphinx-autobuildを起動した後に\ http://127.0.0.1:8000/ \にアクセスすると、ビルドされたHTML文書にアクセスできる。
HTML文書が変更されると、ブラウザが自動更新される。

sphinx-autobuildを終了するには、Ctrl+Cを入力する。


Github Pagesとの連携
--------------------
1. ビルド成果物の出力先フォルダを変更するために、Makefileに追記する。

.. sourcecode:: Makefile
   :linenos:

   # Put it first so that "make" without argument is like "make help".
   html:
	 @$(SPHINXBUILD) -b html "$(SOURCEDIR)" "$(BUILDDIR)/docs"

2. Github上でリポジトリを作成した後、プロジェクトのセットアップを行う。
first-commitを行わないと、gh-pagesブランチの作成ができなかった。

.. sourcecode:: bash
   :linenos:

   $ mkdir docs
   $ cd docs
   $ echo "# docs" >> README.md
   $ git init
   $ git add README.md
   $ git commit -m "first commit"
   $ git remote add origin https://github.com/user/xxxxx
   $ git push origin main

3. 作成したSphinxプロジェクトをGithubPagesと連携する。

.. sourcecode:: bash
   :linenos:

   $ cp ../sphinx/. .
   $ git branch gh-pages
   $ git checkout main
   $ git add .
   $ git commit -m "setup sphinx"
   $ git push origin gh-pages

4. Github上で公開設定を行う。
Githubのリポジトリからgh-pagesブランチの公開設定を行う。Sourceにはdocs配下のファイルを設定する。
