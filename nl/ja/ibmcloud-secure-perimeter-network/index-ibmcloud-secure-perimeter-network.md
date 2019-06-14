---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-04"

keywords: IBM Cloud Container Registry, ibmcloud-secure-perimeter-network, container image, network, Secure Perimeter, public image

subcollection: RegistryImages

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:deprecated: .deprecated}
{:table: .aria-labeledby="caption"}

# `ibmcloud-secure-perimeter-network` イメージの概説
{: #ibmcloud-secure-perimeter-network}

`ibmcloud-secure-perimeter-network` イメージには、Secure Perimeter 内の Vyatta 仮想ルーター・アプライアンスの構成を自動化するためのツールが含まれています。
{:shortdesc}

{{site.data.keyword.IBM}} によって提供されるイメージには、コマンド・ラインを使用してアクセスできます。[IBM のパブリック・イメージ](/docs/services/Registry?topic=registry-public_images#public_images)を参照してください。
{: tip}

## 機能
{: #spn_how-it-works}

`ibmcloud-secure-perimeter-network` を使用すると、Secure Perimeter の Vyatta 仮想ルーター・アプライアンスの構成を自動化できます。

Secure Perimeter について詳しくは、以下のブログ記事を参照してください。

- [Set up a Secure Perimeter in IBM Cloud ![外部リンク・アイコン](../../../icons/launch-glyph.svg "外部リンク・アイコン")](https://developer.ibm.com/dwblog/2018/ibm-cloud-vyatta-set-up-secure-perimeter/)。
- [Set up an automated Secure Perimeter in IBM Cloud ![外部リンク・アイコン](../../../icons/launch-glyph.svg "外部リンク・アイコン")](https://developer.ibm.com/dwblog/2018/set-automated-secure-perimeter-ibm-cloud/)。

`ibmcloud-secure-perimeter-network` イメージは、以下の 2 つの方法で使用できます。

- `ibmcloud-secure-perimeter-network` を Docker コンテナーとして使用して、Secure Perimeter ファイアウォール・ルール構成を初期化する。
- `ibmcloud-secure-perimeter-network` を Kubernetes クラスター上のポッドとして使用して、Secure Perimeter Segment VLAN 上に作成された新規サブネットについて IBM Cloud インフラストラクチャー・アカウントをポーリングし、それらを Vyatta ファイアウォール構成に追加する。

## 含まれている内容
{: #spn_whats_included}

`ibmcloud-secure-perimeter-network` イメージに、以下のソフトウェア・パッケージが用意されています。
{:shortdesc}

- Alpine Linux
- Python ランタイム
- SoftLayer Python クライアント
- Ansible

## 前提条件
{: #spn_prerequisites}

- IBM Cloud インフラストラクチャー・ポータルから Vyatta および VLAN を注文済みで、VLAN が Vyatta に関連付けられていること。
- 自動化された Secure Perimeter デプロイメントによって、`ibmcloud-secure-perimeter-network` がゲートウェイにアクセスするために使用する SSH 鍵を使用して Vyatta がプリロードされること。 SSH 鍵は手動でロードされるか、Secure Perimeter インストール・プロセスを介してロードされる必要があります。 詳しくは、[Set up an automated Secure Perimeter in IBM Cloud ![外部リンク・アイコン](../../../icons/launch-glyph.svg "外部リンク・アイコン")](https://developer.ibm.com/dwblog/2018/set-automated-secure-perimeter-ibm-cloud/)を参照してください。

## {{site.data.keyword.containerlong_notm}} を使用して Secure Perimeter 内で Kubernetes クラスターをプロビジョンする
{: #spn_provision_cluster}

1. IBM Cloud カタログ内の**「コンテナー」**セクションから Kubernetes クラスターをプロビジョンします。
2. **「作成」**をクリックします。
3. VLAN リストから Secure Perimeter Segment パブリックおよびプライベート VLAN を選択します。
4. 必要に応じて他のすべての詳細を入力します。
5. **「クラスターの作成」**をクリックします。

デプロイ後にクラスターにアクセスするには、[{{site.data.keyword.containerlong_notm}}](/docs/containers?topic=containers-getting-started#getting-started)を参照してください。

## Secure Perimeter Vyatta の初期構成を実行する
{: #spn_initial_setup}

1. `config.json` という名前のファイルを作成します。 このファイルには、Vyatta にアクセスするために `ibmcloud-secure-perimeter-network` が必要とする基本パラメーターが含まれます。

   ```
   {
     "slid": "XXXX",
    "apikey": "XXXX",
    "region": "XXXX",
    "inf_name_private": "dp0bond0",
    "inf_name_public": "dp0bond1",
    "gatewayid": "XXXX",
    "vlans": [
      {
         "type": "XXXX",
        "vlan_num": XXXX,
        "vlanid": XXXX
      },
       ...
     ],
    "vyatta_gateway_vip": "X.X.X.X",
    "vyatta_primary": {
       "private_ip": "X.X.X.X",
      "public_ip": "X.X.X.X"
    },
    "vyatta_secondary": {
       "private_ip": "X.X.X.X",
      "public_ip": "X.X.X.X"
    }
   }
   ```
   {: codeblock}

   `config.json` にデータを設定する方法の詳細については、[`config.json` リファレンス表](#spn_reference_config_json)を参照してください。このファイルは、[Kubernetes ポッドとしての `ibmcloud-secure-perimeter-network` のセットアップ](#spn_setup)のプロセスでも使用できます。

2. `ibmcloud-secure-perimeter-network` を Docker コンテナーとして実行して、初期セットアップを開始します。

   ```
   docker run icr.io/ibm/ibmcloud-secure-perimeter-network:1.0.0 python config-secure-perimeter.py -v /path/to/current/dir:/opt/secure-perimeter
   ```
   {: pre}

   この操作により、作業ディレクトリー内に `state.json` ファイルが作成されます。 このファイルを [Kubernetes ポッドとしての `ibmcloud-secure-perimeter-network` のセットアップ](#spn_setup)で使用します。

## Secure Perimeter 内の Kubernetes ポッドをセットアップする
{: #spn_setup}

`ibmcloud-secure-perimeter-network` イメージが Secure Perimeter 上のサブネットを管理するには、Kubernetes ポッドを使用して長期継続プロセスとしてこのイメージを実行できます。Vyatta のポッドを構成するには、`ibmcloud-secure-perimeter-network` イメージのいくつかの構成ファイルとフォルダーをポッドにコピーして、Vyatta 用に構成する必要があります。

1. `pvc.yaml` という名前のファイルを作成します。 この構成ファイルは、ポッドにボリュームとしてマウントできる、永続ボリューム・クレーム (PVC) を作成します。

   ```
   apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: network-pvc
    annotations:
      volume.beta.kubernetes.io/storage-class: "ibmc-file-bronze"
  spec:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 20Gi
   ```
   {: codeblock}

2. PVC を作成します。

   ```
   kubectl apply -f restore-pvc.yaml
   ```
   {: pre}

3. `network-pod.yaml` という名前のファイルを作成します。 この構成ファイルは、`ibmcloud-secure-perimeter-network` イメージを Kubernetes クラスター内のポッドとしてデプロイし、永続ボリューム・クレームをボリュームとしてマウントします。

   ```
   apiVersion: v1
   kind: Pod
   metadata:
     name: network-pod
     labels:
       app: network-pod
   spec:
     template:
       spec:
         containers:
         - name: network-pod
           image: icr.io/ibm/ibmcloud-secure-perimeter-network:1.0.0
           volumeMounts:
           - name: network-vol
             mountPath: /opt/secure-perimeter
         volumes:
         - name: network-vol
           PersistentVolumeClaim:
             claimName: network-pvc
   ```
   {: codeblock}

4. `rules.conf` という名前のファイルを作成します。 この構成ファイルは、パブリック・インターネットからどの外部サブネットおよび外部ポートを Secure Perimeter へのホワイトリストに登録するかを `ibmcloud-secure-perimeter-network` に指示します。

   ```
   {
       "external_subnets": [
         "X.X.X.X/X",
        "X.X.X.X/X"
       ],
      "external_ports": [
         "XX",
        "XX"
       ],
      "userips": [
         "X.X.X.X",
        "X.X.X.X"
      ]
   }
   ```
   {: codeblock}

5. ファイルを `ibmcloud-secure-perimeter-network` ポッドにコピーします。

   ```
   kubectl cp keys network-pod:/opt/secure-perimeter/
  kubectl cp state.json network-pod:/opt/secure-perimeter/state.json
  kubectl cp config.json network-pod:/opt/secure-perimeter/config.json
  kubectl cp rules.conf network-pod:/opt/secure-perimeter/rules.conf
   ```
   {: pre}

   `keys` ディレクトリーに、`ibmcloud-secure-perimeter-network` が Vyatta にアクセスするために必要な SSH 鍵が含まれています。SSH 鍵について詳しくは、[前提条件セクション](#spn_prerequisites)を参照してください。

## `config.json` リファレンス
{: #spn_reference_config_json}

|キー|説明
|---|-------------|---|
|`slid`|IBM Cloud インフラストラクチャー・ユーザー名
|`apikey`|IBM Cloud インフラストラクチャー API 鍵
|`region`|Vyatta がデプロイされる IBM Cloud 地域
|`inf_name_private`|Vyatta プライベート・インターフェースの名前
|`inf_name_public`|Vyatta パブリック・インターフェースの名前
|`gatewayid`|Vyatta ゲートウェイ ID
|`vlans`|タイプ、VLAN 番号、および VLAN ID を含む、Secure Perimeter Segment VLAN のリスト
|`vyatta_gateway_vip`|ゲートウェイの VIP
|`vyatta_primary`|1 次 Vyatta メンバーのプライベートおよびパブリック IP を含んでいるオブジェクト
|`vyatta_secondary`|2 次 Vyatta メンバーのプライベートおよびパブリック IP を含んでいるオブジェクト
{: caption="表 1. <code>config.json</code>" caption-side="top"}

## `rules.conf` リファレンス
{: #spn_reference_rules_conf}

|キー|説明
|---|-------------|---|
|`external_subnets`|Secure Perimeter が使用できるパブリック・インターネット上のサブネットのリスト
|`external_ports`|Secure Perimeter が使用できるポートのリスト
|`userips`|Secure Perimeter へのホワイトリストに登録するユーザー IP のリスト
{: caption="表 2. <code>rules.conf</code>" caption-side="top"}