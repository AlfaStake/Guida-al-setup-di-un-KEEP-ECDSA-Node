# Guida-al-setup-di-un-KEEP-ECDSA-Node

# Keep ECDSA Node 

Questa guida descrive gli step richiesti per programmare un nodo Keep (ECDSA) sulla testnet di RopSten e sulla mainnet di Ethereum. Il sistema operativo della guida è Ubuntu. Se usi una distro diversa come Fedora, la maggior parte dei comandi sono gli stessi ma tieni a mente che qualcosa potrebbe variare. Inoltre, usa anche la guida di Novy4 per configurare il server e renderlo pronto a runnare il nodo. Fornirò in questa guida i dettagli e le variazioni rispetto alla sua. [https://medium.com/@novysf/run-a-keep-network-testnet-node-37096946af35](https://medium.com/@novysf/run-a-keep-network-testnet-node-37096946af35)

## Links per dettagli e altre informazioni

- Pagina Github del progetto: [https://github.com/keep-network/keep-ecdsa](https://github.com/keep-network/keep-ecdsa)
- [https://github.com/keep-network/keep-ecdsa/blob/master/docs/run-keep-ecdsa.adoc](https://github.com/keep-network/keep-ecdsa/blob/master/docs/run-keep-ecdsa.adoc)
- Keep Discord [https://discordapp.com/invite/wYezN7v](https://discordapp.com/invite/wYezN7v) 
- Keep Dashboard [https://dashboard.test.keep.network/](https://dashboard.test.keep.network/)

## Operazioni Preliminari

- Crea un account su Infura [https://infura.io/](https://infura.io/).  Infura si sostituirà alla preparazione in locale di un nodo Ethereum
- Procurati una VPS o un Linux server
- [Installa Docker](https://docs.docker.com/engine/install/ubuntu/) sul tuo server
- Crea un wallet Ethereum dal quale puoi scaricare un JSON (ad esempio su MyEtherWallet)
- Procurati qualche ETH sulla rete Ropsten attraverso il faucet qui: [https://faucet.ropsten.be/](https://faucet.ropsten.be/)
- Carica il tuo file keystore in `~/keep-ecdsa/keystore/` e rinomina il wallet in `keep_wallet.json`.  Puoi usare applicativi come SCP o crearlo direttamente con `touch keep_wallet.json` ed usare `nano` per editarlo e inserirlo nella cartella corretta

## Autorizza il contratto tBTC e stake ETH

### Mainnet

Vai su [https://dashboard.keep.network/applications/tbtc](https://dashboard.keep.network/applications/tbtc) e autorizza il contratto tBTC. Inoltre bonda in stake qualche ETH.

### Testnet

Vai su [https://dashboard.test.keep.network/applications/tbtc](https://dashboard.test.keep.network/applications/tbtc) e autorizza il contratto tBTC. Inoltre bonda qualche ETH precedentemente ricevuto via faucet su Ropsten.

## Server Setup

### Crea il seguente albero di cartelle

Usa il seguente comando per creare una struttura

```bash
mkdir -p $HOME/keep-ecdsa/{config,keystore,persistence}
```

```bash
$HOME
│
└───keep-ecdsa
   │
   │
   └───config
   │    └───config.toml
   │
   └───keystore
   │    └───keep_wallet.json
   │
   └───persistence
```

### Crea le seguenti variabili di ambiente. Se hai letto la guida di [Novy4](https://medium.com/@novysf/run-a-keep-network-testnet-node-37096946af35), linkata in questa guida, avrai già provveduto a impostarle.

Nota bene, gli script e le istruzioni seguenti ti richiedono di inserire al posto del codice preceduto con $, i valori personalizzati, ad esempio, del tuo wallet ETH o della password ecc. 

```bash
## Creiamo le variabili di ambiente
export SERVER_IP=$(curl ifconfig.co)
# Cambialo col tuo ID di Infura
export INFURA_PROJECT_ID="$INFURA-ID"
# Cambialo col tuo address dell' ETH Wallet
export ETH_WALLET="0x..."
# Inserisci la password del tuo wallet ETH
export KEEP_CLIENT_ETHEREUM_PASSWORD="$ENTERPASSWORDHERE"
```

Le precedenti variabili possono anche essere aggiunte al tuo file .bashrc con il seguente comando.  Questo assicura che esse mantengano la persistenza a ogni riavvio del server

```bash
cat <<EOF >>$HOME/.bashrc

## Creiamo le variabili di ambiente
export SERVER_IP=$(curl ifconfig.co)
# Cambialo col tuo ID di Infura.
export INFURA_PROJECT_ID="$INFURA-ID"
# Cambialo col tuo address dell' ETH Wallet
export ETH_WALLET="0x..."
# Inserisci la password del tuo wallet ETH
export KEEP_CLIENT_ETHEREUM_PASSWORD="$ENTERPASSWORDHERE"
EOF
```

Adesso avvia il seguente comando che configura il `config.toml` per il nodo Keep-ECDSA

### MAINNET CONFIG

```bash
cat <<CONFIG >>$HOME/keep-ecdsa/config/config.toml

# Dettagli di connessione alla blockchain di ETH
[ethereum]
  URL = "wss://mainnet.infura.io/ws/v3/$INFURA_PROJECT_ID"
  URLRPC = "https://mainnet.infura.io/v3/$INFURA_PROJECT_ID"


[ethereum.account]
  Address = "$ETH_WALLET"
  KeyFile = "/mnt/keep-ecdsa/keystore/keep_wallet.json"


# Questo address di seguito potrebbe essere soggetto a variazioni future;
# in tal caso, i cambiamenti dell'address di questo contratto verranno listati qui:
# https://github.com/keep-network/keep-ecdsa/blob/master/docs/run-keep-ecdsa.adoc
[ethereum.ContractAddresses]
  BondedECDSAKeepFactory = "0x18758f16988E61Cd4B61E6B930694BD9fB07C22F"


# Questo address di seguito potrebbe essere soggetto a variazioni future;
# in tal caso, i cambiamenti dell'address di questo contratto verranno listati qui:
# https://github.com/keep-network/keep-ecdsa/blob/master/docs/run-keep-ecdsa.adoc
# Addresses of applications approved by the operator.
[SanctionedApplications]
  Addresses = [
    "0x41A1b40c1280883eA14C6a71e23bb66b83B3fB59",
]

[Storage]
  DataDir = "/mnt/keep-ecdsa/persistence"
  
[LibP2P]
  Peers = ["/dns4/bst-a01.ecdsa.keep.boar.network/tcp/4001/ipfs/16Uiu2HAkzYFHsqbwt64ZztWWK1hyeLntRNqWMYFiZjaKu1PZgikN",
"/dns4/bst-b01.ecdsa.keep.boar.network/tcp/4001/ipfs/16Uiu2HAkxLttmh3G8LYzAy1V1g1b3kdukzYskjpvv5DihY4wvx7D",
"/dns4/keep-boot-validator-0.prod-us-west-2.staked.cloud/tcp/3920/ipfs/16Uiu2HAmDnq9qZJH9zJJ3TR4pX1BkYHWtR2rVww24ttxQTiKhsaJ",
"/dns4/keep-boot-validator-1.prod-us-west-2.staked.cloud/tcp/3920/ipfs/16Uiu2HAmHbbMTDDsT2f6z8zMgDtJkTUDJQSYsQYUpaJjdMjiYNEf",
"/dns4/keep-boot-validator-2.prod-us-west-2.staked.cloud/tcp/3920/ipfs/16Uiu2HAmBXoNLLMYU9EcKYH6JN5tA498sXQHFWk4heK22RfXD7wC",
"/ip4/54.39.179.73/tcp/4001/ipfs/16Uiu2HAkyYtzNoWuF3ULaA7RMfVAxvfQQ9YRvRT3TK4tXmuZtaWi",
"/ip4/54.39.186.166/tcp/4001/ipfs/16Uiu2HAkzD5n4mtTSddzqVY3wPJZmtvWjARTSpr4JbDX9n9PDJRh",
"/ip4/54.39.179.134/tcp/4001/ipfs/16Uiu2HAkuxCuWA4zXnsj9R6A3b3a1TKUjQvBpAEaJ98KGdGue67p"]
Port = 3919

# Override the node’s default addresses announced in the network
 AnnouncedAddresses = ["/ip4/$SERVER_IP/tcp/5678"]

[TSS]
# Generazione dei parametri di Timeout delle connessioni TSS
# Il valore dovrebbe essere fornito sulla base delle risorse del server.
# E' un parametro opzionale nel caso non sia fornito di base dalla macchina
  PreParamsGenerationTimeout = "2m30s"
CONFIG

```

### TESTNET CONFIG

```bash
cat <<CONFIG >>$HOME/keep-ecdsa/config/config.toml

# Dettagli di connessione alla blockchain di ETH
[ethereum]
  URL = "wss://ropsten.infura.io/ws/v3/$INFURA_PROJECT_ID"
  URLRPC = "https://ropsten.infura.io/v3/$INFURA_PROJECT_ID"


[ethereum.account]
  Address = "$ETH_WALLET"
  KeyFile = "/mnt/keep-ecdsa/keystore/keep_wallet.json"


# Questo address di seguito potrebbe essere soggetto a variazioni future;
# in tal caso, i cambiamenti dell'address di questo contratto verranno listati qui:
# https://github.com/keep-network/keep-ecdsa/blob/master/docs/run-keep-ecdsa.adoc
[ethereum.ContractAddresses]
  BondedECDSAKeepFactory = "0xe7BF8421fBE80c3Bf67082370D86C8D81D1D77F4"


# Questo address di seguito potrebbe essere soggetto a variazioni future;
# in tal caso, i cambiamenti dell'address di questo contratto verranno listati qui:
# https://github.com/keep-network/keep-ecdsa/blob/master/docs/run-keep-ecdsa.adoc
# Addresses of applications approved by the operator.
[SanctionedApplications]
  Addresses = [
    "0x14dC06F762E7f4a756825c1A1dA569b3180153cB",
]

[Storage]
  DataDir = "/mnt/keep-ecdsa/persistence"
  
[LibP2P]
  Peers = ["/dns4/testnet.keep-client.hashd.dev/tcp/3920/ipfs/16Uiu2HAmJsBiNVFNxsJ27NSQEByv39B1M7AKx5FrAc1htqYhHGhU","/ip4/3.23.88.229/tcp/3919/ipfs/16Uiu2HAmEZpkf1Td8rSBMmgPoa66si2kJLb83Rd2eztJ6f5oLvhp","/dns4/ecdsa-2.test.keep.network/tcp/3919/ipfs/16Uiu2HAmNNuCp45z5bgB8KiTHv1vHTNAVbBgxxtTFGAndageo9Dp",	
"/dns4/ecdsa-3.test.keep.network/tcp/3919/ipfs/16Uiu2HAm8KJX32kr3eYUhDuzwTucSfAfspnjnXNf9veVhB12t6Vf","/dns4/bootstrap-1.ecdsa.keep.test.boar.network/tcp/4001/ipfs/16Uiu2HAmPFXDaeGWtnzd8s39NsaQguoWtKi77834A6xwYqeicq6N"]
Port = 3919

 # Override the node’s default addresses announced in the network
 AnnouncedAddresses = ["/ip4/$SERVER_IP/tcp/3920"]

[TSS]
# Generazione dei parametri di Timeout delle connessioni TSS
# Il valore dovrebbe essere fornito sulla base delle risorse del server.
# E' un parametro opzionale nel caso non sia fornito di base dalla macchina
  PreParamsGenerationTimeout = "2m30s"
CONFIG

```

### Avvio del docker per lancio del nodo

Una volta definito il config.toml entra in `$HOME/keep-ecdsa/config`; puoi usare il seguente comando per avviare il nodo.  Non è necessario lanciare l'immagine, il comando riconosce da solo che non hai nessuna immagine salvata localmente e la scarica automaticamente dal web.

**NOTE**: Io utilizzo la porta 3919 anche se quella di default è la 3920. Questo perchè sulla mia macchina faccio girare sia un nodo ECDSA che uno random beacon, ed entrambi non possono essere messi in ascolto sulla porta di default.

Inoltre, sei libero di rimuovere la linea di comando `--env LOG_LEVEL=debug` se sei interessato a controllare solo eventuali log connessi ad errori o ad avvisi. Ho inserito questa riga di comando per assicurarmi comunque di tenere sotto controllo ogni attività del nodo.  

```bash
sudo docker run -d \
  --restart always \
  --entrypoint /usr/local/bin/keep-ecdsa \
  --volume $HOME/keep-ecdsa:/mnt/keep-ecdsa \
  --env KEEP_ETHEREUM_PASSWORD=$KEEP_CLIENT_ETHEREUM_PASSWORD \
  --env LOG_LEVEL=debug \
  --name ecdsa \
  -p 3920:3919 \
  --log-opt max-size=100m \
  --log-opt max-file=3 \
  keepnetwork/keep-ecdsa-client:v1.1.2-rc \
  --config /mnt/keep-ecdsa/config/config.toml start
```

### Interruzione del comando precedente

```bash
sudo docker run -d \ # avvia il containter e permette al nodo di funzionare in backgroun
  --restart always \ # self-explanator.  Il nodo si riavvierà automaticamente in caso di errore
  --entrypoint /usr/local/bin/keep-ecdsa \ # Questo specifica il file da avviare all'interno del container, che permette di attivare il nodo
  --volume $HOME/keep-ecdsa:/mnt/keep-ecdsa \ # Mappa il filesystem per il container.  LOCAL:CONTAINER
  --env KEEP_ETHEREUM_PASSWORD=$KEEP_CLIENT_ETHEREUM_PASSWORD \ # configura le variabili all'interno del container, in questo caso la variabile password
  --env LOG_LEVEL=debug \ # optional, configura il tipo di log generato dal container
  --name ecdsa \ # nome del container. Se non definito, ne verrà assegnato uno a caso
  -p 3920:3919 \ # inoltra la porta di ascolto del container alla 3919.  LOCAL:CONTAINER
  --log-opt max-size=100m \ # massima dimensione del file di log
  --log-opt max-file=3 \ # solo 3 files di massimo 100MB saranno conservati in locale
  keepnetwork/keep-ecdsa-client:v1.1.2-rc \ # Specifica quale immagine Docker deve essere lanciata
  --config /mnt/keep-ecdsa/config/config.toml start # configura il container usando il tuo config.toml che hai creato precedentemente
```

### Controlla se il tuo nodo è attivo

NOTE: Il nodo impiega pochi minuti per generare un numero di processo. Sii paziente in quanto se non hai visto particolari blocchi, funziona tutto regolarmente.

```bash
docker ps -a
```

Dovresti vedere un output simile:

```bash
ubuntu@ip-172-31-25-217:~$ docker ps -a
CONTAINER ID        IMAGE                                  COMMAND                  CREATED             STATUS
    PORTS                    NAMES
a355b76ca6d4        keepnetwork/keep-ecdsa-client:1.1.2-rc   "/usr/local/bin/keep…"   8 minutes ago       Up 8 minutes        0.0.0.0:3920->3919/tcp   ecdsa
```

### Controlla se il tuo nodo non presenta errori

```bash
docker logs ecdsa --since 10m -f
```
