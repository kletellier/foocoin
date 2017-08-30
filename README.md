# 1\. Introduction

Si vous êtes intéressés par la technologie Bitcoin,voici le tutoriel pour créer « facilement » une monnaie alternative, un « altcoin » basée sur le code source de bitcoin 0.8.6.

L'idée de faire ce tutoriel m'est venu simplement après avoir passé pas mal de temps pour trouver des informations pour créer facilement un « altcoin » afin d'étudier « gratuitement » le fonctionnement de Bitcoin.

Pour ceci j'ai compilé pas mal d'informations issus d'autres sites afin de compiler facilement une version pour Windows.

# 2\. Préparation

Créer son altcoin demande une certaine connaissance de l'utilisation d'outils informatiques.  
Il vous sera nécessaire d'utiliser au moins une machine virtuelle (par exemple sous Virtualbox)

Pour ma part pour ce tutoriel, j'ai utilise des machines virtuelles sous Windows XP 32 bits et Ubuntu Server 14 32 bits (l'idéal est d'avoir la machine virtuelle sous Ubuntu), je n'ai pas testé la compilation Windows sur une machine 64 bits, mais les exécutables compilés fonctionnent sur les 2 versions.

Avant de commencer il faut déterminer certains paramètres :

*   le nom de votre altcoin, et son abréviations.
*   le volume maximal d'unités de monnaie que vous voulez générer et dans quelle durée.

# 3\. Exemple

Le mieux étant de commencer par un exemple… Nous allons créer une monnaie nommé Testcoin avec comme abréviation TSC.

## 3.1 Copie du template

J'ai mis l'ensemble du projet sur un dépôt Github:

