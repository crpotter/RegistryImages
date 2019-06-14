---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-04"

keywords: IBM Cloud Container Registry, ibmcloud-secure-perimeter-health, container image, health, Secure Perimeter, scan, public image

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

# Introduzione all'immagine `ibmcloud-secure-perimeter-health`
{: #ibmcloud-secure-perimeter-health}

L'immagine `ibmcloud-secure-perimeter-health` contiene uno strumento per eseguire la scansione delle vulnerabilità in un Secure Perimeter in {{site.data.keyword.cloud}}.
{:shortdesc}

Puoi accedere alle immagini fornite da {{site.data.keyword.IBM_notm}} utilizzando la riga di comando, consulta [Immagini pubbliche IBM](/docs/services/Registry?topic=registry-public_images#public_images).
{: tip}

## Come funziona
{: #sph_how-it-works}

Per garantire che il tuo Secure Perimeter funzioni correttamente, `ibmcloud-secure-perimeter-health` può eseguire la scansione di reti pubbliche o private nel tuo account dell'infrastruttura {{site.data.keyword.cloud_notm}} e segnalare le vulnerabilità. Puoi usare l'immagine **ibmcloud-secure-perimeter-health** in due modi:

- Usa `ibmcloud-secure-perimeter-health` come un pod su un cluster Kubernetes nel tuo Secure Perimeter per eseguire la scansione per rilevare eventuali esposizioni delle reti private.
- Usa `ibmcloud-secure-perimeter-health` come un contenitore Docker autonomo sulla tua workstation per eseguire la scansione per rilevare eventuali esposizioni delle reti pubbliche.

Per ulteriori informazioni sul Secure Perimeter, consulta questi articoli di blog:

- [Set up a Secure Perimeter in {{site.data.keyword.cloud_notm}} ![Icona link esterno](../../../icons/launch-glyph.svg "Icona link esterno")](https://developer.ibm.com/dwblog/2018/ibm-cloud-vyatta-set-up-secure-perimeter/).
- [Set up an automated Secure Perimeter in {{site.data.keyword.cloud_notm}} ![Icona link esterno](../../../icons/launch-glyph.svg "Icona link esterno")](https://developer.ibm.com/dwblog/2018/set-automated-secure-perimeter-ibm-cloud/).

Dopo la scansione, l'immagine `ibmcloud-secure-perimeter-health` produce un report relativo a quali reti erano raggiungibili dall'interno del Secure Perimeter Segment. Ogni report fornisce i dettagli del nome del gateway di rete, della VLAN, delle relative sottoreti e degli eventuali host che stanno causando la violazione. Il seguente codice è un report di esempio di un utente che ha eseguito la scansione per rilevare le vulnerabilità della rete privata:

```
#-------- Running Secure Perimeter exposure scan 2018-05-24 12:00:00 --------#

RESULTS:

sp-gateway-af6053a9:
	sps-priv-3deb5748:
		10.73.71.168/29:   PASS
	prv-neb1-cc4ee985:
		10.73.84.192/29:   FAIL
			host = 10.73.84.198, ports = [179, 22]
		10.73.63.8/29:     PASS
		10.73.72.128/26:   PASS
sp-gateway-8a9031ab:
	sps-priv-3deb5748:
		10.73.71.168/29:   PASS
	prv-neb1-cc4ee985:
		10.73.84.192/29:   PASS
		10.73.63.8/29:     PASS
```
{: screen}

## Elementi inclusi
{: #sph_whats_included}

L'immagine `ibmcloud-secure-perimeter-health` fornisce i seguenti pacchetti software.
{:shortdesc}

- Alpine Linux
- Runtime Python
- SoftLayer Python Client
- Scanner di porte Nmap

## Esegui il provisioning di un cluster Kubernetes in un Secure Perimeter utilizzando {{site.data.keyword.containerlong_notm}}
{: #sph_provision_cluster}

1. Esegui il provisioning del tuo cluster Kubernetes dalla sezione **Contenitori** nel catalogo {{site.data.keyword.cloud_notm}}.
2. Fai clic su **Crea**.
3. Seleziona le VLAN pubbliche e private del Secure Perimeter Segment dall'elenco di VLAN.
4. Immetti tutti gli altri dettagli necessari.
5. Fai clic su **Crea cluster**.

Per ulteriori informazioni su come ottenere l'accesso al tuo cluster dopo che è stato distribuito, vedi [{{site.data.keyword.containerlong_notm}}](/docs/containers?topic=containers-getting-started#getting-started).

## Esegui la scansione di reti private in un Secure Perimeter
{: #sph_private_networks}

Crea un pod del contenitore dall'immagine `ibmcloud-secure-perimeter-health` e configura una scansione di routine.

**Prima di iniziare**

- Installa le [CLI](/docs/containers?topic=containers-cs_cli_install#cs_cli_install) obbligatorie.
- [Indirizza la tua CLI](/docs/containers?topic=containers-cs_cli_install#cs_cli_configure) al tuo cluster.

1. Crea un file di configurazione denominato `health-pod.yaml`. Questo file crea una distribuzione altamente disponibile del pod del contenitore.

    ```
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: health-pod
      labels:
        app: health-pod
    spec:
      replicas: 1
      selector:
        app: health-pod
      template:
        spec:
          containers:
          - name: health-pod
            image: icr.io/ibm/ibmcloud-secure-perimeter-health:1.0.0
            args:
            - /usr/local/bin/python
            - /run.py
            - --scan
            - private
            - --exclude-vlan-ids
            - <Private Secure Perimeter Segment VLAN ID>
            - --poll-interval
            - 1800
            env:
            - name: SL_USER
              value: <IBM Cloud infrastructure user name>
            - name: SL_APIKEY
              value: <IBM Cloud infrastructure api key>
    ```
    {: codeblock}

2. Crea la distribuzione.

    ```
    kubectl apply -f health-pod.yaml
    ```
    {: pre}

3. Conferma che il pod è in esecuzione.

    ```
    kubectl get pods
    ```
    {: pre}

    ```
    NAME                                    READY     STATUS    RESTARTS   AGE
    health-pod-<random-id>                  1/1       Running   0          1hr
    ```
    {: screen}

## Esegui la scansione di reti pubbliche esterne a un Secure Perimeter
{: #sph_public_networks}

Crea un contenitore Docker dall'immagine `ibmcloud-secure-perimeter-health` ed esegui la scansione delle reti pubbliche.

**Prima di iniziare**

- Installa Docker.

1. Crea un contenitore Docker dalla tua workstation eseguendo il seguente comando:

    ```
    docker run -it -e SL_USER='$SL_USER' -e SL_APIKEY='$SL_APIKEY' icr.io/ibm/ibmcloud-secure-perimeter-health:1.0.0 /usr/local/bin/python run.py --scan public --allowed-public-ports 80 443 9000-9999
    ```
    {: pre}

2. Dopo che il contenitore ha prodotto un report, rivedi la sezione [Analisi dei risultati della scansione](#sph_scan_results) per comprendere i risultati.

## Analisi dei risultati della scansione
{: #sph_scan_results}

`ibmcloud-secure-perimeter-health` produce un report formattato sullo stato di funzionamento di un Secure Perimeter:

```
#-------- Running Secure Perimeter exposure scan 2018-05-24 12:00:00 --------#

RESULTS:

sp-gateway-af6053a9:
	sps-priv-3deb5748:
		10.73.71.168/29:   PASS
	prv-neb1-cc4ee985:
		10.73.84.192/29:   FAIL
			host = 10.73.84.198, ports = [179, 22]
		10.73.63.8/29:     PASS
		10.73.72.128/26:   PASS
sp-gateway-8a9031ab:
	sps-priv-3deb5748:
		10.73.71.168/29:   PASS
	prv-neb1-cc4ee985:
		10.73.84.192/29:   PASS
		10.73.63.8/29:     PASS
```
{: screen}

Il report ha il seguente formato:

```
<gateway name>:
  <vlan name>:
    <subnet>:    PASS
    <subnet>:    FAIL
      host = <ip address>, ports = [<exposed port>, <exposed port>, ...]
```
{: screen}

`ibmcloud-secure-perimeter-health` determina una sottorete come `PASS` se nessun host nella sottorete era raggiungibile, altrimenti esegue la restituzione con `FAIL` ed elenca gli host che erano raggiungibili, insieme alle porte che erano accessibili.

## Guida di riferimento agli argomenti del contenitore
{: #sph_reference_container_arg}

|Chiave|Descrizione|Impostazione predefinita
|---|-------------|---|
|`scan`|Il tipo di scansione dell'esposizione ("public" o "private") |Nessuna (viene eseguita la scansione di entrambe)
|`exclude-vlan-ids`|L'elenco di VLAN in base agli ID per cui evitare la scansione|Nessuna
|`poll-interval`|Imposta il numero di secondi fino alla prossima scansione|0 (eseguire una sola volta)
|`allowed-public-ports`|L'elenco di IP utente da includere nella whitelist nella scansione.|80, 443, 9000-9999
{: caption="Tabella 1. `Argomenti del contenitore" caption-side="top"}

## Riferimenti alla variabile di ambiente
{: #sph_reference_env_var}

|Chiave|Descrizione|
|---|-------------|
|`SL_USER`|Il tuo nome utente dell'infrastruttura IBM Cloud|
|`SL_APIKEY`|La tua chiave API dell'infrastruttura IBM Cloud|
{: caption="Tabella 2. Variabili di ambiente" caption-side="top"}