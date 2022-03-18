# イントロダクション

目的:重要コンポーネントの理解を確認する

## メールとは

[MUA,MTA,MDA](https://ascii.jp/elem/000/000/439/439105/)  
[ヘッダーFrom,エンベロープFrom](https://am.arara.com/blog/header_envelope_20210118)  
上記サイト参照。

最低限でも

* MTAとMUAの違い(MDAはほぼ忘れてよし)
* MTA,MUAがそれぞれ利用するプロトコルとポート番号
* メールヘッダからエンベロープの遷移を読み解く

程度は自主的に構成したテスト環境で学ぶこと。

上記が概ね把握できたら、一度**telnetを用いた直接メール送信**を経験しておくこと。  
[やり方](http://ash.jp/net/telnet_smtp.htm)参照。何よりもこれが一番手っ取り早くSMTPプロトコルを理解する体験になる。

※時間が許せば[RFC](https://sendgrid.kke.co.jp/blog/?p=10609)に目を通すこと。非ExchangeのMTA,MUAの動作推測時、基準となる。

## Microsoft Exchangeとは

### 概要

Active Directory(MS製ディレクトリサーバ)と密接に連携しながら  
大規模組織での運用に耐えられるメールサーバ(MTA,MDA)。  

原則としてMicrosoft Outlook(MUA)からの利用を想定しており、  
SMTP/IMAP/POP3などのRFCプロトコルから外れた拡張プロトコル MAPI を有している。  

### 役割(ServerRole)

Exchangeサーバはインストール時、複数の役割のうちどれを持たせるか選択可能。
Exchange 2010,2013,2016,2019と時間的推移に伴い、役割は集約される傾向にある。

#### CAS(Client Access) and Hub

クライアントからのHTTPS/SMTP/MAPI(独自プロトコル)通信を受け(CAS)、外部との通信も担う(HUB)。  
バックエンドとして機能するMBXサーバへの通信を中継。  

Exchange 2010時代はCAS自体が処理しているエリアも多かった。  
Exchange 2013からは原則MBXへのプロキシとして機能。HUBはCASに統合。  
Exchange 2016からCASはMBXに統合。現状はEdge or MBXのみ考えれば良い。

#### MBX(Mailbox)

メール機能の実体。Mailbox Database(MDB)を保有し、データの実体が格納される。
性質上DiskI/O負荷分散のため別ディスクにMDBを置くことも多い。
冗長性確保のためDAG(Database Availability Group)機能がビルトインで存在し、  
複数のMBXノードがMDBコピーを保有し、障害発生時にフェールオーバーする構成が可能。


#### EdgeTransport

セキュリティ担保のため、DMZなど非信頼NWに置かれるエッジサーバ。  
使われているのを一度も見たことがない。マジで。:(  
定番構成としてPostfixやクラウドメールセキュリティサービスなどを  
上位MTAとしてネクストホップに指定する(=Exchange用語 スマートホスト)ことが多い。

### コアコンポーネント

#### Active Directory

Exchange稼働の前提条件。AD不在の環境へのインストールは不能。  
メールボックスを保有するユーザーやメーリングリスト(グループ)の各種情報について  
ADオブジェクトの属性値(Atribute)に多くを格納する。  

※ちなみに豆知識として、ADは元々Exchangeの1コンポーネントだったが  
独立して別ソフトウェアとして扱われている。  
そのあたりからもExchangeと不可分であることが伺える。

#### IIS(Internet Information Service) 

Windows ServerにビルトインのWebサーバ。  
Exchangeと連携し以下の多様なサービスでHTTP/S通信周りを担う。
* OWA(Outlook Web Access or App)
* OotW(Outlook on the Web, OWAと同義語だがこっちが新しいめ)
* ECP(Exchange Controll Panel)
* AutoDiscover
* EWS(Exchange Web Service)  

#### Transport Service

Exchangeが実際にメールを送受信する際に経由するサービス群。  
[メール フローとトランスポート パイプライン](https://docs.microsoft.com/ja-jp/exchange/mail-flow/mail-flow?view=exchserver-2019)参照。  

概観としては外部から順に以下のような順で内部(DB)へとデータが降りていく。  

```text
外部MTA -> FrontEnd -> Transport -> Mailbox Transport -> Store(DB)
```

送信時は基本逆順で、以下のようにカテゴライザを通る

```text
Store(DB) -> Mailbox Transport -> **Transport(Categorizer/RoutingAgents)**  
  -> FrontEnd Transport -> 外部MTA
```

要点は以下の通り。


* 受信時、TransportServiceは外部サーバに対して送信完了した旨を返答する前に、データ欠損を避けるため、**別の内部MBXサーバにメールを送信する。これをシャドウ冗長**と呼ぶ。つまり、外部サーバとの受信通信が正常終了した後であれば、組織内のMBXサーバが突然電源断してもシャドウ冗長によって受信が継続される。外部サーバと受信通信中のMBXサーバが突然電源断した場合、通信が正常終了しないため相手MTAが送信不能通知を生成できる。もっと噛み砕いて言うと、**複数台のMBXが同時に死ぬ異常事態が起きない限り、受信中のメールが送信不能通知もなく消失することはない**。
* 実際のメール配信(MDA)はStore Driverによって行われる。DBに書きつけたり、DBから読み込む作業はStore Driverがやる。Outlookでクライアントがメール送信を行うと、実際にはStoreの中に送信メールが書き込まれ、Store Driverはそこからメール送信処理を開始し、Mailbox Transport ServiceがこれをSMTPに乗せてTransport Serviceに投げる。
* **トランスポートルールの処理はTransport Service内部で行われる**

### あんまり使われない機能

#### UM(Unified Messaging)

電話と連携し、不在着信等をメールで受けたり出来るとか出来ないとか。  
使っている組織を見たことがない。

#### パブリックフォルダー(Public Folder)

いわゆる「掲示板」。発想からして懐かしさ満載。  
組織によっては社内通知やデータ共有場所として利用されていることがある。  
データの保有方法について、**Exchange 2010以前はPFDB、Exchange 2013以降はPFMBXとしてMDB内部に格納**されるようになった。2010->2013以降へのPF移行はMBXへのコンバートが求められ、作業コストが高い。  
特に古いデータの移行時には文字化け絡みへの注意が必要。

#### 階層型アドレス帳(Hierarchical Address Book)

会社組織の部門階層構造などを配布グループの入れ子で表現し、  
これらの配布グループ情報を階層型アドレス帳としてOutlook上で見せる機能。  
わりと欲しがる声も多く、使われないわけでもないのだが、  組織変更に追従する運用コストが高いので  
積極的には薦めない機能。
