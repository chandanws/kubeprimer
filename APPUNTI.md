# Appunti 14 luglio 2017

### Intro

Storia kubernetes

* borg
* omega
* kubernetes

Docker spacca kubernetes  (Kubernetes usa docker 1.12)

Non c'è un modo sano di mente per tirare su un cluster kubernetes,

Modi per tirarlo su

* kargo (yesss)
* kops (su cloud, aws, gce)
* kubeadm

Sig group per tutti i vari componenti di kubernetes, se si ha qualche problema nei cluster si va a chiedere a questi gruppi di esperti

### Docker

dotcloud -> docker 1.0 -> docker inc

Docker registry

Il registry di google non supporta il multi architettura

Docker run -> pull da registry

Docker daemon prende un immagine, la scompatta, con gli overlay fs per minimizzare l'occupazione dello storage, una volta creata questa porzione di directory isolata viene eseguito l'entrypoint.

Ogni docker container **deve** avere solo un processo 

Docker è passato da LXC a OCI per l'interfacciamento del container

> Guardare runv


### Docker Compose

Gestione multi container , gestendo link, raivvi ecc.

> Guardare docker dub per il deploy in swarm

### Docker swarm

* Comunicazione cifrata
* rotazione certificati
* altro


### Kompose

kompose.io, file automatico per generare da un docker-compose.yml la struttura dei vari yaml

Vedi readme app


# Kubernetes

Fa parte della CNCF nel 2015, che è uno spinoff della cloud linux foundation, google ha donato la sua tecnologia alla CNCF.

### Kubelet

E' il demone principale , gira su tutti i nodi, e si assicura che i container vengano correttamente runnati dove sono stati schedulati.

E' la kubelet che esegue il `docker run` in autonomia, si assicura che i container runnati siano effettivamente healthy, ri ritira su, e applica i limiti.

Kubernetes cerca di riportare i servizi allo stato delle cose.

Il kubelet gira ovunque, master e workers


### Pods

Il pod è una collezione di container, sono una singola unità di esecuzione, che condividono networking e storage, significa che sono all'interno dello stesso networkin namespace. I container all'inerno dello stesso pod possono parlarsi in localhost.

Ogni pod viene schedulato e scalato nello stesso modo e nelle stesse logiche. I pod sono co-located, e devono essere sempre nello stesso nodo.

I pod condividono il loro virtual IP e il loro spazio porte tramite una rete virtuale assegnati dal cluster.

### Repliation Controllers / ReplicaSets

I replication controller si occupano di assicurarsi che il numero dei pod running siano quelli definiti in ogni momento. Il replication controller parla con la Kubelet.

Replication Controller = a ReplicaSets , i replication controllers stanno andando in deprecato.

I replicaset si attaccano ai pod tramite label.

### Deployments

Il deployments è un astrazione del processo di deployment, il deployment mi da la parte di logica del gestione del deployment. 

Nel deployments c'è il concetto di node affinity, se voglio schedulare i pod in posti particolari.

Rolling update e rollbacks


### Kubectl 

E' solo un tool che si occupa di parlare direttamente le api di kubernetes. 


##### Taggare i nodi

Per l'affinity è bene taggare sempre i nodi

##### Comandi utili

* `kubectl get pods`
* `kubectl get deployments`
* `kubectl get rs` (per prendere i replicaset)
* `kubectl get jobs`
* `kubectl get qualsiasicosa`

Fare lo scaling sempre sul deployment


Scordiamoci l'auto scaling, si fa con i deployment e stop, in pratica, si fa solo se serve.



### ConfigMaps e Secrets

Sono delle primitive kubernetes che raccolgono configurazioni e segreti 

Bisogna separare il comportamento delle applicazioni dai valori di configurazione

I configmaps sono salvati in chiaro nel cluster (in etcd) i secrets invece sono salvati in base64

Per i secret, usare Vault di hashicorp, che i secret di kubernetes sono proprio osceni

kaseyhightower per vault e secret


### Service

