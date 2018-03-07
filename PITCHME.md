# g**RPC**

### A modern RPC framework

---

#### HTTP/2.0, ProtoBuf...

### ...but what so special with it?

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

