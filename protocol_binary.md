# Protocol Documentation

## Binary Protocol is not final!  There may be breaking changes at anytime.  Contact me if you wish to write a driver.

### Binary Protocol

The binary protocol is similar to the json protocol, just a different encoding, meant to be faster to encode and decode then json for high performance critical applications (databases, queues, ect). Please read the json protocol spec for more details, as this doc simply describes the encoding. One of the main goals of the binary protocol is make the sharding proxy faster (see goshire-shards project)



Request Format

* All integers are signed big endian.
* Variable length fields (arrays and strings) are all prefixed by a 'length' int16
* All length fields are int16, unless otherwise noted
* All strings are utf8 encoded

### Hello Packet

Clients should send this packet on initial connection.  No response will be sent from the server.  Server will disconnect without a proper hello

```
[encoding (int8)] //same values as param_encoding, currently should always be 0 (json)
[length][json hello (string)]
```

The hello json package should look like the following:

```
{
    "v" : 2.0 //the protocol version (required)
    "useragent" : //the useragent (required)
    "service" : //the service (if used to connect to a shard router only)
}
```

After hello is sent, then the client is free to start sending requests, and recieving responses.
The protocol is async (same as the json protocol), so clients should always listen for new responses.  There are only 2 packet types: REQUEST and RESPONSE.  


### REQUEST

```
[partition (int16)] //set to -1 as default
[length][shard key (string)] 
[router table revision (int64)]

[length][txn_id (string)]
[txn_accept(int8)]
[method(int8)]
[length][uri (string)]
[param_encoding (int8)]
[length][params (array)]
[content_encoding (int8)]
[content_length (int32)][content (array)]
```
 -- Note the shard key and partition sections are only used for goshire-shard router requests. Other requests should zero those fields


### RESPONSE

```
[length][txn_id (string)]
[txn_status(int8)]
[status (int16)]
[length][status_message (string)]
[param_encoding (int8)]
[length (int32)][params (array)]
[content_encoding (int8)]
[content_length (int32)][content (array)]
```


-- Notes:
    *  The param map contains any top level keys for the response packet.  For instance, if the response in json looks like:
```
{
    strest : {...},
    mydata : "this is something I added"
}
```
    then the params map would be { mydata : "this is something I added"}

    This is a bit clunky to implement in clients but it makes the clientside responses exactly the same between the different protocols.