Cos'è? E' un astrazione di un set di pod, il lifecycle del service è scollegato dal lifecicle dei pod. Il ervice ha un virtual ip, lo seleziono tramite una label e farà il routing del traffico ai pod giusti.

Si può puntare direttamente al nome del service, invece di puntare all'ip del service

##### Nota su esporre i pods

I pod non sono raggiungibili dal mondo esterno di default. 

E' equivalente al compose di docker. 

### Networking

Ogni nodo ha un kube-proxy che è un daemon . Il suo lavoro è quello di implementare il routing da virtual ip a ip reale dei nostri servizi. Crea le regole di iptables per gestire questo networking.


### Service discovery

Via variabili di ambiente, oppure tramite kube-dns

### DNS

Kube-dns è un componente del cluster , e gira dentro il master , ed è la componente meno documentata della storia.

### Namespaces

`kubectl get pods --namespace=kube-system`

Cosa sono i namespace? Kubernetes organizza tutto quello che sta facendo in namespace, pod che vivono in namespace diversi non si vedono. Serve per segmentare quelli che si sta facendo.

Multi tenant anche dal punto di vista dell'amministratore. Si può dare accesso a kubectl anche agli sviluppatore bloccando la visualizzazione dei namespace con delle acl

A livello di namespace si possono definire le policy di utilizzo delle risorse

### Tipi di servizi

##### Cluster ip

Stiamo assegnando un ip virtuale interno al cluster kubernetes accessibile solo dal cluster kubernetes

##### NodePort

Rende disponibile il servizio su una porta a "random" o una porta definita da me

Qui la porta viene aperta su TUTTI i nodi **worker** di kubernetes

##### Ingress

Ingress non è un servizio, ma un ingress. Serve per esporre verso l'esterno qualcosa

Il servizio di ingress sta solo sui/sul master, da capire come si fa a fare un load balancing davanti?

##### LoadBalancer

solo su aws, gke



### DaemonSet

Esempio: vogliamo avere un agent che gira in tutti i nodi, il monitoring, logging, altro

Non è un Pod normale.

I Pod normali non possono mai morire, invece un DaemonSet può essere un job batch, che fa quello che deve fare e poi muore. Come è giusto che sia. Insomma, muore e siamo felici.


### Job e CronJobs

E' un pod che deve venire eseguito correttamente e poi deve finire.

il tipo di Pod CronJobs non esiste ancora, ma lo stanno implementando


### Volume e peristent volume

da approfondire su ceph e glusterfs


# Architettura Kubernetes 

> lanciate una ciabatta a chi vuole spodestare i sysadmin

https://speakerdeck.com/thockin/architecture-of-kubernetes



* MASTER : non si schedula nulla sui master, si fa il taint per evitare lo scheduling
* MINION : i worker node

##### Kubelet

E' un componente che ha dentro tutta una serie di pezzi per poter gestire il cluster

La kubelet non parla con etcd

## Servizi CORE

### API 

Fa tante cose, tra cui il control loop , accetta i comandi di kubelet tramite api.

### Scheduler

### etcd

Contiene tutto lo stato del cluster, solo api e scheduler comunicano con etcd
 
### HA

Tutti i controllers sono single tone nel cluster

etcd è bene non clusterizzarlo? se si clusterizza un master e più follower, fa un collo di bottiglia mega. Etcd è meglio flashare le info su disco , gli si fa fare il recovery dal journal se si spacca, i nodi non si fermano se il master si ferma

TODO provare a spaccare il master e sostituirlo

etcd è gigantesco in memory 

Piccole info

1. Lo stato è dichiarativo
2. control loops
3. Simple vs complex (ha ce ne fottiamo?)
4. Il network è a livello di pod


> l'upgrade è un troiaio, è , un , troiaio.

# Monitoring e logging

* Prometheus : metrica e service discovery su kubernetes, serve per fare gli alert, integrazione con grafana, prometheus sa cos'è kubernetes
* fluentd : serve per la collection dei log, è nativo su kubernetes, e sputa su elastic search


prometheus demo, provare





