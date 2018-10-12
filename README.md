# Subspace-Client
The **subspace client** is a javascript library that works in a **node.js**, **browser**, or **react-native** runtime.  It allows you to build client-side applications for web, mobile, or desktop apps to store and retreive data through the subspace protocol and peer-to-peer device network.

Subspace is a decentralized NoSQL database, or key-value store, written entirely in javascript with a developer friendly API. Data hosted on subspace is sharded across a network of end user devices including mobile phones, tablets, desktops, servers, and custom farming hardware.  Subspace hosts pledge free disk space to the network in order to earn subspace credits, which they receive for hosting and serving data for subspace apps. Developers may reserve space on the network by creating a client data contract through the subspace [developer console](https://subspace.network/console). 

To learn more about how the subspace protocol works browse the [FAQ](https://subspace.network/faq) or read the technical [white paper](https://subspace.github.io/paper/).

* [Install](#install)
* [Usage](#usage)
* [API](#api)
* [Promise Support](#promise-support)
* [Events](#events)
* [Contributing](#contributing)
* [License](#license)

## Install

Using NPM

```bash
npm install subspace-client
```

Via a CDN

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/subspace-client.js/1.0/subspace-client.min.js"></script>
```


## Usage

```js
var level = require('level-rocksdb')

// 1) Create our database, supply location and options.
//    This will create or open the underlying RocksDB store.
var db = level('./mydb')

// 2) Put a key & value
db.put('name', 'Level', function (err) {
  if (err) return console.log('Ooops!', err) // some kind of I/O error

  // 3) Fetch by key
  db.get('name', function (err, value) {
    if (err) return console.log('Ooops!', err) // likely the key was not found

    // Ta da!
    console.log('name=' + value)
  })
})
```

## API

* [<code><b>join()</b></code>](#join)
* [<code><b>leave()</b></code>](#leave)
* [<code><b>put()</b></code>](#put)
* [<code><b>get()</b></code>](#get)
* [<code><b>rev()</b></code>](#rev)
* [<code><b>del()</b></code>](#del)
* [<code><b>connect()</b></code>](#connect)
* [<code><b>disconnect()</b></code>](#disconnect)
* [<code><b>send()</b></code>](#send)
* [<code><b>reserveSpace()</b></code>](#reserve-space)
* [<code><b>sendCredits()</b></code>](#send-credits)


<a name="get"></a>
### `client.get(key: string][, callback])`
<code>get()</code> is the primary method for fetching data from the store. The `key` can be of any type. If it doesn't exist in the store then the callback or promise will receive an error. A not-found err object will be of type `'NotFoundError'` so you can `err.type == 'NotFoundError'` or you can perform a truthy test on the property `err.notFound`.

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

## Promise Support

### Using Callbacks

```js

client.join(error => {
  if (error) {
    console.log(error)
    return
  }

  console.log('connected to subspace network')
  client.put('hello subspace', (error, record) => {
    if (error) {
      console.log(error)
      return
    }

    console.log('put a new record to remote host')
    client.get(record.key, (error, value) => {
      if (error) {
        console.log(error)
        return
      }

      assert(record.value === value)
      console.log('got same record back from remote host ')
      return record
    })
  })
})

```

### Using Promises

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
    console.log(error)
    return(error)
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
    console.log(error)
    return(error)
  }
}

```

## Events

## Contributing

## License