[https://github.com/kletellier/foocoin](https://github.com/kletellier/foocoin)

Il suffit de cloner le dépôt ou bien de télécharger le dernier zip du master, le source de ce projet est basé sur une version 0.8.6 de Bitcoin.

L'idéal serait de placer votre projet dans le répertoire c:\testcoin

## 3.2 Renommage

*   On va renommer l'ensemble des occurrences de foocoin vers testcoin, pour ceci il y a fart.exe dans le répertoire contrib, il va falloir le déplacer dans c:\testcoin en le copiant

*   On ouvre une invite de commande cmd (« démarrer → executer → cmd »)

Puis :

> c:  
> cd c:\testcoin  
> fart -r * foocoin testcoin  
> fart -r * Foocoin Testcoin

On renomme le fichier _foocoin-qt.pro_ en _testcoin-qt.pro_

Ensuite on ouvre le fichier _src/qt/bitcoinunits.cpp_

à la ligne 37-39 on modifier :

     case BTC: return QString("BTC");
     case mBTC: return QString("mBTC");
     case uBTC: return Qstring::fromUtf8("μBTC");

Pour devenir

    case BTC: return QString("TSC");
    case mBTC: return QString("mTSC ");
    case uBTC: return QString::fromUtf8("μTSC ");

Ensuite pour les lignes 48-50

    case BTC: return QString("Bitcoins");
    case mBTC: return QString("Milli-Bitcoins (1 / 1,000)");
    case uBTC: return QString("Micro-Bitcoins (1 / 1,000,000)");

deviendra

    case BTC: return QString("Testcoins");
    case mBTC: return QString("Milli-Testcoins (1 / 1,000)");
    case uBTC: return QString("Micro-Testcoins (1 / 1,000,000)");

## 3.3 Renommage des entêtes de messages

L'étape suivante sera de changer les entêtes de messages :

dans le fichier _src/main.cpp_

pour la ligne 3060

    unsigned char pchMessageStart[4] = { 0xf9, 0xbe, 0xb4, 0xd9 };

on change les 4 bits :

    unsigned char pchMessageStart[4] = { 0x07, 0x07, 0x01, 0x01 };

## 3.4 Changement des numéros de ports

L'étape suivante est le changement des numéros de port (RPC et P2P) :

on va dans le fichier _src/init.cpp_ :

à la ligne 305

            "  -port=           " + _("Listen for connections on  (default: 8333 or testnet: 18333)") + "\n" +

devient

            "  -port=           " + _("Listen for connections on  (default: 9333 or testnet: 19333)") + "\n" +

et la ligne 346

    "  -rpcport=        " + _("Listen for JSON-RPC connections on  (default: 9332 or testnet: 19332)") + "\n" +

devient

    "  -rpcport=        " + _("Listen for JSON-RPC connections on  (default: 9332 or testnet: 19332)") + "\n" +

Dans le fichier _src/protocol.h_

à la ligne 23 :

    return testnet ? 18333 : 8333;

devient

    return testnet ? 19333 : 9333;

Dans le fichier _src/bitcoinrpc.cpp_

à la ligne 46 :

     return GetBoolArg("-testnet", false) ? 18332 : 8332;

devient

     return GetBoolArg("-testnet", false) ? 19332 : 9332;

## 3.5 Changement du PUBKEY (première lettre des adresses)

Ensuite on doit changer le PUBKEY, c'est ce qui donnera la première lettre des adresses de votre monnaie. Il suffit d'aller chercher la valeur sur cette page :

[https://en.bitcoin.it/wiki/List_of_address_prefixes](https://en.bitcoin.it/wiki/List_of_address_prefixes "Liste des préfixes")

Dans notre cas on choisira T, ce qui donne le chiffre 65

on ouvre le fichier _src/base58.h_ à la ligne 275

    PUBKEY_ADDRESS = 0,

devient

    PUBKEY_ADDRESS = 65,

## 3.6 Calcul des récompenses et de la masse globale émise

L'étape suivante est la partie qui permet de définir les paramètres de génération de votre monnaie.

L'algorithme de Bitcoin permet de créer un certain volume de monnaie à chaque bloc pour récompenser les mineurs, ce volume diminue avec le temps, il est divisé par 2 tout les x blocs minés sachant que la durée entre les blocs est à définir, au final cela crée une valeur maximale de monnaie crée vers laquelle votre monnaie va tendre. Pour déterminer ces chiffres il faut déjà déterminer quel sera la durée de validation d'un bloc, pour Bitcoin elle est de 10mn, dans notre cas on la définit à 2mn soit 120 sec.

Dans le fichier _src/main.cpp_ aux lignes 1084-1086

    static const int64 nTargetTimespan = 14 * 24 * 60 * 60; // two weeks
    static const int64 nTargetSpacing = 10 * 60;
    static const int64 nInterval = nTargetTimespan / nTargetSpacing;

nTargetSpacing représente le nombre de secondes qui devra être nécessaire pour la création d'un bloc, on la définit dont à 120.

La force de calcul augmentant avec le temps, les blocs risque de se calculer trop vite, Bitcoin implémente un système de difficulté qui est ré-évalué toutes les 2 semaines dans Bitcoin afin d'ajuster en permanence le temps de calcul à 10mn quelque-soit la force de calcul.

Il faut donc ajuster nTargetTimespan, on le mettra à 1h soit 3600.

L'étape suivante consistera à définit à partir de combien de blocs validés les coin générés seront considérés comme mature, dans Bitcoin il faut attendre 100 validations pour utiliser des coins crées à la suite de l'ajout d'un bloc dans la blockchain, on définira le notre à 20.

dans _src/main.h_

à la ligne 55 :

    static const int COINBASE_MATURITY = 100;

devient

    static const int COINBASE_MATURITY = 20 ;

L'étape suivante consiste à définir la progression et le volume de génération des testcoins, pour cela j'ai crée une feuille LibreOffice qui se trouve dans le répertoire contrib, elle se nomme calcul_altcoin.ods

en l'ouvrant on trouve 3 paramètres à gauche :

*   Reward : Le nombre de coins donnés à chaque création de bloc (pour le 1er bloc)
*   Subsidy : Le nombre de blocs nécessaires avant de divisé le reward par 2
*   Duration : la durée de génération d'un bloc.

En ajustant les 3 paramètres on peut déterminer la date vers la laquelle notre monnaie aura généré tout ses coins et le nombre maximal, dans notre cas si l'on donne 125 coins et qu'on divise par 2 cette valeur à chaque 400 blocs pour des blocs de 2 mn on obtient environ 100000 coins maximum en à peu près 1 semaine.

On garde donc ses paramètres de coté.

On va définir la valeur maximale  
dans _src/main.h_ à la ligne 52 :

    static const int64 MAX_MONEY = 21000000 * COIN;

devient

    static const int64 MAX_MONEY = 100000 * COIN;

Ensuite on doit définir que l'on donne 125 coins au début et qu'on divise à chaque 400 blocs (800 mn)

on va donc dans _src/main.cpp_ ligne 1074

    int64 static GetBlockValue(int nHeight, int64 nFees)
    {
        int64 nSubsidy = 50 * COIN;

        // Subsidy is cut in half every 210000 blocks, which will occur approximately every 4 years
        nSubsidy >>= (nHeight / 210000);

        return nSubsidy + nFees;
    }

devient

    int64 static GetBlockValue(int nHeight, int64 nFees)
    {
        int64 nSubsidy = 125 * COIN;

        // Subsidy is cut in half every 400 blocks
        nSubsidy >>= (nHeight / 400);

        return nSubsidy + nFees;
    }

## 3.7 Génération du Genesis Block

L'étape suivante est un peu plus complexe et fait entrer en jeu pour la première fois notre machine virtuelle, ils s'agit de calculer le bloc original, le genesis block.

Ce bloc à pour but de définir le début de votre blockchain, pour cela l'idéal est d'avoir une phrase situant la date de début de votre monnaie dans l'exemple de Bitcoin : « The Times 03/Jan/2009 Chancellor on brink of second bailout for banks » est issu d'un article de journal, ce qui prouve facilement la date initiale du bloc (et donc qu'aucun bloc et aucune transaction ne peut exister auparavant).

Nous prendrons « Le 25 noel 2014 le pere noel est passe »

Il faut aussi une date au format UNIX en allant sur le site [http://www.epochconverter.com/](http://www.epochconverter.com/ "Epoch time")

nous prenons la date UNIX actuelle : 1419416375

Ensuite j'utilise un script issus de ce dépôt : [https://github.com/lhartikk/GenesisH0](https://github.com/lhartikk/GenesisH0)

j'ai modifié le script pour passer la difficulté de 32 à 20 bits car sinon la puissance de calcul sur un petit réseau de 2 pc rend le minage trop long…

Ce script se trouve dans contrib il s'appelle _genesis_ld.py_ la version 32 bits _genesis.py_

Pour l' exécuter sur notre machine Ubuntu il faut installer les paquets python pycrypto et construct

puis lancer

    python genesis_ld.py -z "Le 25 noel 2014 le pere noel est passe " -t 1419416375

après quelques minutes (ou quelques heures en version 32 bits) il va vous générer les données suivantes :

> algorithm: SHA256  
> merkle hash: 0410191ecb81668ab68f707ee4e105242fd776ff34a99206fb9717b00135038a  
> pszTimestamp: Le 25 noel 2014 le pere noel est passe  
> pubkey: 04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f  
> time: 1419416375  
> bits: 0x1e0ffff0  
> nonce: 1540800  
> genesis hash: 0000006036353e4336d653b2d82f1630a26c2cae6a1e2f09b8ee0a9f023ed9bd

Nous allons maintenant le définir dans le source

dans _src/main.cpp_

à la ligne 2760

    const char* pszTimestamp = "The Times 03/Jan/2009 Chancellor on brink of second bailout for banks";

devient

    const char* pszTimestamp = "Le 25 noel 2014 le pere noel est passe ";

à la ligne 2766

    txNew.vout[0].scriptPubKey = CScript() << ParseHex("04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f") << OP_CHECKSIG;

on recopie le pubkey trouvé par le genesis.py

    txNew.vout[0].scriptPubKey = CScript() << ParseHex("04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f") << OP_CHECKSIG;

ensuite aux lignes 2772-2774

    block.nTime    = 1231006505;
    block.nBits    = 0x1d00ffff;
    block.nNonce   = 2083236893;

devient avec les valeurs trouvés par le genesis.py

    block.nTime    = 1419416375;
    block.nBits    =  0x1e0ffff0 ;
    block.nNonce   = 1540800;

à la ligne 2787 on rentre le merkle root

     assert(block.hashMerkleRoot == uint256("0x4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b"));

devient

     assert(block.hashMerkleRoot == uint256("0x410191ecb81668ab68f707ee4e105242fd776ff34a99206fb9717b00135038a"));

On va rentrer le hash du bloc de genèse :  
à la ligne 34

    uint256 hashGenesisBlock("0x000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f");

devient

    uint256 hashGenesisBlock("0x0000006036353e4336d653b2d82f1630a26c2cae6a1e2f09b8ee0a9f023ed9bd");

si vous avez décidé d'utiliser la version 20 bits, on va aussi diminuer la difficulté à la ligne 35 en la passant de 32 à 20 bits.

    static CBigNum bnProofOfWorkLimit(~uint256(0) >> 32);

devient

    static CBigNum bnProofOfWorkLimit(~uint256(0) >> 20);

## 3.8 Définition des checkpoints

Ensuite dans _src/checkpoints.cpp_

à la ligne 36 on va définir les points de passage dans la chaine des blocs, dans notre cas seul le genesis block :

     static MapCheckpoints mapCheckpoints =
            boost::assign::map_list_of
            ( 11111, uint256("0x0000000069e244f73d78e8fd29ba2fd2ed618bd6fa2ee92559f542fdb26e7c1d"))
            ( 33333, uint256("0x000000002dd5588a74784eaa7ab0507a18ad16a236e7b1ce69f00d7ddfb5d0a6"))
            ( 74000, uint256("0x0000000000573993a3c9e41ce34471c079dcf5f52a0e824a81e7f953b8661a20"))
            (105000, uint256("0x00000000000291ce28027faea320c8d2b054b2e0fe44a773f3eefb151d6bdc97"))
            (134444, uint256("0x00000000000005b12ffd4cd315cd34ffd4a594f430ac814c91184a0d42d2b0fe"))
            (168000, uint256("0x000000000000099e61ea72015e79632f216fe6cb33d7899acb35b75c8303b763"))
            (193000, uint256("0x000000000000059f452a5f7340de6682a977387c17010ff6e6c3bd83ca8b1317"))
            (210000, uint256("0x000000000000048b95347e83192f69cf0366076336c639f9b7228e9ba171342e"))
            (216116, uint256("0x00000000000001b4f4b433e81ee46494af945cf96014816a4e2370f11b23df4e"))
            (225430, uint256("0x00000000000001c108384350f74090433e7fcf79a606b8e797f065b130575932"))
            (250000, uint256("0x000000000000003887df1f29024b06fc2200b55f8af8f35453d7be294df2d214"))
            ;
        static const CCheckpointData data = {
            &mapCheckpoints,
            1375533383, // * UNIX timestamp of last checkpoint block
            21491097,   // * total number of transactions between genesis and last checkpoint
                        //   (the tx=... number in the SetBestChain debug.log lines)
            60000.0     // * estimated number of transactions per day after checkpoint
        };

devient

     static MapCheckpoints mapCheckpoints =
            boost::assign::map_list_of
            ( 0, uint256("0x0000006036353e4336d653b2d82f1630a26c2cae6a1e2f09b8ee0a9f023ed9bd"))

            ;
        static const CCheckpointData data = {
            &mapCheckpoints,
            1419416375, // * UNIX timestamp of last checkpoint block
            0,   // * total number of transactions between genesis and last checkpoint
                        //   (the tx=... number in the SetBestChain debug.log lines)
            300.0     // * estimated number of transactions per day after checkpoint
        };

## 3.9 Définition des nœuds majeurs

Ensuite on définit les nœuds majeurs (ce sont les adresses ip des clients sur qui posséderont une copie certifiée de la blockchain ), cette opération est nécessaire si vous rendez public votre altcoin.  
Il suffira alors de lancer sur cette IP une instance de votre daemon que nous allons compiler dans le chapitre suivant.

dans _src/net.cpp_ à la ligne 1193

    static const char *strMainNetDNSSeed[][5] = {
        {"bitcoin.sipa.be", "seed.bitcoin.sipa.be"},
        {"bluematt.me", "dnsseed.bluematt.me"},
        {"dashjr.org", "dnsseed.bitcoin.dashjr.org"},
        {"xf2.org", "bitseed.xf2.org"},
        {NULL, NULL}
    };

devient

    static const char *strMainNetDNSSeed[][5] = {

        {NULL, NULL}
    };

# 4\. Compilation

## 4.1 Compilation sous Ubuntu

L'étape suivante est la compilation dans un premier temps sur Ubuntu il faudra installer composant nécessaire :

    sudo apt-get install build-essential libboost-all-dev libcurl4-openssl-dev libdb5.1-dev libdb5.1++-dev git qt-sdk libminiupnpc-dev libssl-dev

Copier vos sources dans un répertoire de votre machine virtuelle Ubuntu

Puis ensuite en allant dans le répertoire src /levedb

    chmod 755 build *

et on compile leveldb

    make libleveldb.a libmemenv.a

puis de nouveau dans src et tapez

    make -f makefile.unix

vous devriez avoir un testcoind dans le répertoire à la fin de la compilation,  
ensuite on execute strip testcoind afin de rétrecir l'executable

on l'execute :

    ./testcoind &

au bout de quelques secondes un message apparaît :

> /home/administrateur/.testcoin/testcoin.conf  
> It is recommended you use the following random password:  
> rpcuser=testcoinrpc  
> rpcpassword=595S2Fd4EM8KT3ktnUmooE81zLc4n1e73rmmdCT7q4YJ  
> (you do not need to remember this password)  
> The username and password MUST NOT be the same.

Vous devez créer un fichier _testcoin.conf_ dans le répertoire .testcoin de votre profil actuel (ici administrateur) et ajouter :

> rpcuser=testcoinrpc  
> rpcpassword=595S2Fd4EM8KT3ktnUmooE81zLc4n1e73rmmdCT7q4YJ

Une fois ce fichier crée vous relancer votre instance du daemon :

    ./testcoind &

puis vous lancer

    ./testcoind getinfo

vous obtenez normalement :

> {  
> "version" : 80600, "protocolversion" : 70001, "walletversion" : 60000, "balance" : 0.00000000, "blocks" : 0, "timeoffset" : 0, "connections" : 0, "proxy" : "", "difficulty" : 0.00024414, "testnet" : false, "keypoololdest" : 1419426388, "keypoolsize" : 101, "paytxfee" : 0.00000000, "errors" : "" }

Votre daemon fonctionne !!!

Stoppons le avec la commande

    ./testcoind stop

## 4.2 Compilation sous Windows

Désormais on s'attaque à la compilation du daemon et du Wallet QT sur Windows

Pour ça j'ai récupérer un script qui permet d'automatiser cette tache, je l'ai pris dans ce dépôt :

[https://github.com/phelixbtc/bitcoin/tree/0.8.5-EWB](https://github.com/phelixbtc/bitcoin/tree/0.8.5-EWB)

Dans mon projet il se trouve dans contrib/easywinbuilder

Il suffit alors simplement d'éxecuter le fichier : ___all_easywinbuilder.bat_

et de suivre les instructions à l'écran, à la fin vous devez avoir un testcoind.exe et un répertoire de build avec votre Wallet QT

# 5\. Minage de votre altcoin

Étape suivante et la plus intéressante le test pour vérifier le minage de vos coins :

On va lancer l’exécutable Windows , il faut prendre le fichier testcoind.exe présent dans le répertoire source et le mettre dans un autre répertoire, on va aussi y copier les fichiers :

*   libgcc_s_dw2-1.dll
*   libstdc++-6.dll
*   mingwm10.dll

présents dans le répertoire release (celui ou se trouve le protefeuille Qt)

On ouvre une fenêtre de commande cmd et on lance testcoind

On a le même message d'erreur que sur la version ubuntu qui demande de créer un mot de passe RPC, il faudra donc créer le fichier testcoin.conf dans _C:\Users\utilisateur\AppData\Roaming\Testcoin_

Il faut lancer ensuite la machine virtuelle avec Ubuntu, et connaître son adresse IP , pour cela on utilise la commande ifconfig.

Dans notre cas 192.168.0.153

Puis celle de notre PC sous windows (avec ce coup ci la commande ipconfig que l'on execute dans une fenêtre de commande)

Dans notre cas 192.168.0.150

On va donc lancer nos 2 testcoind,

On va rajouter une ligne addnode=192.168.0.150 dans _/home//.testcoin/testcoin.conf_

on lance donc

     ./testcoind  &

On rajoute aussi une ligne addnode=192.168.0.153 dans _C:\Users\utilisateur\AppData\Roaming\Testcoin\testcoin.conf_

Puis on lance sur windows dans une fenêtre de commande, en le connectant sur la machine Ubuntu

    testcoind

on retourne sur la machine Ubuntu et on execute la commande

    ./testcoind getinfo

et on a le message suivant :

> {  
> "version" : 80600, "protocolversion" : 70001, "walletversion" : 60000, "balance" : 0.00000000, "blocks" : 0, "timeoffset" : 0, "connections" : 1, "proxy" : "", "difficulty" : 0.00024414, "testnet" : false, "keypoololdest" : 1420445245, "keypoolsize" : 101, "paytxfee" : 0.00000000, "errors" : "" }

connections = 1 spécifie que la connexion est effectué entre les 2 machines.

On va pouvoir lancer le minage sur la machine ubuntu :

    ./testcoind getnewaddress ""

permet de créer une adresse dans le portefeuille

on lance le minage :

    ./testcoind setgenerate true 16

On lance la commande :

    ./testcoind getmininginfo

la valeur hashpersec est supérieure à zéro on est donc en train de miner, si vous avez choisi une difficulté restreinte les blocs vont se créer rapidement…

en lançant

    ./testcoind getbalance

au bout de quelques minutes vous devriez avoir un chiffre supérieur à zéro ça y est vous avez miné vos premiers altcoins….

Vos daemon peuvent être utilisés pour l'ensemble des opérations sur votre réseau d'altcoin, vous pouvez trouver une liste des commandes disponible à cette adresse :

[https://en.bitcoin.it/wiki/Original_Bitcoin_client/API_calls_list](https://en.bitcoin.it/wiki/Original_Bitcoin_client/API_calls_list)

L'étape suivante consistera à mettre en « production » votre altcoin, pour ceci vous devrez donc mettre votre daemon sur un serveur accessible, mettre son adresse ip dans dans src/net.cpp à la ligne 1193 et ensuite c'est la grande aventure, sachez néanmoins qu'il est très risqué d'utiliser à petite échelle votre monnaie…

# 6\. Transfert de vos altcoins minés

La dernière étape est de transférer vos altcoins crée vers votre portefeuille Qt sur votre machine Windows.

Pour ceci vous allez récupérer votre répertoire release, il contient testcoin-qt.exe

En lançant votre programme vous obtenez une interface de portefeuille dans l'onglet Recevoir vous avez une adresse en faisant clic droit dessus vous pouvez la copier dans votre clipboard, l'étape suivante et de transférer des fonds vers cette adresse. On va sur la machine ubuntu et on tape la commande suivante :

    ./testcoind sendtoaddress  

Aussitôt vous devriez voir arriver vos fonds dans votre portefeuille avec une notification dans la barre de taches…

Voilà vous avez une monnaie pour faire des tests ou utiliser entre amis...