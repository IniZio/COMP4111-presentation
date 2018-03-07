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
const dnode = require('dnode');

const server = dnode({
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

```js
const dnode = require('dnode');

const d = dnode.connect(5004);
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

<p><span class="slide-title">gRPC (helloworld.proto)</span></p>

```protobuf
syntax = "proto3";

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
const grpc = require('grpc');
const { helloworld: hello_proto } = grpc.load('./helloworld.proto');

function main() {
  const server = new grpc.Server();
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

<p><span class="slide-title">gRPC (client.js)</span></p>

```js
const grpc = require('grpc');
const { helloworld: hello_proto } = grpc.load('./helloworld.proto');

const client = new hello_proto.Greeter(
    'localhost:50051',
    grpc.credentials.createInsecure()
);
const user = process.argv[2] || 'world';

client.sayHello({name: user}, function(err, response) {
    console.log('Greeting:', response.message);
});
```

@[4-7](Connect with server)

@[10-12](Call a procedure on server)

@[2, 4](Loads the proto too)

---

### Why ProtoBuf?

---

<p><span class="slide-title">ProtoBuf</span></p>

```protobuf
// Unary response, as configured by the request.
message Response {
  required int32 id = 1;
  // The user the request came from, for verifying authentication was
  // successful.
  string username = 2;
  // OAuth scope.
  string oauth_scope = 3;
}
```

<p><span class="slide-title">JSON</span></p>

```json
{
    "id": 1324234,
    "username": "tom",
    "oauth_scope": "public, email"
}
```

---

Advantages:

- Schemas are awesome

- Versioning:

  ```protobuf
  int32 id = 1;
  // later...
  string responseId = 4
  ```

- Validations: 

  ```protobuf
  required int32 id = 1;
  ```

---

- Even better performance
  ![](https://cdn.auth0.com/blog/protobuf-json/java-times.png)

---

#### Other advantages of gRPC:

- HTTP/2.0: header compression, multi-resourse request
- Extendable:
  - Can use JSON / XML,
  - Security, health-checking, load-balancing and failover, monitoring, tracing, logging ....
- Multi-language: C++, Java, Objective-C, Python, Ruby, Go, C#, Node.js

---

#### Scenerios for gRPC:

- Low latency, highly scalable, distributed systems.
- Developing mobile clients which are communicating to a cloud server.
- Need a protocol that needs to be accurate, efficient and language independent.
- Want fully extendable system eg. authentication, load balancing, logging and monitoring etc.

