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

```js

```

<a name="join"></a>
### `client.join([, callback])`

```js

```

<a name="put"></a>
### `client.put(value: any][, callback])`

```js

```


<a name="get"></a>
### `client.get(key: string][, callback])`
`get()` is the primary method for fetching data from the store. The `key` can be of any type. If it doesn't exist in the store then the callback or promise will receive an error. A not-found err object will be of type `'NotFoundError'` so you can `err.type == 'NotFoundError'` or you can perform a truthy test on the property `err.notFound`.

```js
db.get('foo', function (err, value) {
  if (err) {
    if (err.notFound) {
      // handle a 'NotFoundError' here
      return
    }
    // I/O or other error, pass it up the callback chain
    return callback(err)
  }

  // .. handle `value` here
})
```

`options` is passed on to the underlying store.

If no callback is passed, a promise is returned.

<a name="rev"></a>
### `client.rev(value: any][, callback])`

```js

```

<a name="del"></a>
### `client.del(key: string][, callback])`

```js

```

<a name="leave"></a>
### `client.leave([, callback])`

```js

```

## Promise Support

subspace-cleint ships with native Promise support out of the box.

Each function taking a callback also can be used as a promise, if the callback is omitted. This applies for:

client.join()
client.get(key)
client.put(key, value)
client.rev(key, value)
client.del(key)
client.leave()


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
