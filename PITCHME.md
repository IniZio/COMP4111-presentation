# g**RPC**

### A modern RPC framework

---

#### HTTP/2.0, ProtoBuf...

### ...but why does it even matter?

<br />

Let's see how current RPC looks like

---

@title[RPC in server]

<p><span class="slide-title">RPC (server.js)</span></p>

```js
var dnode = require('dnode');

var server = dnode({
    transform : function (s, cb) {
        cb(s.replace(/[aeiou]{2,}/, 'aa').toUpperCase())
    }
});
server.listen(5004);
```

@[4-6](Procedures that server handles)

@[8-8](Listen to port)

---

@title[RPC in client]

<p><span class="slide-title">RPC (client.js)</span></p>

```js
const dnode = require('dnode');

var d = dnode.connect(5004);
d.on('remote', function (remote) {
    remote.transform('beep', function (s) {
        console.log('beep => ' + s);
        d.end();
    });
});
```

@[3-3](Connect with server)

@[5-8](Call a procedure on server)

---

### What if....

- ... we do not want to use Node.js for either side?
- we want to know what kind of parameters are allowed?
- the procedure is blocking?

---

#### How gRPC is like

@title[Protobuf]

```protobuf
syntax = "proto3";

option java_multiple_files = true;
option java_package = "io.grpc.examples.helloworld";
option java_outer_classname = "HelloWorldProto";
option objc_class_prefix = "HLW";

package helloworld;

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

@[16-19](Request parameters)

@[21-24](Reply parameters)

@[10-14](A  service to greet)

---

@title[gRPC in Server]

<p><span class="slide-title">gRPC (server.js)</span></p>

```js
var grpc = require('grpc');
var { helloworld: hello_proto } = grpc.load('./helloworld.proto');

function main() {
  var server = new grpc.Server();
  server.addService(hello_proto.Greeter.service, {
      sayHello (call, callback) {
        callback(null, {message: 'Hello ' + call.request.name});
      }
  });
  server.bind('0.0.0.0:50051', grpc.ServerCredentials.createInsecure());
  server.start();
}

main();
```

@[7-9](Procedure that server handles)

@[12-13](Listen to port)

@[2, 6](Loads proto into server service)

---

@title[gRPC in Client]

<p><span class="slide-title">RPC (client.js)</span></p>

```js
var grpc = require('grpc');
var { helloworld: hello_proto } = grpc.load('./helloworld.proto');

var client = new hello_proto.Greeter(
    'localhost:50051',
    grpc.credentials.createInsecure()
);
var user = process.argv[2] || 'world';

client.sayHello({name: user}, function(err, response) {
    console.log('Greeting:', response.message);
});
```

@[4-7](Connect with server)

@[10-12](Call a procedure on server)

@[2, 4](Loads the proto too)