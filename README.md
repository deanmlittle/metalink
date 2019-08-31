# Overview
Metalink is a proposed, work-in-progress protocol standard that aims to simplify the Bitcoin app development process and create seamless, Bitcoin-powered experiences for users without having to rely upon intermediaries. It aims to be robust enough to protect the privacy and funds of users, decentralised enough that no individual actor holds power over it, flexible and extensible enough to be used by any wallet, web app, smart contract, business or service imaginable, and most importantly, handles all of the above in the only sensible way any Bitcoin-based protocol should ever attempt to do so: On-chain.

Don't like it? ~stiff~ Fork it, update it and create a merge request.


# Protocol

The blockchain protocol standard is broken down into the following three sub-sections:

1. [**Identity and Access Management (IdAM)**](#_1-identity-management-idam) - Linking human-readable pseudonyms to signing keys.
2. [**Public Key Infrastructure (PKI)**](#_2-public-key-infrastructure-pki) - Scalable generation and tracking of public keys.
3. [**Application Binary Interface (ABI)**](#_3-application-binary-interface-abi) - Providing addressing for specific functions in Bitcoin apps that are authorised by wallets to create seamless interactions for users in trusted environments.

# 1. Identity Management (IdAM)

## Overview

Using a simple `OP_RETURN` namespace protocol, we can handle the creation and maintenance of identities completely on-chain like so:

**Create:** 
`OP_FALSE OP_RETURN <network> <pseudonym> <permissions>`

**Update:** 
`OP_FALSE OP_RETURN <network> <pseudonym> <permissions> <sig>`

The individual parts that make up the protocol are as follows:

1. `<network>` - An `OP_RETURN` namespace, such as a Bitcom address or similar, for storing and maintaining signing identities. The specification advocates the use of a single namespace throughout the Bitcoin network for interoperability, without procluding the possibility that businesses and developers may wish to create their own on-chain IdAM _"intranet"_ on Bitcoin for privacy, testing or internal use.
2. `<pseudonym>` - A 1-32 character long, unique, human-readable name, comprised of any utf8 string such as: `_unwriter`, `_莫寫者` or `_アンライター`. It is important for Bitcoin to become a global currency that we can support many different locales when it comes to naming.
3. `<permissions>` - A valid JSON schema of permissions and signing keys with two compulsory keys, `o` (owner) and`a` (active), and one compulsory attribute, `v` (version), that is optionally extensible to enable further functionality, like so:

```json
{
    "o": "0320b5f5a...", //Owner public key
    "a": "02f64f033...", //Active public key
    "v": 1
}
```

By default, users will sign with an active permission key. The owner key can update any attribute or key, including the active key, but the active key cannot change the owner key. In this way, should the active key used for signing actions become compromised, you still retain full ownership over your identity so long as you have the owner key.

This specification can also be optionally extensible with additional identities, properties or attributes in the future as needed:

```json
{
    "o": "02f8s9234...",
    "a": "02e7fa103...",
    "v": 1337,
    "custom": "value",
    "other": "value",
    "attributes": ["some attribute", "some other attribute"]
}
```

By using an extensible JSON object, it is possible to extend functionality in the future on an opt-in basis without breaking the protocol with additional `OP_RETURN` fields.

4. `<signature>` - To update an identity's permissions schema, the update transaction must be signed by the `active` or `owner` key of the first person to claim a pseudonym. The `active` key is used to update the entire schema with the exception of the `owner` key. Only the `owner` key may update the entire schema. This is to enable used to have a hot key that is used for signing things, and a cold key that is used to secure ownership of their identity. You could even choose not to give your wallet access to the `owner` key! While not recommended, `active` and `owner` may also share the same key.

## Examples

### Create a new identity
By putting these things together, we can create a new identity like so:

`OP_FALSE` `OP_RETURN` `1b8asjda8sdj8a...` `_unwriter` `{ o: '02f8s9234...', a: '02e7fa103...', v: 0 }`

### Update an identity
For any future updates to this identity, the `v` attribute must be incremented by `1` and the resulting JSON schema must be signed by the current active key in order to provide double spend protection. The signature is appended to the end like so:

`OP_FALSE` `OP_RETURN` `1b8asjda8sdj8a...` `_unwriter` `{ o: '02f8s9234...', a: '02e7fa103...', v: 1 }` `48304502e...`

# 2. Public Key Infrastructure (PKI)

In the scope of this protocol, there are two main types of actors:

1. Users - Individual wallet users that must manually initiate consent for payment requests to/from new providers.
2. Services - Oracles that automatically react to payment requests on behalf of users or businesses.

They can interact on a `user<->service`, `user<->user` or `service<->service` basis. For example, the user `_unwriter` can interact with the service `twetch`. The service `twetch` can interact with the service `ron`. The user `_unwriter` can interact with the user `atilla`. 

The way each party's payment addresses relate to one another can be summarised as follows:
_"One mnemonic, many xprivs. One xpriv, one xpub. One authorisation, one ECIES-encrypted xpub and derivation path."_

In simple terms, a wallet holds a mnemonic that can generates multiple xprivs. For each service that a wallet interacts with, a new xpriv and xpub pair are generated. The xpub and a derivation path are sent to the service.

The protocol may look something like this:

`OP_FALSE` `OP_RETURN` `<pseudonym>` `<ECIES-encrypted JSON schema>`

The schema may look something like this before encrypted, and is signed with the active key of the sender:

```json
{
    "name": "_unwriter",
    "xpub": "xpub19dkja9...",
    "path": "m/19482/1248/7414/0",
    "signature": "8dk23e9osa..."
}
```
An example of how this would look on-chain would be something like this:

`OP_FALSE` `OP_RETURN` `twetch` `23049tujf...`


There are two ways in which xpubs can be transferred; synchronously and asynchronously.

## Synchronous Authorisation
Synchronous requests are possible from a `user->service` or `service<->service` scenario. In a synchronous scenario, a user sends can initiate an authorisation by sending an ECIES-encrypted xpub to a service's IdAM namespace, either encrypted with their current active key, or some other agreed upon key. In response, the service automatically sends a similar ECIES-encrypted xpub of their own to the user's IdAM namespace. In doing so, the two are able to maintain a list of all of their transactions between one another, but cannot use the shared xpub to see any of their other xpub addresses or activities.

## Asynchronous Authorisation
It is also possible to have asynchronous requests, where one user requests the xpub and derivation path of another user via an ECIES-encrypted OP_RETURN request to their namespace. The user's wallet finds this request by periodically scanning their IdAM namespace and pushing them a notification to accept or reject the request. A similar request can also be sent back to the sender and similarly responded to in due course to enable two-directional exchanges.

## Unknowns
Still considering alternative options for handling things like xpubs revealing transaction history in the case of compromised active keys. One way of handling this would be a Diffie-Helmann exchange. Another would be a three way handshake where the initial request sends a second public key with which the xpub can be ECIES encrypted, along with a third public key with which the responder may send a return xpub for two-directional exchanges.

# 3. Application Binary Interface (ABI)
In Bitcoin applications, there often emerges the need for small, repetitive microtransactions which a user may opt to whitelist as safe. Some examples may include liking, posting or commenting on a social network, sending someone a tip, or pay as you go video streaming. Imagine what a terrible UX it would be to have to confirm every 5 seconds that you wish to continue watching. would create a terrible UX. At the same time, simply authorising *any* transaction creates a security risk for this funds. Having an ABI standard is one solution to end the tradeoff between safety and convenience. 

## Examples

Let's imagine we're building something like Twitter as an example, where users send a post request to the blockchain, and the service validates their post and publishes it on their behalf to the Bitcom address of their app.

You'd need a simple server that scans the blockchain for post requests then posted them to a Bitcom namespace in Node.js like this:

```javascript
function readPostRequests(){
    bsv.get(namespace, posts => {
        posts.forEach(p => {
            if(isValid(p)){
                post(p.sender, p.args);
            }
        });
    });
}

function post(sender, [message]) {
    bsv.publishPost(sender, message);
}

readPostRequests();
```

An ABI for posts could look something like this:

```json
{
    "language": "javascript",
    "version": "0.0.1",
    "namespace": "18ads921a...",
    "abi": [
        {
            "function": "post",
            "namespace": "18ads921a...",
            "args": [
                "string"
            ],
            "limits": [
                {
                    "token": "bsv",
                    "max": 10000,
                    "min": 1000
                }
            ]
        }
    ]
}
```

By creating an ABI like this as a header for your post function, a wallet is able to determine that the ABI is asking you to consent to:

- A post function
- From the TXID of this ABI
- To a certain namespace
- With a minimum value of 1,000 satoshis, and
- a maximum value of 10,000 satoshis

Your wallet could also allow you to set your own max limit within the bounds of this ABI. By defining the bounds of what you're asked to consent to, as well as what you personally agree to consent to within those bounds, wallets are able to provide seamless experiences where you consent once and optionally whitelist until you either opt out, or the ABI is modified and consent is renewed.

Now let's say we wanted to add on some additional functions like commenting and liking:

```javascript
function like(sender, [txid]) {
    bsv.publishLike(sender, txid);
}
function comment(sender, [txid, message]){
    bsv.publishComment(sender, txid, message);
}
```

We could then expand our ABI like so:

```json
{
    "language": "javascript",
    "version": "0.0.1",
    "namespace": "18ads921a...",
    "abi": [
        {
            "function": "post",
            "args": [
                "string"
            ],
            "limits": [
                {
                    "token": "bsv",
                    "max": 10000,
                    "min": 10000
                }
            ]
        },
        {
            "function": "comment",
            "args": [
                "string", 
                "string"
            ],
            "limits": [
                {
                    "token": "bsv",
                    "min": 10000,
                    "max": 10000
                }
            ]
        },
        {
            "function": "like",
            "args": [
                "string"
            ],
            "limits": [
                {
                    "token": "bsv",
                    "min": 10000,
                    "max": 10000
                }
            ]
        }
    ]
}
```

Although an app is not limited to a single ABI, in order to conserve space on-chain, we could also consider uploading the ABI in a compressed format:

```json
{"l":"javascript","v":"0.0.1","n":"18ads921a...","a":[{"f":"post","a":["string"],"l":[{"t":"bsv","mn":10000,"mx":10000}]},{"f":"comment","a":["string","string"],"l":[{"t":"bsv","mn":10000,"mx":10000}]},{"f":"like","a":["string"],"l":[{"t":"bsv","mn":10000,"mx":10000}]}]}
```

No specific `OP_RETURN` namespace is required for posting an ABI. Provided you structure it as: `OP_FALSE` `OP_RETURN` `<schema>` and provide the TXID of the ABI, a wallet will be able to interact with it and authorise its actions.

Don't like Javascript? No problem. We can simply swap out the language and variable types. Here's a few examples for alternative languages:

### C++

```json
{
    "language": "cpp",
    "version": "0.0.1",
    "namespace": "18ads921a...",
    "abi": [
        {
            "function": "tip",
            "args": [
                "char[34]",
                "uint32_t",
            ],
            "limits": [
                {
                    "token": "bsv",
                    "min": 1
                }
            ]
        }
    ]
}
```

### WebAssembly

```json
{
    "language": "wasm",
    "version": "0.0.1",
    "namespace": "18ads921a...",
    "abi": [
        {
            "function": "tip",
            "args": [
                "string",
                "i32",
            ],
            "limits": [
                {
                    "token": "bsv",
                    "min": 1
                }
            ]
        }
    ]
}
```

### Python

```json
{
    "language": "python",
    "version": "0.0.1",
    "namespace": "18ads921a...",
    "abi": [
        {
            "function": "tip",
            "args": [
                "str",
                "long",
            ],
            "limits": [
                {
                    "token": "bsv",
                    "min": 1
                }
            ]
        }
    ]
}
```

### Solidity

Love Ethereum? Of course not, but consider that a bunch of devs may want to jump ship and try something that actually scales in a familiar environment. We can use `OP_RETURN` data to feed a solidity ABI in an off-chain version of EVM like truffle. Consider the following example where a function is treated as a solidity `payable` function. Due to the flexibility of JSON schemas, we can add the additional property of `"type"` to the schema that is solidity-specific without breaking the base schema that enables wallets to interpret which actions are being authorised, for how much and to whom.

```json
{
    "language": "solidity",
    "version": "0.0.1",
    "namespace": "18ads921a...",
    "abi": [
        {
            "function": "tip",
            "type": "payable",
            "args": [
                "string",
            ],
            "limits": [
                {
                    "token": "bsv",
                    "min": 1
                }
            ]
        }
    ]
}
```

## Wallet integration
The beauty of this approach is that any wallet can make use of ABIs by simply setting up a proxy that intercepts post requests to a certain URL, such as `POST https://bsvabi/<txid>/<action name> <data>`, and turns them into a protocol-compliant `OP_RETURN`. This can be handled by injecting a Javascript file into an Android or iOS webview, a Chrome plugin, a desktop proxy app, or any kind of proxyable service imaginable. It means that any wallet can be compliant with any app by just injecting this tiny piece of code instead of each having to create their own solution and forcing apps to integrate with them.
