!SLIDE bullets incremental

# What's Riak? #

* Key-Value Database
* Distributed
* Scalable
* Really, really scalable
* Made by [Basho](http://basho.com)

!SLIDE center

    brew install riak

!SLIDE

## Heavily inspired by [Amazon's Dynamo](http://www.allthingsdistributed.com/2007/10/amazons_dynamo.html) ##

!SLIDE bullets incremental

# Basics #

* Data is partitioned into slices
* Partition = virtual node/vnode
* Hashed on a 160 bit integer space
* One physical node = multiple vnodes

!SLIDE center

![The Ring](riak-ring.png)

!SLIDE bullets incremental

# Consistent Hashing #

* Determines partition from key
* Used to find an object's node

!SLIDE bullets incremental

# Dynamo/Riak Cluster #

* All nodes are equal
* Nodes can join and leave anytime
* Cluster rebalances partitions
* Gossip protocol

!SLIDE bullets incremental

# Replication #

* Ideally...
* Data is replicated across N nodes
* Data must be read from R nodes
* Read Repairs update outdated nodes
* Data must be written to W nodes

!SLIDE bullets incremental

# Failover #

* Servers do fail
* Hinted handoff to neighbors during downtimes
* Other nodes accept writes for failed nodes
* Resync when node comes back up

!SLIDE center

![Replication](riak-data-distribution.png)

!SLIDE bullets incremental

# Riak Basics #

* Buckets, keys, values
* Buckets are like flat namespaces
* Sane, RESTful HTTP interface
* Or Protobufs for speed

!SLIDE bullets incremental

# Riak Basics #

* Erlang (Munctional!!!)
* Pluggable storage
* Default: [Bitcask](http://blog.basho.com/2010/04/27/hello,-bitcask/)

!SLIDE bullets incremental

# Objects #

* Any type (binary)
* Or JSON documents
* Limited to 50-60 MB

!SLIDE bullets incremental

# Object Versioning #

* Vector clocks track object changes
* New vector clock for every change
* Automatic or manual conflict resolution

!SLIDE center

## Vector Clocks are [easy](http://blog.basho.com/2010/01/29/why-vector-clocks-are-easy/) and [hard](http://blog.basho.com/2010/04/05/why-vector-clocks-are-hard/). ##

!SLIDE bullets incremental

# Querying Data #

* By key
* Map/Reduce
* Link walking

!SLIDE bullets incremental

# Links #

* Riak objects can have links to other objects
* Links can be tagged
* Objects always have an uplink to their bucket

!SLIDE bullets incremental

# A conversation with Riak #

* Warning: There will be HTTP

!SLIDE commandline incremental small

# Get Bucket Properties #

    $ curl -D - http://localhost:8098/riak/rubies
    HTTP/1.1 200 OK
    Vary: Accept-Encoding
    Server: MochiWeb/1.1 WebMachine/1.7.1 (participate in the frantic)
    Date: Thu, 02 Sep 2010 11:06:47 GMT
    Content-Type: application/json
    Content-Length: 370
    {"props":{"name":"rubies","n_val":3,"allow_mult":false,
    last_write_wins":false,"precommit":[],"postcommit":[],
    chash_keyfun":{"mod":"riak_core_util",
    "fun":"chash_std_keyfun"},"linkfun":
    ...

!SLIDE smaller

# Set Bucket Properties #

    $ curl -D - PUT -H "Content-Type: application/json" \
      -d '{"props":{"n_val":5}}' \
      http://localhost:8098/riak/rubies

!SLIDE smaller commandline incremental

# Fetch an Object #

    $ curl -D - http://localhost:8098/riak/rubies/rbx
    HTTP/1.1 404 Object Not Found
    Server: MochiWeb/1.1 WebMachine/1.7.1 (participate in the frantic)
    Date: Thu, 02 Sep 2010 12:42:23 GMT
    Content-Type: text/plain
    Content-Length: 10

    not found

!SLIDE center

# Whoops! #

!SLIDE smaller

# Create an Object #

    $ curl -D - -X PUT -H "Content-Type: application/json" \
    -d '{"version": "1.0"}' localhost:8098/riak/rubies/rbx

!SLIDE smaller

# Fetch an Object (again) #

    $ curl -D - localhost:8098/riak/rubies/rbx
    HTTP/1.1 200 OK
    X-Riak-Vclock: a85hYGBgzGDKBVIszLIxwhlMiYx5rAzTJL4c5csCAA==
    Vary: Accept-Encoding
    Server: MochiWeb/1.1 WebMachine/1.7.1 (participate in the frantic)
    Link: </riak/rubies>; rel="up"
    Last-Modified: Thu, 02 Sep 2010 12:46:14 GMT
    ETag: 70SLW1IjL0YvRvW8pDECMN
    Date: Thu, 02 Sep 2010 12:49:32 GMT
    Content-Type: application/json
    Content-Length: 18

    {"version": "1.0"}

!SLIDE smaller

# Create some more #

    $ curl -D - -X PUT -H "Content-Type: application/json" \
    -d '{"version": "0.6"}' localhost:8098/riak/rubies/macruby

    $ curl -D - -X PUT -H "Content-Type: application/json" \
    -d '{"version": "1.9.2"}' localhost:8098/riak/rubies/mri

!SLIDE smaller

# Links #

    $ curl -D - -X POST -H "Content-Type: application/json" \
    -H 'Link: </riak/rubies/macruby>; riaktag="llvm"' \
    -d '{"version": "1.0.0"}' \
    localhost:8098/riak/rubies/rbx

!SLIDE smaller

    $ curl -D - -X POST -H "Content-Type: application/json" \          
    -H 'Link: </riak/rubies/mri>; riaktag="incompatible"' \
    -d '{"version": "0.6"}' \  
    localhost:8098/riak/rubies/macruby

!SLIDE smaller commandline incremental

# Links #

    $ curl -D - http://localhost:8098/riak/rubies/rbx        
    HTTP/1.1 200 OK
    X-Riak-Vclock: a85hYGBgzmDKBVIszLIxwhlMiYx5rAzTJL4c5YMKM3oGrIMKH5CGC7M1JzGd1C9DlsgCAA==
    Vary: Accept-Encoding
    Server: MochiWeb/1.1 WebMachine/1.7.1 (participate in the frantic)
    Link: </riak/rubies/macruby>; riaktag="llvm", </riak/rubies>; rel="up"
    Last-Modified: Thu, 02 Sep 2010 12:59:44 GMT
    ETag: 5R0AS2o0y81CIB4LpcHD5y
    Date: Thu, 02 Sep 2010 12:59:50 GMT
    Content-Type: application/json
    Content-Length: 20

    {"version": "1.0.0"}

!SLIDE smaller commandline incremental

# Walking Links #

    $ curl -D - localhost:8098/riak/rubies/rbx/_,llvm,_
    HTTP/1.1 200 OK
    ...
    Content-Type: multipart/mixed; boundary=QVzlBS0eP4wma1FgTQHG48pWsNO
    --2AvZHUf1P6Vu1loZ2c6QW8ywk6s
    X-Riak-Vclock: a85hYGBgzGDKBVIszNvtV2QwJTLmsTKYyX05ypcFAA==
    Location: /riak/rubies/macruby
    Content-Type: application/json
    Link: </riak/rubies>; rel="up"
    Etag: z0hqVp2kYkT0vFYyWtub6
    Last-Modified: Thu, 02 Sep 2010 13:10:14 GMT

    {"version": "0.6"}
    --2AvZHUf1P6Vu1loZ2c6QW8ywk6s--

!SLIDE smaller commandline incremental

# Walking Nested Links #

    $ curl -D - localhost:8098/riak/rubies/rbx/_,llvm,_/_,incompatible,_/
    HTTP/1.1 200 OK
    ...
    Content-Type: multipart/mixed; boundary=7D3wLfYH6zjsblEzVA0rvIhi7wS
    --7D3wLfYH6zjsblEzVA0rvIhi7wS
    X-Riak-Vclock: a85hYGBgzGDKBVIsTAs02TOYEhnzWBk05L4c5csCAA==
    Location: /riak/rubies/mri
    Content-Type: application/json
    Link: </riak/rubies>; rel="up"
    Etag: 4JQRo8nYoGM6W4lFKWErxQ
    Last-Modified: Thu, 02 Sep 2010 13:10:00 GMT

    {"version": "1.9.2"}
    --7D3wLfYH6zjsblEzVA0rvIhi7wS--
    
!SLIDE bullets incremental

# Use Cases for Links #

* Associations (has_one, has_many)
* Chain similar objects
* More on [link walking](http://blog.basho.com/2010/02/24/link-walking-by-example/)
* Tag objects with similar context

!SLIDE bullets incremental

# Map/Reduce #

* Query data with Erlang or JavaScript
* Transform data (map)
* Group data (reduce)
* Don't think CouchDB

!SLIDE bullets incremental

# Map/Reduce #

* Requires an input list
* One or more map phases
* None or more reduce phases
* None or more link phases

!SLIDE bullets incremental

# Map/Reduce #

* Map phases are run near the data
* Reduce phase on the coordinating node
* [More on Map/Reduce in Riak](http://wiki.basho.com/display/RIAK/MapReduce)

!SLIDE code smaller

# Map/Reduce (real simple)#

    $ curl -D - -X POST localhost:8098/mapred \
    -H "Content-Type: application/json" \
    -d @-
    {"inputs": [["rubies", "rbx"], ["rubies", "macruby"]],
      "query": [
        {"map":
          {"language": "javascript",
           "source": "Riak.mapValues"}},
        {"reduce":
          {"language": "javascript",
           "source": "Riak.reduceSort"}}
    ]}

