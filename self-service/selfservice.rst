.. _selfservice:

------------------
HYCU: セルフサービス
------------------

概要
++++++++
HYCUセルフサービスは、マルチテナントのロールベースのユーザーアクセス管理を可能にする強力な機能セットです。これは、2つの主なユースケースを念頭に置いて設計されています。
- サービスプロバイダーが顧客にBaaSを提供し、データ保護機能にアクセスできるようにします。ITを関与させることなく、ワークロードが保護されていること、またはお客様が自分で高度なバックアップと復元のアクティビティを実行できることを保証します。
- IT環境の管理の負荷分散と他のグループへの所有権の委任（例えば、devopsやdb admins等…）。

HYCUユーザーグループの作成
========================

新しいHYCUユーザーグループを作成することから始めます：

#. 左側パネル > Self-Service > New（画面の左上）

   - Name – HYCU内部ユーザーグループの任意の名前

   .. figure:: images/1.png

#. **Save** をクリックします。

これで、Self-serviceインベントリに新しくeng-grpユーザーグループが作成されました。各ユーザーグループはHYCU内部データベースで異なるスキーマを取得し、HYCUはセキュリティとデータの機密保持のためにデータを明確に分離することに注意してください。

次のステップは、このグループにユーザーを設定して追加することです。HYCUは、ユーザーを単独で管理するか、Active Directoryと統合できます。この演習では、HYCU Backup ControllerをActive Directoryドメインと統合するところから始めましょう。 **adminユーザーでHYCUにログインしてください。**

#. 上部パネル > **管理（歯車アイコン）** > **Active Directory**

   .. figure:: images/2.png

#. **New** をクリックします。

   .. figure:: images/3.png

#. 次の値を入力します：

   - Name – ADドメインの分かりやすい名前
   - Domain – FQDN
   - Provider URL – https://<ホスト名またはIP>
   - ポート番号はオプションの為、入力の必要はありません。

   .. figure:: images/4.png

#. **Save** をクリックします。

   .. figure:: images/5.png

#.  これにより、HYCUは先ほど追加したドメインからADユーザーとADグループを追加できるようになります。単一のHYCU Backup Controller内に複数のADドメインを追加できることに注意してください。これは、共有インフラストラクチャに複数のドメインを持つマルチテナンシー環境を実装するときに非常に役立つ機能です。


Active Directory グループとユーザーの追加
=====================================

HYCUのユーザー管理のすべての機能は、Self-serviceセクションで利用できます。

#. 左側パネル > **Self-Service** > **Users**（画面の左上）

   .. figure:: images/6.png

#. **New** をクリックします。
   ここで、ローカルHYCUユーザー（HYCUによって管理される資格情報）、ADユーザー、およびADグループを追加できます。この演習では、Active Directoryでユーザーを構成済みのADグループを使用します。

   - Common Name – ADサーバーに存在する正確なADグループ名

   .. figure:: images/7.png

#. Active Directory下のドロップダウンで使用可能な作成済みのADドメインを指定し、 **Save** をクリックします。

   .. figure:: images/8.png

#. ユーザーが実際にHYCUユーザーグループに追加されるまで、HYCUにログインすることはできません。それでは、以前に定義したHYCUユーザーグループにユーザーを追加しましょう。 **eng-grp** グループ >  **Add to Group** をクリックします。

   .. figure:: images/9.png

#. HYCUに登録したADグループを追加し、Backup Operatorのロールを割り当てます。HYCUには4つのロールがあることがわかります：

   - Viewer - 読み取り専用権限を持つロール
   - Backup Operator - VMの検出とバックアップの権限を持つロール
   - Restore operator - 復元の権限を持つロール
   - Administrator - バックアップと復元の権限、およびユーザー管理とレポートの権限を持つロール

#. **Add User** をクリックします。これにより、バックアップのみの権限を持つADグループ **hycugrp** がHYCU内部ユーザーグループ **eng-grp** に追加されます。


リソースの所有権をグループに割り当てる
=======================================

#. VMと共有フォルダの所有権をeng-grpに割り当てます。左側パネル> **Virtual Machines** > 対象となるVMを選択し、 **Owner**（画面の右上）ボタンをクリックします。

   .. figure:: images/10.png

#. 作成したグループ **eng-grp** を選択し、 **Assign** をクリックします。

   .. figure:: images/11.png

#. これにより、VMの所有権がeng-groupに割り当てられます。同じことは、共有フォルダについても行うことができます。左側パネル>  **Shares** > 対象となるShareを選択 > **Owner**（画面の右上）ボタンをクリックします。

   .. figure:: images/12.png

#. **eng-grp** を選択し、 **Assign** をクリックします。

   .. figure:: images/13.png

