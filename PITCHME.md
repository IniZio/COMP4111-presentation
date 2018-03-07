# g**RPC**

### A modern RPC framework

---

## What so special with it?

Let's see how current RPC looks like

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

@title[RPC in server]

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

