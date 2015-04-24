# A server in your pocket

## Introduction

The proliferation of inexpensive connected devices has created a situation where a person, at any given moment, is surrounded by multiple interactive computers: personal and other peoples' devices or appliances with built in processors and network capabilities.

Today, there's no way for web-applications to utilize the devices in a user's proximity. Native applications on Android and iOS have these capabilities via propietary SDKs.

We believe the browser can become the platform agnostic channel to build interactions that take advantage of multiple devices and screens. This document contains a proposal for a high level API and security model for devices to consume and expose services and manage the connections of devices nearby.

Careful attention has been paid to the API ergonomics to make sure experiences that span multiple devices align with web principles and as well as developer knowledge and expectation.

## Consuming and exposing nearby services

HTTP is the universal language of the web. It is used by clients to consume, and servers expose services. We want to keep this model for multi-device web applications. Our proposition will enable the user-agent to discover and connect to 'nearby' services, as well as exposing its own.

## Use cases

#### Home media backup

Tom is an engineer from Berlin. He takes photos on his device everyday, so he frequently has to backup his photo to his desktop machine to free up storage space. Tom gets frustrated when he finds it hard to get at his own data  and doesn't like using third-party services.

Fortunately for Tom Firefox OS is a hackable phone and exposes all its content via secure HTTP endpoints. With a nodejs server and a few NPM modules, Tom is able to write a script that backs-up his photos to his desktop whenever he's at home.

He shares his app on Github and finds many other devlopers start to use and contribute. It quickly develops from a quick hack to an installable desktop app anyone can use.

#### Local chat

Sally and Peter live in the countryside and ride the same bus to school each morning. The bus is busy, so they often don't get seats next to each other. The want to chat about the cliffhanger from last night's series finale but network signal is patchy and messages often don't send.

Sally tells Peter to visit p2pmessenger.com before they next ride the bus. The next day Peter pulls out his phone and searches the nearby area for other p2pchat users, the search returns one result: "Sally's Flame". Peter taps to connect, Sally gets a notification: "Peter's Tarako wants to connect with p2pchat.com". She accepts and they chat away, no internet required.

#### Multiplayer gaming

George, Phil and Clare all enjoy playing games on their phones, but because they are on different operating systems they don't always have access to the same games.

Phil finds a great racing game on a web-app marketplace and sends Clare and George a link. They quickly all have the game running and Phil initiates a multi-player game. Clare and George scan the local area for active multi-player games and quickly find "Phil's phone". With Phil's phone acting as the central real-time server they begin racing each other around the track.

#### Other

- Ability to serve a remote client to the local network from a device (eg. remote SMS app being served to IE6 on the same LAN).
- Apps can expose specific data endpoints to other nearby devices (eg. gallery serving images/videos, a contacts app exposing contact JSON/vcard).
- Uploading (pushing) files to another device (eg. sharing media between devices).
- Communicate with existing non-web remote hardware (eg. FitBit).
- Subscription to uni-directional 'server' push (eg. low-energy heart-rate monitor, smart-watch displaying arbitrary notifications from phone).
- Bi-directional socket connection between two apps (eg. multi-player gaming, local messaging, remote controlling a flying drone).

## Transport agnostic

Wireless transport approaches are changing at a rapid pace. At Mozilla our approach to date has been to spec and expose transport mechanisms one-by-one as Web APIs in the browser.

Although 'certified' and sometimes 'privileged' FirefoxOS apps have access to some communication APIs, little progress has been made to land them in Firefox Desktop, Firefox Android or other web-platforms. This greatly limits their potential/usefulness.

Our proposal aims to provide an abstraction on top of underlying transport types to provide web-applications with a means to search for and connect to nearby 'services', ignorant of the communication mechanism.

[insert google doc quote]

Initially we would suggest that the user-agent scan for IP based services advertising on LAN, Wifi-Direct and Bluetooth (PAN). As future transport types emerge they could be added without the need for new Web-APIs which often take years to become widely usable (if ever).

#### This transport abstraction should:

  - Allow developers to easily **leverage complex communication channels**
  - **Mitigate risk** around exposing low-level network APIs via a secure browser abstraction.
  - Empower browsers to **immediately adopt future transport types**, side-stepping painfully slow specification process.

## Limitations of existing discovery API proposals

### Service Discovery API

