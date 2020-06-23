# Getting Started <a id="getting-started"></a>

This documentation is for developers using caver-js v1.5.0 or higher. If you are using an older version, see [Getting Started (~v1.4.1)](v1.4.1/getting-started.md).

## Prerequisites <a id="prerequisites"></a>

### Dependencies <a id="dependencies"></a>

The following packages are required to use the caver-js library.

* [Node.js](https://nodejs.org/en/download/)
* [npm](https://www.npmjs.com/get-npm)
* [gcc-c++](https://gcc.gnu.org/)
* [Solidity compiler](https://solidity.readthedocs.io/en/develop/installing-solidity.html)

**Note** caver-js can run on Node.js versions 8, 10 and 12, and the recommended versions are:

* lts/carbon \([8.16.0](https://nodejs.org/dist/latest-v8.x/)\)
* lts/dubnium \([10.16.0](https://nodejs.org/dist/latest-v10.x/)\)
* lts/erbium \([12.18.0](https://nodejs.org/dist/latest-v12.x/)\)

If you are already using a different version of the Node \(for example, Node v14\), use the Node Version Manager\([NVM](https://github.com/nvm-sh/nvm)\) to install and use the version supported by caver-js.

### Installation <a id="installation"></a>

To try it out, install caver-js with npm using the following command:

```text
$ npm install caver-js
```

**Note**: `package.json` file should exist on the same install path. If it does not exist, `package.json` can be generated via `npm init`.

To install a specific version of caver-js, try the following command:

```text
$ npm install caver-js@X.X.X
```

## Sending KLAY at a glance <a id="sending-klay-at-a-glance"></a>

This section describes a simple example of using a `keystore file` to send KLAY with a value transfer transaction. The keystore file can be created in [Klaytn Wallet](../../../toolkit/klaytn-wallet.md#how-to-receive-baobab-testnet-klay). If you need KLAY for testing, you can get Baobab testnet KLAY from the [Klaytn Wallet](../../../toolkit/klaytn-wallet.md#how-to-receive-baobab-testnet-klay).

```javascript
const fs = require('fs')
const Caver = require('./index')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	// Read keystore json file
	const keystore = fs.readFileSync('./keystore.json', 'utf8')

	// Decrypt keystore
	const keyring = caver.wallet.keyring.decrypt(keystore, 'password')
	console.log(keyring)

	// Add to caver.wallet
	caver.wallet.add(keyring)
	
	// Create value transfer transaction
	const vt = new caver.transaction.valueTransfer({
		from: keyring.address,
		to: '0x8084fed6b1847448c24692470fc3b2ed87f9eb47',
		value: caver.utils.toPeb(1, 'KLAY'),
		gas: 25000,
	})

	// Sign to the transaction
	const signed = await caver.wallet.sign(keyring.address, vt)

	// Send transaction to the Klaytn blockchain platform (Klaytn)
	const receipt = await caver.rpc.klay.sendRawTransaction(signed)
	console.log(receipt)
}

testFunction()
```

## Starting with caver-js <a id="starting-with-caver-js"></a>

Once you have finished installing caver-js, you can now connect to a Klaytn Node using caver-js.

To practice the examples below, first create a test file in the working directory.

```bash
$ touch test.js
```
You can see the `test.js` file created in the working directory.

Write the following code in test.js.
```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	const version = await caver.rpc.klay.getClientVersion()
	console.log(version)
}

testFunction()
```

Running the above code gives you the following result.

```bash
$ node ./test.js
Klaytn/v1.4.0/linux-amd64/go1.14.1
```

If you see the output of console.log like above, proceed with the steps below. The version number can be different according to the version of the connected Klaytn node.

### Connecting to a Klaytn Node <a id="connecting-to-a-klaytn-node"></a>

You can import the caver-js module and connect it to a Klaytn Node in the Baobab testnet as shown in the example below:

```javascript
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')
```

If you are running an EN, you can connect it to your own node by changing the host and port like below:

```javascript
const Caver = require('caver-js')
const caver = new Caver('http://localhost:8551/')
```

## Managing Keyrings <a id="managing-keyrings"></a>

[Keyring] is a structure that contains the address of the Klaytn account and the private key(s). 

[Keyring] can be classified into three types depending on the type of key being stored: [SingleKeyring] to store one address and one private key, [MultipleKeyring] to store one address and multiple private keys, and [RoleBasedKeyring] to store one address and one or more private keys for each role.

[SingleKeyring] defines `key` property inside, and this `key` store one private key.

[MultipleKeyring] defines `keys` property inside, and this `keys` is implemented as an array to store multiple private keys.

The `keys` property defined in [RoleBasedKeyring] is implemented as a two-dimensional array (empty `keys` will look like `[ [], [], [] ]`) that can include multiple keys for each [role]. The first element of the array is filled with the private key(s) to be used for `roleTransactionKey`, the second element the private key(s) to be used for `roleAccountUpdateKey`, and the third element the private key(s) to be used for `roleFeePayerKey`.

### Creating a Keyring <a id="creating-a-keyring"></a>

#### Generating a SingleKeyring <a id="generating-a-singlekeyring"></a>

You can randomly generate a single keyring as shown below.

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	const keyring = caver.wallet.keyring.generate()
	console.log(keyring)
}

testFunction()
```

Running the above code gives you the following result.

```bash
$ node ./test.js
SingleKeyring {
	_address: '0x3d263c3c0df60c5516f932d244531742f45eed5c',
	_key: PrivateKey { _privateKey: '0x{private key}' }
}
```

The execution result is shown above. Member variables defined inside the instance can be accessed through `keyring.address` and `keyring.key`.

#### Creating a SingleKeyring from private key <a id="creating-a-singlekeyring-from-private-key"></a>

Also, if you own a specific private key, you can use it to create a keyring as shown below.

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	// Create a keyring from a private key
	const keyringFromPrivateKey = caver.wallet.keyring.createFromPrivateKey('0x{private key}')
	console.log(keyringFromPrivateKey)
}

testFunction()
```

Running the above code gives you the following result.

```bash
$ node ./test.js
SingleKeyring {
	_address: '0xf5a9079f311f9ec55170af351627aff0c5d2e287',
	_key: PrivateKey { _privateKey: '0x{private key}' } 
}
```

The result of `caver.wallet.keyring.createFromPrivateKey`, like the result of `caver.wallet.keyring.generate` above, is a [SingleKeyring] instance with an address defined inside it and a [PrivateKey] instance in `keyring.key`.

#### Creating a SingleKeyring with a private key and an address <a id="creating-a-singlekeyring-with-a-private-key-and-an-address"></a>

If your private key for your Klaytn account is decoupled from the address, you can create a keyring using the given address and the given private key like below.

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	// Create a keyring with an address and a private key
	const keyring = caver.wallet.keyring.createWithSingleKey('0x{address in hex}', '0x{private key}')
	console.log(keyring)

	// Create a keyring from a KlaytnWalletKey
	const keyringFromKlaytnWalletKey = caver.wallet.keyring.createFromKlaytnWalletKey('0x{private key}0x{type}0x{address in hex}')
	console.log(keyringFromKlaytnWalletKey)
}

testFunction()
```

Run the code in your console like below.

```bash
$ node ./test.js
SingleKeyring {
	_address: '0x17e7531b40ad5d7b5fa7b4ec78df64ce1cb36d24',
	_key: PrivateKey { _privateKey: '0x{private key}' }
}
SingleKeyring {
	_address: '0x17e7531b40ad5d7b5fa7b4ec78df64ce1cb36d24',
	_key: PrivateKey { _privateKey: '0x{private key}' }
}
```

#### Creating a MultipleKeyring with multiple private keys <a id="creating-a-multiplekeyring-with-multiple-private-keys"></a>

If you want to use multiple private keys, you can create a [MultipleKeyring] using an address and multiple private keys. The below examples show how to create a [MultipleKeyring] with multiple private keys.

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	// Create a keyring with an address and private keys
	const keyring = caver.wallet.keyring.createWithMultipleKey('0x{address in hex}', [ '0x{private key1}', '0x{private key2}' ])
	console.log(keyring)
}

testFunction()
```

Running the above code gives you the following result.

```bash
$ node ./test.js
MultipleKeyring {
	_address: '0x17e7531b40ad5d7b5fa7b4ec78df64ce1cb36d24',
	_keys: [
		PrivateKey { _privateKey: '0x{private key1}' },
		PrivateKey { _privateKey: '0x{private key2}' } 
	]
}
```

As you can see, `_keys` has multiple PrivateKey instances in the array. Member variables defined inside the instance can be accessed through `keyring.address` and `keyring.keys`.


#### Creating a RoleBasedKeyring with private keys <a id="creating-a-rolebasedkeyring-with-role-based-private-keys"></a>

To use different private key(s) for each [role], `caver.wallet.keyring.createWithRoleBasedKey` is used instead. Each array element represents a role described in [RoleBasedKeyring]. The example below shows how to create a [RoleBasedKeyring] instance from different keys for each role.

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	// Create a keyring with an address and private keys defined by each roles
	const keyring = caver.wallet.keyring.createWithRoleBasedKey('0x{address in hex}', [
		[ '0x{private key1}', '0x{private key2}', '0x{private key3}' ],
		[ '0x{private key4}'],
		[ '0x{private key5}', '0x{private key6}' ],
	])
	console.log(keyring)
}

testFunction()
```

Running the above code gives you the following result.

```bash
$ node ./test.js
RoleBasedKeyring {
	_address: '0x17e7531b40ad5d7b5fa7b4ec78df64ce1cb36d24',
	_keys: [ 
		[ 
			PrivateKey { _privateKey: '0x{private key1}' },
			PrivateKey { _privateKey: '0x{private key2}' },
			PrivateKey { _privateKey: '0x{private key3}' }
		],
		[ PrivateKey { _privateKey: '0x{private key4}' } ],
		[ 
			PrivateKey { _privateKey: '0x{private key5}' },
			PrivateKey { _privateKey: '0x{private key6}' }
		]
	]
}
```

Looking at the output above, the first element of the keys array, `roleTransactionKey`, has three PrivateKey instances, and the second element, `roleAccountUpdateKey`, has one PrivateKey instance. And the last element of the array, `roleFeePayerKey`, has two PrivateKey instances.

**Note**: Calling functions related to keyring ([caver.wallet.keyring]) or wallet ([caver.wallet]) do not affect the actual Klaytn blockchain platform (Klaytn).

### Adding Keyrings to caver-js <a id="adding-keyrings-to-caver-js"></a>

You can use a keyring easily by using the in-memory wallet provided by caver-js. The following examples illustrate how to add a keyring to a wallet using a keyring instance and a keystore file generated by [Klaytn Wallet](https://wallet.klaytn.com/).

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	// Using a keyring instance
	const keyring = caver.wallet.keyring.generate()
	caver.wallet.add(keyring)
	console.log(caver.wallet.getKeyring(keyring.address))

	// Using a keystore file
	const decrypted = caver.wallet.keyring.decrypt({ 
		version: 4,
		id: '9c12de05-0153-41c7-a8b7-849472eb5de7',
		address: '0xc02cec4d0346bf4124deeb55c5216a4138a40a8c',
		keyring: [
			{ 
				ciphertext: 'eacf496cea5e80eca291251b3743bf93cdbcf7072efc3a74efeaf518e2796b15',
				cipherparams: { iv: 'd688a4319342e872cefcf51aef3ec2da' },
				cipher: 'aes-128-ctr',
				kdf: 'scrypt',
				kdfparams: {
					dklen: 32,
					salt: 'c3cee502c7157e0faa42386c6d666116ffcdf093c345166c502e23bc34e6ba40',
					n: 4096,
					r: 8,
					p: 1
				},
				mac: '4b49574f3d3356fa0d04f73e07d5a2a6bbfdd185bedfa31f37f347bc98f2ef26'
			}
		]
	}, 'password')

	caver.wallet.add(decrypted)
	console.log(caver.wallet.getKeyring(decrypted.address))
}

testFunction()
```

Run the in your console.

```bash
$ node ./test.js
SingleKeyring {
	_address: '0x66391720b488a3fb2c7c69d99cd4cd6e23ca18e3',
	_key: PrivateKey { _privateKey: '0x{private key}' }
}
SingleKeyring {
	_address: '0xc02cec4d0346bf4124deeb55c5216a4138a40a8c',
	_key: PrivateKey { _privateKey: '0x{private key}' }
}
```

Looking at the output above, you can query your keyring from `caver.wallet` after adding it to `caver.wallet`.

If you have an address and private key(s) to use, you can easily create a keyring and add it directly to [caver.wallet] via [caver.wallet.newKeyring].

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	// Add to wallet with an address and a private key
	const addedSingle = caver.wallet.newKeyring('0x{address in hex}', '0x{private key1}')
	console.log(caver.wallet.getKeyring(addedSingle.address))

	// Add to wallet with an address and private keys
	const addedMultiple = caver.wallet.newKeyring('0x{address in hex}', ['0x{private key2}', '0x{private key3}', '0x{private key4}'])
	console.log(caver.wallet.getKeyring(addedMultiple.address))

	// Add to wallet with an address and private keys defined by each roles
	const addedRoleBased = caver.wallet.newKeyring('0x{address in hex}', [
		['0x{private key5}', '0x{private key6}', '0x{private key7}'],
		['0x{private key8}', '0x{private key9}'],
		['0x{private key10}', '0x{private key11}']
	])
	console.log(caver.wallet.getKeyring(addedRoleBased.address))
}

testFunction()
```

Running the above code gives you the following result. The result of the above code execution is shown below. When `caver.wallet.newKeyring` is executed with a private key, a Keyring instance with one private key is created and added to `caver.wallet`. For multiple private keys, a Keyring instance with multiple private keys is created. When passing one or more private keys for each role as arguments, a Keyring instance with a different private key(s) for each role is created and also added to the `caver.wallet`.

```bash
$ node ./test.js
SingleKeyring {
	_address: '0x651f6ae6b45750082b22805583acc989399c6552',
	_key: PrivateKey { _privateKey: '0x{private key1}' }
}
MultipleKeyring {
	_address: '0xce3ee92aeb4d600a41c98bdf92e8b337e186bf58',
	_keys: [ 
		PrivateKey { _privateKey: '0x{private key2}' },
		PrivateKey { _privateKey: '0x{private key3}' },
		PrivateKey { _privateKey: '0x{private key4}' }
    ]
}
RoleBasedKeyring {
	_address: '0x626d5b94ec76a105c5afa370bb7e59050a22b8b5',
	_keys: [ 
		[ 
			PrivateKey { _privateKey: '0x{private key5}' },
			PrivateKey { _privateKey: '0x{private key6}' },
			PrivateKey { _privateKey: '0x{private key7}' }
		],
		[ 
			PrivateKey { _privateKey: '0x{private key8}' },
			PrivateKey { _privateKey: '0x{private key9}' }
		],
		[ 
			PrivateKey { _privateKey: '0x{private key10}' },
			PrivateKey { _privateKey: '0x{private key11}' }
		]
	]
}
```

`caver.wallet.add` or `caver.wallet.newKeyring` returns a Keyring instance after adding it to `caver.wallet`.

## Sending a Transaction <a id="sending-a-transaction"></a>

This section will show you how to send KLAY using caver-js on the Baobab network.

### Getting KLAY via Baobab Faucet <a id="getting-klay-via-baobab-faucet"></a>

If you need KLAY for testing, you can get Baobab testnet KLAY from the [Klaytn Wallet](../../../toolkit/klaytn-wallet.md#how-to-receive-baobab-testnet-klay). Log in to the Klaytn Wallet using the private key or the keystore file and receive Baobab testnet KLAY via the faucet for testing.

### Sending a Value Transfer Transaction <a id="sending-a-value-transfer-transaction"></a>

You can use a caver-js wallet to generate a signature of a transaction. You have to go through two steps below to send the transaction to the network.

1. Sign a transaction
	- If the keyring you want to use is added to [caver.wallet], you can use `caver.wallet.sign` function to sign.
	- If you manage the keyring separately without adding it to `caver.wallet`, you can sign the transaction through `transaction.sign` function.
2. Send the RLP-encoded string of the signed transaction to the Klaytn via `caver.rpc.klay.sendRawTransaction`.

**Note:** The sender should have enough number of KLAY.

#### Sign a transaction

Before sending a transaction to Klaytn, you should sign a transaction first. 

Below is an example of how to sign a transaction if a keyring is added to the [caver.wallet].

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	// Add a keyring to caver.wallet
	const keyring = caver.wallet.keyring.createFromPrivateKey('0x{private key}')
	caver.wallet.add(keyring)

	// Create a value transfer transaction
	const valueTransfer = new caver.transaction.valueTransfer({
		from: keyring.address,
		to: '0x176ff0344de49c04be577a3512b6991507647f72',
		value: 1,
		gas: 30000,
	})

	// Sign the transaction via caver.wallet.sign
	await caver.wallet.sign(keyring.address, valueTransfer)

	const rlpEncoded = valueTransfer.getRLPEncoding()
	console.log(`RLP-encoded string: ${rlpEncoded}`)
}

testFunction()
```

The above code adds a keyring to `caver.wallet`, creates a transaction, and signs the transaction through `caver.wallet.sign`. 

Running the above code gives you the following result. When the above code is executed, the RLP-encoded string of the transaction is shown below. (The RLP-encoded string output you got could be different from the string output shown below.)

```bash
RLP-encoded string: 0x08f87e808505d21dba0082753094176ff0344de49c04be577a3512b6991507647f720194ade4883d092e2a972d70637ca7de9ab5166894a2f847f845824e44a0e1ec99789157e5cb6bc691935c204a23aaa3dc049efafca106992a5d5db2d179a0511c421d5e508fdb335b6048ca7aa84560a53a5881d531644ff178b6aa4c0a41
```

#### Send the RLP-encoded string of the signed transaction to the Klaytn

Now you can send a signed transaction to the network like below. If you want to run the below example, replace `0x{RLP-encoded string}` with the value of `rlpEncoded` above.

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	const rlpEncoding = `0x{RLP-encoded string}`

	// Send the transaction using `caver.rpc.klay.sendRawTransaction`.
	const receipt = await caver.rpc.klay.sendRawTransaction(rlpEncoding)
	console.log(receipt)
}

testFunction()
```

Running the above code gives you the following result. When the above code is executed, the receipt of the transaction is shown below.

```bash
$ node ./test.js
{ 
	blockHash: '0xd20066b448da77a41a46fbf0856792b85b60c42213126f661f6434b5b1263072',
	blockNumber: '0x1efb',
	contractAddress: null,
	from: '0x09a08f2289d3eb3499868908f1c84fd9523fe11b',
	gas: '0x7530',
	...
	signatures: [
		{ 
			V: '0x4e43',
			R: '0x5737aa8c88f019a3ee184faed6d34d103f77773bd5434cb0328c11738c8d9755',
			S: '0x578b118f4400999e5232bd0860cfbdbf89622f6e11cc6bd9722a86767d2723b7'
		}
	],
	status: '0x1',
	to: '0x176ff0344de49c04be577a3512b6991507647f72',
	transactionHash: '0x43e8ab1a2365ad598448b4402c1cfce6a71b3a103fce3a69905613e50b978113',
	transactionIndex: 0,
	type: 'TxTypeValueTransfer',
	typeInt: 8,
	value: '0x1'
}
```

If you want to sign a transaction and send it to the network without `caver.wallet`, see the example below.

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	// Create a value transfer transaction
	const keyring = caver.wallet.keyring.createFromPrivateKey('0x{private key}')
	const valueTransfer = new caver.transaction.valueTransfer({
		from: keyring.address,
		to: '0x176ff0344de49c04be577a3512b6991507647f72',
		value: 1,
		gas: 30000,
	})

	// Sign the transaction via transaction.sign
	await valueTransfer.sign(keyring)

	// Send the transaction to the Klaytn using `caver.rpc.klay.sendRawTransaction`.
	const receipt = await caver.rpc.klay.sendRawTransaction(valueTransfer)
	console.log(receipt)
}

testFunction()
```

When the above code is executed, the receipt of the transaction is printed like the previous example.

### Checking Receipts <a id="checking-receipts"></a>

You can use the promise or event emitter to get the receipt of the transaction when you transfer the transaction to the Klaytn by [caver.rpc.klay.sendRawTransaction].

The following example shows how to get a receipt using promises and event emitters.

```javascript
// Using a promise - async/await
const receipt = await caver.rpc.klay.sendRawTransaction(rawTransaction)
console.log(receipt)

// Using a promise
caver.rpc.klay.sendRawTransaction(rawTransaction).then(console.log)

// Using an event emitter
caver.rpc.klay.sendRawTransaction(rawTransaction).on('receipt', console.log)
```

As described in the example above, you can get the result of sending a transaction through the promise and event emitter. The `transactionHash` field is defined inside the receipt object. You can use [caver.rpc.klay.getTransactionReceipt] RPC call with `receipt.transactionHash` to query the receipt of a transaction at any time from the network after the transaction is included in a block. The example below shows how to get a receipt using the [caver.rpc.klay.getTransactionReceipt] RPC call.

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	const receipt = await caver.rpc.klay.getTransactionReceipt('0x40552efbba23347d36f6f5aaba6b9aeb6602e004df62c1988d9b7b1f036e676a')
	console.log(receipt)
}

testFunction()
```

Running the above code gives you the following result. When the above code is executed, the receipt of the transaction is shown below.

```bash
$ node ./test.js
{ 
	blockHash: '0x65d041011440e04643c546eb8bbb1dcabb659c3b3216e01473cb0712e47b5f69',
	blockNumber: '0x20db',
	contractAddress: null,
	from: '0x09a08f2289d3eb3499868908f1c84fd9523fe11b',
	gas: '0x7530',
	...
	signatures: [
		{ 
			V: '0x4e43',
			R: '0xfabe48071a8b72f0c340b2ee9d948a496cce467aebe027159d66a175e6b4b5b4',
			S: '0x1d4e503f1b084cda15edeba6b7b8eba15057b9d2484f7f3d095c980c2d98f13'
		}
	],
	status: '0x1',
	to: '0x176ff0344de49c04be577a3512b6991507647f72',
	transactionHash: '0x40552efbba23347d36f6f5aaba6b9aeb6602e004df62c1988d9b7b1f036e676a',
	transactionIndex: 0,
	type: 'TxTypeValueTransfer',
	typeInt: 8,
	value: '0x1'
}
```

The result of the transaction can be found through the `status` of the receipt. For the details of the return values, see [caver.rpc.klay.getTransactionReceipt]. If a transaction is failed, you can check more about the error in `txError` of the receipt. For more information about `txError`, see [txError: Detailed Information of Transaction Failures].

## Executing Other Transaction Types <a id="executing-other-transaction-types"></a>

Klaytn provides various transaction types for extensibility and performance. For more information, see [Transactions](../../../klaytn/design/transactions/README.md). This section describes some examples that can be used with caver-js.

### Fee Delegation <a id="fee-delegation"></a>

Klaytn provides [Fee Delegation] feature. Here's an example of making a RLP-encoded transaction when you are a sender of this kind of transaction:

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	const sender = caver.wallet.keyring.createFromPrivateKey('0x{private key}')
	caver.wallet.add(sender)

	const feeDelegatedTx = new caver.transaction.feeDelegatedValueTransfer({
		from: sender.address,
		to: '0x176ff0344de49c04be577a3512b6991507647f72',
		value: 5,
		gas: 50000,
	})

	await caver.wallet.sign(sender.address, feeDelegatedTx)

	const rlpEncoded = feeDelegatedTx.getRLPEncoding()
	console.log(rlpEncoded)
}

testFunction()
```

When the above code is executed, the RLP-encoded string will be printed. (The RLP-encoded string output you got could be different from the string output shown below.)

```bash
$ node ./test.js
0x09f884028505d21dba0082c35094176ff0344de49c04be577a3512b6991507647f720594f5a9079f311f9ec55170af351627aff0c5d2e287f847f845824e43a0f4b53dbd4c915cb73b9c7fa17e22106ee9640155a06ab4a7ed8661f846d2a5cca035b5bba6a26d4ccd20c65e8f31cce265c193f1c874806f9fae6b0ee9df0addf080c4c3018080
```

The fee payer can send the transaction to the Klaytn after attaching the `feePayerSignatures` to the RLP-encoded string (`rawTransaction`) signed by the transaction sender. If `caver.wallet` also has the fee payer's key, the fee payer's signature can be injected into `feeDelegatedTx` by calling `caver.wallet.signAsFeePayer(feePayer.address, feeDelegatedTx)`. Otherwise, the fee payer has to create a `feeDelegatedTx` from the RLP-encoded string signed by the sender and add the fee payer's sign onto it, as illustrated below. If you want to run the below example, replace `0x{RLP-encoded string}` with the value of `rlpEncoded` above.

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	const feePayer = caver.wallet.keyring.createFromPrivateKey('0x{private key}')
	caver.wallet.add(feePayer)

	const rlpEncoded = '0x{RLP-encoded string}'

	const feeDelegateTxFromRLPEncoding = new caver.transaction.feeDelegatedValueTransfer(rlpEncoded)

	// Set the fee payer address.
	feeDelegateTxFromRLPEncoding.feePayer = feePayer.address
	await caver.wallet.signAsFeePayer(feePayer.address, feeDelegateTxFromRLPEncoding)

	console.log(feeDelegateTxFromRLPEncoding.getRLPEncoding())
}

testFunction()
```

When the above code is executed, the RLP-encoded string including the sender's signatures and fee payer's signatures is printed like below. (The output you got could be different from the string output shown below.)

```bash
$ node ./test.js
0x09f8dc028505d21dba0082c35094176ff0344de49c04be577a3512b6991507647f720594f5a9079f311f9ec55170af351627aff0c5d2e287f847f845824e43a0f4b53dbd4c915cb73b9c7fa17e22106ee9640155a06ab4a7ed8661f846d2a5cca035b5bba6a26d4ccd20c65e8f31cce265c193f1c874806f9fae6b0ee9df0addf09417e7531b40ad5d7b5fa7b4ec78df64ce1cb36d24f847f845824e44a0921b7c3be69db96ce14134b306c2ada423613cb66ecc6697ee8067983c268b6ea07b86b255d1c781781315d85d7904226fb2101eb9498c4a03f3fbd30ba3ec5b79
```

The transaction is now signed by both the sender and the fee payer, and it can now be sent over the network. Replace `0x{RLP-encoded string}` with the RLP-encoded string output of the example code above.

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	const rlpEncoded = '0x{RLP-encoded string}'
	const receipt = await caver.rpc.klay.sendRawTransaction(rlpEncoded)
	console.log(receipt)
}

testFunction()
```

Running the above code gives you the following result. Through the execution result of the above code, you can check the FeeDelegatedValueTransfer transaction result.

```bash
$ node ./test.js
{ 
	blockHash: '0xb6a76163c4c558f50bdae77968a0f35dcfececf78b5cb780c3514a30a1c0a864',
	blockNumber: '0xede',
	contractAddress: null,
	feePayer: '0x17e7531b40ad5d7b5fa7b4ec78df64ce1cb36d24',
	feePayerSignatures: [
		{
			V: '0x4e44',
			R: '0x921b7c3be69db96ce14134b306c2ada423613cb66ecc6697ee8067983c268b6e',
			S: '0x7b86b255d1c781781315d85d7904226fb2101eb9498c4a03f3fbd30ba3ec5b79'
		}
	],
	from: '0xf5a9079f311f9ec55170af351627aff0c5d2e287',
	gas: '0xc350',
	...
	signatures: [
		{
			V: '0x4e43',
			R: '0xf4b53dbd4c915cb73b9c7fa17e22106ee9640155a06ab4a7ed8661f846d2a5cc',
			S: '0x35b5bba6a26d4ccd20c65e8f31cce265c193f1c874806f9fae6b0ee9df0addf0'
		}
	],
	status: '0x1',
	to: '0x176ff0344de49c04be577a3512b6991507647f72',
	transactionHash: '0x1878cc27b7f259a98d3248b41bffb6158640b4a07c503095deac1913fb3856c2',
	transactionIndex: 0,
	type: 'TxTypeFeeDelegatedValueTransfer',
	typeInt: 9,
	value: '0x5'
}
```

### Account Update <a id="account-update"></a>

If you want to change the private key(s) for your Klaytn account, there are 3 important things you need to remember:

1. Klaytn validates every transaction you send to it.
2. The validation requires your public keys which exactly corresponds to your private key(s).
3. Thus, changing your private key(s) into the new one(s) is **always be** **preceded** by changing your old public key(s) to the new one(s). The new public key(s) must be derived from the new private key(s).

Keeping the 3 things above in your mind, you can change your private key(s) by following the steps below:

1. Prepare the new private key(s) to create a new keyring.
2. Create a keyring by its type (Single keyring, Multiple keyring, or Role-based keyring) you need.
3. Generate an Account instance from the new keyring. This Account instance holds the new public key(s) for your Klaytn account.
4. Send AccountUpdate transaction including Account instance to Klaytn.
5. Finally, replace your old keyring to the new one that was created in Step 2.

Please check [Account Update] for the details.

To change your AccountKey, you must provide an [Account] instance for the `account` field in the input argument object of `caver.transaction.accountUpdate`. An [Account] instance contains the address of the Klaytn account and the AccountKey to be updated.

The code below is an example code that changes the private key(s) you use for your Klaytn account along with changing AccountKey of your Klaytn account to [AccountKeyPublic]. Don't forget to prepare your new private key(s).

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	let sender = caver.wallet.keyring.createFromPrivateKey('0x{private key}')
	caver.wallet.add(sender)

	const newPrivateKey = caver.wallet.keyring.generateSingleKey()
	console.log(`new private key string: ${newPrivateKey}`)
	const newKeyring = caver.wallet.keyring.createWithSingleKey(sender.address, newPrivateKey)

	// create an Account instance
	const account = newKeyring.toAccount()

	const updateTx = new caver.transaction.accountUpdate({
		from: sender.address,
		account: account,
		gas: 50000,
	})
	await caver.wallet.sign(sender.address, updateTx)
	const receipt = await caver.rpc.klay.sendRawTransaction(updateTx)
	console.log(receipt)

	// Update the keyring in caver.wallet for signing afterward.
	sender = caver.wallet.updateKeyring(newKeyring)
}

testFunction()
```

If the above code is executed successfully, you no longer are able to use the old private key(s) to sign any transaction with the old keyring. So you must update the old keyring with the `newKeyring` through `caver.wallet.updateKeyring(newKeyring)`. Once it is updated, the signing will be done by the newly updated private key(s).

Running the above code gives you the following result. In the execution result of the above code, the result of the private key and the account update that you should newly use are printed like below.

```bash
$ node ./test.js
new private key string: 0x{private key}
{ 
	blockHash: '0x4c0221245e7c810cc19b05257e8d7cd34f24cc829f8787a832c08682640173f5',
	blockNumber: '0x26d6',
	contractAddress: null,
	from: '0xeec694a4143e05945823b216d0c62ab91c192a63',
	gas: '0xc350',
	gasPrice: '0x5d21dba00',
	gasUsed: 41000,
	key: '0x02a1024cc461670797071be16c34b22df1a3588653da5c1e9279b1d9e4b24fbcba07d8',
	...
	signatures: [
		{
			V: '0x4e43',
			R: '0xd0fa2d25711de4bfc3a7a6a660d307264fa3b2cacbb7eb71ab68f47661ebcfaf',
			S: '0x4652102241e61968988a22f9fa2d5d38d4e654d1f4b193fba5627c0856c9da7b'
		} 
	],
	status: '0x1',
	transactionHash: '0x4efdeeb1bb1e52ace11d64a19f564a973b36c29a0d85899a215621659b793665',
	transactionIndex: 0,
	type: 'TxTypeAccountUpdate',
	typeInt: 32
}
```


Here comes how to update AccountKey of your Klaytn account with multiple [AccountKeys]? The example below explains how to create an [Account] instance with multiple private keys that what you want to use (You can create an [Account] instance with multiple public keys via [caver.account.create]). Same again, after feeding the account instance created to the `account` field inside the transaction object, the left rest of the updating process is just the same as the above example.

First, let's create an Account instance to update with [AccountKeyWeightedMultiSig]. For [AccountKeyWeightedMultiSig], a threshold and a weight for each key must be defined. To do this, use [caver.account.weightedMultiSigOptions]. The first parameter is the threshold, and the second parameter is an array containing the weight for each key.

```javascript
// Create an account instance with three private keys using AccountKeyWeightedMultiSig
const newPrivateKeys = caver.wallet.keyring.generateMultipleKeys(3)
const newKeyring = caver.wallet.keyring.createWithMultipleKey(sender.address, newPrivateKeys)

// threshold = 3, the weights of the three keys = [1, 2, 1]
const options = new caver.account.weightedMultiSigOptions(3, [1, 2, 1])

const account = newKeyring.toAccount(options)
```

Now let's update AccountKey using [AccountKeyRoleBased]. [AccountKeyRoleBased] is an `AccountKey` type that defines the key to use for each [role].

```javascript
// Create an account instance with roles using AccountKeyRoleBased. In the account instance created, each role has a public key that corresponds to one private key.
const newPrivateKeys = caver.wallet.keyring.generateRoleBasedKeys([1, 1, 1])
const newKeyring = caver.wallet.keyring.createWithRoleBasedKey(sender.address, newPrivateKeys)

const account = newKeyring.toAccount()
```

The AccountKeyRoleBased above is an example of using one public key for each role. As you can see from the code above, each of them corresponds to one private key. If you want to use multiple private keys for each role, [caver.account.weightedMultiSigOptions] must be defined for each role as shown below.

```javascript
// Create an account instance with [3, 2, 3] keys for each role using AccountKeyRoleBased
const newPrivateKeys = caver.wallet.keyring.generateRoleBasedKeys([3, 2, 3])
const newKeyring = caver.wallet.keyring.createWithRoleBasedKey(sender.address, newPrivateKeys)

const options = [
	// thresold = 4, weights of keys = [2, 2, 4] for roleTransactionKey
	new caver.account.weightedMultiSigOptions(4, [2, 2, 4]),
	// threshold = 2, weights of keys = [1, 1]
	new caver.account.weightedMultiSigOptions(2, [1, 1]),
	// threshold = 3, weights of keys = [1, 1, 1]
	new caver.account.weightedMultiSigOptions(3, [1, 1, 1]),
]

const account = newKeyring.toAccount(options)
```

If you want to update AccountKey to [AccountKeyLegacy] or [accountKeyFail], create an Account instance as shown below and assign it to the `account` field of the transaction.

```javascript
// Create an account with AccountKeyLegacy
const accountWithLegacyKey = caver.account.createWithAccountKeyLegacy(keyringToUpdate.address)

// Create an account with AccountKeyFail
const accountWithFailKey = caver.account.createWithAccountKeyFail(keyringToUpdate.address)
```

### Smart Contract <a id="smart-contract"></a>

The [caver.contract] package makes it easy to interact with smart contracts on Klaytn. It automatically converts all methods of a smart contract into javascript calls when its low-level ABI \(Application Binary Interface\) is given. This allows you to interact with smart contracts as if they were JavaScript objects.

First, we make a simple solidity example like the below. Create a 'test.sol' file and write down the below example.

```
pragma solidity ^0.5.6;

contract KVstore {
    mapping(string=>string) store;
    function get(string memory key) public view returns (string memory) {
        return store[key];
    }
    function set(string memory key, string memory value) public {
        store[key] = value;
    }
}
```

Now we can compile a smart contract to get its bytecode and ABI.

```text
> solc --abi --bin ./test.sol
======= ./test.sol:KVstore =======
Binary: 
608060405234801561001057600080fd5b5061051f806100206000396000f3fe608060405234801561001057600080fd5b50600436106100365760003560e01c8063693ec85e1461003b578063e942b5161461016f575b600080fd5b6100f46004803603602081101561005157600080fd5b810190808035906020019064010000000081111561006e57600080fd5b82018360208201111561008057600080fd5b803590602001918460018302840111640100000000831117156100a257600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f8201169050808301925050505050505091929192905050506102c1565b6040518080602001828103825283818151815260200191508051906020019080838360005b83811015610134578082015181840152602081019050610119565b50505050905090810190601f1680156101615780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b6102bf6004803603604081101561018557600080fd5b81019080803590602001906401000000008111156101a257600080fd5b8201836020820111156101b457600080fd5b803590602001918460018302840111640100000000831117156101d657600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f8201169050808301925050505050505091929192908035906020019064010000000081111561023957600080fd5b82018360208201111561024b57600080fd5b8035906020019184600183028401116401000000008311171561026d57600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f8201169050808301925050505050505091929192905050506103cc565b005b60606000826040518082805190602001908083835b602083106102f957805182526020820191506020810190506020830392506102d6565b6001836020036101000a03801982511681845116808217855250505050505090500191505090815260200160405180910390208054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156103c05780601f10610395576101008083540402835291602001916103c0565b820191906000526020600020905b8154815290600101906020018083116103a357829003601f168201915b50505050509050919050565b806000836040518082805190602001908083835b6020831061040357805182526020820191506020810190506020830392506103e0565b6001836020036101000a0380198251168184511680821785525050505050509050019150509081526020016040518091039020908051906020019061044992919061044e565b505050565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061048f57805160ff19168380011785556104bd565b828001600101855582156104bd579182015b828111156104bc5782518255916020019190600101906104a1565b5b5090506104ca91906104ce565b5090565b6104f091905b808211156104ec5760008160009055506001016104d4565b5090565b9056fea165627a7a723058203ffebc792829e0434ecc495da1b53d24399cd7fff506a4fd03589861843e14990029
Contract JSON ABI 
[{"constant":true,"inputs":[{"name":"key","type":"string"}],"name":"get","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"key","type":"string"},{"name":"value","type":"string"}],"name":"set","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"}]
```

**NOTE**: To compile a smart contract, you must have a [solidity compiler](https://solidity.readthedocs.io/en/develop/installing-solidity.html) installed.

For the smart contract deployment, you can use [caver.contract] to deploy it, or you can deploy it using [caver.transaction.smartContractDeploy], [caver.transaction.feeDelegatedSmartContractDeploy] or [caver.transaction.feeDelegatedSmartContractDeployWithRatio] transaction. Here is an example of using [caver.contract].

You can create a contract instance as below using the result of compiling the smart contract.

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	const abi = [{"constant":true,"inputs":[],"name":"count","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getBlockNumber","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_count","type":"uint256"}],"name":"setCount","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"}]
	const contractInstance = new caver.contract(abi)
	console.log(contractInstance)
	console.log(contractInstance.options.address)
}

testFunction()
```

Running the above code gives you the following result.

```bash
$ node ./test.js
Contract {
	...
  methods: {
		get: [Function: bound _createTxObject],
		'0x693ec85e': [Function: bound _createTxObject],
		'get(string)': [Function: bound _createTxObject],
		set: [Function: bound _createTxObject],
		'0xe942b516': [Function: bound _createTxObject],
		'set(string,string)': [Function: bound _createTxObject]
	},
  events: { allEvents: [Function: bound ] },
  _address: null,
  _jsonInterface: [ ... ],
  _keyrings: KeyringContainer { ... }
}
null
```

Looking at the output above, you can see that the methods are managed through abi inside the Contract instance. And since it hasn't been deployed yet, you can see that the result of `contractInstance.options.address` is output as null.

If the smart contract has already been deployed and you know the contract address where the smart contract was deployed, please pass the contract address to the second parameter as shown below.

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	const abi = [{"constant":true,"inputs":[{"name":"key","type":"string"}],"name":"get","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"key","type":"string"},{"name":"value","type":"string"}],"name":"set","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"}]
	
	const contractInstance = new caver.contract(abi, '0x3466D49256b0982E1f240b64e097FF04f99Ed4b9')

	console.log(contractInstance)
	console.log(contractInstance.options.address)
}

testFunction()
```

Running the above code gives you the following result.

```bash
$ node ./test.js
Contract {
	...
  methods: {
		get: [Function: bound _createTxObject],
		'0x693ec85e': [Function: bound _createTxObject],
		'get(string)': [Function: bound _createTxObject],
		set: [Function: bound _createTxObject],
		'0xe942b516': [Function: bound _createTxObject],
		'set(string,string)': [Function: bound _createTxObject]
	},
  events: { allEvents: [Function: bound ] },
  _address: '0x3466D49256b0982E1f240b64e097FF04f99Ed4b9',
  _jsonInterface: [ ... ],
  _keyrings: KeyringContainer { ... }
}
0x3466D49256b0982E1f240b64e097FF04f99Ed4b9
```

Since this contract instance received the address of the smart contract, it stores the contract address in `contractInstance.options.address`.

If the contract instance is created, you can deploy it by passing the bytecode to the `data` field as shown below.

Note that [caver.contract] sends transactions for deployment and execution. It uses keyrings in `caver.wallet` to sign transactions. The keyring to be used must be added to `caver.wallet` before.

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	const deployer = caver.wallet.keyring.createFromPrivateKey('0x{private key}')
	caver.wallet.add(deployer)
	
	const abi = [{"constant":true,"inputs":[{"name":"key","type":"string"}],"name":"get","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"key","type":"string"},{"name":"value","type":"string"}],"name":"set","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"}]
	const contractInstance = new caver.contract(abi)
	
	const deployedInstance = await contractInstance.deploy({
		data:  '608060405234801561001057600080fd5b5061051f806100206000396000f3fe608060405234801561001057600080fd5b50600436106100365760003560e01c8063693ec85e1461003b578063e942b5161461016f575b600080fd5b6100f46004803603602081101561005157600080fd5b810190808035906020019064010000000081111561006e57600080fd5b82018360208201111561008057600080fd5b803590602001918460018302840111640100000000831117156100a257600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f8201169050808301925050505050505091929192905050506102c1565b6040518080602001828103825283818151815260200191508051906020019080838360005b83811015610134578082015181840152602081019050610119565b50505050905090810190601f1680156101615780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b6102bf6004803603604081101561018557600080fd5b81019080803590602001906401000000008111156101a257600080fd5b8201836020820111156101b457600080fd5b803590602001918460018302840111640100000000831117156101d657600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f8201169050808301925050505050505091929192908035906020019064010000000081111561023957600080fd5b82018360208201111561024b57600080fd5b8035906020019184600183028401116401000000008311171561026d57600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f8201169050808301925050505050505091929192905050506103cc565b005b60606000826040518082805190602001908083835b602083106102f957805182526020820191506020810190506020830392506102d6565b6001836020036101000a03801982511681845116808217855250505050505090500191505090815260200160405180910390208054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156103c05780601f10610395576101008083540402835291602001916103c0565b820191906000526020600020905b8154815290600101906020018083116103a357829003601f168201915b50505050509050919050565b806000836040518082805190602001908083835b6020831061040357805182526020820191506020810190506020830392506103e0565b6001836020036101000a0380198251168184511680821785525050505050509050019150509081526020016040518091039020908051906020019061044992919061044e565b505050565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061048f57805160ff19168380011785556104bd565b828001600101855582156104bd579182015b828111156104bc5782518255916020019190600101906104a1565b5b5090506104ca91906104ce565b5090565b6104f091905b808211156104ec5760008160009055506001016104d4565b5090565b9056fea165627a7a723058203ffebc792829e0434ecc495da1b53d24399cd7fff506a4fd03589861843e14990029',
	}).send({
		from: deployer.address,
		gas: '0x4bfd200',
		value: '0x0',
	})
	console.log(deployedInstance)
	console.log(deployedInstance.options.address)
}

testFunction()
```

In the code above, the `deployer` deploys the contract to the Klaytn and returns the deployed contract instance.

```bash
$ node ./test.js
Contract {
	...
  methods: {
		get: [Function: bound _createTxObject],
		'0x693ec85e': [Function: bound _createTxObject],
		'get(string)': [Function: bound _createTxObject],
		set: [Function: bound _createTxObject],
		'0xe942b516': [Function: bound _createTxObject],
		'set(string,string)': [Function: bound _createTxObject]
	},
  events: { allEvents: [Function: bound ] },
  _address: '0x3466D49256b0982E1f240b64e097FF04f99Ed4b9',
  _jsonInterface: [ ... ],
  _keyrings: KeyringContainer { ... }
}
0x3466D49256b0982E1f240b64e097FF04f99Ed4b9
```

One way to invoke a specific method of a smart contract is to use it with `caver.contract` or use [caver.transaction.smartContractExecution], [caver.transaction.feeDelegatedSmartContractExecution] or [caver.transaction.feeDelegatedSmartContractExecutionWithRatio] transaction.

To transact with a smart contract:

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	const keyring = caver.wallet.keyring.createFromPrivateKey('0x{private key}')
	caver.wallet.add(keyring)

	const abi = [{"constant":true,"inputs":[{"name":"key","type":"string"}],"name":"get","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"key","type":"string"},{"name":"value","type":"string"}],"name":"set","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"}]
	
	const contractInstance = new caver.contract(abi, '0x{address in hex}')
	const receipt = await contractInstance.methods.set('testKey', 'testValue').send({ from:keyring.address, gas:'0x4bfd200' })
	console.log(receipt)
}

testFunction()
```

When the above code is executed, the transaction result from executing `set` arrives as below.

```bash
$ node ./test.js
{ 
	blockHash: '0x610336d43644abc5ab71156f7334ff67deabdd8de27778faa9dec99d225927e6',
  blockNumber: 4724,
  contractAddress: null,
  from: '0xbbfa9e3f76ddafedc28197e0f893366dd3c5c74a',
  gas: '0x4bfd200',
  gasPrice: '0x5d21dba00',
  gasUsed: 62351,
  input: '0xe942b...',
  ...
  status: true,
  to: '0x3466d49256b0982e1f240b64e097ff04f99ed4b9',
  transactionHash: '0x3a354703ab4a7b32492edab454b446dd3e92eec81ecbdaf2c3d84ffdd5cf9948',
  transactionIndex: 0,
  type: 'TxTypeSmartContractExecution',
  typeInt: 48,
  value: '0x0',
  events: {}
}
```

To load a contract instance and call one of its functions:

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	const abi = [{"constant":true,"inputs":[],"name":"count","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getBlockNumber","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_count","type":"uint256"}],"name":"setCount","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"}]
	const contractInstance = new caver.contract(abi, '0x{smart contract address}')

	const value = await contractInstance.methods.get('testKey').call()
	console.log(value)
}

testFunction()
```

When the above code is executed, the value is shown as an output below.

```bash
$ node ./test.js
testValue
```

To find more information, see [caver.contract].

## Sending a Transaction with multiple signers<a id="sending-a-transaction-with-multiple-signers"></a>

If the Klaytn account's AccountKey is AccountKeyMultiSig or AccountKeyRoleBased, the person who manages each key can vary.

This section describes how to collect signatures and send the transaction if there are multiple signers.

To run this example, you need to update AccountKey of the Klaytn account you use for testing with [AccountKeyWeightedMultiSig]. Please refer to [Account Update](#account-update) for how to update your Klaytn account.

### Signing sequentially <a id="signing-sequentially"></a>

When a transaction is signed using `caver.wallet` or the transaction's `sign` function, signatures (or feePayerSignatures) are defined (or appended) inside the transaction. You can obtain the RLP-encoded string (`rawTransaction`) containing the signatures (and feePayerSignatures) by calling the `transaction.getRLPEncoding()` function of the signed transaction instance.

The following example shows how to sign a transaction sequentially with multiple private keys.
Let's assume that AccountKey of the account who sends this transaction is AccountKeyWeightedMultiSig of two public keys, which means this Klaytn account can use two private key strings, one private key for each user. This is a case that two users share the same Klaytn account.

In the example below, user1 and user2 create a `Keyring` instances to be used. After that, each uses its own keyring to sign the transaction. The example below uses `transaction.sign` to sign it.

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	const user1 = caver.wallet.keyring.createWithSingleKey('0x{address in hex}', '0x{private key1}')
	const user2 = caver.wallet.keyring.createWithSingleKey('0x{address in hex}', '0x{private key2}')

	const transaction = new caver.transaction.valueTransfer({
		from: user1.address,
		to: '0x45c2a1e3a1c3957a06dae73ad516461c2d2c7ccc',
		value: 1,
		gas: 70000,
	})

	await transaction.sign(user1)
	console.log(transaction.signatures)

	await transaction.sign(user2)
	console.log(transaction.signatures)
}

testFunction()
```

Running the above code gives you the following result. Looking at the execution result of the code above, if user1 signs, one signature is created. If user2 signs, user2's signature is appended. [SignatureData] is an object that stores a signature.

```bash
$ node ./test.js
[ 
	SignatureData { _v: '0x4e43', _r: '0x3f3d3...', _s: '0x24f94...' }
]
[ 
	SignatureData { _v: '0x4e43', _r: '0x3f3d3...', _s: '0x24f94...' },
	SignatureData { _v: '0x4e44', _r: '0xd6a94...', _s: '0x72dc8...' }
]
```

Then let's see how to sign sequentially without sharing the same transaction object. In the below example, user1 passes RLP-encoded string that is the result of getRLPEncoding function of the signed transaction to user2.

The code below explains how to sign and append signatures with RLP-encoded string.

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	// Create user1's keyring
	const user1 = caver.wallet.keyring.createWithSingleKey('0x{address in hex}', '0x{private key1}')
	
	// Create a value transfer transaction
	const transaction = new caver.transaction.valueTransfer({
		from: user1.address,
		to: '0x45c2a1e3a1c3957a06dae73ad516461c2d2c7ccc',
		value: 1,
		gas: 70000,
	})
	
	// Sign the transaction
	await transaction.sign(user1)

	// Create user2's keyring
	const user2 = caver.wallet.keyring.createWithSingleKey('0x{address in hex}', '0x{private key2}')

	// Create a value transfer transaction from the RLP-encoded string
	const rlpEncoding = transaction.getRLPEncoding()
	const transactionFromRLP = new caver.transaction.valueTransfer(rlpEncoding)

	await transactionFromRLP.sign(user2)
	console.log(transactionFromRLP.signatures)
}

testFunction()
```

Running the above code gives you the following result.

```bash
$ node ./test.js
[ 
	SignatureData { _v: '0x4e43', _r: '0x3f3d3...', _s: '0x24f94...' },
	SignatureData { _v: '0x4e44', _r: '0xd6a94...', _s: '0x72dc8...' }
]
```

If you run the above code, you can see that user2's signature has been appended in `transactionFromRLP.signatures` and a total of two signatures are included in it.

When all users have signed, send a transaction to the network through `await caver.rpc.klay.sendRawTransaction(transactionFromRLP)`.

If you send a fee-delegated transaction, and the fee payer uses multiple keys, you can proceed with the above logic using `caver.wallet.signAsFeePayer`.

### Combining signed raw transactions <a id="combining-signed-rawtransactions"></a>

If you receive multiple signed RLP-encoded raw transaction strings from several people, you can combine them into a single RLP-encoded raw transaction string that contains all the signatures.

The example below shows how to combine and send the RLP encoded transactions.

```javascript
// test.js
const Caver = require('caver-js')
const caver = new Caver('https://api.baobab.klaytn.net:8651/')

async function testFunction() {
	const vt = new caver.transaction.valueTransfer({
		from: '0x0fa355263f37f5a54d9179452baa8b63b8b2cdde',
		to: '0x45c2a1e3a1c3957a06dae73ad516461c2d2c7ccc',
		value: 1,
		gas: 70000,
	})
	const rlpEncodedStrings = [
		'0x08f87f018505d21dba00830111709445c2a1e3a1c3957a06dae73ad516461c2d2c7ccc01940fa355263f37f5a54d9179452baa8b63b8b2cddef847f845824e44a01aa72b883ca540c8a63de244cd061ec4f9efb139541e8db304c07ec27bc9d272a06a4ca54f6269f2ddfe3648eb9ed57b0c5739f0077e1a38449f3ae3cc0b20dc3e',
		'0x08f8c6018505d21dba00830111709445c2a1e3a1c3957a06dae73ad516461c2d2c7ccc01940fa355263f37f5a54d9179452baa8b63b8b2cddef88ef845824e44a01aa72b883ca540c8a63de244cd061ec4f9efb139541e8db304c07ec27bc9d272a06a4ca54f6269f2ddfe3648eb9ed57b0c5739f0077e1a38449f3ae3cc0b20dc3ef845824e43a0fd76dfc53c812ec6aa860076f731e3913936088a1518cc34f2d176bcbe0ac772a071491c938458fffe106dde485fc8b26cbebe8a517c46bd185b126930f480d773',
		'0x08f8c6018505d21dba00830111709445c2a1e3a1c3957a06dae73ad516461c2d2c7ccc01940fa355263f37f5a54d9179452baa8b63b8b2cddef88ef845824e44a01aa72b883ca540c8a63de244cd061ec4f9efb139541e8db304c07ec27bc9d272a06a4ca54f6269f2ddfe3648eb9ed57b0c5739f0077e1a38449f3ae3cc0b20dc3ef845824e43a021e84a4740b374cdcf0cc38f93225f6d2f77388a9d90302d47b4f3ed84e4db5fa072ff5e77d2506d5222081c4d2a341c6ee5d258500030564f985951472f247b7d',
	]
	const combined = vt.combineSignedRawTransactions(rlpEncodedStrings)
	console.log(combined)
}

testFunction()
```

Running the above code gives you the following result.

```bash
$ node ./test.js
0x08f9010d808505d21dba00830111709445c2a1e3a1c3957a06dae73ad516461c2d2c7ccc01940fa355263f37f5a54d9179452baa8b63b8b2cddef8d5f8458207f5a094ce13c39d25d44ad1d07ba2fd89f476c4dc6eef6071a2ef1f496f9b04d049e5a00f7abddd548998b0a55e53600a48286c38262fffc6c153a64e8f65a77c11c722f8458207f6a0ceeea7287b2670719d8ac15cf3b21b36fcaf74d58cce99935ce17e100064037aa0499067788d5db5e7c09ed7bfe19764d66684abc06b81e8f54ea254377bc81066f8458207f5a05c3ba89336c7d84d4ce08104cfd6f7ef33cd9f29766a1955baae8bcf964fd50fa015accbbce6bb11847a3b0660491775d64ef6d692ea709b768f64f12968c09240
```

Running the code above outputs one RLP-encoded raw transaction string with all the signature information combined.

When executing `combineSignedRawTransactions `, the signed RLP-encoded raw transaction strings to be combined must be exactly the same to each other except the signatures and the optional variables in the transaction instance. Optional variables without any given value in the base transaction instance (the caller of `combineSignedRawTransactions`) will be redeemed with the corresponding ones in the following raw transaction string to be merged right next. If there is any inconsistency among all raw transaction strings including the values of optional variables of them to be merged, an error occurs.

The combineSignedRawTransactions returns an RLP-encoded string containing all signatures (and feePayerSignatures if the transaction is a fee-delegated transaction) as a result. You use this to send a transaction to the network through `await caver.rpc.klay.sendRawTransaction(combined)`.


## Sample Projects <a id="sample-projects"></a>

The BApp \(Blockchain Application\) Development sample projects using caver-js are the following:

* [Count BApp](../../tutorials/count-bapp/README.md)
* [Klaystagram](../../tutorials/klaystagram/README.md)

## Links <a id="links"></a>

* caver-js [GitHub repository](https://github.com/klaytn/caver-js)
* caver-js on [npm](https://www.npmjs.com/package/caver-js)



[caver.contract]: api-references/caver.contract.md

[caver.account]: api-references/caver.account.md
[Account]: api-references/caver.account.md
[AccountKey]: ../../../klaytn/design/accounts.md#account-key
[AccountKeyLegacy]: ../../../klaytn/design/accounts.md#accountkeylegacy
[AccountKeyPublic]: ../../../klaytn/design/accounts.md#accountkeypublic
[AccountKeyFail]: ../../../klaytn/design/accounts.md#accountkeyfail
[AccountKeyWeightedMultiSig]: ../../../klaytn/design/accounts.md#accountkeyweightedmultisig
[AccountKeyRoleBased]: ../../../klaytn/design/accounts.md#accountkeyrolebased
[role]: ../../../klaytn/design/accounts.md#roles
[caver.account.weightedMultiSigOptions]: api-references/caver.account.md#weightedmultisigoptions
[caver.account.create]: api-references/caver.account.md#caver-account-create

[caver.wallet]: api-references/caver.wallet.md
[caver.wallet.newKeyring]: api-references/caver.wallet.md#newkeyring
[caver.wallet.keyring]: api-references/caver.wallet.keyring.md
[Keyring]: api-references/caver.wallet.keyring.md
[SingleKeyring]: api-references/caver.wallet.keyring.md#singlekeyring
[MultipleKeyring]: api-references/caver.wallet.keyring.md#multiplekeyring
[RoleBasedKeyring]: api-references/caver.wallet.keyring.md#rolebasedkeyring
[KlaytnWalletKey]: ../../../klaytn/design/accounts.md#klaytn-wallet-key-format
[SignatureData]: api-references/caver.wallet.keyring.md#signaturedata

[caver.rpc.klay.getTransactionReceipt]: api-references/caver.rpc/klay.md#gettransactionreceipt
[getTransactionReceipt]: api-references/caver.rpc/klay.md#gettransactionreceipt
[caver.rpc.klay.sendRawTransaction]: api-references/caver.rpc/klay.md#sendrawtransaction

[txError: Detailed Information of Transaction Failures]: ../../json-rpc/transaction-error-codes.md

[Fee Delegation]: ../../../klaytn/design/transactions/README.md#fee-delegation
[Account Update]: api-references/caver.transaction.md#accountupdate
[caver.transaction.smartContractDeploy]: api-references/caver.transaction.md#smartcontractdeploy
[caver.transaction.feeDelegatedSmartContractDeploy]: api-references/caver.transaction.md#feedelegatedsmartcontractdeploy
[caver.transaction.feeDelegatedSmartContractDeployWithRatio]: api-references/caver.transaction.md#feedelegatedsmartcontractdeploywithratio
[caver.transaction.smartContractExecution]: api-references/caver.transaction.md#smartcontractexecution
[caver.transaction.feeDelegatedSmartContractExecution]: api-references/caver.transaction.md#feedelegatedsmartcontractexecution
[caver.transaction.feeDelegatedSmartContractExecutionWithRatio]: api-references/caver.transaction.md#feedelegatedsmartcontractexecutionwithratio