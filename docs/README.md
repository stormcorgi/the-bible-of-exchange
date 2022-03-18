# Exchange Server 虎の巻

## これはなに？

これから Exchange サーバの設計・構築・運用を行う人間に向けた(実践的)解説ドキュメントです。

Microosft Exchange の仕組みや固有名称(CAS、MBX、DAG、HAB…)、ならびにメールプロトコル(SMTP,POP3,IMAP)そのものの解説については避け、他の良質な書籍やサイトへの言及にとどめます。

実戦の中で遭遇した事象と解法、ならびに現地調査、設計、構築、運用における"勘所"の公開を意図しています。

また、実際の調査、構築、テスト作業時に利用した自作ツール(Powershell)についても公開します(MIT ライセンス)。

## 誰が書いている？

Exchange 2010,2013,2016 あたりを主戦場としているメールエンジニア。  
経験は 5 年程度、主な環境サイズは 1000 名程度。

## 目次

- [Exchange Server 虎の巻](#exchange-server-虎の巻)
  - [これはなに？](#これはなに)
  - [誰が書いている？](#誰が書いている)
  - [目次](#目次)
  - [イントロダクション](#イントロダクション)
  - [現地調査](#現地調査)
  - [基本設計](#基本設計)
  - [詳細](#詳細)
  - [構築作業](#構築作業)
  - [テスト](#テスト)
  - [運用](#運用)
  - [ハイブリッド構成](#ハイブリッド構成)
  - [DAG 構成](#dag-構成)
  - [推薦書籍、サイト、ツール](#推薦書籍サイトツール)
    - [ツール](#ツール)
    - [情報源](#情報源)
  - [小技](#小技)

## [イントロダクション](/introduction/)

## [現地調査](/field-survey/)

## [基本設計](/basic-design/)

## [詳細](/detailed-design/)

## [構築作業](/build-environment/)

## [テスト](/testing/)

## [運用](/operation/)

## [ハイブリッド構成](/hybrid/)

## [DAG 構成](/DAG/)

## 推薦書籍、サイト、ツール

### ツール

[MX Toolbox](https://mxtoolbox.com/)  
オープンリレーの確認、レコード不備のチェック、ブラックリスト状態の串刺し検索など  
メール環境の調査、確認、運用に有用。

[Mail Header Analyzer](https://mha.azurewebsites.net/)  
名前の通り、ヘッダ情報(テキスト)を入れると視覚的にどの MTA を経由して  
中継に何秒掛かったか、暗号化状況はどうか、各ヘッダの値が何か、見やすく表示してくれる。  
MS のエンジニアが作ってるツールなので、セキュリティ的にもそれほど懸念はなさそう。

[Exchange Connectivity Analyzer](https://testconnectivity.microsoft.com/)  
特にハイブリッド時、MS 側のサーバからオンプレミス Exchange への各種接続が通っているか  
チェックするのに使えるツール。

[EWSEditor](https://github.com/dseph/EwsEditor)  
オフラインツール。EWS(Exchange Web Service)のクライアント。特にハイブリッド構成時など、  
外部からの EWS 通信状況をチェックする際に利用。

### 情報源

[Exchange Team Blog](https://techcommunity.microsoft.com/t5/exchange-team-blog/bg-p/Exchange)  
ノータイムで RSS に突っ込んでウォッチすべし。  
最速最強の情報はいつもここにある。開発チームのブログ。当然英語。

[Practical 365](https://practical365.com/)  
O365,M365 関連の情報多数。ハイブリッド、ExO 単品運用の双方で情報源として有用。  
コメント欄でトラブルシューティングしていることも多々あり、他環境を伺い知れる。 当然英語。

[SendGrid](https://sendgrid.kke.co.jp/blog/)  
クラウドメール配信サービスのブログ。企業が発信する DM などの送信代行等を行うメールのプロ。  
基礎知識やトレンドの解説など有用な記事多数。

[ナリタイ](https://www.naritai.jp/)  
SPF,DKIM,DMARC などメールなりすましに関連する基礎知識確認に最適  
日本語だから対顧客の説明にも。

## 小技

- 他人が書いた手順を調べたい場合は"step by step"を検索ワードに入れるべし。  
  例) exchange 2016 DAG create step by step
