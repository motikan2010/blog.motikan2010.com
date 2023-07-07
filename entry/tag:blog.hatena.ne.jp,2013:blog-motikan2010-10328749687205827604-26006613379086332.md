<div style="text-align: center;">[f:id:motikan2010:20190728232522j:plain]</div>

<div class="contents-box">
  <p>[:contents]</p>
</div>

## はじめに

　プロジェクト管理ツールとし有名な Jira Server の脆弱性が見つかりました。  

　脆弱性を含んでいるバージョンを利用しているかつ、特定の設定を有効にしている場合に、<b>未ログインユーザでも</b>「サーバーサイド・テンプレート・インジェクション(server-side template injection)」攻撃が可能ということで少し調べてみました。  

　Poc も存在しており、実際にその検証している様子もありますので、<b>Jiraサイトをパブリックに公開している</b>運用している場合には、バージョンの確認・設定の確認をしてみたほうがよいでしょう。  

<!-- more -->

## 概要

　脆弱性を含んでいるバージョンや影響が記載されています。  
[https://confluence.atlassian.com/jira/jira-security-advisory-2019-07-10-973486595.html:embed:cite]

[https://jp.tenable.com/blog/cve-2019-11581-critical-template-injection-vulnerability-in-atlassian-jira-server-and-data:embed:cite]

#### 脆弱性についてより詳しく

[https://paper.seebug.org/982/:title]

## 検証情報（キャプチャ画像有り）

　実際に検証された方のレポートです。  
フォームの入力値に含まれている `curl`コマンド が実行されていることが確認できます。
[https://github.com/jas502n/CVE-2019-11581/:embed:cite]

### PoC（検証コード）

　<span style="color: #ff0000">PoC は既に公開されています。</span>  

　私は下記の PoC を利用して脆弱性を確認しました。特定のリクエスト1つ送ることで攻撃できるので、攻撃が容易であることが分かります。  
[https://github.com/kobs0N/CVE-2019-11581:embed:cite]

### 検証

#### 検証パッケージ（Jira Server 7.13.4）

　脆弱性を含んでいるJiraは下記のリンクから取得しました。  
※ダウンロードリンク  
https://product-downloads.atlassian.com/software/jira/downloads/atlassian-jira-software-7.13.4-x64.bin

#### Jira Server インストール手順(AWS)

[https://dev.classmethod.jp/cloud/aws/atlassian-jira-5_2_5-on-amazon-linux/:embed:cite]

### 攻撃可能な設定

[https://confluence.atlassian.com/jira/jira-security-advisory-2019-07-10-973486595.html#JIRASecurityAdvisory2019-07-10-Description:embed:cite]

　下記**いずれか**の条件に当てはまる設定時に悪用可能とのことです。  
条件1.
> an SMTP server has been configured in Jira and the Contact Administrators Form is enabled; or

条件2.
> an SMTP server has been configured in Jira and an attacker has "JIRA Administrators" access.

　条件1. ⇨ 「Contact Administrators Form」は未ログインユーザでもJira管理者に対してメールを送ることができる機能であり、必須な機能ではないと思われるので OFF にしても問題ないでしょう。

　条件2. ⇨ 攻撃者が「JIRA 管理者権限」を持つ必要があり、攻撃の対象なりづらいことから、緊急を要して対応する必要もないでしょうか。

## 対応

　脆弱性への対応方法について、バージョンを上げる以外にも下記のサイトのように対応する方法あります。  
[https://confluence.atlassian.com/jira/jira-security-advisory-2019-07-10-973486595.html#JIRASecurityAdvisory2019-07-10-WhatYouNeedtoDo:embed:cite]

## まとめ

　「Contact Administrators Formが有効になっている」かつ「Jiraサイトがパブリックに公開されている」場合はいつ攻撃を受けてもおかしくない状態ですので、早めに対応した方がよいでしょう。

## 追記

　警視庁によると当脆弱性のスキャンが観測されたとのことです。  
  
[f:id:motikan2010:20191003193439p:plain:w500]

[https://www.npa.go.jp/cyberpolice/important/2019/201910021.html:embed:cite]

## 更新履歴

- 2019年7月28日 新規作成
- 2019年10月3日 警視庁リンク追加
