<a name="top"></a>

# Microsoft GIGAスクールパッケージ 利用状況可視化テンプレート

Microsoft GIGAスクールパッケージ導入後の端末やMicrosoft 365の利用状況を可視化するためのジョブとレポートを実装したサンプルです。

画像を挿入

## 目次

- [本プロジェクトについて](#-本プロジェクトについて)
- [前提条件と利用可能なレポート](#-前提条件と利用可能なレポート)
- [事前準備](#-事前準備)
- [レポート利用方法](#-レポート利用方法)
- [ドキュメント](#-ドキュメント)

## 🚀 本プロジェクトについて

### 背景

文部科学省の「GIGAスクール構想の実現 学習者用コンピュータ最低スペック基準の一部訂正について（依頼）」には、GIGAスクール構想で普及した端末の利用状況を把握する機能が必要であると記載されています。

> **2.9. 端末の稼働状況を把握できる機能について**
> 
> 本機能は、プライバシー保護に十分留意した上で、端末の利活用状況を客観的に把握するために具備する必要がある（文部科学省による端末の利活用状況の調査において、こうした客観的データに基づく回答を求めることとなる。）...

詳細は[こちら](https://www.mext.go.jp/content/20240201-mxt_shuukyo01-000033777_01.pdf)を参照ください。

しかし、Microsoft 365管理センターで確認できるログは保管期間が制限されており、長期間のログ保管機能は標準では提供されていません。  
※ 例：Microsoft 365利用状況レポートのログは最大180日までしか参照できません。

そこで、Microsoft 365のログやユーザー情報を定期的にSharePointに蓄積し、Power BIを使って可視化できるようにするため、本プロジェクトが始まりました。

### 目的

本プロジェクトの目的は以下の通りです。

- **テナント全体のMicrosoft 365利用ログ蓄積**: GIGAスクール構想で導入したMicrosoft 365テナントの利用状況を文部科学省に報告する必要がある可能性があるため、SharePoint上で長期的なログの保管を行います。
- **Microsoft 365利用状況の可視化**: 蓄積されたログをPower BIを使って視覚的にわかりやすく可視化します。
- **組織別の利用状況可視化**: 学校や学級単位での利用状況を把握するために、所属情報をもとにした可視化もサポートします。

## 🎓 前提条件と利用可能なレポート

テナントのライセンスや名簿情報の運用状況に応じて利用可能なレポートが異なります。

| レポート種別 | 必要なライセンス | ログ匿名化設定 | 名簿情報の運用 |
|:-|:-|:-|:-|
| 010_テナントの利用状況可視化サンプル | Microsoft 365 A1 | 有 | 無 |
| 020_学校毎の利用状況可視化サンプル | Microsoft 365 A1 | 無 | 有 |
| 030_学級内の利用状況可視化サンプル | Microsoft 365 A5（またはMicrosoft 365 A1 + Power BI Pro） | 無 | 有 |

## 📝 事前準備

### オプション: 環境構築に必要なツールのインストール

必要に応じて以下のツールをインストールしてください。インストールされていない場合は、次の手順を実行することをお勧めします。

- **Gitのインストール**

  以下のコードを実行するか、[こちら](https://gitforwindows.org/)の手順に従ってインストールしてください。
  ```shell
  winget install --id Git.Git -e --source winget
  ```

- **GitHub CLIのインストール**

  以下のコードを実行するか、[こちら](https://github.com/cli/cli/releases/)の手順に従ってインストールしてください。
  ```shell
  winget install --id GitHub.cli
  ```

- **Azure CLIのインストール**

  以下のコードを実行するか、[こちら](https://learn.microsoft.com/ja-jp/cli/azure/install-azure-cli-windows?tabs=azure-cli)の手順に従ってインストールしてください。
  ```shell
  Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi
  Start-Process msiexec.exe -ArgumentList '/I AzureCLI.msi /quiet' -Wait
  ```

- **Power BI Desktopのインストール**

  詳細は[こちら](https://learn.microsoft.com/ja-jp/power-bi/fundamentals/desktop-get-the-desktop)の手順に従ってください。

- **GitHubアカウントと組織の作成**
  1. [GitHub](https://github.com/)にアクセス
  2. サインアップをクリックしてアカウントを作成
  3. 「Your Organization」から新規組織を作成

### 必須手順

1. **リポジトリの作成**  
   GitHub上でプライベートリポジトリを作成します。  
   「Your Organization > {組織名} > repository > New repository」から作成してください。

  ※**必ず"Private"を選択**し、**"Add a README file"にチェック**を付けてください。

2. **設定ファイルの編集**  
   `params.json`ファイル内で"Organization name", "Repository name"の2項目を編集し、組織名とリポジトリ名を入力します。

   例) Organization name = "TestOrganization", Repository name = "TestRepository"の場合
   ```json
   {
     "githubOrganizationName": "TestOrganization",
     "githubRepositoryName": "TestRepository",
     "concealed": true,
     "appId": "",
     "tenantId": "",
     "tenantDomain": "",
     "siteUrl": ""
   }
   ```

3. **デプロイスクリプトの実行**  
   PowerShellで次のコマンドを実行し、認証などの指示に従います。

   ```shell
   git clone https://github.com/{~}/ms-device-usage-report.git
   cd ms-device-usage-report/setup
   .\deploy.ps1
   ```

4. **GitHubシークレットの設定**  
   GitHubにアクセスし、リポジトリのシークレットを追加します。

   | Name | Value |
   |:-|:-|
   | AZURE_TENANT_ID | {「params.json」の"tenantId"} |
   | AZURE_TENANT_NAME | {「params.json」の"tenantDomain"} |
   | AZURE_CLIENT_ID | {「params.json」の"appId"} |
   | SITE_NAME | M365UsageRecords |
   | DOC_LIB | Shared%20Documents |

5. **動作確認**  
   SharePointにデータが出力されているか確認してください。データが正しく出力されていない場合は、設定を再確認してください。
   SharePointサイトのURLは`params.json`ファイル内を参照してください。

## 📃 レポート利用方法

前提条件別に以下3種のレポートを公開しています。リンク先の手順に従ってそれぞれ利用してください。

1. [Microsoft 365 テナント全体の利用状況可視化サンプル](./010_テナントの利用状況可視化サンプル/Readme.md)
2. [学校毎の利用状況可視化サンプル](./020_学校毎の利用状況可視化サンプル/Readme.md)
(利用のためには各Microsot 365 IDがどの学校に所属しているのかを示す名簿ファイルの作成が必要)
3. [学級・児童生徒毎の利用状況可視化サンプル](./030_学級内の利用状況の可視化と共有サンプル/Readme.md)
(2同様名簿ファイルの作成が必要)


## 📚 ドキュメント

本プロジェクトに関連するドキュメントはこちらです。

- aaa
- bbb
- 推奨設定

ご覧いただき、ありがとうございます。Microsoft 365の利用状況把握に少しでもお役立ていただければ幸いです。

[Back to top](#top)
