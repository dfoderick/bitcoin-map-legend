# Bitcoin protocol queries

Knowing Bitquery helps form a mental map of what is being written to the bitcoin blockchain. Some maps need a legend.

The purpose of this repo is to document specific queries that would be useful for real applications
using the various bitcoin protocols. It is not intended to be a general guide to learning bitquery.

Nevertheless, we begin with some background materials for getting up to speed with bitquery.  
>[Bitquery](https://docs.bitdb.network/docs/query_v3) is the language of 
[bitdb](https://docs.bitdb.network/docs/bitdb) and 
[bitsocket](https://bitsocket.org/) from 
[unwriter](https://bitgraph.network/21). 
It is based on the [mongo db query over documents](https://docs.mongodb.com/manual/tutorial/query-documents/).

The structure of the bitcoin transactions are called [tna - as documented here](https://github.com/21centurymotorcompany/tna).

The following links are useful to discover more query examples.  
[Bitquery playlist](https://github.com/21centurymotorcompany/bitplaylist)  
[More Bitsocket Examples](https://bitsocket.org/examples)  
[More Bitdb Examples](https://docs.bitdb.network/docs/intro_v3)

## Sites
Genesis is starting point for all queries.  
https://genesis.bitdb.network/

Explorer Endpoint:  
```https://genesis.bitdb.network/query/1FnauZ9aUH2Bex6JzdcV4eNX7oLSSEbxtN/<query>```  

API Endpoint:  
```https://genesis.bitdb.network/q/1FnauZ9aUH2Bex6JzdcV4eNX7oLSSEbxtN/<query>```  

## General queries
A few queries not specific to any protocol can be helpful for debugging or building apps on bitcoin.

Find specific transaction by txid. When you just created a transaction and you want to see how bitdb shreaded the transaction.  
[Try it](https://genesis.bitdb.network/query/1FnauZ9aUH2Bex6JzdcV4eNX7oLSSEbxtN/ew0KICAidiI6IDMsDQogICJxIjogeyANCiAgICAgICJmaW5kIjogeyJ0eC5oIjogIjZmNzE0N2E3YTY2NTYxMzlhMWEzZmIxZTQ1YzI2NmMyNTM0Yzg4MjdmYmE5ZjBlNzFjMDdlMmI0NTI3NjI2NWYifSANCiAgICB9DQp9)
```
{
  "v": 3,
  "q": { 
      "find": {"tx.h": "6f7147a7a6656139a1a3fb1e45c266c2534c8827fba9f0e71c07e2b45276265f"} 
    }
}
```
Find transactions going to an address. When your app needs to find out when it is being called.
> *Important!* Genesis uses original bitcoin format for addresses. Make sure you are querying for the correct address format.  

[Try it](https://genesis.bitdb.network/query/1FnauZ9aUH2Bex6JzdcV4eNX7oLSSEbxtN/ew0KICAidiI6IDMsDQogICJxIjogew0KICAgICJmaW5kIjogew0KICAgICAgIm91dC5lLmEiOiAiMTlmaFB3OXJoZU5UOWtUNEJjTHNOQ3laaGpvMVFSaXZkOCINCiAgICB9DQogIH0NCn0=)
```
{
  "v": 3,
  "q": {
    "find": {
      "out.e.a": "19fhPw9rheNT9kT4BcLsNCyZhjo1QRivd8"
    }
  }
}
```
Use [Raw Stream](https://github.com/21centurymotorcompany/bitplaylist/blob/master/bitsocket/basic/raw.json) for BitSocket only to live stream all transactions when you just want to look at something. Makes for great Saturday night entertainment.
```
{
  "v": 3,
  "q": { "find": {} }
}
```
Get random transactions. Might useful for your SPV wallet to obfuscate your address monitoring in case someone is watching you.  
[Try it](https://genesis.bitdb.network/query/1FnauZ9aUH2Bex6JzdcV4eNX7oLSSEbxtN/ew0KICAidiI6IDMsDQogICJxIjogew0KICAgICJkYiI6IFsiYyJdLA0KICAgICJhZ2dyZWdhdGUiOiBbeyAiJHNhbXBsZSI6IHsgInNpemUiOiAxMCB9IH1dDQogIH0NCn0=)
```
{
  "v": 3,
  "q": {
    "db": ["c"],
    "aggregate": [{ "$sample": { "size": 10 } }]
  }
}
```

## Protocols
| Protocol            | Docs           |
| :-----------------------: |:---------------------:|
| [memo](#memo)| [memo docs](https://sv.memo.cash/protocol) |
| [matter](#matter)      | [matter docs](https://www.mttr.app/p/0777a0e61c1de4b7a39d85c1072413f382ca45bdf0f9c217d9ee7884b0c488f7) |
| [blockpress](#blockpress)      | [blockpress docs](https://www.blockpress.com/developers/blockpress-protocol) |
| [tokenized](#tokenized)| [tokenized docs](https://github.com/tokenized/specification) |
| [bitcoin token](#bitcointoken)| [bitcointoken docs](https://github.com/simpleledger/slp-specifications/blob/master/slp-token-type-1.md) |
| SLP | [SLP docs](http://bitcointoken.com/) |
| bitcoinfiles| [bitcoinfiles docs](https://github.com/simpleledger/slp-specifications/blob/master/bitcoinfiles.md) |

## memo
[Memo Protocol document](https://sv.memo.cash/protocol)

Example: Find memo posts.
```
{
  "v": 3,
  "q": {
    "find": { 
        "out.b0": { "op": 106 },
        "out.h1": "6d02"  
    },
    "limit": 10
  }
}
```

Example: Find memo posts and posts with topics.
```
{
  "v": 3,
  "q": {
    "find": { 
        "out.b0": { "op": 106 },
        "out.h1": { "$in": ["6d02", "6d0c"] }  
    },
    "limit": 10
  }
}
```
## tokenized
[Tokenized Protocol document](https://github.com/tokenized/specification)

Tokenized is a plethora of smart contract definitions where all the use cases have been carefully analyzed.
Tokenized protocol is implemented as a fixed width message with a prefix ```00000020```

Example: Find tokenized data. This example is great because it shows how to transform the transactions to just select the outs with the data that you want.
```
{
  "v": 3,
  "q": {
    "db": ["c"],
    "find": {
      "out.b0": { "op": 106 },
      "out.str": { "$regex": "^OP_RETURN 00000020" }
    },
  "limit": 3,
  "project": { "tx": 1, "out": 1, "_id": 0 }
  },
  "r": {
    "f": "[ .[] | (.out[] | select(.b0.op==106)) as $outWithData | {tx: .tx.h, msg: $outWithData.s1}]"
  }
}
```
## bitcointoken
[bitcointoken site](http://bitcointoken.com/)  
[bitcointoken docs](http://bitcointoken.com/docs.html)

BitcoinToken is interesting because the project is working on the concept of data ownership, data that is "updatable" on the blockchain by the data owner and can be "shared" with other co-owners. 

>*Data stored in this way comes with a very strong guaranty: only the person in possession of the private key for 0x03a70e0e5f7... can spend the output containing the data. We call that user the owner of the data. By default the user to issue the command is the owner, to designate a different owner, pass in that users public key as the second argument. Multiple public keys can be passed in to model co-ownership of data.*

Also, check out how they are storing json on the blockchain (i.e. key-value pairs).
```
const outputId = await db.put({
  text: 'Lorem ipsum',
  author: 'Alice'
})
```
The above json encoded on the blockchain.
```
 4 0x74657874 OP_DROP // text
11 0x4c6f72656d20697073756d OP_DROP // Lorem ipsum
 6 0x617574686f72 OP_DROP // author
13 0x416c69636520616e6420426f62 OP_DROP // Alice and Bob
```

## matter
[Matter Protocol document](https://www.mttr.app/p/0777a0e61c1de4b7a39d85c1072413f382ca45bdf0f9c217d9ee7884b0c488f7)

Example: Find all posts.
```
{
  "v": 3,
  "q": {
    "find": { 
        "out.b0": { "op": 106 },
        "out.h1": "9d02"  
    },
    "limit": 10
  }
}
```

## blockpress
[blockpress protocol document](https://www.blockpress.com/developers/blockpress-protocol)

Example: Find blockpress posts.
```
{
  "v": 3,
  "q": {
    "find": { 
        "out.b0": { "op": 106 },
        "out.h1": "8d02"  
    },
    "limit": 10
  }
}
```


