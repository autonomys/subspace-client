# Subspace-Client
The **subspace client** is a javascript library that works in a **node.js** or **browser** runtime.  It allows you to build client-side applications for web, mobile, or desktop apps to store and retreive data through the subspace protocol and peer-to-peer device network.

Subspace is a decentralized NoSQL database, or key-value store, written entirely in javascript with a developer friendly API. Data hosted on subspace is sharded across a network of end user devices including mobile phones, tablets, desktops, servers, and custom farming hardware.  Subspace hosts pledge free disk space to the network in order to earn subspace credits, which they receive for hosting and serving data for subspace apps. Developers may reserve space on the network by creating a client data contract through the subspace [developer console](https://subspace.network/console). 

To learn more about how the subspace protocol works browse the [FAQ](https://subspace.network/faq) or read the technical [white paper](https://subspace.github.io/paper/).

* [Usage](#usage)
* [API](#api)
* [Promise Support](#promise-support)
* [Events](#events)


## Usage

Install with NPM

```bash
npm install subspace-client
```

Fetch from a CDN

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/subspace-client.js/1.0/subspace-client.min.js"></script>
```

Example workflow

```js
const Subspace = require('subspace-client')

// create the client instance
// a contract is only required to put new records
const client = new Subspace({
  contract: '5cd2ac01f8f5bd5f149455925b3023e1c1c54133042477b59059e388df1366a4'
})

// connect to the network
client.join((error => {
  if (error) throw(error)
  
  // put a record to your database contract
  client.put('hello subspace', (error, record) => {
    if (error) throw(error)
    
    console.log(record)
    
    // {
    //   key: '678e749011f8a911f011e105c618db898a71f8759929cc9a9ebcfe7b125870ee',
    //   value: 'hello subspace'
    // }
    
    // get a record from your database contract
    client.get(record.key, (error, value) => {
      if (error) throw(error)
      
      console.log(value) // -> 'hello subspace'      
    })
  })
})

```

## API

* **[`client`](#client)**
* **[`client.join()`](#join)**
* **[`client.put()`](#put)**
* **[`client.get()`](#get)**
* **[`client.rev()`](#rev)**
* **[`client.del()`](#del)**
* **[`client.leave()`](#leave)**

<a name="client"></a>
### `const client = new Subspace([options])`
Creates a new `client` instance that is used to interface with the subspace network.

* `options`: optional arguments to set identity and configure connection parameters

If no options are passed to the constructor a new subspace identity will be generated allowing the client to `connect` and `get` records from the subspace network. If a valid `contract_id` is provided the client will be able to `put` new record and `rev` and `del` records they own. Remaining options are purely optional, with suitable defaulst provided. 

```js
let options = {
  contract: '5cd2ac...',     // optional id for storage contract, obtained through the subspace console
  name: 'Gavin Belson',      // optional name to associate with profile
  email: 'gavin@hooli.com',  // optional email to associate with profile
  passphrase: 'box_three',   // optional passhprase to associate with profile
  gateway_nodes: [],         // optional, overirdes the default gateways for bootstrapping onto subspace
  gateway_count: 1,          // optional, default is 1
  delegated: true,           // optional, will delegate put/get of replicas to a first host for improved performance
}
```

<a name="join"></a>
### `async client.join([callback])`
Joins the subspace network by connecting to one or more gateway nodes, authenticating your public key, and fetching the latest tracker. 

Returns a callback or promise on authentication with the first gateway. 
Emit a `connected` event when fully authenticated with all gateway nodes and tracker sync is complete.
Returns an error if failed

```js
client.connect(error => {
  if (error) throw(error)
  console.log('Authenticated with a gateway node')
})

client.on('connected', () => {
  console.log('Authenticated with all gateway nodes and fetched the tracker')
})

```

<a name="put"></a>
### `async client.put(value: any][, callback]): record: object`
Writes some data to the subspace network. Data is encoded and encrypted based on the clients public key. Requires a signature from a valid data contract private key. Record type (mutable or immutable) is based on the underlying contract. Resolves once the record is written to the first host. Emits an event when the record is fully replicated on SSDB or if error during replication.

* `value` - the data to be stored. Valid types include boolean, number, string, array, json, binary, and buffer.

Returns the record object, a key-value pair with the derived key and value in the same format it was provided. 
Returns an error if failed. 
Emits an event when fully resolved.

Example
```js
client.put('hello subspace', (error, record) => {
  if (error) throw(error)
  
  console.log('Record successfully put to subspace')
  console.log(record)
    
    // {
    //   key: '678e749011f8a911f011e105c618db898a71f8759929cc9a9ebcfe7b125870ee',
    //   value: 'hello subspace'
    // }
})

client.on('put', (record, hosts) => {
  console.log(`New record with id: ${record.key} has been fully replicated to all hosts, including: `)
  hosts.forEach(hostId => console.log(`\n ${hostId}`))
})

```

<a name="get"></a>
### `async client.get(key: string][, callback]): value: any`
Retrieves some data from the subspace network with a given record key (id). Data is decoded and decrypted using the clients private key. No data contract is required to `get` records. Resolves once the record is retrieved from the first host. Emits an event when the record is fully retrieved and validated or if error during replication.

* `key` - 32 byte record key as a hex encoded string 

Returns the record value in the same format it was provided. 
Returns an error if failed. 
Emits an event when fully resolved.

Example
```js
client.get('678e749011f8a911f011e105c618db898a71f8759929cc9a9ebcfe7b125870ee', (error, value) => {
  if (error) throw(error)
  
  console.log('Succesfully got value from subspace')
  console.log(value)
    
    // 'hello subspace'
})

client.on('get', (record, hosts) => {
  console.log(`Existing record with id: ${record.key} has been fully retrieved to all hosts, including: `)
  hosts.forEach(hostId => console.log(`\n ${hostId}`))
})

```

<a name="rev"></a>
### `async client.rev(key: string, value: any][, callback]): record: object`
Updates a mutable record. Resolves once the record is validated and replicated to the first valid host. Emits an event when the update is fully replicated on SSDB or if error during replication.

* `key` - 32 byte record key as a hex encoded string 
* `value` - the new value to be assigned to this key. Does not have be the same type as previous value. Valid types include boolean, number, string, array, json, binary, and buffer.

Returns the new record object, a key-value pair with the same key and updated value in the same format it was provided. 
Returns an error if failed. 
Emits an event when fully resolved.

Example
```js

client.rev('goodbye subspace', (error, record) => {
  if (error) throw(error)
  
  console.log('Record successfully updated on subspace')
  console.log(record)
    
    // {
    //   key: '678e749011f8a911f011e105c618db898a71f8759929cc9a9ebcfe7b125870ee',
    //   value: 'goodbye subspace'
    // }
})

client.on('rev', (record, hosts) => {
  console.log(`Update to existing record with id: ${record.key} has been fully replicated to all hosts, including: `)
  hosts.forEach(hostId => console.log(`\n ${hostId}`))
})

```

<a name="del"></a>
### `async client.del(key: string][, callback])` : 
Deletes a mutable record from a mutable contract. Resolves once the record is deleted from the first host. Emits an event when the delete is fully replicated on SSDB or if error replication.

* `key` - 32 byte record key as a hex encoded string 

Returns an error if failed. 
Emits an event when fully resolved.

Example
```js
client.del('678e749011f8a911f011e105c618db898a71f8759929cc9a9ebcfe7b125870ee', (error) => {
  if (error) throw(error)
  console.log('Succesfully started record deletion from subspace')
})

client.on('del', (record, hosts) => {
  console.log(`Existing record with id: ${record.key} has been fully deleted from all hosts, including: `)
  hosts.forEach(hostId => console.log(`\n ${hostId}`))
})

```

<a name="leave"></a>
### `async client.leave([, callback])`
Leaves the subspace network by gracefully disconnecting from all nodes you are directly connected to. 

Returns a callback or promise on send of all disconnect notifications. 
Emits a `disconnected` event when all connections are fully closed.

```js
client.disconnect(error => {
  if (error) throw(error)
  console.log('Notified all peers I am leaving')
})

client.on('disconnected', () => {
  console.log('All peers connections have been closed')
})

```

## Promise Support

subspace-client ships with native Promise support out of the box.

Each function taking a callback also can be used as a promise, if the callback is omitted. This applies for:

* `client.join()`
* `client.get(key)`
* `client.put(key, value)`
* `client.rev(key, value)`
* `client.del(key)`
* `client.leave()`

### Traditional callback syntax

```js

client.join(error => {
  if (error) throw(error)
  console.log('connected to subspace network')
  
  client.put('hello subspace', (error, record) => {
    if (error) throw(error)
    console.log('put a new record to remote host')
    
    client.get(record.key, (error, value) => {
      if (error) throw(error)
      assert(record.value === value)
      console.log('got same record back from remote host ')
      return record
    })
  })
})

```

### Using chained promises

```js

client.join()
  .then(() => {
    console.log('connected to subspace network')
    client.put('hello subspace')
  })
  .then(record => {
    console.log('put a new record to remote host')
    client.get(record.key)
  })
  .then(value => {
    assert(record.value === value)
    console.log('got same record back from remote host ')
    return(record)
  })
  .catch(error => {
    console.log('Subspace Error')
    throw(error)
  })

```

### Using async/await syntax

```js

const testSubspace = async () => {
  try {
    await client.join()
    console.log('connected to subspace network')

    const record = await client.put('hello subspace')
    console.log('put a new record to remote host')

    const value = await client.get(record.key)
    assert(record.value === value)
    console.log('got same record back from remote host ')

    return record
  }
  catch (error) {
    console.log('Subspace Error')
    throw(error)
  }
}

```

## Events

`client` is an [`EventEmitter`](https://nodejs.org/api/events.html) and emits the following events.

| Event          | Description                             | Arguments                         |
|:---------------|:----------------------------------------|:----------------------------------|
| `connected`    | Client has fully joined network         | -                                 |
| `put`          | Record replicated to all hosts          | `record: object, hosts: array`    |
| `get`          | Record retrieved from all hosts         | `record: object, hosts: array`    |
| `rev`          | Updated record replicated on all hosts  | `record: object, hosts: array`    |
| `del`          | Record deleted from all hosts           | `record: object, hosts: array`    |
| `disconnected` | Client has fully left the network       | -                                 |


For example you can do:

```js
client.on('put', (record, hosts) => {
  console.log(`New record with id: ${record.key} has been fully replicated to all hosts, including: `)
  hosts.forEach(hostId => console.log(`\n ${hostId}`))
})
```
