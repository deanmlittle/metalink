# Metalink
**A proposal for better fullstack apps and wallets on BSV.**

The metalink proposal is made up of three main parts: 
1. Metalink ID
2. BSVABI
3. Unplanaria

### 1. Metalink ID
Metalink ID is a decentralised, pseudonymous ID system, using TXIDs to represent unique identities with both an owner and active key, like so:
``` json
{
    "o": "<public key 1>",
    "a": "<public key 2>"
}
```
By publishing a TXID with this JSON schema in the first output after an `OP_0 OP_RETURN`, we now have the bare minimum implementation of a unique identity system with verifiable, on-chain data. By storing public keys in TXs, we end up with a far superior solution to relying upon any individual identity provider, or systems like DNS, as identities can now be immutable, decentralised, cryptographically verifiable, enforcably unique, uniquely addressable and infinitely scalable. 

Here's a worked example for the Metalink ID [6dbffa521cd7c63aab630ee5b458f998eb6152f81f23fb571cf36094fb663b1f](https://whatsonchain.com/tx/6dbffa521cd7c63aab630ee5b458f998eb6152f81f23fb571cf36094fb663b1f)

Here, we have taken the following pair of owner/active keys and put them on chain:

```json
{
    "o":"038eaed0ec5a92b5d7480a8a790c8d3e4beae363e4da733b6d98a3697500be9f4b",
    "a":"0277d75325ec3a251165bd15fdb7c8919621ce5b13b965da399011ffe8315ac5d6"
}
```

Now that the Metalink ID is on chain, it's possible for anyone to serve it up off-chain securely with SPV proofs, meaning anyone can now consume it without ever having to go back to chain to check that it exists, provided they just keep a copy of the block headers. Here's an example of an SPV proof for the above Metalink ID:

```js
{
    rawtx: "01000000011d65721d6d5e6f0e6db39635be5bb7437412d07be7db1c73607f3363ac204289010000006a473044022041f43a3c9fb2ce3614955176843d60bb721ba33827718ab6bc90b160b7b6ead102204db54582e0af66b534215691dbc288b57c06a865fc6aa12b158a41b88f23dd01412102844d5867139ecf1d898863e1e58fa61a61aaafc80f98b2699ea2a1335743b008ffffffff02000000000000000097006a4c937b226f223a22303338656165643065633561393262356437343830613861373930633864336534626561653336336534646137333362366439386133363937353030626539663462222c2261223a22303237376437353332356563336132353131363562643135666462376338393139363231636535623133623936356461333939303131666665383331356163356436227d55280000000000001976a914e2a1c0b7da0612e457b08a19253f9ca8424ceab788ac00000000",
    txid: "6dbffa521cd7c63aab630ee5b458f998eb6152f81f23fb571cf36094fb663b1f",
    branches: [
        "e6c80714e945e65cd29e041d41068877d3b60918994cc94428b34ddf67d32b8f",
        "995cdc1c5eb1d7eeb0082b4d5e3efa94cfe32b1a1abcdbc6cc685379b49927d8",
        "a03bb64a811b09d70e2514793b348683c7474d7df1df37a0addf93b0c2b36379",
        "daa4d66a43468093af7ec90da332da4253fd17054c0e63087780da7bcbc87f6e",
        "2dfad80ebdd32dba310e2215d335bef47dc854de4c8f7fbf72ddd2e89b452d40",
        "20537bfd3ead14f5a4cd1b9ce3f8a71359b6fbc54b8861f04dc1c2c7bc11b2d9",
        "b6a729151fa40e0e39adfebbadaccde0529bec1ddef55f44e1fe6a3327ae7588",
        "e79560f199171a748db1ee4ee9a9c714ed38007404691f4335484765ca3fdd64",
        "7ad59e46abafb973b6a76eddb60555417f62c88db90ff83559f1b725c3a3ed3c",
        "ce6ca14091812da9b8a3b9110f81dea7831b125b6eb0ce71750fa66f4ef24ded",
        "3b2feb13859e9c8399ec9ad92c585e80c8a7d257fe9317aa555ead81884547e8"
    ],
    merkleRoot: "97e7d6256b9642bf998c9d1c954a98032be325b71e400f42e425ef84bcd613b5",
    block: "0000000000000000029e1f8782f8470965513f898f5564b561daf32fdd99dc42"
}
```

