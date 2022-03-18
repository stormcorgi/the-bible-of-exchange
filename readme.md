# Exchange Server 虎の巻

## これはなに？

これからExchangeサーバの設計・構築・運用を行う人間に向けた(実践的)解説ドキュメントです。

Microosft Exchangeの仕組みや固有名称(CAS、MBX、DAG、HAB…)、ならびにメールプロトコル(SMTP,POP3,IMAP)そのものの解説については避け、他の良質な書籍やサイトへの言及にとどめます。

実戦の中で遭遇した事象と解法、ならびに現地調査、設計、構築、運用における"勘所"の公開を意図しています。

また、実際の調査、構築、テスト作業時に利用した自作ツール(Powershell)についても公開します(MITライセンス)。

## 誰が書いている？

Exchange 2010,2013,2016あたりを主戦場としているメールエンジニア。  
経験は5年程度。

## 目次

- [Exchange Server 虎の巻](#exchange-server-虎の巻)
  - [これはなに？](#これはなに)
  - [誰が書いている？](#誰が書いている)
  - [目次](#目次)
  - [イントロダクション](#イントロダクション)
  - [推薦書籍、サイト、ツール](#推薦書籍サイトツール)
    - [ツール](#ツール)
    - [情報源](#情報源)
  - [現地調査](#現地調査)
  - [設計](#設計)
  - [構築作業](#構築作業)
  - [テスト](#テスト)
  - [運用](#運用)
  - [ハイブリッド構成](#ハイブリッド構成)
  - [小技](#小技)

## イントロダクション

see 00_introduction.md


## 推薦書籍、サイト、ツール

### ツール

[MX Toolbox](https://mxtoolbox.com/)  
オープンリレーの確認、レコード不備のチェック、ブラックリスト状態の串刺し検索など  
メール環境の調査、確認、運用に有用。

[Mail Header Analyzer](https://mha.azurewebsites.net/)  
名前の通り、ヘッダ情報(テキスト)を入れると視覚的にどのMTAを経由して  
中継に何秒掛かったか、暗号化状況はどうか、各ヘッダの値が何か、見やすく表示してくれる。  
MSのエンジニアが作ってるツールなので、セキュリティ的にもそれほど懸念はなさそう。

[Exchange Connectivity Analyzer](https://testconnectivity.microsoft.com/)  
特にハイブリッド時、MS側のサーバからオンプレミスExchangeへの各種接続が通っているか  
チェックするのに使えるツール。

[EWSEditor](https://github.com/dseph/EwsEditor)  
オフラインツール。EWS(Exchange Web Service)のクライアント。特にハイブリッド構成時など、  
外部からのEWS通信状況をチェックする際に利用。


### 情報源
[Exchange Team Blog](https://techcommunity.microsoft.com/t5/exchange-team-blog/bg-p/Exchange)  
ノータイムでRSSに突っ込んでウォッチすべし。  
最速最強の情報はいつもここにある。開発チームのブログ。当然英語。

[Practical 365](https://practical365.com/)  
O365,M365関連の情報多数。ハイブリッド、ExO単品運用の双方で情報源として有用。  
コメント欄でトラブルシューティングしていることも多々あり、他環境を伺い知れる。 当然英語。

[SendGrid](https://sendgrid.kke.co.jp/blog/)  
クラウドメール配信サービスのブログ。企業が発信するDMなどの送信代行等を行うメールのプロ集団。  
基礎知識やトレンドの解説など有用な記事多数。

[ナリタイ](https://www.naritai.jp/)  
SPF,DKIM,DMARCなどメールなりすましに関連する基礎知識確認に最適  
日本語だから対顧客の説明にも。

## 現地調査

see 01_field-survey.md

## 設計

see 02_basic-design.md

see 03_detailed-design.md

## 構築作業

see 04_build-environment.md

## テスト

see 05_test.md

## 運用

see 06_operation.md

## ハイブリッド構成

WIP

## 小技

* 他人が書いた手順を調べたい場合は"step by step"を検索ワードに入れるべし。  
例) exchange 2016 DAG create step by step
