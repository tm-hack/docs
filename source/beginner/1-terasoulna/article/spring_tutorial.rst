TERASOLUNA Server Framework
=====================================================
本章ではTERASOLUNA Server Frameworkのチュートリアルを実施時の内容をまとめる。

前提事項
--------
検証環境は以下の通りである。

* Windows11
* JDK 17.0.2
* apache-tomcat 9.0.55

また、本ページで掲載している図については `TERASOLUNA Server Framework <http://terasolunaorg.github.io/guideline/current/ja/index.html>`_ から引用している。

TERASOLUNA Server Frameworkの概要
------------------------------------------
TERASOLUNA Server Frameworkは独自のフレームワークではなく、SpringFrameworkを中心としたOSSの組み合わせである。
ライブラリ構成を以下の図に示す。

.. image:: http://terasolunaorg.github.io/guideline/current/ja/_images/introduction-software-stack.png

Spirng MVCのアーキテクチャ
------------------------------------------
リクエストを受けてからレスポンスを返すまでのSpringMVCの処理フローを以下の図に示す。

.. image:: http://terasolunaorg.github.io/guideline/current/ja/_images/RequestLifecycle.png

ControllerとServiceの境界について
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
チュートリアルを実施していて、ControllerとServiceの境界について曖昧になりうると考えたので、
ドキュメントの記述を参照した。

   Controllerは以下の5つの役割を担う。
   
   #. リクエストを受け取るためのメソッドを提供する。（@RequestMapping）
   
   #. リクエストパラメータの入力チェックを行う。（@Validated）
   
   #. 業務処理の呼び出しを行う。（Sericeの呼び出し）
   
   #. 業務処理の処理結果をModelに反映する。（Viewからの処理結果を参照可能にする）
   
   #. 処理結果に対応するView名を返却する。（描画処理はJSPで行うことによる責任分離）
   
   Serviceは以下の2つの役割を担う。
   
   #. Controllerに対して業務ロジックを提供する。（Serviceはビジネスルールに関わる処理の実装に専念する）
   
   #. トランザクション境界を設ける。（@Transactional)
   
   ControllerとSericeで実装するロジックについては以下のルールに則ることを推奨する。
   
   #. クライアントからリクエストされたデータに対する単項目チェック、相関項目チェックはController側(Bean ValidationまたはSpring Validator)で行う。
   
   #. Serviceに渡すデータへの変換処理(Bean変換、型変換、形式変換など)は、ServiceではなくController側で行う。
   
   #. ビジネスルールに関わる処理はServiceで行う。業務データへのアクセスは、RepositoryまたはO/R Mapperに委譲する。
   
   #. ServiceからControllerに返却するデータ（クライアントへレスポンスするデータ）に対する値の変換処理(型変換、形式変換など)は、Serviceではなく、Controller側（Viewクラスなど）で行う。



O/R Mapperについて
--------------------
O/R Mapperはオブジェクト指向言語で書かれたアプリケーションと非オブジェクト指向であるデータベースやSQLの間インピーダンスミスマッチを解消させるためのフレームワークである。
TERASOLUNA Server Frameworkでは、O/R Mapperとして以下のいずれかを想定している。

* JPA（Java Persistance API）
* MyBatis

JPA（Java Persistance API）
^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Domainに@Entity等のアノテーションを付与することで設定する
* CRUDに関わる基本的なメソッドがEntityManagerから提供されるため、冗長なSQLの記載が不要になる
* 独自のクエリを発行する際は、SQLライクな独自のクエリ言語であるJPQL（Java Persistance Query Language）を使用する

以下にDomainの実装例を示す。

