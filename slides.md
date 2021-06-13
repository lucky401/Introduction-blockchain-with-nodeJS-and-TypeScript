---
theme: default
title: 'Introduction blockchain with nodeJS and TypeScript'
download: true
monaco: false
highlighter: Prism
routerMode: 'hash'
colorSchema: 'dark'
info: |
  ## Introduction blockchain with nodeJS and TypeScript
  By Lucky Dewa Satria.
---

<style>
@import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');
</style>

# Simple Blockchain

With NodeJS and TypeScript

<div class="pt-12">
  <span
    @click="$slidev.nav.next"
    class="px-2 p-1 rounded cursor-pointer" 
    hover="bg-white bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span>
</div>

<img
  src="/bitcoin-img.svg"
  class="absolute right-0 top-0 h-full order-first filter blur-sm" 
  title="bitcoin"
  alt="bitcoin background"
/>

---

# Crypto Library from NodeJS

The crypto module provides cryptographic functionality that includes a set of wrappers for OpenSSL's hash, HMAC, cipher, decipher, sign, and verify functions.

```ts {all}
import crypto from 'crypto';
```

<img
  src="/crypto.png"
  class="h-60"
  border="rounded-lg"
  title="crypto"
  alt="crypto"
/>

Read more about [Crypto](https://nodejs.org/api/crypto.html#crypto_crypto)

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent; 
  -moz-text-fill-color: transparent;
}
</style>

---

# Create Transaction Class

The fundamental purpose of any cryptocurrencies transfer fund from one user to another user in a transaction

```ts {all|2,12|4-5|6|9-11|all}
// Transfer of funds between two wallets
class Transaction {
  constructor(
    public amount: number,
    public payer: string,
    public payee: string // public key
  ) {}

  toString() {
    return JSON.stringify(this);
  }
}
```

---

# Create Block Class

Block is container for multiple transactions

<div class="flex justify-between">
  <div class="w-1/2">

```ts {all|4|5-6}
// Individual block on the chain
class Block {
  constructor(
    public prevHash: string,
    public transaction: Transaction,
    public ts = Date.now()
  ) {}

  get hash() {
    const str = JSON.stringify(this);
    const hash = crypto.createHash('SHA256');
    hash.update(str).end();
    return hash.digest('hex');
  }
}
```

  </div>
  <div class="w-1/2 ml-6">
  </div>
</div>

<span v-click="1" class="absolute right-65 top-44 text-green-300 text-xl">
  link to previous block
</span>

<arrow v-click="1" x1="510" y1="190" x2="300" y2="195" color="#564" width="3" arrowSize="1" />

---

# Create Block Class

Block is container for multiple transactions

<div class="flex justify-between">
  <div class="w-1/2">

```ts {9,14|10|11-13|all}
// Individual block on the chain
class Block {
  constructor(
    public prevHash: string,
    public transaction: Transaction,
    public ts = Date.now()
  ) {}

  get hash() {
    const str = JSON.stringify(this);
    const hash = crypto.createHash('SHA256');
    hash.update(str).end();
    return hash.digest('hex');
  }
}
```

  </div>
  <div class="w-1/2 ml-6">
    <img
      src="/hash-how.png"
      class="h-50 bg-gray-500"
      border="rounded-lg"
      title="hash how"
      alt="hash how"
    />
    <div class="grid grid-cols-2">
      <img
        v-click="1"
        src="/nsa.png"
        class="h-36 mt-6 mx-2"
        border="rounded-lg"
        title="NSA logo"
        alt="NSA logo"
      />
      <img
        v-click="1"
        src="/oneway.jpeg"
        class="h-34 mt-6 mx-"
        border="rounded-lg"
        title="One way"
        alt="One way"
      />  
    </div>
  </div>
</div>

---

# Create Chain Class

Linked-list of block with singleton instance (So we have only one chain)

<div class="text-xs h-10">

```ts {all}
// The blockchain
class Chain {
  // Singleton instance
  public static instance = new Chain();
  chain: Block[];
  constructor() {
    this.chain = [
      // Starting block
      new Block('', new Transaction(100, 'fed', 'satoshi')),
    ];
  }
  // Most recent block
  get lastBlock() {
    return this.chain[this.chain.length - 1];
  }
  addBlock(transaction: Transaction) {
    const newBlock = new Block(this.lastBlock.hash, transaction);
    this.chain.push(newBlock);
  }
}
```

</div>

---

# Create Wallet Class

Allow people to securely transfer fund back and forth by implementing wallet