You can use the following script to [verify the Merkle root of an SPV proof](https://www.bitpaste.app/tx/49c518a597f4704e943913db1936a924465832484358f92fe4642204d5dd51f9). While there may be some cases where apps still want/need to verify SPV proofs for Metalink IDs, by using them in conjunction with BSVABI, we can actually enable complete separation of concerns by outsourcing this task to wallets. This comes at no incremental cost to wallets, as they will already be maintaining their own BIP270 infrastructure and thus a copy of the block headers.

When using a Metalink-enabled wallet, you can import the TXID and active key for your identity, and save your owner key somewhere safe. In doing so, your wallet can now `sign`, `login`, `encrypt`, `decrypt`, `verify` and `send` on behalf of your active key, but does not have a way of cryptographically proving it is the owner, insulating the owner of the identity from a myriad of potential attack vectors and removing the need to trust your wallet with ultimate ownership of your identity. An active key is used to interact with Metalink-enabled Bitcoin apps for all functions other than changing ownership.

Should a user wish to migrate to a new Metalink ID, they may tell this to each service they use individually, or through a syndication network, by signing with their owner key. Alternatively, if the user is well-known to the service, they may also choose to migrate identities based upon ***non-cryptographic*** evidence of ownership. Such a system is sufficiently robust to serve as the basis for any auxiliary functions of online identity, such as reputation, ownership of property, social security, licensing, or KYC, without compromising the principle of keyholder sovereignty in cases where this is not required.

Although this is how the system works on its most basic level, it is also possible for apps and wallets to build their own abstraction layers on top of it so that they may offer users custodial identity services tied to an email/password, paymail, handle, social media account, or phone number, enabling us to cater to any experience level, from beginners all the way up to power-users.

### 2. BSVABI
ABI, or Application Binary Interface, is a term referring to a schema that tells computers where they can look in machine code to execute certain functions, and what the expected inputs of those functions will be. Implementing an ABI system in Bitcoin enables us to solve a plethora of problems, the most important being the **"every/any problem"**; the painful dichotomy of having to choose between the poor of UX of asking the user to authorise **every** single action, or the poor security of having them authorise **any** action up to a certain value.

BSVABI presents a simple method of type/value checking OP_RETURN data with price limits to end this dichotomy once and for all through creating a common protocol through which apps and wallets can communicate. Some examples are included in the following two sub-sections.

#### 2a. BSVABI for Apps
For apps, there are three steps involved: Creation, Deployment and Injection. For simplicity, we will focus on the example of a microblogging platform called "MicroSV" with the ability to `post` and `like` posts. Let's also assume that it uses the bitcom namespace `1MicroSVej1Cupp9XRYnh5FChNoRcVYKANZs` and pays out a small fee to both the platform provider, and content providers.

##### ABI Creation
The app must create a valid JSON ABI schema, such as the example below.

```json
{
	"name": "MicroSV",
	"hostname": "microsv.com",
	"endpoint": "api.microsv.com/publish",
	"actions": {
		"post": {
			"limit": 10000,
			"description": "Submit a post to MicroSV",
			"args": [{
					"name": "namespace",
					"type": "String",
					"value": "1MicroSVej1Cupp9XRYnh5FChNoRcVYKANZs"
				},
				{
					"name": "text",
					"type": "String",
					"min": "1",
					"max": "256"
				},
				{
					"name": "signature",
					"type": "Signature"
				}
			]
		},
		"like": {
			"limit": 10000,
			"description": "Like a post on MicroSV",
			"args": [{
					"name": "namespace",
					"type": "String",
					"value": "1MicroSVej1Cupp9XRYnh5FChNoRcVYKANZs"
				},
				{
					"name": "postId",
					"type": "Hex",
					"min": "64",
					"max": "64"
				},
				{
					"name": "signature",
					"type": "Signature"
				}
			]
		}
	}
}
```

**Let's break down what the four top-level properties do:**

`name` - This is an optional value that helps users to easily identify the app they are interacting with, and enables wallets to provide them with a clearer UX. 

`hostname` - This is a required value that is used by wallets to ensure that the site calling ABI actions matches the site in the ABI schema to prevent against XSS attacks

`endpoint` - Unlike Planaria apps where users must submit transactions directly to the Bitcoin SV network, and apps must scan the blockchain for these transactions, Metalink apps default to a BIP270 implementation where users `signAndSend` their transactions directly to the API of the app which then validates their transaction, extracts the relevant data needed for the app, then publishes it to the Bitcoin SV network on their behalf. Once published, these transactions are still compatible with Planaria apps. The benefits of this approach are further outlined in the Unplanaria section of the proposal.

`actions` - This is where we define the action(s) that users may perform with an app, the inputs of those actions, and type/value matching to help ensure protocol compliance and minimise garbage data.

**Actions**

Any property directly below `actions` is treated as the name of an action that a user may call. In the above example, we had the action name `post`. Each action name has the following properties:

`limit` - This is the default upper limit, denominated in satoshis as an integer, of how much this TX, with the sum of its outputs, may cost the user in total.

`description` - This is an optional value used to explain to the user what they are consenting to, in this case, submitting a post to MicroSV.

`args` - This is where we define the arguments that are passed into the action, optionally including a variable `name`, what `type` of data the variable is, the `min` and `max` value for how large/small/long an input may be, and optionally, an explicit `value` they must match. The index of each of these actions map to the 0th array element after an `OP_RETURN` in a transaction output.

**Type/value matching args**

Args contain the following properties:

`name` - An optional value that is useful for debugging, as it allows BSVABI.js to point out which input has failed type/value checking.

`type` - An optional value that allows BSVABI.js to check that data is of a certain type. Current types include:
- `*` - Match all
- `String` - A string
- `Integer` - A whole number of arbitrary length
- `Float` - A decimal of arbitrary length and precision
- `Hex` - A hexadecimal string
- `Binary` - A binary string
- `Base58` - A base58 string
- `JSON` - An escaped JSON object
- `Signature` - A valid signature signed by a Metalink ID

##### ABI Deployment
In order for people to be able to consume this ABI, it must first be published to chain with the format: `OP_0 OP_RETURN <abi schema>`. There is a simple ABI deployment tool available on bsvabi.org/deploy to assist with this. After publishing the ABI to chain, we may now safely refer to it by its TXID.

##### ABI Injection
Now that you've deployed your ABI to the Bitcoin SV network, it can be consumed by apps and wallets, enabling you to make use of the six functions BSVABI performs; namely, `login`, `signAndSend`, `sign`, `encrypt`, `decrypt` and `balance`.

**Getting started**

To begin using BSVABI, simply inject the BSVABI.js library into your web app and instantiate it with your TXID, like so:

```js
const abi = new BSVABI("<abi txid>");
```

Now that you've instantiated BSVABI, you can make start calling actions.

**Login**

You can get your active Metalink ID from your wallet by calling `login()` like so:

```js
try {
    const user = await abi.login();
} catch(error) {
    console.warn("Oops, looks like your wallet dosn't implement Metalink ID");    
}
```

This sends a window postMessage to your wallet, asking for your Metalink ID, and will return the following object:
```js
{
    id: '<txid>'
    pubkey: '<active key>'
}
```

You can now use this ID to interact with other ABI actions using `signAndSend()`.

**Sign and Send**

You can call ABI actions using the signAndSend function, like so:
`abi.signAndSend({ name: 'actionname', args: [arg1, arg2, arg3...]}).`

You can also add one or more payment outputs by defining an object containing the properties `address` and `amount` to an array under the `pay` property, as well as any additional data you would like the wallet to forward to your server, but don't necessarily want to put into the TX itself (like a return address for example) using the `data` property. Let's take the `post` action from the above schema as an example:

```js
try {
    const tx = await abi.signAndSend({
        action: {
            name: 'post', 
            args: ['Just setting up my MicroSV', user.id]
        },
        pay: [
            {
                address: '18sd9jdks...', //Any valid bitcoin or paymail address
                amount: '2000' //Payment amount denominated in satoshis
            }
        ],
        data: {
            returnAddress: 'someone@microsv.com'
        }
    });
} catch(error) {
    console.warn(error);
}
```

Upon calling `signAndSend()`, a window postMessage will be emited by your browser window, enabling this transaction to be intercepted, signed and published by your Metalink-enabled wallet. Your Metalink-enabled wallet will also automatically convert Metalink IDs in a signature field into a stringified signature object like so:

```js
{
    id: '<Metalink ID txid>'
    signature: '<signature string>'
}
```

**Sign**

Sometimes you just want to `sign` something without having to publish a transaction. You can do this by calling the `sign()` function like so:

```js
try {
    const signature = await abi.sign(user.id, '<String to sign>');
} catch(error) {
    console.warn('')
}
```

This will prompt your wallet to use your Metalink ID to sign an arbitrary string beginning with _"BSVABI:"_ to ensure you can't be tricked into signing something you don't consent to. This can used for things like logging in to a website.

**Encrypt**

Sometimes you might want to `encrypt` something with someone else's public key. You can do this by calling the `encrypt()` function like so:

```js
try {
    const encryptedString = await abi.encrypt('<Metalink TXID>', '<String to encrypt>');
} catch(error) {
    console.warn('Something went wrong');
}
```

This will prompt your wallet to look up the active key of the Metalink ID you wish to encrypt for, ECIES encrypt the string, and return it back to your app as a JSON object, like so:

```js
{
    message: '<Encrypted string>'
}
```

**Decrypt**

You may also wish to `decrypt` an encrypted string someone has sent to you. You can do this by calling the `decrypt()` function like so:

```js
try {
    const decryptedString = await abi.decrypt(user.id, '<Encrypted string>');
} catch(error) {
    console.warn('Something went wrong');
}
```

This will prompt your wallet to decrypt an ECIES encrypted string with your Metalink ID's active key and return the decrypted string as a JSON object, like so:

```js
{
    message: '<Decrypted string>'
}
```

**Balance**

As BSVABI transitions BSV apps from having to consent to every single action to making large amounts of frictionless microtransactions, in order to provide a clearer understanding how much users are spending and earning, you can choose to display their current balance using the `balance()` function like so:

```js
try {
    const balance = await abi.balance();
} catch(error) {
    console.warn('Something went wrong');
}
```

This will prompt your wallet to show the balance of the wallet you're currently using to pay for each transaction. Notice that a Metalink ID is not used, as BSVABI treats funds and identity as totally separate.

#### 2b. BSVABI for Wallets
BSVABI enables any existing wallet to become a Metalink-enabled browser by simply intercepting `window.postMessage` messages from Metalink-enabled apps.

- **Mobile Wallets** - On mobile, it is as simple as injecting a piece of Javascript code into a regular Android or iOS webview.
- **Web wallets** - Web wallets can listen for and intercept postMessages using an `<iframe>`. 
- **Desktop wallets** - Desktop wallets can observes postMessages in certain browsers, roll their own browser, or utilise a browser plugin as a proxy service.
- **Browser plugins** - Browser plugins, such as those for Chrome/Firefox/Edge have direct access to postMessages.

This means that all types of BSV wallets will be able to follow a broadly-recognised, standard way of validating user actions, whitelisting them so that they may be repeatable without prompting for consent, and limiting spending so that they remain safe. No more swiping buttons every time you perform an action. 

Each time a user performs a BSVABI action, it will be sent to the wallet via window postMessages with the following schema:

```js
{
    uuid: "<random uuid>", //A random UUID used to map a function call to a promise on the frontend
    action: "BSVABI_SIGNANDSEND", //The function the user is calling in the wallet
    accound: "<Metalink ID TXID>",
    args: [
        "<ABI TXID>", 
        "<action>", 
        ["arg1", "arg2", "arg3"]
    ]
}
```

This can be implemented by whitelisting based upon:

1. TXID of an ABI
2. Hostname calling an ABI function
3. Function name
4. Spend limit

With the following information from the ABI above, we can prompt a user one time to find out whether they would like to whitelist the action `post` from `MicroSV.com` to the value of `10,000 satoshis`. Here's an example of how that could look in a mobile wallet:

![](https://i.imgur.com/gc9mFwX.jpg)

Should the user whitelist this action, we can now locally store a record of consent:

```js
{
    abi: '<ABI TXID>',
    hostname: 'microsv.com',
    account: '<Metalink ID TXID>',
    action: 'post'
    limit: 10000
}
```

Now, in the future if the `post` action is ever called with the same `txid` and `hostname` within the `limit` of 10,000 satoshis, the wallet can automatically authorise this action without needing to prompt for consent a second time. Consent would only be broken if:

1. The app updates their ABI, forcing them to serve up a new `txid`
2. The `hostname` calling the action changes (required to prevent XSS)
3. The user clears the action from their whitelist
4. The spend `limit` is exceeded

In cases 1, 2 and 3, a completely new consent window is required the next time the action is called, however in case 4, it is possible to simply prompt the user, asking whether or not they wish to increase their maximum spend limit for the action they have already consented to.

It is important that wallets provide compliant implementations of all 6 basic BSVABI actions, `login`, `signAndSend`, `sign`, `encrypt`, `decrypt` and `balance`. Worked examples will be provided on the BSVABI github. Consent should only be required for `sign`, `signAndSend`, `encrypt` and `decrypt`, all of which may be whitelisted for specific hostnames.

### 3. Unplanaria
Unplanaria is BIP270 inspired solution, designed to solve the problems Planaria solves in a way that is:

- Infinitely scalable
- Less resource-intensive
- Doesn't require spamming the p2p broadcast network with tx/block requests
- Doesn't require digging through large amounts of irrelevant data for the <1% of TXs your app or user is interested in

Instead of data transactions following the order of:

`User > Wallet > Miner > App`

It treats the app as the TX publisher, so transactions flow in the order of:

`User > Wallet > App > Miner`

This enables apps to extract all the information from `OP_RETURN` outputs without having to ask miners or third party services to collect this data for them. It also helps to reduce `OP_RETURN` spam by removing the incentive to fill up namespaces with garbage data as it will never be seen by the app server, and any garbage data sent to the app server for publishing can be validated before posting.

An Unplanaria server accepts raw transactions in the same way Insight API does; a JSON POST request with the property `rawtx`, however it also has the ability to accept optional `data` forwarded from the `signAndSend` action, like so:

```json
{
    "rawtx": "<raw tx data>",
    "data": "<some optional additonal data forwarded from the app>"
}
```

It then saves the data from the transaction to some kind of local database, publishes the transaction to the Bitcoin SV network via a miner API, another insight API or the p2p broadcast network. This means an Unplanaria server can be as simple as:

```js
const express = require('express')
const app = express()

async function broadcastTX(tx){
    //broadcast to a miner, p2p network or another Insight API
}

async function saveTxToDB(tx){
    //Do some validation, throw on error, return true upon save
}

app.post('/publish', async(req, res) => {
    try {
        if(!req.body.rawtx) throw ("Invalid transaction");
        await saveTxToDB(req.body.rawtx);
        await broadcastTx(req.body.rawtx);
        res.send("Success!");
    } catch(error) {
        res.status(500).send({ error: error });
    }
})

app.listen(1337, () => console.log(`Unplanaria listening on port 1337`))
```

### 4. Future possibilities
Having a reliable, on-chain identity system, and a means by which apps can communicate with wallets exposes several possibilities for the future:

- SPV-powered ID and ABI syndication
- Encrypted, P2P PKI
- P2P ECDH
- Integrating BSVABI into the plethora of existing smart wallets, establishing Bitcoin SV as a serious competitor to the likes of Ethereum, EOS and Tron in the dApp space.
