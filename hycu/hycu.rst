.. _hycu:

----
HYCU
----

**この演習の推定所要時間は60分です。**

概要
++++++++

HYCUは、Nutanix AHVおよびESXiクラスターに完全なバックアップ機能を提供するためにゼロから構築された唯一のソリューションであり、AHVの導入を検討する顧客の懸念を排除します。 HYCUは、Nutanix VM、Volumes、Nutanix Files環境のバックアップに利用できます。

ソフトウェアのみのアプライアンスにより、HYCUは、追加のノードがバックアップターゲット（保存場所）として機能するため、Nutanixの案件拡大に役立ちます。

HYCUは、Nutanix以外のESXiおよび物理Windows Server環境も完全にサポートしているため、顧客はレガシープラットフォームからNutanixの統合バックアップソリューションに移行できます。また、 HYCUはNutanix上の次のアプリケーションとファイルサービスをバックアップできます：

- Microsoft SQL Server (フェールオーバークラスタおよびAvailability Groupを含む)
- Microsoft Exchange Server (Database Availability Group (DAG)を含む)
- Microsoft Active Directory
- Oracle Database
- Microsoft Failover Cluster
- SAP HANA
- Nutanix Files
- Nutanix Volume Groups
- A-syncおよびNear-sync スナップショット

HYCUは、Nutanixのセカンダリストレージ製品であるNutanix Mineとバンドルできます。

.. figure:: images/mine_hycu_logo.png

**この演習では、HYCU仮想アプライアンスを展開して構成し、Nutanix環境のさまざまなワークロードのデータのバックアップと復元について学びます。**

HYCUアプライアンスの構成
++++++++++++++++++++++++++

#. **Prism > VM > Table** から **+ Create VM** をクリックします。

#. フィールドに値を入力し、 **Save** をクリックします：

   - **Name** - *Initials*\ **-HYCU**
   - **vCPU(s)** - 4
   - **Number of Cores per vCPU** - 1
   - **Memory** - 4 GiB
   - **+ Add New Disk** を選択

     - **Operation** - Clone from Image Service
     - **Image** - hycu-\*.qcow2

     - **Add** を選択

   - **+ Add New Disk** を選択

     - **Operation** - Allocate on Storage Container
     - **Storage Container** - Default
     - **Size (GiB)** - 32
     - **Add** を選択
   - **CD-ROM** Diskを削除
   - **Add New NIC** を選択

     - **VLAN Name** - Secondary
     - **Add** を選択

#. *Initials*\ **-HYCU** VMを選択し、 **Power on** をクリックします。

   保護対象VMの数に応じてバックアップアプライアンスのCPUおよびメモリリソースを増やします。サイジングの詳細については `HYCU User Guide <https://support.hycu.com/hc/en-us/sections/115001018365-Product-documentation>`_ を参照してください。

   アプライアンスに追加されたセカンダリディスクは、ローカルHYCUデータベースおよびメンテナンス用のデータディスクとして使用されます。 VMバックアップのデータは保存しません。

#. **Launch Console** をクリックします。

#. **1 HYCU Backup Controller** を選択します。

   .. figure:: images/0.png

#. 以下のフィールドを値を入力し、 **OK** をハイライトして **Enter** キーをクリックします：

   - **Hostname** - *Initials*\ **-HYCU**
   - **IPv4 address** - *Specify the IP Assigned by IPAM DHCP*
   - **Subnet mask** - 255.255.255.128
   - **Default gateway** - *Secondary VLAN Gateway* (例： 10.XX.YY.129)
   - **DNS server** - *DC VM IP*
   - **Search domain** - ntnxlab.local

   .. note:: フィールドの切り替えにはTabキーをクリックします。

   .. figure:: images/1.png

   内部データベースが初期化され、バックアップ・コントローラーのサービスが開始されるまで、約1分待ちます。

#. デフォルトの資格情報をメモします。 **Enter** キーをクリックして **HYCU** VMコンソールを閉じます。

   .. figure:: images/2.png

Backup Source（保護対象）の追加
++++++++++++++++++++++

HYCUは、AHVまたはESXiホストのNutanixクラスターとの緊密な統合を提供します。 HYCUは、従来のハイパーバイザーの「スタン」スナップショットに依存するのではなく、Nutanix分散ストレージファブリックと直接APIを介して変更されたブロックを特定し、効率的なNutanixのスナップショットを活用します。スナップショットについてはこちら：`Redirect-on-write snapshots <https://nutanixbible.com/#anchor-book-of-acropolis-snapshots-and-clones>`_

HYCU仮想アプライアンスが展開されているクラスターがNutanix Mineの場合、Nutanix MineクラスターをHYCU内のソース（保護対象）とターゲット（保存場所）の両方として追加する必要があります。

HYCUをMineに展開する際、Nutanixクラスターをソースとして追加した後、ワンクリックでHYCUダッシュボードをPrismに登録できます。

#. ブラウザから \https://<*HYCU-VM-IP*>:8443/ を開きます。 既定の資格情報を使ってログインします：

   - **Username** - admin
   - **Password** - admin

