TERASOLUNA Server Framework（REST編）
=====================================================
本章ではTERASOLUNA Server Framework（REST編）のチュートリアルを実施時の内容をまとめる。

前提事項
--------
検証環境は以下の通りである。

* Windows11
* JDK 17.0.2
* apache-tomcat 9.0.55

また、本ページで掲載している図については `TERASOLUNA Server Framework <http://terasolunaorg.github.io/guideline/current/ja/index.html>`_ から引用している。

SpringMVCを利用したRESTful Web Serviceの開発
--------------------------------------------------
Spring MVCの機能を利用してRESTful Web Serviceを開発した場合のアプリケーション構成を以下に示す。

.. image:: https://terasolunaorg.github.io/guideline/current/ja/_images/RESTOverviewApplicationConstitutionOnSpringMVC.png

DomainオブジェクトとResourceオブジェクトを分ける意義
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
チュートリアルを実施していて、DomainオブジェクトクラスとResourceクオブジェクトを分ける意義について参考になったので、
ドキュメントの記述を引用する。平たく言うとアプリケーション層とドメイン層のロジックを分離し、保守性を高めるため。

   Domainオブジェクトはビジネスを行う上で必要な資源や、ビジネスを行っていく過程で発生するものを表現する。
   Resourceオブジェクトはクライアントとの入出力で使用するUI上の情報を表現する。

   両者を混同するとアプリケーション層の影響がドメイン層におよび、保守性を低下させる原因となる。
   DomainObjectとResourceクラスは別々に作成し、Dozer等のBeanMapperを利用してデータ変換を行うことを推奨する。

