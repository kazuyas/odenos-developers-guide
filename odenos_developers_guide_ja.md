### LogicComponent/Driver作成ガイドライン

----
#### overview

本ドキュメントはODENOSの基本的な動作について理解しており、
ODENOSを拡張して新たなLogicComponent,Driverを実装する人向けのガイドラインである.    

基本的な動作は、doc/QUICKSTART.mdを参照。

本ドキュメントでは、component/Driver実装にて必要となるメソッドとその実装方針を示す.
具体的な実装は"Aggregator","DummyDriver","LearningSwitch"の同メソッドを参照すること.    

言語はjavaを例に説明する.    

----
#### Index
    
 * 1. [project構成](#project)
 * 2. [apps開発時のディレクトリ構成例](#packege)
 * 3. [BaseClassの決定](#baseclass)
 * 4. [connection処理](#connection)
 * 5. [Event subscription](#subscription)
 * 6. [Event method override](#event)
 * 7. conversion機能の活用
 * 7.1. [conversionTableの設定](#conversionTable)
 * 7.2. [Event method override](#Event2)
 * 8. Request Event作成
     

----
#### <a name="project">project構成</a>

  ディレクトリ構成を下記のように変更
 (主要なディレクトリ、ファイルのみ記載)
 ---------------------------------
	odenos/
	 |
	 +-- odenos -- 起動スプリクト
	 +-- run-unittests.sh -- testスプリクト
	 |
	 |-- apps/
	 |   +-- example/      --- sampleアプリ群
	 |   +-- java/         --- java componentを作成する場合のsample(dummy_driver)
	 |   +-- python/       --- python componentを作成する場合のsample(dummy_driver)
	 |   +-- rest_sample/  --- rest実行時のサンプルスプリクト
	 |
	 |-- doc/  -- apiドキュメント
	 |
	 |-- etc/  -- ODENOS の config
	 |    + -- odenos.conf --  起動プロセスの設定
	 |                         (app配下のディレクトリを指定することで作成したcomponentを登録可能)
	 |    + -- log_*.conf　--  log出力先(default : ./var/log)
	 |
	 |-- lib/   -- ODENOSのライブラリ(jar,python,ruby) <-buildファイルの格納
	 |
	 |-- src/   -- ODENOSのソース(java,python,ruby)
	 |
	 |-- var/
	       +-- log  -- log格納

 ---------------------------------

 * build方法の変更
  mvnコマンドにてbuild(doc/QUICKSTART.md参照)

 * 起動スプリクト
  - ./odenos start で起動
  - ./odenos stop で終了
  - ./odenos restart で再起動になります。

 * 設定ファイル
  ./etc/odenos.conf に起動するコンポーネントマネージャを記載してください。

 ---------------------------------
  		    [odenos.confの初期情報]
		    PROCESS romgr1,java,apps/java/sample_components/target/classes
		    PROCESS romgr2,python,apps/python/sample_components

		    [設定内容]
		    PROCESS %1, %2, %3
		    %1: compoment_managerの名前(任意に設定可能)
		    %2: 言語を指定 java or python (Ruby未対応)
		    %3: 独自作成したconponentの格納先
 ---------------------------------

----
#### <a name="packege">apps開発時のディレクトリ構成例</a>

  apps 配下に作成ソフトウェアを格納する。
  作成ソフトウェアは、odenos/lib配下のファイルをimportし使用する。(classpathに指定)
  ※odenosはetc(configファイル)以外は基本的に変更することはない。

	|-- apps/
	    +-- Project-A/
	    	+--- run_splict  実行スプリクト.コンポーネント生成/connectionなど
		     +--- src/main/java/xxx/proj-a/
		     |                       + component/
		     |                          +--- ExtLinkLayerlizer.java
		     |                          +--- ExtAggregator.java
		     |                          +--- driver/
		     |                                +------ XXXXDriver.java
		     |                                +------ YYYYDriver.java
		     +--- target/classes  -- classファイル格納

----
#### <a name="baseclass">BaseClassの決定</a>

Driver : Driver.classを継承して実装する。(参考：DummyDriver)    
LogicComponent : Logic classを継承して実装する。(参考：Aggregator)   

----
#### <a name="connection">connectionの実装ガイドライン</a>

* Conponent/Driverと"Network"のconnection 変化時(*)の処理を記載する.
 
(*)POST/PUT/DELETE \<base_uri>/connections が実施されたとき.

"ConnectionChanged"イベントがSystemManagerより各componentに通知され,次のメソッドがコールされるので自Componentにてoverrideして実装すること.    

method                       | Note
-----------------------------|------------------------------ 
onConnectionChangedAddedPre  | Connectionが追加されたとき
onConnectionChangedUpdatePre | Connectionが更新されたとき
onConnectionChangedDeletePre | Connectionが削除されたとき

本メソッドにて,要求されたConnectionに対し,自Componentが接続（更新、削除）可能かチェックすること.    
onConnectionChanged~Preを"true"で返すと、下記メソッドがコールされる.    
(falseを返した場合は下記メソッドはコールされずにconnection処理が終了する)

method                       | Note
-----------------------------|------------------------------ 
onConnectionChangedAdded     | Connectionが追加されたとき
onConnectionChangedUpdate    | Connectionが更新されたとき
onConnectionChangedDelete    | Connectionが削除されたとき 

本メソッドでは, connection時に実施する処理を記載する.    
通常は,下記[Event subscription](#subscription)処理を記載するこを想定している.    
自Componentにてoverrideして実装すること.    

    
----

#### <a name="subscription">Event subscriptionの設定ガイドライン</a>

Event subscriptionでは、受信するイベントを登録します。    
netwrokからの通知が必要なイベントを登録します。

##### networkのイベント一覧

event            | Note
-----------------|------------------------------ 
NODE_CHANGED     | "NodeChanged"
PORT_CHANGED     | "PortChanged"
LINK_CHANGED     | "LinkChanged" 
FLOW_CHANGED     | "FlowChanged"
IN_PACKET_ADDED  | "InPacketAdded"
OUT_PACKET_ADDED | "OutPacketAdded"

##### subscription method

method                       | Note
---------------------|---------------
addEntryEventSubscription | イベント通知登録(add,delete) 
updateEntryEventSubscription | イベント通知登録(update)  ArrayListに伝搬するattributesを指定する(*)
removeEventSubscription  | イベント通知解除(addEntryEventSubscription,updateEntryEventSubscriptionで登録したイベントの解除)
applyEventSubscription  | 登録したイベントを実際にeventManagerに反映する(~EventSubscriptionだけでは反映されないので本メソッドをコールすること)

イベントのActionとして "add","update","delete"がある。
\<nwcId>にnetwork のidを指定する（通常はconnectionのnetwork_idとなる）

(*)後述のconversionTable機能を使用する場合は、こで指定したattributesが更新対象となる,Driver開発などconversionTable機能を使用しない場合はnullを指定。

----
#### <a name="event">Eventの実装ガイドライン</a>

　networkのtopology,flow,packetが更新されると、イベントが通知される。
  処理が必要なメソッドをoverrideして処理を記述すること。

* Topology

          |  add method      |  update      | delete 
----------|------------------|--------------|------------- 
Node      | onNodeAdded      | onNodeUpdate | onNodeDelete
Port      | onPortAdded      | onPortUpdate | onPortDelete
Link      | onLinkAdded      | onLinkUpdate | onLinkDelete
Flow      | onFlowAdded      | onFlowUpdate | onFlowDelete
InPacket  | onInPacketadded  | -            | -
OutPacket | onOutPacketadded | -            | -

 後述のconversionTable機能を使用する場合は、overrideするメソッドが追加されるのでそちらも参照すること。

----


#### <a name="conversionTable">conversionTableの設定ガイドライン</a>

conversionTableにtopology,flow関連を登録しておくと,
network間のtopology,flowの更新をベースクラスのLogic内でルールに従って自動で行う。
(前述のConversion メソッドによって動作する)

Slicer,Fedeletorなどで、network間でのtopology,flowの更新処理の実装を軽減するための機能である。
複数のNetworkと接続がないComponentでは本機能の効果は無い.(Driver、LerningSwitchなど..)

  [network1] ---  [LogicConponent] --- [network2]    

onConnectionChangedAddedにて、network間の関連付けを行う。
上記、[network1] と [network2] が関連付けられると、一方networkでtopology,flowに変化があった時に
他方のnetworkへの反映を容易に行うことができる。

* 関連性を設定するメソッド群

method                         | Note                       |  コールタイミング
-----------------------|---------------|-----------------
addEntryConnectionType   | Network の ConnectionTypeの登録 \<id>にnetworkのidを指定。 \<type>にconnection_typeを指定。 | onConnectionChangedAdded
addEntryNetwork               | networkの関連付け  | onConnectionChangedAdded
addEntryNode                 | nodeを関連付け      | onNodeAdded(*)
addEntryPort                  | portの関連付け       | onPortAdded(*)
addEntryLink                  | Linkの関連付け      | onLinkAdded(*)
addEntryFlow                 | Ｆｌｏｗの関連付け     | onFlowAdded(*)

* 関連性を削除するメソッド群

method                         | Note                         |  コールタイミング
-----------------------|----------------------|-----------------
delEntryConnectionType  | Network の ConnectionTypeの削除   |  onConnectionChangedDelete
delEntryNetwork                | networkの関連付けを削除               | onConnectionChangedDelete
delEntryNode                 | nodeを関連付けを削除                   | onNodeDelete(*)
delEntryPort                  | portの関連付けを削除                    | onPortDelete(*)
delEntryLink                  | Linkの関連付けを削除                    | onLinkDelete(*)
delEntryFlow                 | Ｆｌｏｗの関連付けを削除                   | onFlowDelete(*)

 
(*) Logic.java内でaddEntry~ , delEntry~ をコールして関連付けを行っている。
該当メソッドをoverrideする場合は本処理も合わせて実装する必要がある。

----
#### <a name="Event2">conversionTableの設定時のEventの実装ガイドライン</a>

##### Event処理の流れ（Nodeの例）

----
  
    Event発生(NodeChanged イベント発行) [Network]
     +--> [Subscribe]している[LogicComponent]    
               +---> onNodeAdded   <--------------------------- 1. overrideしてイベント処理を記述
                       |onNodeAddedPre  | <--- 前処理 <--- 2. overrideしてイベント処理を記述
                       | Conversion     | <--- ConversionTableに従って動作(class Logic内で処理)
                       |onNodeAddedPost | <--- 後処理 <--- 3. overrideしてイベント処理を記述
  

----

上記メソッドをオーバライドしないと、conversionTableで記載した関連性でオブジェクトの更新をlogic内で行う。
Nodeの追加、更新、削除がlogicのConversion()によって行われる。

 1. onNodeAdded : イベント処理を全て記述する場合は本メソッドをoverrideする。
 2. onNodeAddedPre処理にてフィルタリングが可能（伝搬したくないイベントをこの処理で落とす）本メソッドをoverrideして"false"を返すと以降の処理を行わない。
 3. onNodeAddedPostにて後処理を実施。Conversionのrequestに対するResponseがパラメータrespListに格納されている。Responseをチェックしてエラー処理などを行う。

  
Tips:
意図的にイベントを落としたいときは、onNodeAddedPreで"False"を返すように実装する。
単純なコピーではなく、LogicComponent特有の処理を行う場合は、onNodeAddedをオーバライドすること（その場合は、onNodeAddedPreはコールされない）
Conversion後に処理を行いたい場合は、onNodeAddedPostをオーバライドすること。
引数 respListにResponseコードが格納されている。
（エラーチェックなどに使われることを想定）



* Topology

-  |  add method    |  update          | delete 
---|----------------|------------------|------------------- 
1  | onNodeAdded    | onNodeUpdate     | onNodeDelete
2  | onNodeAddedPre | onNodeUpdatePre  | onNodeDeletePre
3  | onNodeAddedPost| onNodeUpdatePost | onNodeDeletePost

 * Port,Link,Flowも相当のメソッドあり
 * onInPacket,onOutPacketはaddedのみ

----

#### Requestの実装ガイドライン

Conponent/Driverに独自のREST IFを追加する場合の実装方法です。

具体的な実装はAggregator,LearningSwitchなどを参照してください。

<必要な処理>    

 1. クラス変数として 下記parserを定義  
   protected final RequestParser<IActionCallback> parser;    

 2. コンストラクタにてparserを初期化    
        parser = createParser();   
    
 3. createParserメソッドにRequest処理を記載する。    
　　A. B. C. について必要分記載する。

        private RequestParser<IActionCallback> createParser() {
          return new RequestParser<IActionCallback>() { {
            addRule(Method.GET, 　<---A. Actinの指定(GET,POST,PUT,DELETE)
                    "fdb",        <---B. pathの指定 
                    new IActionCallback() {    
                    @Override    
                    public Response process(
                      final RequestParser<IActionCallback>.
                      ParsedRequest parsed) throws Exception {
                        return getFdb(); <---C. 実処理を行うメソッドの記述
                      }
                   });
            }};
        }
    


 4. onRequestメソッドをoverrideする。    
　　3 のメソッドをコールする

----

