=====================================================
SpringBoot
=====================================================
本章ではSpringBootのチュートリアル時の作業メモを残す。
実施したチュートリアルは2019-12-27に策定されたものである。

前提事項
========
検証環境

* Windows11
* JDK 17
* Spring Boot 2.6.7

注意事項

* dozer-spring-boot-starterについては依存関係が解決できないため導入しない

Spring Bootの概要
====================

SpringとSpringBootの違い
==========================


プロジェクトの初期化
======================
Spring initializrに必要項目を入力して、generateボタンを押下すると、
zip形式のプロジェクトファイルがダウンロードできる。

https://start.spring.io/


Spring Bootのコマンド
========================
Spring Bootアプリケーションを実行する。

.. sourcecode:: bash
   :linenos:

   mvn spring-boot:run


デバッグメモ
======================

validationの有効化
--------------------
resourceに@NotNullなどのvalidationを付与する場合は、
pom.xmlに以下のbean定義を追加する必要がある。

.. sourcecode:: xml
   :linenos:

   <dependency>
      <groupId>org.hibernate.validator</groupId>
      <artifactId>hibernate-validator</artifactId>
   </dependency>