.. sourcecode:: java
   :linenos:

   package com.example.todo.domain.model;

   import java.io.Serializable;
   import java.util.Date;   
   import javax.persistence.Column;
   import javax.persistence.Entity;
   import javax.persistence.Id;
   import javax.persistence.Table;
   import javax.persistence.Temporal;
   import javax.persistence.TemporalType;   
   
   @Entity
   @Table(name = "todo")
   public class Todo implements Serializable {
       private static final long serialVersionUID = 1L;   
       
       @Id
       @Column(name = "todo_id")
       private String todoId;   
       @Column(name = "todo_title")
       private String todoTitle;   
       @Column(name = "finished")
       private boolean finished;   
       
       @Temporal(TemporalType.TIMESTAMP)
       @Column(name = "created_at")
       private Date createdAt;   
       public String getTodoId() {
           return todoId;
       }   
       public void setTodoId(String todoId) {
           this.todoId = todoId;
       }   
       public String getTodoTitle() {
           return todoTitle;
       }   
       public void setTodoTitle(String todoTitle) {
           this.todoTitle = todoTitle;
       }   
       public boolean isFinished() {
           return finished;
       }   
       public void setFinished(boolean finished) {
           this.finished = finished;
       }   
       public Date getCreatedAt() {
           return createdAt;
       }   
       public void setCreatedAt(Date createdAt) {
           this.createdAt = createdAt;
       }
   }

以下にRepositoryの実装例を示す。
なお、RepositoryImplは自動生成されるため作成不要である。

.. sourcecode:: java
   :linenos:

   package com.example.todo.domain.repository.todo;
   
   import org.springframework.data.jpa.repository.JpaRepository;
   import org.springframework.data.jpa.repository.Query;
   import org.springframework.data.repository.query.Param;
   
   import com.example.todo.domain.model.Todo;
   
   public interface TodoRepository extends JpaRepository<Todo, String> {
   
       @Query("SELECT COUNT(t) FROM Todo t WHERE t.finished = :finished")
       long countByFinished(@Param("finished") boolean finished);
   
   }

MyBatis
^^^^^^^^^^^^^^^^^^^^^^^^^^^

* O/R MapperというよりもSQLとオブジェクトをマッピングするSQL Mapperという表現が正しい
* Mapperファイル(xml)にSQLとオブジェクトのメソッドとのマッピングを定義する
* SQLでデータベースの操作ができつつも、ビジネスロジックからSQL自体を隠蔽できる

以下にMapperファイルの例を示す。
なお、Repositoryについては通常と同様にエンティティとプロパティを定義する。

.. sourcecode:: xml
   :linenos:

   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   
   <mapper namespace="com.example.todo.domain.repository.todo.TodoRepository">
   
       <resultMap id="todoResultMap" type="Todo">
           <id property="todoId" column="todo_id" />
           <result property="todoTitle" column="todo_title" />
           <result property="finished" column="finished" />
           <result property="createdAt" column="created_at" />
       </resultMap>
   
       <select id="findById" parameterType="String" resultMap="todoResultMap">
       <![CDATA[
           SELECT
               todo_id,
               todo_title,
               finished,
               created_at
           FROM
               todo
           WHERE
               todo_id = #.{todoId}
       ]]>
       </select>
   
       <select id="findAll" resultMap="todoResultMap">
       <![CDATA[
           SELECT
               todo_id,
               todo_title,
               finished,
               created_at
           FROM
               todo
       ]]>
       </select>
   
       <insert id="create" parameterType="Todo">
       <![CDATA[
           INSERT INTO todo
           (
               todo_id,
               todo_title,
               finished,
               created_at
           )
           VALUES
           (
               #.{todoId},
               #.{todoTitle},
               #.{finished},
               #.{createdAt}
           )
       ]]>
       </insert>
   
       <update id="update" parameterType="Todo">
       <![CDATA[
           UPDATE todo
           SET
               todo_title = #.{todoTitle},
               finished = #.{finished},
               created_at = #.{createdAt}
           WHERE
               todo_id = #.{todoId}
       ]]>
       </update>
   
       <delete id="delete" parameterType="Todo">
       <![CDATA[
           DELETE FROM
               todo
           WHERE
               todo_id = #.{todoId}
       ]]>
       </delete>
   
       <select id="countByFinished" parameterType="Boolean"
           resultType="Long">
       <![CDATA[
           SELECT
               COUNT(*)
           FROM
               todo
           WHERE
               finished = #.{finished}
       ]]>
       </select>
   
   </mapper>


MyBatisとJPAの使い分け
^^^^^^^^^^^^^^^^^^^^^^^^^^^
* 複雑なクエリが必要であり、今後もカスタマイズが想定される場合はカスタマイズ性の高いMyBatisを使用する
* 単純なクエリしか不要である、即座にモックアップ的なアプリケーションを構築する場合はJPAを使用する
* JPAの場合、JPQLという独自のクエリ言語を用いるため、JPAに習熟している開発者が必要になる
