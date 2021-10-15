gaang-solr-zk-client
======================

This is a fork from  [node-solr-smart-client](https://github.com/jijordre/node-solr-smart-client)
* Periodic maintainence of the package and dependencies update to prevent software composition Analysis (SCA) security flaws
* Updates to support master aliases pointing to multiple solr collections

A node.js solr smart client. In short it serves as a smart constructor for [solr-client](https://github.com/lbdremy/solr-node-client). The constructor queries a ZooKeeper
ensemble for live nodes of the input Solr collection. When more than one live node is returned by ZooKeeper, the constructor picks one at random and hands over to solr-client.

Features
--------

* a smart constructor for [solr-client](https://github.com/lbdremy/solr-node-client)
* retrieval of live Solr nodes from ZooKeeper cluster using [node-zookeeper-client](https://github.com/alexguan/node-zookeeper-client)
* HTTP requests towards Solr nodes using [node-rest-client](https://github.com/aacerox/node-rest-client)
* supports SSH tunneling
* supports Solr collection aliases

Installation
------------

    npm install gaang-solr-zk-client
    
Include option `--save` to add it to your package.json file in the same go:

    npm install gaang-solr-zk-client --save
    
Usage
-----

### Basic

    // Load dependency
    var solrSmartClient = require('gaang-solr-zk-client');
    
    // Define options
    options = {
        zkConnectionString: 'localhost:2181',
        zkLiveNodes: '/live_nodes',
        zkAliases: '/aliases.json',
        solrProtocol: 'http',
        solrCollectionsGetEndPoint: '/admin/collections?action=LIST', // Supports XML and JSON writer types
        ssh: {},
        // Passed verbatim to node-zookeeper-client
        zk: {
            sessionTimeout: 3000,
            spinDelay : 1000,
            retries : 1
        },
        // Passed verbatim to node-rest-client
        rest: {
            requestConfig: {
                timeout: 3000
            },
            responseConfig: {
                timeout: 3000
            },
            mimetypes: {
                json: ["application/json", "application/json;charset=utf-8", "application/json; charset=utf-8", "application/json;charset=UTF-8", "application/json; charset=UTF-8"],
                xml: ["application/xml", "application/xml;charset=utf-8", "application/xml; charset=utf-8", "application/xml;charset=UTF-8", "application/xml; charset=UTF-8"]
            }
        }
    };
    
    // Create Solr client, execute query and print number of documents in response.
    solrSmartClient.createClient('my_solr_collection', options, function (err, solrClient) {
        if (err) {
            return console.log(err);
        }
        solrClient.search('q=*:*', function (err, obj) {
            if (err) {
                return console.log(err);
            }
            console.log('Number of documents found: %d', obj.response.numFound);
        })
    });
    
### SSH tunneling

Assuming SSH tunnels have been set up in the following manner

    ssh -f -N -L 2181:my_zookeeper_node_1:2181 my_user@my_zookeeper_node_1
    ssh -f -N -L 2182:my_zookeeper_node_2:2181 my_user@my_zookeeper_node_2
    ssh -f -N -L 8080:my_solr_node_1:8080 my_user@my_solr_node_1
    ssh -f -N -L 8081:my_solr_node_2:8080 my_user@my_solr_node_2
    
The options' `ssh` field may be set as

    ssh: {
        tunnels: {
            'my_solr_node_1:8080': 'localhost:8080',
            'my_solr_node_2:8080': 'localhost:8081'
        }
    }

or alternatively in SSH style as

    ssh: {
        tunnels: '8080:my_solr_node_1:8080,8081:my_solr_node_2:8080'
    }
    
As well `zkConnectionString` in `options` must have the tunneled value of `'localhost:2181,localhost:2182'`.
    
Test
----

    npm test
