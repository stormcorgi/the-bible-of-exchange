# イントロダクション

目的:重要コンポーネントの理解を確認する

## メールとは

[MUA,MTA,MDA](https://ascii.jp/elem/000/000/439/439105/)  
[ヘッダー From,エンベロープ From](https://am.arara.com/blog/header_envelope_20210118)  
上記サイト参照。

最低限でも

- MTA,MDA,MUA の違いを説明できる
- MTA,MUA がそれぞれ利用するプロトコルとポート番号を列挙できる
- メールヘッダからエンベロープの遷移を読み解く

程度は自主的に構成したテスト環境(Postfix/Dovecot ペアを推奨)で学ぶこと。

上記が概ね把握できたら、一度**telnet を用いた直接メール送信**を経験しておくこと。  
[やり方](http://ash.jp/net/telnet_smtp.htm)参照。何よりもこれが一番手っ取り早く SMTP プロトコルを理解する体験になる。

※時間が許せば[RFC](https://sendgrid.kke.co.jp/blog/?p=10609)に目を通すこと。非 Exchange の MTA,MUA の動作推測時、基準となる。

## Microsoft Exchange とは

### 概要

Active Directory(MS 製ディレクトリサーバ)と密接に連携しながら  
大規模組織での運用に耐えられるメールサーバ(MTA,MDA)。

原則として Microsoft Outlook(MUA)からの利用を想定しており、  
SMTP/IMAP/POP3 などの RFC プロトコルから外れた拡張プロトコル MAPI を有している。

### 役割(ServerRole)

Exchange サーバはインストール時、複数の役割のうちどれを持たせるか選択可能。
Exchange 2010,2013,2016,2019 と時間的推移に伴い、役割は集約される傾向にある。

#### CAS(Client Access) and Hub

クライアントからの HTTPS/SMTP/MAPI(独自プロトコル)通信を受け(CAS)、外部との通信も担う(HUB)。  
バックエンドとして機能する MBX サーバへの通信を中継。

Exchange 2010 時代は CAS 自体が処理しているエリアも多かった。  
Exchange 2013 からは原則 MBX へのプロキシとして機能。HUB は CAS に統合。  
Exchange 2016 から CAS は MBX に統合。現状は Edge or MBX のみ考えれば良い。

#### MBX(Mailbox)

メール機能の実体。Mailbox Database(MDB)を保有し、データの実体が格納される。
性質上 DiskI/O 負荷分散のため別ディスクに MDB を置くことも多い。
冗長性確保のため DAG(Database Availability Group)機能がビルトインで存在し、  
複数の MBX ノードが MDB コピーを保有し、障害発生時にフェールオーバーする構成が可能。

#### EdgeTransport

セキュリティ担保のため、DMZ など非信頼 NW に置かれるエッジサーバ。  
使われているのを一度も見たことがない。マジで。:(  
定番構成として Postfix やクラウドメールセキュリティサービスなどを  
上位 MTA としてネクストホップに指定する(=Exchange 用語 スマートホスト)ことが多い。

### コアコンポーネント

#### Active Directory

Exchange 稼働の前提条件。AD 不在の環境への Exchange インストールは不能。  
メールボックスを保有するユーザーやメーリングリスト(グループ)の各種情報について  
AD オブジェクトの属性値(Atribute)に多くを格納する。

※ちなみに豆知識として、AD は元々 Exchange の 1 コンポーネントだったが  
独立して別ソフトウェアとして扱われている。  
そのあたりからも Exchange と不可分であることが伺える。

#### IIS(Internet Information Service)

Windows Server ビルトインの Web サーバ。  
Exchange と連携し以下の多様なサービスで HTTP/S 通信周りを担う。

- OWA(Outlook Web Access or App)
- OotW(Outlook on the Web, OWA と同義語だがこっちが新しいめ)
- ECP(Exchange Controll Panel)
- AutoDiscover
- EWS(Exchange Web Service)

#### Transport Service(Exchagnge 内部サービス)

Exchange が実際にメールを送受信する際に経由するサービス群。  
最初から全てを理解する必要はないが、少しでも踏み込んだ理解をしたいなら  
[メール フローとトランスポート パイプライン](https://docs.microsoft.com/ja-jp/exchange/mail-flow/mail-flow?view=exchserver-2019)参照。

概観としては外部から順に以下のような順で内部(DB)へとデータが降りていく。

```text
外部MTA -> FrontEnd Transport Service(CAS)
 -> Transport Service(MBX) -> Mailbox Transport Service(MBX)
 -> Store(DB)
```

送信時は基本逆順で、以下のようにカテゴライザを通る

```text
Store(DB)
 -> Mailbox Transport Service
 -> **Transport Service(Categorizer/RoutingAgents)**
 -> FrontEnd Transport Service
 -> 外部MTA
```

重要ポイントは以下の通り。

- 受信時、Transport Service は外部サーバに対して送信完了した旨を返答する前に、データ欠損を避けるため、**別の内部 MBX サーバにメールを送信する。これをシャドウ冗長**と呼ぶ。つまり、外部サーバとの受信通信が正常終了した後であれば、組織内のとある MBX サーバが突然電源断しても、シャドウ冗長によって受信が継続される。外部サーバと受信通信中の MBX サーバが突然電源断した場合、通信が正常終了しないため相手 MTA が送信不能通知を生成できる。もっと噛み砕いて言うと、**複数台の MBX が同時に死ぬ異常事態が起きない限り、受信中のメールが送信不能通知もなく消失することはない**。
- 実際のメール配信(MDA)は Mailbox Transport Service 内部の Store Driver によって行われる。DB に書きつけたり、DB から読み込む作業は Store Driver がやる。Outlook でクライアントがメール送信を行うと、実際には Store の中に送信メールが書き込まれ、Store Driver はそこからメール送信処理を開始し、Mailbox Transport Service がこれを SMTP に乗せて Transport Service に投げる。
- **トランスポートルールの処理は Transport Service 内部のカテゴライザで行われる**

### あまり使われない機能

#### UM(Unified Messaging)

電話と連携し、不在着信等をメールで受けたり出来るとか出来ないとか。  
使っている組織を見たことがない。

#### パブリックフォルダー(Public Folder)

いわゆる「掲示板」。発想からして懐かしさ満載。  
組織によっては社内通知やデータ共有場所として利用されていることがある。  
データの保有方法について、**Exchange 2010 以前は PFDB、Exchange 2013 以降は PFMBX として MDB 内部に格納**されるようになった。2010->2013 以降への PF 移行は MBX へのコンバートが求められ、作業コストが高い。  
特に古いデータの移行時には文字化け絡みへの注意が必要。
Microsoft も積極的に利用して欲しくないスタンス。新規に使い始めることを考慮しているなら、別のソリューションを絶対に選択すべき。

#### 階層型アドレス帳(Hierarchical Address Book)

会社組織の部門階層構造などを配布グループの入れ子で表現し、  
これらの配布グループ情報を階層型アドレス帳として Outlook 上で見せる機能。  
わりと欲しがる声も多く、使われないわけでもないのだが、 組織変更に追従する運用コストが高いので  
積極的には薦めない機能。
