# A server in your pocket

The proliferation of inexpensive connected devices has created a situation where a person, at any given moment, is surrounded by multiple interactive computers: Personal and other people devices or appliances with built in processors and network capabilities. Today, there's a limited number of ways that developers can take advantage of all the devices in user's proximity. The few options available are locked in vendor specific APIs. The browser can be the platform agnostic channel to build interactions that take advantage of multiple devices and screens.

This document contains a proposal for a high level API and security model for devices to consume and expose services and manage the connections with other devices. Careful attention has been paid to the API ergonomics to make sure experiences that span multiple devices align with Web principles and developers knowledege and expectations.

## Exposing and consuming services
Web clients consume and servers expose services using HTTP as the transport layer. We want to keep this model for multi device Web applications. We just provide a mechanism for user agents to expose HTTP services to other user agents.

## Managing connections (security model)
We want to provide a secure mecanism to facilitate the management of cross device relationships. It has to be clear and easy to use for both users and developers.

## Advertising and discovering services
Asuming that all devices are connected to the same network we need a way to find and advertise services to other devices.

## Network setup
In many cases we can't assume that a network where all the devices are connected exists. We need a way to create networks that devices can freely join and leave without interrupting the connections of the devices already connected

## Discovery

### HTTP GET

```js
var query = { type: 'json/contact' };

navigator.service.discover(query).then(callback);
navigator.service.watch(query).then(callback);

function callback(matches) {
  var service = matches[0];
  
  service.connect().then((connection) => {
    var url = serivice.endpoints.get; // http://diego-phone.local/contacts-app/
    var request = new XMLHttpRequest();
    request.open('get', url, true);
    request.send();
    
    request.onload = (e) => {
      console.log(request.responseText);
      connection.close();
    };
  });
});
```

```js
navigator.service.discover({ type: 'image/*' }).then((imageServices) => {
  
  imageServices.map((service) => {
    var request = new XMLHttpRequest();
    request.open('get', service.endpoint, true);
    request.send();
    return request;
  }).then((result) => {
    console.log(result);
  });
});
```

### Watching Services

```js
var watcher = navigator.service.watch('fitbit.com');

watcher.addEventListener('change'
```

### Real-time Socket

```js
navigator.service.discover({ uri: 'fitbit.com' }).then((matches) => {
  var fitbitService = matches[0];
  var socket = new WebSocket(fitbit.endpoints.heartrate);
  
  socket.onmessage = (event) => {
    console.log(event.data);
  };
});
```

## Registration

### HTTP Routes

```js
navigator.service.register.get('/data', function(req, res) {
  res.send({ some: 'data' });
});

navigator.service.register.post('/upload', function(req, res) {
  // ...
});
```

### Socket

```js
navigator.service.register.socket('/heartbeat', {
  onRequest: function(request) {
    var connection = request.accept('accepted', request.origin);
  
    connection.onmessage = (event) => { ... };
    connection.onclose = (event) => { ... };
  }
});
