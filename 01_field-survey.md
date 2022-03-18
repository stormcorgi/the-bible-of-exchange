# 現地調査すべきポイント

## AD

ドメイン機能レベル（動作前提条件チェック）

```PowerShell
Get-ADDomain | fl
```

フォレスト機能レベル（動作前提条件チェック）

```PowerShell
Get-ADForest | fl
```

## 主要機能

バージョン確認（各サーバで実施） ProductVersion を想定バージョンとチェック

```PowerShell
Gcm exsetup | fl
```

全台見られない場合、組織が認識する全 Exc サーバのバージョン(AdminDisplayVersion)をチェック

```PowerShell
Get-ExchangeServer | fl
```

受信コネクタ

```PowerShell
Get-ReceiveConnector | fl
Get-ADPermission "受信コネクタ" -User "NT AUTHORITY\ANONYMOUS LOGON" |
    where {($_.Deny -eq $false ) -and ($_.IsInherited -eq $false)} |
    ft User,ExtendedRights
# Ms-Exch-SMTP-Accept-Any-RecipientがExtendedRightsに含まれるかどうか
```

- SMTP リレー設定把握

```PowerShell
Get-ReceiveConnector Default ○○ | Get-ADPermission -User "NT AUTHORITY\ANONYMOUS LOGON"
```

- 送信コネクタ（ポイント：スマートホストに対して優先度をつけてコントロールしていないか）

```PowerShell
Get-SendConnector | fl
```

- トランスポート設定

```PowerShell
Get-TransportRule
Get-TransportServer
```

- OWA/ActiveSync の URL

```PowerShell
Get-OwaVirtualDirectory
Get-ActiveSyncVirtualDirectory
```

- オフラインアドレス帳

```PowerShell
Get-OfflineAddressBook
```

- ログ関係

```PowerShell
Get-MailboxServer
Get-ClientAccessServer
Get-TransportServer
```

- 組織設定

```PowerShell
Get-OrganizationConfig
Get-OrganizationUnit
```

- AD 関連

```PowerShell
Get- <Server name> | Get-MailboxStatistics -ResultSize Unlimited | Sort -property TotalItemSize -Descending | select Displayname,DatabaseName,ItemCount,TotalItemSize
```

- EMC/パラメータシート確認
  MBX 数など、オブジェクト数
  グループの種類（ユニバーサルでないとメールヒントが動作しない）

```PowerShell
Get-DistributionGroup -Filter {(GroupType -ne "Universal") -and (GroupType -ne "Universal,SecurityEnabled")}
```

- 監査系/冗長周りの確認（アーカイブメールボックス・ジャーナルなど）
- 組織の構成>ハブトランスポート＞ジャーナリング
- 保存期間ポリシー（削除済みアイテムなど）
- Exc 周りの管理者名（組織の構成>Exchange 管理者)

## OS

- ドライブ構成（Explorer スクショで OK）
- OS 情報（稼働 OS のバージョン/エディションは MS 問い合わせ時必須）
- .NET Framework のバージョン(動作前提条件チェック)  
  HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full の Release
- インターネットに出られる環境？そうでない場合も多く、無視できるエラーか判断基準になる
- FW(Windows FireWall)設定は無効化されている？いない？
- IPv6 は無効化されていない？完全無効化してあると不具合の経験有

## サブ機能

- MBX 状態確認

```PowerShell
Get-Mailbox -ResultSize Unlimited | where{$_.IsLinked -eq $True } | fl Alias,*Linked*
```

※リンク済み MBX は過去の仕様上存在する可能性あり。  
エラーなく移行要求作成も同期も可能だが、いつまで経っても完了しない。

- タスクスケジューラによる自動実行処理の確認
- 持ち上がり環境のスキーマ破損チェック
  1. ADSIEdit.msc を起動。
  2. スキーマを選択し、`rangeUpper`の値が想定値か確認
  3. 以下のスキーマ属性の lDAPDisplayName の値が`DUP-<CNの値>-xxxx`のような値であれば破損している
  - CN=ms-Exch-Assistant-Name
  - CN=ms-Exch-LabeledURI
  - CN=ms-Exch-House-Identifier
  - CN=secretary