```ts {all}
class Wallet {
  public publicKey: string; // For receiving money
  public privateKey: string; // For spending money
  constructor() {
    const keypair = crypto.generateKeyPairSync('rsa', {
      modulusLength: 2048,
      publicKeyEncoding: { type: 'spki', format: 'pem' },
      privateKeyEncoding: { type: 'pkcs8', format: 'pem' },
    });

    this.privateKey = keypair.privateKey;
    this.publicKey = keypair.publicKey;
  }
  sendMoney(amount: number, payeePublicKey: string) {
    const transaction = new Transaction(amount, this.publicKey, payeePublicKey);
    const sign = crypto.createSign('SHA256');
    sign.update(transaction.toString()).end();
    const signature = sign.sign(this.privateKey);
    Chain.instance.addBlock(transaction, this.publicKey, signature);
  }
}
```

---

# Alter Chain Class

Add wallet in chain class to improve security

<div class="text-xs h-10">

```ts {11-17}
// The blockchain
class Chain {
  // ... //
  // Add a new block to the chain if valid signature
  addBlock(
    transaction: Transaction,
    senderPublicKey: string,
    signature: Buffer
  ) {
    // ... //
    const verify = crypto.createVerify('SHA256');
    verify.update(transaction.toString());
    const isValid = verify.verify(senderPublicKey, signature);
    if (isValid) {
      const newBlock = new Block(this.lastBlock.hash, transaction);
      this.chain.push(newBlock);
    }
  }
  // ... //
}
```

</div>

---

<div class="absolute top-1/2 left-1/2 transform-gpu -translate-x-1/2 -translate-y-1/2">
  <span class="text-center block text-4xl leading-loose" style="font-family: 'Press Start 2P', sans-serif">
    One more issue <twemoji-skull class="inline" />
  </span>
</div>
---

# Double Spending!

<div class="absolute top-1/2 left-1/2 transform-gpu -translate-x-1/2 -translate-y-1/2">
  <img
    src="/double-spending.jpeg"
    class="h-xs bg-gray-500"
    border="rounded-lg"
    title="hash how"
    alt="hash how"
  />
</div>

---

# Alter chain Class with proof-of-work concepts

```ts {4-18}
class Chain {
  // ... //
  // Proof of work system
  mine(nonce: number) {
    let solution = 1;
    console.log('mining...');
    while (true) {
      const hash = crypto.createHash('MD5');
      hash.update((nonce + solution).toString()).end();
      const attempt = hash.digest('hex');

      if (attempt.substr(0, 4) === '0000') {
        console.log(`Solved: ${solution}`);
        return solution;
      }

      solution += 1;
    }
  }
  // ... //
}
```

---

# add nonce prop to block class

```ts {2}
class Block {
  public nonce = Math.round(Math.random() * 999999999);
  // ... //
}
```

---

# Alter add block method in chain class

```ts {13}
// The blockchain
class Chain {
  // ... //
  // Add a new block to the chain if valid signature
  addBlock(
    transaction: Transaction,
    senderPublicKey: string,
    signature: Buffer
  ) {
    // ... //
    if (isValid) {
      const newBlock = new Block(this.lastBlock.hash, transaction);
      this.mine(newBlock.nonce); // add this
      this.chain.push(newBlock);
    }
  }
  // ... //
}
```

---

# Usage

```ts {all}
const satoshi = new Wallet();
const lucky = new Wallet();

satoshi.sendMoney(50, lucky.publicKey);
```

<img
  src="/result.png"
  class="h-66"
  border="rounded-lg"
  title="result"
  alt="result"
/>

---

# Thanks, Any feedback would be appreciated

<div class="flex w-sm my-10 items-center">
  <img src="/lucky.jpg" class="h-20 rounded-full shadow-2xl">
  <div class="ml-5">
    <span class="block mb-2" style="font-family: 'Press Start 2P', sans-serif">Lucky DS</span>
    <span class="block mb-2">Fullstack Javascript Engineer</span>
    <a href="tel:085156501865" class="text-xs">085156501865</a>
  </div>
</div>

### Tech Stack for this presentation

<div border="rounded" class="bg-white mt-2">
  <div class="flex justify-around p-5">
    <img src="/vue-logo.png" alt="vue" class="h-20 mx-2" title="vue"/>
    <img src="/slide_dev.png" alt="slidev" class="h-20 mx-2 " title="slidev"/>
    <img src="/cloudfare.png" alt="cloudfare" class="h-20 mx-2" title="cloudfare"/>
    <img src="/github.png" alt="github" class="h-20 mx-2" title="github"/>
    <img src="/vite.svg" alt="vite" class="h-20 mx-2" title="vite"/>
  </div>
</div>
