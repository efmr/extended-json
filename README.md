﻿# Extended-JSON

[Extended JSON][ejson] forked from [mongodb-js/extended-json.git][github] which is a spec of [MongoDB Extended JSON][ejson] parse and stringify that is friendly with [bson][bson] and is actually compliant with the [kernel][json_cpp]. This fork extends the types to $boolean and $number to protects against coerced numbers and booleans to strings (eg. [Redis][redis]) of inflated objects values.

## Install

Install as a submodule in your project
`git submodule add git@github.com:efmr/extended-json.git lib/extended-json`

## Example

```javascript
var EJSON = require('./lib/extended-json');
var BSON = require('bson');

var doc = {
  _id: BSON.ObjectID(),
  last_seen: new Date(),
  display_name: undefined,
  count: 12.3e-10,
  isValid: true
};

console.log('Doc', doc);
// > Doc { _id: 53c2ab5e4291b17b666d742a,
//  last_seen: Sun Jul 13 2014 11:53:02 GMT-0400 (EDT), 
//  display_name: undefined, 
//  count: 12.3e-10,
//  isValid: true }

console.log('JSON', JSON.stringify(doc));
// > JSON {"_id":"53c2ab5e4291b17b666d742a","last_seen":"2014-07-13T15:53:02.008Z",\
// "count":12.3e-10,"isValid":true}

console.log('EJSON', EJSON.stringify(doc));
// > EJSON {"_id":{"$oid":"53c2ab5e4291b17b666d742a"},"last_seen":{"$date":1405266782008},\
// "display_name":{"$undefined":true},"count":{$number:12.3e-10},"isValid":{$boolean:true}}

// And likewise, EJSON.parse works just as you would expect.
// Even if numbers or booleans are coerced into string (Redis)
EJSON.parse('{"_id":{"$oid":"53c2ab5e4291b17b666d742a"},"last_seen":{"$date":1405266782008},\
"display_name":{"$undefined":true},"count":{"$number":"12.3e-10"},"n":123,\
"isValid":{$boolean:"true"},"isParsed":false}');
// { _id: 53c2ab5e4291b17b666d742a,
//  last_seen: Sun Jul 13 2014 11:53:02 GMT-0400 (EDT),
//  display_name: undefined ,
//  count: 12.3e-10,
//  n: 123 ,
//  isValid: true,
//  isParsed: false}

console.log('Inflated Object', EJSON.inflate(doc));
// > Inflated Object { _id: { $oid: 53c2ab5e4291b17b666d742a },
//  last_seen_at: { $date: 1405266782008 },
//  display_name: { $undefined: true},
//  count: { $number: 12.3e-10 } ,
//  isValid: { $boolean: true} }
```

### Streams

```javascript
var request = require('request');
var url = 'https://cdn.rawgit.com/imlucas/mongodb-extended-json/master/test/data.json';
var EJSON = require('./lib/extended-json');
var util = require('util');
var es = require('event-stream');

request.get(url)
  .pipe(EJSON.createParseStream('*'))
  .pipe(es.through(function(doc){
    this.emit('data',
      util.format('_id `%s` has timestamp %s\n',
        doc._id, doc._id.getTimestamp()));
  }))
  .pipe(process.stdout);

// Prints out:
//
// _id `54a1e4c798c1120000bb8b0f` has timestamp Mon Dec 29 2014 18:33:27 GMT-0500 (EST)
// _id `54a1e42f94042000008ac2f3` has timestamp Mon Dec 29 2014 18:30:55 GMT-0500 (EST)
// _id `54a1e3e68f038b0000631fe2` has timestamp Mon Dec 29 2014 18:29:42 GMT-0500 (EST)
// _id `54a1e3bc124968000052af15` has timestamp Mon Dec 29 2014 18:29:00 GMT-0500 (EST)
// _id `54a1e3a8124968000052af14` has timestamp Mon Dec 29 2014 18:28:40 GMT-0500 (EST)
// _id `54a1e2117b93d000002ee9bb` has timestamp Mon Dec 29 2014 18:21:53 GMT-0500 (EST)
// _id `54a1e1d6360382000019c110` has timestamp Mon Dec 29 2014 18:20:54 GMT-0500 (EST)
// _id `54a1e12ea5a5bb0000561dda` has timestamp Mon Dec 29 2014 18:18:06 GMT-0500 (EST)
// _id `54a1e0c14f3dc50000ba1afc` has timestamp Mon Dec 29 2014 18:16:17 GMT-0500 (EST)
// _id `54a1df90edbfd100001943b3` has timestamp Mon Dec 29 2014 18:11:12 GMT-0500 (EST)

```

[ejson]: http://docs.mongodb.org/manual/reference/mongodb-extended-json/
[github]: https://github.com/mongodb-js/extended-json
[redis]: http://redis.io/
[bson]: http://github.com/mongodb/js-bson
[json_cpp]: https://github.com/mongodb/mongo/blob/master/src/mongo/db/json.cpp