- トランスポートルールについて（GUI 上で必ずルール一覧を取得）
- 電子メールアドレスポリシの取得

```PowerShell
　Get-EmailAddressPolicy
```

- ActiveSync 設定の確認

```PowerShell
　Get-ActiveSyncMailboxPolicy
```

- リモートドメインの確認

```PowerShell
　Get-RemoteDomain
```

- 階層型アドレス帳か否か

```PowerShell
get-organizationconfig | ft hierarchicaladdressbookroot
```

利用がなければ Null。無効化するときに$null 指定するので
貼り付け元 <https://answers.microsoft.com/ja-jp/msoffice/forum/msoffice_outlook/%E9%9A%8E%E5%B1%A4%E5%9E%8B%E3%82%A2%E3%83%89/e7723a50-bf1c-4d8f-8ff3-532365f133fd>

- Set-Group -identity hoge –ishierarchicalgroup $true
  で階層型アドレス帳に表示するか否か選択できるみたいだから
  Get-Group してから ishierarchicalgroup が真のものを検索すればわかりそう

- [Exchange 組織名の確認](https://blogs.technet.microsoft.com/exchangeteamjp/2016/12/29/migration-batch-fails-in-an-envitonment-wichi-organizationname-with-2byte-characters/)  
  ダブルバイト文字が組織名に含まれていると移行バッチおかしくなるみたい。  
  Exc2007 以降はシングルバイト限定なので、基本それ以降のバージョンで現行稼働中なら問題はないはず？
- IRM を使っているかどうか。（RMS サーバは存在する？） \*普通のコマンドでは抜けてこないので注意（Get-IRMConfiguration）
  クライアントサイドの IRM（Outlook とか）については既定値オンなので Exc 側の設定を見ただけでは分からないことに留意。多分、AD 側の IRM 機能と役割が入ってるかどうかとかそういう見分けになるのか…
- リソースメールボックスが存在しない場合、会議室や設備予約は別システムが担っている可能性あり。そことの連携、役割分担について確認必要
- 共有メールボックスが存在しない場合、一般的なユーザメールボックスにアクセス権限を付与することで共有を行っている可能性あり。運用について確認し、移行時に考慮すべき点が無いか洗い出すこと
- 出来れば全台に接続して確認すること。デスクトップに特定のマシンだけアプリやショートカット、タスクがないかなど、同一バージョンのマシンであっても差異があると後々困るかもしれない。
- MDB 数と名前、DAG がある場合アクティブの状況はどう？（get-databaseavailabilitygroup コマンドでは抜けてこないので要注意！）

### PF

```PowerShell
# DBファイルの実サイズ確認(100GB超の場合PFMBXの分割検討）
# PFオブジェクト数
$PFStat = Get-PublicFolderStatistics -ResultSize Unlimited
$PFStat | Measure-Object
# Countが50万を超えていると一括で移行しきれない
$PFStat | Measure-Object -property Itemcount -Sum -Maximum -Average
# Maximumが100万を超える場合、PFMBXの分割を検討
# アイテムサイズ
$PFStat | sort TotalItemSize -Descending | select -First 10 Name,TotalSize
# アイテムサイズ上位10番までで10GBを超過していないか
```

- 移行対象の文字化け確認  
  AD ユーザとコンピュータで拡張機能を表示、Microsoft Exchange SystemObjects を選択。  
  列の追加と削除で「Exchange エイリアス」「Windows2000 より以前のログオン名」  
  「X.400 電子メールアドレス」「電子メールアドレス」を追加。一覧をエクスポート。  
  終わったら列を元に戻す。
- Exchange 2010 調査時スクショしたポイント
  - 管理された既定のフォルダ
  - アドレス一覧（少し）
  - ジャーナリング設定
  - MDB の設定（ジャーナル受信者の有無、保守スケジュール、送受信サイズ制限、削除済みアイテム保存期間）
  - 承認済みドメイン
  - 送受信コネクタの設定