#. ツールバーから、:fa:`cog` **> Sources** をクリックします。

   .. figure:: images/3.png

#. **+ New** をクリックし、 以下のフィールドに値を入力します：

   - **URL** - *Prism ElementのURL* (例：https://10.XX.YY.37:9440)
   - **User** - admin
   - **Password** - *Prism Element Password*

#. **Next** をクリックします。

#. HYCUがNutanixクラスターを検証します。 **Save** をクリックします。

   .. figure:: images/4.png

#. ジョブの開始後、 **Close** をクリックします。

   すべてのジョブは非同期で実行され、 **Jobs** ページで確認できます。

   .. figure:: images/5.png

   .. note:: **Nutanix Mine with HYCUにおける注意点：** Nutanix Mineクラスターの場合、ダッシュボードをMine Prismに展開できます。ソースの下でMineクラスターを強調表示し、[Register with Prism]をクリックしてHYCUダッシュボードをPrismに展開します。 この環境はGlobal Tech Summitの共有クラスターであるため、Mine Prismにダッシュボードを展開しないでください。

   .. figure:: images/6.png

    HYCUダッシュボードをPrism Elementに展開すると、クラスターのPrismサービスが自動的に再起動します。

    .. figure:: images/7.png

#. **HYCU** サイドバーから、:fa:`bars` **> Virtual Machines** をクリックし、クラスターのVMがリスト表示されていることを確認します。

Backup Target（保存場所）の追加
++++++++++++++++++++++

ターゲットはバックアップデータを保存するために使用されます。HYCUは以下のターゲットをサポートします。
   - Nutanix (Nutanix独自のiSCSI)
   - iSCSI
   - NFS (Nutanix Filesを含む)
   - SMB (Nutanix Files含む)
   - AWS, S3 (Nutanix Bucketsを含む)
   - Azure
   - Google Cloud Platform (GCP)

この演習では、NutanixをVMバックアップデータのターゲットとして使用します。 Nutanix VolumesとNutanix Objectsを通じて、2つの異なるターゲットストレージを利用できます。


Nutanix Volumesをターゲットとして設定
+++++++++++++++++++++++++++++++++++++++

HYCUはNutanixクラスター上でネイティブに実行されます。 本番クラスターまたはセカンダリストレージクラスターのどちらにも展開できます：
   - Nutanix Mine環境では、HYCUアプライアンスとターゲットストレージは同じクラスターに存在します。
   - Nutanix Mine以外の環境では、HYCUアプライアンスはソースVMと同じクラスター上に展開し、ターゲットストレージはソースVMと異なるクラスターに作成します。

HYCUを使用すると、Nutanixクラスター（Mineかどうかに関係なく）をターゲットとして非常に簡単に構成できます。 Prism Elementの資格情報を指定した後、HYCUは複数のvDiskでボリュームグループを自動的に構成し、外部iSCSIアクセスを有効にします。 次に、ボリュームグループはXFSでフォーマットされ、基盤となるvDisk全体にデータをストライプできるため、書き込みパフォーマンスが最大化され、バックアップ時間を最小化できます。 そして、HYCUはこのVolume Groupをバックアップターゲットとして活用します。

.. note:: Nutanixをターゲットとして登録する前に、iSCSI Data Services IPが構成されていることを確認してください。

   .. figure:: images/8.png

#. **HYCU** サイドバーから、:fa:`bars` **> Targets** をクリックします。

#. **+ New** をクリックします。以下のフィールドに値を入力し、最後に **Save** をクリックします。

.. note:: この手順では、Nutanixストレージコンテナの設定を構成できます。Nutanixが推奨するバックアップワークロードのベストプラクティスに従います。原則として、ハードウェア圧縮は有効にできますが、重複排除は無効のままにしておく必要があります。クラスターに4つ以上のノードがある場合は、Erasure Coding を有効にすることを検討してください。

   - **Name** - Nutanix_VG
   - **Concurrent Backups** - 4
   - **Description** - *Nutanix Cluster Name* HYCU-Target VG
   - **Type** - Nutanix
   - **URL** - *Prism ElementのURL* (例：https://10.XX.YY.37:9440)
   - **Username** - admin
   - **Password** - *Prism Element Password*

   .. figure:: images/9.png

複数のターゲットを登録することも可能です。

#. ターゲットの展開は約3分で完了します。HYCUの"Jobs"メニューから進行状況を確認できます。

#. HYCUはVolume Groupを自動的に展開します。 ターゲットの設定が完了すると、HYCU ContainerとVolume Groupが作成されたことが、Prism Elementから確認できます：

.. figure:: images/10.png


Nutanix Objectsをターゲットとして設定
+++++++++++++++++++++++++++++++++++++++

HYCUは、S3互換オブジェクトにバックアップする機能があり、Nutanix Objectsは最適なユースケースになります。 HYCUはNutanix Objectsにネイティブで対応しており、プロキシ等を使用することなくバックアップやコピー、そしてアーカイブすることができます。さらに、Nutanix Objects WORM機能（オブジェクトロック）とシームレスに統合し、ランサムウェアからデータを適切に保護します。

Nutanix Objectsは3つのユースケースがあります。
   - Mine with HYCUをセカンダリストレージとして使用し、Nutanix Objectsを2次コピーとアーカイブ用途で使用します。
   - 既存のお客様のストレージと組み合わせて、Nutanix Objectsを2次コピーとアーカイブ用途で使用します。
   - HYCUアプライアンスをNutanix Objects上に展開し、ランサムウェア対策として使用します。

Nutanix ObjectsとHYCUの組み合わせによるセキュリティ対応は
   - HYCUは、ロックダウンされたCentOSバージョン8ベースのアプライアンスであり、リリースごとに最新のセキュリティパッチで更新しています。
   - HYCUは、Fast Restoreオプション機能により、Nutanixスナップショットを追加の保護レイヤーとして保持できます。
   - HYCUのソフトウェアWORM機能は、バックアップデータを人的ミスもしくは悪意のある削除から保護します。
   - エンドツーエンドの暗号化をサポートします。

HYCU内でのObjectsの設定はとてもシンプルで、Objectsへの書き込みパフォーマンスは、従来のiSCSIバックアップターゲットを使用した場合と同等です。

.. note:: 時間を節約するために、Prism Central内でObjectsを有効にし、"ntnx-objects"という名前のObject storeを事前に展開しています。このObject store内にBucketを作成します。

Access Keysの作成
..................

#. Prism Central > Services > Objectsに進みます。

#. 左上のメニューから"Access Keys"をクリックします。

#. "+ Add People"をクリックし、 "Add people not in a directory service"を選択します。次に" *Initials*-hycu@ntnxlab.local." をEmail Addresses欄に入力し、Nextをクリックします。

   .. note:: ローカルユーザーではなく、ここではユーザー認証用のディレクトリサービスを設定できます。

   .. figure:: images/32.png

#. “Download Keys“をクリックし、 ユーザー認証キーをローカルマシンにダウンロードします。 次にCloseをクリックします。 後ほどHYCU内でバケットを構成するときにこのキーを使用します。

   .. figure:: images/33.png

Bucketの構成
....................

#. "ntnx-objects"をクリックし、"Create Bucket"を選択します。

#. バケットの名前を "*initials*-hycu-bucket"とし、デフォルトオプションのまま"Create"をクリックします。

   .. figure:: images/34.png

#. 作成後に "*initials*-hycu-bucket"をクリックし、"User Access"を選択します。次に"Edit User Access"をクリックします。

#. "*initials*-hycu@ntnxlab.local" と入力し、"Read"と"Write"オプションの両方を選び、Saveをクリックします。

   .. figure:: images/35.png

#. ランサムウェア対策には、"*initials*-hycu-bucket"バケットを作成し、Actions > Configure WORMに進みます。

   .. figure:: images/42.pnp

#. WORM機能を有効にするには、Retention periodを7 daysと入力し、"Enable WORM"をクリックします。

   .. figure:: images/41.png

HYCU内でNutanix Objectsを設定
.....................................

#. 新しいブラウザタブでHYCUインターフェースに戻り、ログインします（必要な場合）。 HYCU WebインターフェースがTCPポート8443を使用してHTTPSでリッスンすることを思い出してください。

#. 左側のメニューからTargetsに進みます。

   .. figure:: images/36.png

#. 右上の"+ Add"ボタンをクリックします。

#. ターゲットの名前をNTNX_Objectsにします。

#. **Use for Archiving** オプションを有効にします。

#. Typeで"AWS S3/Compatible"を選択します。

#. Service endpointとして、 `http://[objects client used IP]` を入力します。このIPは Prism  CentralでObject storeをクリックすることで確認できます。

   .. figure:: images/37.png

#. バケット名として "*initials*-hycu-bucket" を入力します。

#. 前にダウンロードしたファイルからAccess KeyとSecret Accessを取得し、Nutanix Objectsのユーザーとして使用します。"Save"をクリックします。

   .. figure:: images/38.png

既存のHYCUポリシーを変更するか、Objectsへアーカイブする新しいポリシーを作成できるようになりました。


Backupポリシーの構成
+++++++++++++++++++++++++++

HYCUポリシーは、データが失われる可能性のある最大許容期間を指定することにより、ビジネスのサービスレベル目標（SLO）要件をデータ保護要件にマップするように設計されています。- 目標復旧時間（RTO）。
ポリシーで、RPO (Backup Every)、RTO (Recover Within)、Retention、そしてBackup target(s)を定義することにより、これらのSLAをVMグループに簡単に適用できます。

#. **HYCU** サイドバーから、:fa:`bars` **> Policies** をクリックします。

   既定で4つのポリシーが構成されています:

   - **Gold** - RPO4時間、RTO4時間
   - **Silver** - RPO12時間、RTO12時間
   - **Bronze** - RPO24時間、RTO24時間
   - Exclude - バックアップから除外

#. カスタムポリシーを作成するには、 **+ New** をクリックします。

#. 以下のフィールドに値を入力し、 **Save** をクリックします:

   - **Name** - Platinum
   - **Description** - 2 Hour RPO/RTO, Fast Restore Enabled (1 Week)
   - **Enabled Options** - Backup, Fast Restore
   - **Backup Every** - 2 Hours
   - **Recover Within** - 2 Hours
   - **Retention** - 2 Weeks
   - **Targets** - Nutanix_VG
   - **Backup Threshold** - 25%
   - **Fast Restore Retention** - 1 Weeks

   .. figure:: images/11.png

   HYCUは、管理者がRTO目標を定義できる点が特徴的です。希望の **Recover Within** 期間を指定し、ターゲットで **Automatic** を選択すると、HYCUはVMを転送するのに適したターゲットを計算します。ターゲットのパフォーマンスは常に監視され、構成された時間内でデータを復元できることが保証されます。 HYCUインスタンスに複数のターゲットが設定されている場合、サブセットを選択でき、HYCUはターゲットからインテリジェントに選択します。

   バックアップポリシーには、次のような複数の詳細構成があります。

   - **Backup Windows** - 管理者は、ジョブ実行の細かい時間帯と曜日のスケジュールを定義し、バックアップポリシーに適用できます。
   - **Copy** - ピーク外の時間帯で、データをプライマリターゲットからセカンダリターゲットに非同期でコピーします。
   - **Archiving** - 管理者は、フルバックアップを長期間保存するために、コールドストレージを使用することができます。
   - **Fast Restore** - Nutanixクラスターのローカルスナップショットを保持し、復元時に利用することで、迅速な復元を実現します。
   - **Auto-assignment** - Prism CentralのVM CategoriesもしくはvCenterのカスタム属性により、HYCUは新たに見つけた仮想マシンへ自動的に適切なポリシーを割り当てます。

   HYCUは、管理者がRTOを定義する機能も特徴的です。 **Recover Within** を指定してターゲットで **Automatic** を選択すると、HYCUはバックアップを転送する適切なターゲットを計算します。 ターゲットのパフォーマンスは常に監視され、構成されたウィンドウ内でデータを復元できることが保証されます。 HYCUインスタンスに複数のターゲットが設定されている場合、サブセットを選択でき、HYCUは選択されたターゲットから適切に選択します。

#. Nutanix Objectsへのアーカイブを構成するには、右上のメニューから"Archiving"をクリックして、アーカイブプロンプトを開きます。次に+Newをクリックします。

#. アーカイブの名前を"Nutanix_Objects"にします。

#. Monthly Archiveを有効にし、先の手順で作成した"Nutanix_Objects"を選択します。

   .. figure:: images/39.png

#. Saveをクリックし、次にPlatinumポリシーを編集（Edit）します。

   .. figure:: images/43.png

#. Archivingにチェックを入れて、有効化します。

   - **Enabled Options** - Archiving
   - **Data Archive** - Nutanix_Objects

   .. figure:: images/40.png

#. Saveをクリックします。

#. **Exclude** ポリシーを選択し、  **Set Default > Yes** をクリックします。

   .. figure:: images/12.png

   このデフォルトポリシーにより、VMがHYCUによって既定でバックアップされないようにします。本番環境では、適切なポリシーを選択し、既定ですべてのVMをバックアップできます。 ソースクラスター上で作成された新しいVMには、デフォルトのポリシーが自動的に適用されます。

仮想マシンのバックアップ
+++++++++++++++

この演習では、iSCSI Volume GroupがマウントされたWindows Server VMをバックアップします。 ゲスト内のiSCSIディスクは、高可用性のために共有ストレージを必要とするSQL Serverなどのエンタープライズアプリケーションでは一般的です。

Windows VMを作成し、Nutanix Prismを介してVMにNutanix Volume Groupを追加します。これは、VM iSCSIイニシエーターを使用して行うこともできます。

#. **Prism > VM > Table** に進み、 **+ Create VM** をクリックします。

#. 以下のフィールドに値を入力し、 **Save** をクリックします:

   - **Name** - *Initials*\ -HYCUBackupTest
   - **vCPU(s)** - 2
   - **Number of Cores per vCPU** - 1
   - **Memory** - 4 GiB
   - **+ Add New Disk** を選択

     - **Operation** - Clone from Image Service
     - **Image** - Windows2012R2.qcow2
     - **Add** を選択
   - **Add New NIC** を選択

     - **VLAN Name** - Secondary
     - **Add** を選択

#. *Initials*\ **-HYCUBackupTest** を選択し、 **Power on** をクリックします。

#. VM起動後、 **Launch Console** をクリックします。

#. Sysprepプロセスを完了し、ローカル管理者アカウントのパスワードを入力します。 (例：nutanix/4u)

#. **Prism Element > Storage > Table > Volume Groups** から、 **+ Volume Group** を選択します。

#. 以下のフィールドに値を入力します:

   - **Name** - *Initials*\ -BackupTestVG
   - **iSCSI Target Name Prefix** - *Initials*\ -HYCU-Target
   - **Description** - VG attached to HYCUBackupTest VM
   - **+ Add New Disk** を選択
     - **Storage Container** - Default
     - **Size (GiB)** - 10
   - **Save** を選択
   - 新たに作成したVolume Groupをダブルクリック

   - **+ Attach to a VM** を選択

     - **Available VMs** - *Initials*\ -HYCUBackupTest の前に作成されたVMを選択
     - **Attach** を選択

#. **Save** をクリックします。

#. *Initials*\ **-HYCUBackupTest** コンソールまたはRDPセッションに戻ります。

#. PowerShellを開いて次のコマンドを実行し、ディスクを有効にしてフォーマットします：

   .. code-block:: powershell

     Get-Disk -Number 1 | Initialize-Disk -ErrorAction SilentlyContinue
     New-Partition -DiskNumber 1 -UseMaximumSize -AssignDriveLetter -ErrorAction SilentlyContinue | Format-Volume -Confirm:$false

#. 次のコマンドを実行し、WinRMを有効にします。

   .. code-block:: powershell

    Enable-PSRemoting –force
     # Set start mode to automatic
     Set-Service WinRM -StartMode Automatic
     Set-Item WSMan:localhost\client\trustedhosts -value *

#. 最後に、iSCSI（E:)ディスクだけでなく、OS（C:)ディスク（デスクトップ上のテキストファイルなど）に複数のファイルを作成します。

   .. figure:: images/13.png

#. **HYCU** サイドバーから、 :fa:`bars` **> Virtual Machines** を選択します。

   VMにポリシーを割り当てる前に、HYCUがゲストOSへの認証に使用する資格情報を作成し保存します。これは、ファイルとアプリケーションに対応したバックアップを実行し、iSCSIディスクを検出できるようにします。
   Prismを介してVMに接続されたVolume Groupは、Nutanix APIを介して自動的に検出され、認証情報を割り当てなくても保護されます。 ゲスト内のiSCSIイニシエーターを介してVMを接続する場合、検出プロセスは接続されたVolume Groupも検出します。

#. 上部ツールバーから、 **(鍵アイコン) Credentials > + New** をクリックします。

#. 以下のフィールドに値を入力します:

   - **Name** - Local Windows Admin
   - **Username** - Administrator
   - **Password** - *HYCUBackupTest VM作成時に入力したパスワード*

#. **Save** をクリックします。

#. *Initials*\ **-HYCUBackupTest** VMを選択し、 **(鍵アイコン) Credentials** をクリックします。 **Local Windows Admin** 資格情報を選択し、 **Assign** をクリックすることでVMに割り当てます。

   .. note::

     HYCUは定期的に自動同期を行います。仮想マシンのリストに *Initials*\ **-HYCUBackupTest** が表示されない場合は、 **Synchronize** をクリックして、更新されたリストをPrismから取得します。

   HYCUは、資格情報がVMへの認証に使用できることを検証します。しばらくすると、 **Discovery** 列に、検出が成功したことを示す緑色のチェックが表示されます。

   .. figure:: images/16.png

   .. note::

     HYCUは、VMまたは共有フォルダにOwner（所有者）を割り当てることもできます。 この割り当てにより、セルフサービスポリシーの適用が可能になり、Active Directoryユーザーまたはグループが任意のリソースにアクセスできるようになります。 セルフサービスで使用可能な役割には、Viewer（読み取り専用）、Administrator、Backup Operator、およびRestore Operatorが含まれます。

     .. figure:: images/19.png

#. *Initials*\ **-HYCUBackupTest** VMを選択し、 **(盾アイコン) Policies** をクリックします。

#. カスタムの **Platinum** ポリシーを選択し、 **Assign** をクリックします。

#. **HYCU** サイドバーから、:fa:`bars` **> Jobs** をクリックし、バックアップの進捗を確認します。

   HYCUがNutanix Change Block Tracking APIを利用してOSディスクだけでなく、iSCSIを介してマウントされたVolume Groupもバックアップされていることを、バックアップジョブの詳細から確認できます。さらに、Volume GroupをAHVのVMに（ゲスト内のiSCSIイニシエーターを使用せずに）直接接続する場合、HYCUはゲスト内の検出資格情報を必要とせずにVolume Groupをバックアップおよび復元できます。

   .. figure:: images/17.png

#. 最初のフルバックアップが完了したら、サイドバーから **Dashboard** を選択し、すべてのポリシーが準拠していること、VMが100%保護されていることを確認します。

#. **Virtual Machines** に戻り、 *Initials*\ **-HYCUBackupTest** VMを選択します。 **Backup** をクリックし、手動で増分バックアップを実行します。

   .. figure:: images/18.png

レプリカからのバックアップ
..................

マルチクラスターNutanix環境では、ディザスタリカバリの目的で、Nutanix保護ドメイン（PD）のレプリケーションを構成することがよくあります。 HYCUは、VMが実行されているクラスターから直接バックアップを実行する代わりに、レプリカから本番VMをバックアップできるように、Nutanix保護ドメインを認識できます。 この結果、次の価値が得られます：
 - データを2回コピーしない為、帯域幅要件を半分に削減
 - リモートのクラスターにエージェントを配置して維持する必要がない
 - 元のクラスターまたは指定した他のクラスターへの復元が可能

これは、いくつかのシナリオでとても有益です:
 -  ROBO (リモートオフィス/ブランチオフィス)の保護
 -  複数の本番サイトから中央のデータセンターにレプリケートする環境
 -  セカンダリコピーの取得を回避するため、レプリカからバックアップするアクティブ/アクティブの2拠点環境
 -  HYCUがDRサイトで実行され、本番サイトに触れることなく本番VMを保護できる本番サイトとDRサイトの環境

 .. figure:: images/13b.png

バックアップからの復元
+++++++++++++++++

#. **HYCU** サイドバーから、:fa:`bars` **> Virtual Machines >** に進み、 *Initials*\ **-HYCUBackupTest** をクリックします。

#. 下の **Details** テーブルから、 **Compliancy** 列と **Backup Status** 列のアイコンにカーソルを合わせると、サイズ、バックアップを実行する時間、バックアップのタイプなど、各リストアポイントに関する追加情報が表示されます。

   .. figure:: images/21.png

#. 最新の増分リストアポイントを選択し、 **Restore VM** をクリックします。

   HYCUは、VM全体を上書きまたはクローンする機能と、個々のVMディスクまたはVolume Groupを個別に復元またはクローンする機能を提供します。Volume Groupの復元は、ディスクを既存のVMにマウントしたい場合に役立ちます。

   さらに、任意のリストアポイントのローカルディスクとVolume Groupの両方をNFSまたはSMB共有にエクスポートできます。

#. **Clone VM** を選択し、 **Next** をクリックします。

   .. figure:: images/20.png

   .. note:: HYCUはVMのクローンを作成しますが、VMにはVolume Groupが接続されているため、警告が表示されます。 この警告は安全に無視できます。

#. 以下のフィールドに値を入力し、 **Restore** をクリックします:

   - **Select a Storage Container** - Original location
   - **New VM Name** - *Initials*\ -HYCUBackupTest-Clone
   - **Power Virtual Machine On** - Disabled
   - **Restore Instance** - Automatic

   .. note::

     複数のNutanixクラスターで構成されている場合、VMの復元先として別クラスターを指定できます。

     インスタンスの復元でAutomaticを選択すると、デフォルトで最速のオプションが選択されます。 このポリシーでは、 **NutanixVG** Volume Groupに保存されたバックアップとは対照的に、ローカルのNutanixスナップショットになります。 手動でインスタンスを選択すると、バックアップまたはアーカイブターゲットからRTOをテストするのに役立ちます。

#. **Prism > VM > Table** から、元の *Initials*\ **-HYCUBackupTest** VMをPower offし、 **その後** *Initials*\ **-HYCUBackupTest-Clone** をPower onします。

   .. note::

     元の仮想マシンと復元された仮想マシンは、同じネットワークおよびiSCSI設定を持つため、潜在的な問題を回避するために、両方の仮想マシンが同時に起動しないことを確認してください。

#. VMコンソールを起動し、すべてのファイルとディスクがVM内で期待どおりに表示されることを確認します。 Nutanix Volumeのクローンが作成されたことも確認できます。

   *おめでとうございます！ HYCUを使用して、最初のVMとVolume Groupを復元しました。*

#. **Prism > VM > Table** から、 *Initials*\ **-HYCUBackupTest-Clone** VMとクローンの *Initials*\ **-BackupTestVG-**\ *Timestamp* Volume Groupを削除します。

   .. note::

      アタッチされていることでVolume Groupの削除が失敗する場合、Volume Groupを **Update** し、Client下の *Initials*\ **-HYCUBackupTest-Clone** VM IQNの選択を解除します。 **Save** クリックして、再度Volume Groupを削除します。

#. 元の *Initials*\ **-HYCUBackupTest** VMをPower onします。

#. **HYCU** サイドバーから、:fa:`bars` **> Jobs** をクリックします。VMの復元には時間が掛かることがあります。

   バックアップポリシーは、Nutanixクラスターのローカルスナップショットを保持するように設定されているため、復元作業ははほぼ瞬時に行われます。

VMファイルの復元
..................

VMまたはディスク全体を復元するだけでなく、HYCUを使用して、バックアップされたVMまたはVolume Groupからファイルを直接復元することもできます。 多くの場合、VMを復元する必要性は、不注意で削除または破損したファイルを取得することのみを目的としています。ファイルを直接復元する機能は、同じ最終結果を達成するために必要な時間とリソースを削減します。

#. **HYCU** サイドバーから、:fa:`bars` **> Virtual Machines >** を選択し、 *Initials*\ **-HYCUBackupTest** をクリックします。

#. 最新の増分スナップショットを選択し、 **Restore Files** をクリックします。

   これにより、バックアップがマウントされ、ユーザーがローカルファイルシステムを参照できるようになります。

#. 以前にボリュームグループ（E :)に作成した1つ以上のファイルを選択し、 **Next** をクリックします。

   .. figure:: images/22.png

#. **Restore to Virtual Machine** を選択し、 **Next** をクリックします。

#. 以下のフィールドに値を入力し、 **Restore** をクリックします:

   - **Path** - Original location
   - **Mode** - Rename restored
   - **Restore ACL** (デフォルトのまま)

#. *Initials*\ **-HYCUBackupTest** コンソールを開き、ファイルが復元されていることを確認します。

   .. figure:: images/23.png

   HYCUは、非常にシンプルでPrismのようなワークフローを維持しながら、Nutanix VM、VG、およびファイルデータを復元する柔軟性を提供します。 HYCUはネイティブNutanixストレージAPIを利用して、高速で効率的なバックアップおよび復元を可能にします。


.. _hycu-files:

(オプション) Nutanix Files 統合
++++++++++++++++++++++++++++++++++++

HYCUは、ネイティブのNutanix Change File Tracking（CFT）APIを使用してNutanix Filesに完全に統合されたバックアップおよび復元機能を提供する最初のソリューションです。 さらに、HYCUはNutanix FilesのSMB共有とNFS共有の両方をバックアップできます。

従来のバックアップソリューションは、ネットワークデータ・マネジメント・プロトコル（NDMP）を使用してファイルサーバーに大きな負荷をかけており、変更されたファイルを識別するためにファイルツリー全体を読み取る必要がありますが、HYCUはNutanixストレージレイヤースナップショットとCFTを使用して、変更されたファイル情報を即座に取得します。 つまり、HYCUバックアップは、ファイルサーバーへの影響を排除し、従来の夜間バックアップと比較して、変更ファイルを1時間ごとにバックアップすることにより、データ損失リスクを大幅に軽減します。

この演習では、Nutanix Filesをバックアップソースとして構成し、Nutanix Files SMB共有をターゲットにします。

SMB共有をターゲットとして追加
.......................

.. note:: この演習では、Nutanix Files SMB共有を使用しますが、HYCUはNFS共有もサポートしています。

この演習では、1つのファイル共有ソースをファイル共有ターゲットにバックアップします。 最初に、バックアップデータのターゲットとして使用するファイルクラスター上の共有を定義します。

Filesのバックアップターゲットには、NFSエクスポート、SMB共有、またはS3（クラウド）ターゲットが必要です。つまり、Nutanix Bucketsも使用できます。 バックアップ対象のファイルをブロックストレージに直接書き込むことができないため、iSCSIターゲットは現在サポートされていません。

#. **Prism > File Server** から、 **+ Share/Export** をクリックします。

#. 以下のフィールドに値を入力し、 **Next > Next > Create** をクリックします:

   - **Name** - *Initials*\ -HYCUTarget
   - **File Server** - *Initials*\ -Files
   - **Select Protocol** - SMB

#. **HYCU** サイドバーから、 :fa:`bars` **> Targets** をクリックします。

#. **+ New** をクリックし、以下のフィールドに値を入力し、 **Save** をクリックします:

   - **Name** - Files-HYCUTarget
   - **Concurrent Backups** - 1
   - **Description** - *Nutanix Files Cluster Name* HYCUTarget Share
   - **Type** - SMB
   - **Domain** - NTNXLAB
   - **Username** - Administrator
   - **Password** - nutanix/4u
   - **SMB Server Name** - BootcampFS.ntnxlab.local
   - **Shared Folder** - /\ *Initials*\ -HYCUTarget

   .. figure:: images/24.png

APIアクセスの設定
......................

HYCUがNutanix Files REST APIsおよびCFTにアクセスするためには資格情報が必要です。

#. **Prism > File Server** から、 *Initials*\ **-Files** サーバーを選択し、 **Manage roles** をクリックします。

   .. figure:: images/25.png

#. **REST API Access Users** の下で、 **+ New user** をクリックします。

#. 以下のフィールドに値を入力し、 **Save > Close** をクリックします:

   - **Username** - *Initials*\ -hycu
   - **Password** - nutanix/4u

   .. figure:: images/26.png

Nutanix Filesをソースとして追加
...........................

Filesの保護は、ハイパーバイザーをHYCUに追加することと似ていますが、Filesをソースとして追加すると、Filesを実行しているNutanixクラスター上にHYCU Instanceが展開される点が異なります。 この追加インスタンスの目的は、HYCU Backup Controllerからファイルコピー操作をオフロードすることです。

DHCPが有効になっているAHVクラスターの場合、Filesソースを追加すると、追加のHYCU Instanceを自動的に展開できます。 ESXiまたはDHCPが無効の環境では、HYCU Instanceを手動で展開する必要があります。（HYCU Backup Controllerの展開と同様）。手動による展開の詳細については `HYCU User Guide <https://support.hycu.com/hc/en-us/sections/115001018365-Product-documentation>`_ を参照してください。

#. **HYCU** ツールバーから、:fa:`cog` **> Sources** をクリックします。

#. 上部メニューから **Nutanix Files** をクリックします。

   .. figure:: images/26a.png

#. **+ New** をクリックし、以下のフィールドに値を入力します:

   - **URL** - https://bootcampfs.ntnxlab.local:9440
   - **Nutanix Files Server Credentials > Username** - *Initials*\ -hycu
   - **Nutanix Files Server Credentials > Password** - nutanix/4u
   - **Backup Credentials > Username** - NTNXLAB\\Administrator
   - **Backup Credentials > Password** - nutanix/4u

   **Nutanix Files Server Credentials** は先の演習で構成したREST API資格情報になります。HYCUはAPIを使用して、前回のバックアップ以降に更新されたファイルを把握します。 **Backup Credentials** は、HYCUが共有フォルダにアクセスしてファイルコピーをするためのものです。このユーザーには、HYCUによってバックアップされるすべての共有フォルダへの読み取りアクセス権が必要です。

   .. figure:: images/27.png

   .. note::

     ファイルのコピーのために共有フォルダにアクセスする必要があるため、HYCUが **Secondary** ネットワークに展開されています。:ref:`files` の演習では、 **Primary** ネットワークがストレージネットワークとして選択されました。つまり、 **Primary** ネットワーク上の他のVMは共有にアクセスできません。

#. **Save** をクリックしてFilesソースを追加し、HYCU instanceの展開を開始します。

   Prismから *Initials*\ **-HYCU-1** VMの作成を確認し、HYCU **Jobs** ページで全体的なステータスを監視できます。 このプロセスは完了するまでに約3分かかります。

   .. figure:: images/28.png

Filesのバックアップと復元
............................

Filesのバックアップと復元は、VM / VGワークフローと非常に似た動作をします。同じカスタマイズ可能なポリシーやOwner（所有者）/セルフサービス構造を使用します。

#. 作成したSMBターゲット *Initials*\-HYCUTargetをカスタム **Platinum** ポリシーに追加します。

#. **HYCU** サイドバーから、:fa:`bars` **> Shares** をクリックします。

#. **Marketing** 共有を選択し、 **(盾アイコン) Policies** をクリックします。

   .. note::

     Prismに戻り、"Marketing"という名前のSMB共有を作成する必要がある場合があります。Filesで他の共有フォルダを作成した場合は、それらのいずれかを選択することもできます。

#. カスタム **Platinum** ポリシーを選択し、 **Assign** をクリックします。

#. **Jobs** に戻り、初期バックアップが正常に完了したことを確認します。

#. Windows Tools VMまたは *Initials*\ **-HYCUBackupTest** VMを使用し、Marketing共有フォルダ（例：``\\<Initials>-Files\Marketing``）にアクセスし、以下を実行します:

   - ファイルの更新
   - 新しいファイルの追加
   - 既存ファイルの削除

#. **HYCU** サイドバーから、:fa:`bars` **> Shares** をクリックします。

#. **Marketing** フォルダを選択し、 **Backup** から増分バックアップを実行します。

   追加ファイルのサイズ次第ですが、増分バックアップは1分以内に完了するはずです。

#. **Restore Points** 下で最新のリストアポイントを選択し、 **Backup Status** にカーソルを合わせると、前回のバックアップ以降に変更されたファイルの数と、バックアップの増分サイズの両方を確認できます。

   .. figure:: images/29.png

   これらの値は、Marketing共有フォルダに追加/変更されたファイルを正確に反映していますか？

   上の画面のターゲットは **Files-HYCUTarget** であることに注意してください。これは、バックアップポリシーを編集せずにどのように決定されましたか？

#. 元のフルバックアップのリストアポイントを選択し、 **Browse & Restore Files** をクリックします。

   .. figure:: images/30.png

#. 以前にMarketing共有フォルダから削除したファイルを選択し、 **Next** をクリックします。

#. 元の場所をターゲットにして、 **Restore** をクリックします。

#. クライアントVMのコンソールに戻り、Marketing共有フォルダを更新して、以前に削除したファイルを表示します。

   .. figure:: images/31.png

   数回のクリックで、管理者またはエンドユーザーは、HYCUおよびCFT APIを使用して、個々のファイル、フォルダ、またはNutanix Files共有全体を簡単に復元できます。

重要なポイント
+++++++++++++

**HYCU** について知っておくべき重要なことは何ですか？

- HYCUは、AHVおよびESXiのVM、VG、およびアプリケーションに対する完全なバックアップ機能を提供します。

- HYCUは、バックアップと復元の両方にNutanixスナップショットを活用する最初の製品であり、VMスタンをなくし、ローカルのNutanixスナップショットから迅速に復元できるようにします。

- HYCUは、Nutanixノードをバックアップストレージターゲットとして使用することもでき、Nutanixビジネスの規模拡大に貢献します。

- Prismと同様に、HYCUは使いやすいHTML5管理コンソールを提供します。

- HYCUは、VMレプリカからバックアップすることによりネットワーク帯域幅を最大50％削減する、ROBO環境向けの唯一のソリューションです。

- HYCUはNutanix Filesに初めてスケールアウトのバックアップと復元を提供した製品で、リソース要件とバックアップ時間を最大90％削減します。
