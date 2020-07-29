.. title:: Hycu and Nutanix

.. toctree::
   :maxdepth: 3
   :caption: HYCU
   :name: _hycu
   :hidden:

   hycu/hycu

.. toctree::
   :maxdepth: 3
   :caption: HYCU Add-On Labs
   :name: _addon
   :hidden:

   protecting-applications/protectingapps
   protecting-physical/protectingphysical
   reporting/reporting
   self-service/selfservice

.. toctree::
   :maxdepth: 2
   :caption: Appendix
   :name: _appendix
   :hidden:

   tools_vms/windows_tools_vm
   tools_vms/linux_tools_vm
   taskman/taskman
   wordpress/wordpress


.. _getting_started:

---------------
はじめに
---------------

HycuおよびNutanixアドオンラボへようこそ！このワークブックには、講師主導のNutanixとHycuソフトウェアを組み合わた連携ソリューションのセッションが含まれています。

この演習ではHycuとNutanixを使って学びます。

新機能
++++++++++
- ワークショップはこちらのバージョンにアップグレードされています:
    - AOS & PC 5.11.2.x

- オプションラボの更新:


アジェンダ
++++++

- Hycu

はじめに
+++++++++++++

- 名前
- Nutanixの知識

初期設定
+++++++++++++

- 使用されている *Passwords* をメモします。
- 仮想デスクトップにログインします。（接続情報はこちら）

環境の詳細情報
+++++++++++++++++++

Nutanix Workshopsは、Nutanix Hosted POC環境で実行することを目的としています。演習を完了するために必要なすべてのイメージ、ネットワーク、VMがクラスターにプロビジョニングされます。

ネットワーク
..........

ホストされたPOCクラスターは、標準の命名規則に従います：

- **Cluster Name** - POC\ *XYZ*
- **Subnet** - 10.**21**.\ *XYZ*\ .0
- **Cluster IP** - 10.**21**.\ *XYZ*\ .37

マーケティングプールからプロビジョニングされた場合：

- **Cluster Name** - MKT\ *XYZ*
- **Subnet** - 10.**20**.\ *XYZ*\ .0
- **Cluster IP** - 10.**20**.\ *XYZ*\ .37

例：

- **Cluster Name** - POC055
- **Subnet** - 10.21.55.0
- **Cluster IP** - 10.21.55.37

ワークショップ全体を通して、 *XYZ* をサブネットの正しいオクテットに置き換える必要があるインスタンスが複数あります。次に例を示します:

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - IP Address
     - Description
   * - 10.21.\ *XYZ*\ .37
     - Nutanix Cluster Virtual IP
   * - 10.21.\ *XYZ*\ .39
     - **PC** VM IP, Prism Central
   * - 10.21.\ *XYZ*\ .40
     - **DC** VM IP, NTNXLAB.local Domain Controller

各クラスターは、VMで使用できる2つのVLANで構成されています:

.. list-table::
  :widths: 25 25 10 40
  :header-rows: 1

  * - Network Name
    - Address
    - VLAN
    - DHCP Scope
  * - Primary
    - 10.21.\ *XYZ*\ .1/25
    - 0
    - 10.21.\ *XYZ*\ .50-10.21.\ *XYZ*\ .124
  * - Secondary
    - 10.21.\ *XYZ*\ .129/25
    - *XYZ1*
    - 10.21.\ *XYZ*\ .132-10.21.\ *XYZ*\ .253

資格情報
...........

.. note::

   *<Cluster Password>* はクラスタに固有であり、ワークショップのリーダーによって提供されます。

.. list-table::
   :widths: 25 35 40
   :header-rows: 1

   * - Credential
     - Username
     - Password
   * - Prism Element
     - admin
     - *<Cluster Password>*
   * - Prism Central
     - admin
     - *<Cluster Password>*
   * - Controller VM
     - nutanix
     - *<Cluster Password>*
   * - Prism Central VM
     - nutanix
     - *<Cluster Password>*

各クラスターには専用のドメインコントローラーVM、 **DC**があり、 **NTNXLAB.local**ドメインにADサービスを提供します。ドメインには次のユーザーとグループが作成されています:

.. list-table::
   :widths: 25 35 40
   :header-rows: 1

   * - Group
     - Username(s)
     - Password
   * - Administrators
     - Administrator
     - nutanix/4u
   * - SSP Admins
     - adminuser01-adminuser25
     - nutanix/4u
   * - SSP Developers
     - devuser01-devuser25
     - nutanix/4u
   * - SSP Consumers
     - consumer01-consumer25
     - nutanix/4u
   * - SSP Operators
     - operator01-operator25
     - nutanix/4u
   * - SSP Custom
     - custom01-custom25
     - nutanix/4u
   * - Bootcamp Users
     - user01-user25
     - nutanix/4u

アクセス手順
+++++++++++++++++++

NutanixがホストするPOC環境にはさまざまな方法でアクセスできます：

ラボアクセスのユーザー資格情報
...........................

PHXベースのクラスター:
**Username:** PHX-POCxxx-User01 (up to PHX-POCxxx-User20), **Password:** *<Provided by Instructor>*

RTPベースのクラスター:
**Username:** RTP-POCxxx-User01 (up to RTP-POCxxx-User20), **Password:** *<Provided by Instructor>*

VDIフレーム
.........

ログイン先: https://frame.nutanix.com/x/labs

**Nutanix社員** -  **NUTANIXDC** 資格情報を使用
**社員以外** -  **Lab Access User** 資格情報を使用

Parallels VDI
.................

PHXベースのクラスター ログイン先: https://xld-uswest1.nutanix.com

RTPベースのクラスター ログイン先: https://xld-useast1.nutanix.com

**Nutanix社員** -  **NUTANIXDC** 資格情報を使用
**社員以外** -  **Lab Access User** 資格情報を使用

社員用 Pulse Secure VPN
..........................

クライアントのダウンロード:

PHXベースのクラスター ログイン先: https://xld-uswest1.nutanix.com

RTPベースのクラスター ログイン先: https://xld-useast1.nutanix.com

**Nutanix社員** -  **NUTANIXDC** 資格情報を使用
**社員以外** -  **Lab Access User** 資格情報を使用

クライアントのインストール

Pulse Secure Clientで、接続を **追加** :

PHXの場合:

- **Type** - Policy Secure (UAC) または Connection Server
- **Name** - X-Labs - PHX
- **Server URL** - xlv-uswest1.nutanix.com

RTPの場合:

- **Type** - Policy Secure (UAC) または Connection Server
- **Name** - X-Labs - RTP
- **Server URL** - xlv-useast1.nutanix.com


Nutanix バージョン情報
++++++++++++++++++++

- **AHV Version** - AHV 20170830.337
- **AOS Version** - 5.11.2.3
- **PC Version** - 5.11.2.1