There is already a [draft spec]() for a service discovery API in the browser. The API is almost identical to what we are proposing for discovery, but it only provides discovery of devices on a pre-existing LAN.

We'd like to expand upon this by adding *Wifi-Direct* and *Bluetooth* service discovery. For these transports we'd need to create a network before passing back the chosen `NetworkService`, object so that advertised HTTP endpoints can be reached.

The user-agent would also have to 'dedupe' identical services that may have been advertised on more than one transport type.

### Presentation API (2 UA)

Again this spec only covers the discovery of devices already on the same LAN network.

## Discovery & connection

The most important thing to note about our service discovery/connection proposition is that the app developer never specifies a desired transport type or discovery protocol.

The first iteration of this API would only be able to query services by a unique service-id (eg. the service's URL). In later iterations it could be possible to query for a more abstract service type (similar to web-activity types in FirefoxOS), but this would be out of scope for the initial proof of concept.

1. App queries for a service-name id via `navigator.service.discover`
2. Trusted user-agent UI takes over to allow user to pick their service (think `<input type="file">`)
3. Once the user has chosen the service they wish to connect to, the `.discover()` promise is resolved with a `Service` object.
4. The app chooses once of the service endpoints to hit. This endpoint could be HTTP (`GET`, `POST`, `PUT`, `DELETE`) or some kind of socket (eg. Websocket, EventSource).

```js
navigator.service.discover('contacts.gaiamobile.org').then(service => {
  var request = new XMLHttpRequest();

  // GET http://<device-ip>:<app-port>/
  request.open('get', service.url, true);

  request.send();
  request.onload = e => {
    var contacts = JSON.parse(request.responseText);
    console.log(contacts);
    service.close();
  };
});
```

## Exposing a service

We're all too used to our services living in the cloud. Even to communicate with the person next to you it requires network connectivity, lengthy round-trips and sometimes costly data.

Exposing a services in the browser is relatively unheard of. In order to make it commonplace we'd need to find a way to securely expose a TCP-like socket in the browser. A raw TCP Socket API (`mozTCPSocket`) may be available to 'privileged' apps, but this doesn't provide a solution for desktop or browsers other than Firefox.

Web apps would be able to use this extensible low-level API to build exciting abstractions. Our original plan was to provide full HTTP and WebSocket server APIs in the browser, but we came to the conclusion that this would require the specification and approval of a very large API surface area. This would be unlikely to be approved as it involves baking high-level/inextensible APIs into the web-platform (see [open web manifesto]).

The real question to solve is: *How can we allow client-side web-applications to listen and respond to incoming connections on a device's port in a secure enough way to expose to the open-web?*

```js
navigator.port.onconnection = connection => {
  connection.send('ping');
  connection.close();
});
```

This low-level unopinionated Web API will enable JavaScript libraries implement existing (and future) TCP/IP based protocols, moving innovation off the slow standards track and into agile 'user-land'.

### How can we trust incoming connections?

For this to be secure we'd require additional trusted user-agent intervention to decide whether to allow/deny incoming requests to an application. This could come in the form of:

- Requiring a token/key assigned by the user agent be part of every request.
- Providing a way the user can open their device to allow incoming connections for a short period of time to 'pair' (like WPS mode on routers).
- Requiring user grant access via a user-agent prompt on connection attempts *"A device nearby ([ip-address]) wants to connect with someapp.com"*. Although this is tricky as using HTTP headers alone we don't have much information to show the user. Although it's worth considering 90% of the time the user, being nearby, is likely playing a part in initiating the connection, so will understand/be-expecting the prompt.

Even if we can't decide on the perfect solution straight away we can start with the most secure option (probably something private key related) and iterate from there.

## Advertising a service

The socket based API would allow devices to expose a service, but without an advertisement, this wouldn't be discoverable by nearby devices. The Web API might look something like:

```js
  navigator.service.advertise(<unique-service-name>);
  navigator.service.stopAdvertising(<unique-service-name>);
```
Again we'd propose that this be an abstraction on top of whatever transport types the user-agent may have at its disposal.

Under the hood the user-agent would advertise on LAN (*mDNS/uPnP/DIAL*), Wifi-direct and Bluetooth to reach as many devices as possible. For example if Device A only has *Bluetooth* turned on, it is still able to discover Device B which is advertising on LAN, *Bluetooth* and *Wifi-Direct*.

## References

* [W3C Working Draft - Network Service Discovery](http://www.w3.org/TR/discovery-api/)