#. この後すぐに、**Owner** 列にそれぞれのVM/Shareを所有するユーザーグループが表示されます。デフォルトのInfrastructure Groupの管理者メンバーは、このデータの所有権を失うため、ポリシーを割り当ててバックアップを開始したり、復元したりできなくなります。ユースケースによっては、顧客がデータ保護を完全に実行できることだけでなく、ときどき特定の管理アクティビティを依頼される場合があります。この場合は、管理者ユーザーをそれぞれのテナントユーザーグループにも追加する必要があります：

   #. eng-grpグループ > Add to Groupをクリックし、Administratorロールを持つ管理者ユーザーを追加します。

   #. 次に、右上隅に移動してInfrastructure Groupグループをクリックし、テナントユーザーグループを選択して、Switchをクリックします。これで、それぞれのユーザーグループのメンバーとしてログインし、ユーザーに代わってアクションを実行できます。真のマルチテナンシーから予想されるように、どのユーザーも複数のユーザーグループに属していて、それらの間を移動することができます。

   .. figure:: images/14.png

   .. note:: VM/Shareの所有権をグループに（再）割り当てると、機密性の制約により、前のグループで行われたバックアップは削除されます。ユーザーグループが最初から計画されていることを確認してください。

ロールベースアクセス・コントロールのデモ
=======================================

**eng-grp** グループのメンバーとしてログインしてみましょう。この場合はADグループhycugrpのメンバーになります。この演習では、 **hycuusr1** という名前のADユーザーが、ADグループ **hycugrp** のメンバーとしています。ADユーザーを使用してログインするには、username@FQDN（例：hycuusr1@ntnxlab.local）と入力してください。

.. figure:: images/15.png

#. ログイン後、Virtual Machines/Sharesに移動し、Infrastructure Groupによって割り当てられたVM/共有フォルダのみが表示されることを確認します。TargetsとSelf-serviceオプションはグレー表示されていることに注意してください。デフォルトの **Infrastructure Group** とそのメンバーのみが、ターゲットを構成する明示的な権限を持っています。他のすべてのグループとそのメンバーは、ターゲットを表示できません。

   .. figure:: images/16.png

#. ユーザーが **Backup Operator** のロールではなく **Administrator** のロールを持っている場合は、Self-serviceオプションが有効になります。ユーザーには、それぞれのHYCUユーザーグループからユーザーを追加または削除する権限しかありません。ユーザーをHYCUに追加する権限はまだありません（ **Infrastructure Admin** グループの管理者権限がない場合）。

#. Policiesに移動し、ユーザーグループのメンバーがバックアップポリシーを変更できないことを確認します。それらを表示して割り当てるだけです。デフォルトでは、HYCUにログインすると、テナントはすべてのポリシーを表示して割り当てることができます。これは、サービスプロバイダーが自身のGold/Silver/Bronzeポリシーを作成し、それらの使用に基づいて課金するシナリオで役立ちます。


マルチテナント向けカスタムポリシー
######################################


場合によっては、各ユーザーグループ（テナント）ごとに特定のポリシーを定義し、グループごとに異なるターゲットを許可することが理にかなっています。その場合、各ユーザーグループが自分のポリシーのセットのみを参照できるようにする必要があります。これを実現し、特定のユーザーグループにポリシーの所有権を割り当てるには、HYCU構成ファイルを調整する必要があります。
これは、管理画面からはまだサポートされていません：

- ユーザーグループの名前をプレフィックスとして使用し、バックアップポリシーを作成します。

  - 例えば、ユーザーグループ名が **eng-grp** の場合、バックアップポリシーは **eng-grp** <policy_name>のような名前にする必要があります。

- ユーザーグループに適切なポリシーを作成したら、SSHを使用してBackup Controllerに接続します。

  - ユーザー名：hycu、パスワード：hycu/4u

- /opt/grizzlyに移動します。

  - viエディターを使用してconfig.properties ファイルを開きます。

- 次のオプションを追加します:

.. code-block:: powershell

    policies.group.specific.synchronized=true

- grizzly serviceを再起動します:

.. code-block:: powershell

    services grizzly restart

#. 完了後、各ユーザーグループのメンバーは、自分用に構成されたポリシーのみを表示して割り当てることができます。

   .. figure:: images/17.png

#. 最後に、VM/共有フォルダにポリシーを割り当てて、バックアップを実行します。Backup OperatorまたはAdministratorのロールを持つユーザーは、資格情報を設定し、アプリケーションを検出して保護することもできます。Restore OperatorとAdministratorのロールを持つユーザーは、復元を実行し、ファイルとアプリケーションの粒度の細かな復元を実行することもできます。
