### LogicComponent/Driver�쐬�K�C�h���C��

----
#### overview

�{�h�L�������g��ODENOS�̊�{�I�ȓ���ɂ��ė������Ă���A
ODENOS���g�����ĐV����LogicComponent,Driver����������l�����̃K�C�h���C���ł���.    

��{�I�ȓ���́Adoc/QUICKSTART.md���Q�ƁB

�{�h�L�������g�ł́Acomponent/Driver�����ɂĕK�v�ƂȂ郁�\�b�h�Ƃ��̎������j������.
��̓I�Ȏ�����"Aggregator","DummyDriver","LearningSwitch"�̓����\�b�h���Q�Ƃ��邱��.    

�����java���ɐ�������.    

----
#### Index
    
 * 1. [project�\��](#project)
 * 2. [apps�J�����̃f�B���N�g���\����](#packege)
 * 3. [BaseClass�̌���](#baseclass)
 * 4. [connection����](#connection)
 * 5. [Event subscription](#subscription)
 * 6. [Event method override](#event)
 * 7. conversion�@�\�̊��p
 * 7.1. [conversionTable�̐ݒ�](#conversionTable)
 * 7.2. [Event method override](#Event2)
 * 8. Request Event�쐬
     

----
#### <a name="project">project�\��</a>

  �f�B���N�g���\�������L�̂悤�ɕύX
 (��v�ȃf�B���N�g���A�t�@�C���̂݋L��)
 ---------------------------------
	odenos/
	 |
	 +-- odenos -- �N���X�v���N�g
	 +-- run-unittests.sh -- test�X�v���N�g
	 |
	 |-- apps/
	 |   +-- example/      --- sample�A�v���Q
	 |   +-- java/         --- java component���쐬����ꍇ��sample(dummy_driver)
	 |   +-- python/       --- python component���쐬����ꍇ��sample(dummy_driver)
	 |   +-- rest_sample/  --- rest���s���̃T���v���X�v���N�g
	 |
	 |-- doc/  -- api�h�L�������g
	 |
	 |-- etc/  -- ODENOS �� config
	 |    + -- odenos.conf --  �N���v���Z�X�̐ݒ�
	 |                         (app�z���̃f�B���N�g�����w�肷�邱�Ƃō쐬����component��o�^�\)
	 |    + -- log_*.conf�@--  log�o�͐�(default : ./var/log)
	 |
	 |-- lib/   -- ODENOS�̃��C�u����(jar,python,ruby) <-build�t�@�C���̊i�[
	 |
	 |-- src/   -- ODENOS�̃\�[�X(java,python,ruby)
	 |
	 |-- var/
	       +-- log  -- log�i�[

 ---------------------------------

 * build���@�̕ύX
  mvn�R�}���h�ɂ�build(doc/QUICKSTART.md�Q��)

 * �N���X�v���N�g
  - ./odenos start �ŋN��
  - ./odenos stop �ŏI��
  - ./odenos restart �ōċN���ɂȂ�܂��B

 * �ݒ�t�@�C��
  ./etc/odenos.conf �ɋN������R���|�[�l���g�}�l�[�W�����L�ڂ��Ă��������B

 ---------------------------------
  		    [odenos.conf�̏������]
		    PROCESS romgr1,java,apps/java/sample_components/target/classes
		    PROCESS romgr2,python,apps/python/sample_components

		    [�ݒ���e]
		    PROCESS %1, %2, %3
		    %1: compoment_manager�̖��O(�C�ӂɐݒ�\)
		    %2: ������w�� java or python (Ruby���Ή�)
		    %3: �Ǝ��쐬����conponent�̊i�[��
 ---------------------------------

----
#### <a name="packege">apps�J�����̃f�B���N�g���\����</a>

  apps �z���ɍ쐬�\�t�g�E�F�A���i�[����B
  �쐬�\�t�g�E�F�A�́Aodenos/lib�z���̃t�@�C����import���g�p����B(classpath�Ɏw��)
  ��odenos��etc(config�t�@�C��)�ȊO�͊�{�I�ɕύX���邱�Ƃ͂Ȃ��B

	|-- apps/
	    +-- Project-A/
	    	+--- run_splict  ���s�X�v���N�g.�R���|�[�l���g����/connection�Ȃ�
		     +--- src/main/java/xxx/proj-a/
		     |                       + component/
		     |                          +--- ExtLinkLayerlizer.java
		     |                          +--- ExtAggregator.java
		     |                          +--- driver/
		     |                                +------ XXXXDriver.java
		     |                                +------ YYYYDriver.java
		     +--- target/classes  -- class�t�@�C���i�[

----
#### <a name="baseclass">BaseClass�̌���</a>

Driver : Driver.class���p�����Ď�������B(�Q�l�FDummyDriver)    
LogicComponent : Logic class���p�����Ď�������B(�Q�l�FAggregator)   

----
#### <a name="connection">connection�̎����K�C�h���C��</a>

* Conponent/Driver��"Network"��connection �ω���(*)�̏������L�ڂ���.
 
(*)POST/PUT/DELETE \<base_uri>/connections �����{���ꂽ�Ƃ�.

"ConnectionChanged"�C�x���g��SystemManager���ecomponent�ɒʒm����,���̃��\�b�h���R�[�������̂Ŏ�Component�ɂ�override���Ď������邱��.    

method                       | Note
-----------------------------|------------------------------ 
onConnectionChangedAddedPre  | Connection���ǉ����ꂽ�Ƃ�
onConnectionChangedUpdatePre | Connection���X�V���ꂽ�Ƃ�
onConnectionChangedDeletePre | Connection���폜���ꂽ�Ƃ�

�{���\�b�h�ɂ�,�v�����ꂽConnection�ɑ΂�,��Component���ڑ��i�X�V�A�폜�j�\���`�F�b�N���邱��.    
onConnectionChanged~Pre��"true"�ŕԂ��ƁA���L���\�b�h���R�[�������.    
(false��Ԃ����ꍇ�͉��L���\�b�h�̓R�[�����ꂸ��connection�������I������)

method                       | Note
-----------------------------|------------------------------ 
onConnectionChangedAdded     | Connection���ǉ����ꂽ�Ƃ�
onConnectionChangedUpdate    | Connection���X�V���ꂽ�Ƃ�
onConnectionChangedDelete    | Connection���폜���ꂽ�Ƃ� 

�{���\�b�h�ł�, connection���Ɏ��{���鏈�����L�ڂ���.    
�ʏ��,���L[Event subscription](#subscription)�������L�ڂ��邱��z�肵�Ă���.    
��Component�ɂ�override���Ď������邱��.    

    
----

#### <a name="subscription">Event subscription�̐ݒ�K�C�h���C��</a>

Event subscription�ł́A��M����C�x���g��o�^���܂��B    
netwrok����̒ʒm���K�v�ȃC�x���g��o�^���܂��B

##### network�̃C�x���g�ꗗ

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
addEntryEventSubscription | �C�x���g�ʒm�o�^(add,delete) 
updateEntryEventSubscription | �C�x���g�ʒm�o�^(update)  ArrayList�ɓ`������attributes���w�肷��(*)
removeEventSubscription  | �C�x���g�ʒm����(addEntryEventSubscription,updateEntryEventSubscription�œo�^�����C�x���g�̉���)
applyEventSubscription  | �o�^�����C�x���g�����ۂ�eventManager�ɔ��f����(~EventSubscription�����ł͔��f����Ȃ��̂Ŗ{���\�b�h���R�[�����邱��)

�C�x���g��Action�Ƃ��� "add","update","delete"������B
\<nwcId>��network ��id���w�肷��i�ʏ��connection��network_id�ƂȂ�j

(*)��q��conversionTable�@�\���g�p����ꍇ�́A���Ŏw�肵��attributes���X�V�ΏۂƂȂ�,Driver�J���Ȃ�conversionTable�@�\���g�p���Ȃ��ꍇ��null���w��B

----
#### <a name="event">Event�̎����K�C�h���C��</a>

�@network��topology,flow,packet���X�V�����ƁA�C�x���g���ʒm�����B
  �������K�v�ȃ��\�b�h��override���ď������L�q���邱�ƁB

* Topology

          |  add method      |  update      | delete 
----------|------------------|--------------|------------- 
Node      | onNodeAdded      | onNodeUpdate | onNodeDelete
Port      | onPortAdded      | onPortUpdate | onPortDelete
Link      | onLinkAdded      | onLinkUpdate | onLinkDelete
Flow      | onFlowAdded      | onFlowUpdate | onFlowDelete
InPacket  | onInPacketadded  | -            | -
OutPacket | onOutPacketadded | -            | -

 ��q��conversionTable�@�\���g�p����ꍇ�́Aoverride���郁�\�b�h���ǉ������̂ł�������Q�Ƃ��邱�ƁB

----


#### <a name="conversionTable">conversionTable�̐ݒ�K�C�h���C��</a>

conversionTable��topology,flow�֘A��o�^���Ă�����,
network�Ԃ�topology,flow�̍X�V���x�[�X�N���X��Logic���Ń��[���ɏ]���Ď����ōs���B
(�O�q��Conversion ���\�b�h�ɂ���ē��삷��)

Slicer,Fedeletor�ȂǂŁAnetwork�Ԃł�topology,flow�̍X�V�����̎������y�����邽�߂̋@�\�ł���B
������Network�Ɛڑ����Ȃ�Component�ł͖{�@�\�̌��ʂ͖���.(Driver�ALerningSwitch�Ȃ�..)

  [network1] ---  [LogicConponent] --- [network2]    

onConnectionChangedAdded�ɂāAnetwork�Ԃ̊֘A�t�����s���B
��L�A[network1] �� [network2] ���֘A�t������ƁA���network��topology,flow�ɕω�������������
������network�ւ̔��f��e�Ղɍs�����Ƃ��ł���B

* �֘A����ݒ肷�郁�\�b�h�Q

method                         | Note                       |  �R�[���^�C�~���O
-----------------------|---------------|-----------------
addEntryConnectionType   | Network �� ConnectionType�̓o�^ \<id>��network��id���w��B \<type>��connection_type���w��B | onConnectionChangedAdded
addEntryNetwork               | network�̊֘A�t��  | onConnectionChangedAdded
addEntryNode                 | node���֘A�t��      | onNodeAdded(*)
addEntryPort                  | port�̊֘A�t��       | onPortAdded(*)
addEntryLink                  | Link�̊֘A�t��      | onLinkAdded(*)
addEntryFlow                 | �e�������̊֘A�t��     | onFlowAdded(*)

* �֘A�����폜���郁�\�b�h�Q

method                         | Note                         |  �R�[���^�C�~���O
-----------------------|----------------------|-----------------
delEntryConnectionType  | Network �� ConnectionType�̍폜   |  onConnectionChangedDelete
delEntryNetwork                | network�̊֘A�t�����폜               | onConnectionChangedDelete
delEntryNode                 | node���֘A�t�����폜                   | onNodeDelete(*)
delEntryPort                  | port�̊֘A�t�����폜                    | onPortDelete(*)
delEntryLink                  | Link�̊֘A�t�����폜                    | onLinkDelete(*)
delEntryFlow                 | �e�������̊֘A�t�����폜                   | onFlowDelete(*)

 
(*) Logic.java����addEntry~ , delEntry~ ���R�[�����Ċ֘A�t�����s���Ă���B
�Y�����\�b�h��override����ꍇ�͖{���������킹�Ď�������K�v������B

----
#### <a name="Event2">conversionTable�̐ݒ莞��Event�̎����K�C�h���C��</a>

##### Event�����̗���iNode�̗�j

----
  
    Event����(NodeChanged �C�x���g���s) [Network]
     +--> [Subscribe]���Ă���[LogicComponent]    
               +---> onNodeAdded   <--------------------------- 1. override���ăC�x���g�������L�q
                       |onNodeAddedPre  | <--- �O���� <--- 2. override���ăC�x���g�������L�q
                       | Conversion     | <--- ConversionTable�ɏ]���ē���(class Logic���ŏ���)
                       |onNodeAddedPost | <--- �㏈�� <--- 3. override���ăC�x���g�������L�q
  

----

��L���\�b�h���I�[�o���C�h���Ȃ��ƁAconversionTable�ŋL�ڂ����֘A���ŃI�u�W�F�N�g�̍X�V��logic���ōs���B
Node�̒ǉ��A�X�V�A�폜��logic��Conversion()�ɂ���čs����B

 1. onNodeAdded : �C�x���g������S�ċL�q����ꍇ�͖{���\�b�h��override����B
 2. onNodeAddedPre�����ɂăt�B���^�����O���\�i�`���������Ȃ��C�x���g�����̏����ŗ��Ƃ��j�{���\�b�h��override����"false"��Ԃ��ƈȍ~�̏������s��Ȃ��B
 3. onNodeAddedPost�ɂČ㏈�������{�BConversion��request�ɑ΂���Response���p�����[�^respList�Ɋi�[����Ă���BResponse���`�F�b�N���ăG���[�����Ȃǂ��s���B

  
Tips:
�Ӑ}�I�ɃC�x���g�𗎂Ƃ������Ƃ��́AonNodeAddedPre��"False"��Ԃ��悤�Ɏ�������B
�P���ȃR�s�[�ł͂Ȃ��ALogicComponent���L�̏������s���ꍇ�́AonNodeAdded���I�[�o���C�h���邱�Ɓi���̏ꍇ�́AonNodeAddedPre�̓R�[������Ȃ��j
Conversion��ɏ������s�������ꍇ�́AonNodeAddedPost���I�[�o���C�h���邱�ƁB
���� respList��Response�R�[�h���i�[����Ă���B
�i�G���[�`�F�b�N�ȂǂɎg���邱�Ƃ�z��j



* Topology

-  |  add method    |  update          | delete 
---|----------------|------------------|------------------- 
1  | onNodeAdded    | onNodeUpdate     | onNodeDelete
2  | onNodeAddedPre | onNodeUpdatePre  | onNodeDeletePre
3  | onNodeAddedPost| onNodeUpdatePost | onNodeDeletePost

 * Port,Link,Flow�������̃��\�b�h����
 * onInPacket,onOutPacket��added�̂�

----

#### Request�̎����K�C�h���C��

Conponent/Driver�ɓƎ���REST IF��ǉ�����ꍇ�̎������@�ł��B

��̓I�Ȏ�����Aggregator,LearningSwitch�Ȃǂ��Q�Ƃ��Ă��������B

<�K�v�ȏ���>    

 1. �N���X�ϐ��Ƃ��� ���Lparser���`  
   protected final RequestParser<IActionCallback> parser;    

 2. �R���X�g���N�^�ɂ�parser��������    
        parser = createParser();   
    
 3. createParser���\�b�h��Request�������L�ڂ���B    
�@�@A. B. C. �ɂ��ĕK�v���L�ڂ���B

        private RequestParser<IActionCallback> createParser() {
          return new RequestParser<IActionCallback>() { {
            addRule(Method.GET, �@<---A. Actin�̎w��(GET,POST,PUT,DELETE)
                    "fdb",        <---B. path�̎w�� 
                    new IActionCallback() {    
                    @Override    
                    public Response process(
                      final RequestParser<IActionCallback>.
                      ParsedRequest parsed) throws Exception {
                        return getFdb(); <---C. ���������s�����\�b�h�̋L�q
                      }
                   });
            }};
        }
    


 4. onRequest���\�b�h��override����B    
�@�@3 �̃��\�b�h���R�[������

----

