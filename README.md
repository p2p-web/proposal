# H1 A server in your pocket

The proliferation of inexpensive connected devices has created a situation where a person, at any given moment, is surrounded by multiple interactive computers: Personal and other people devices or appliances with built in processors and network capabilities. Today, there's a limited number of ways that developers can take advantage of all the devices in user's proximity. The few options available are locked in vendor specific APIs. The browser can be the platform agnostic channel to build interactions that take advantage of multiple devices and screens.

This document contains a proposal for a high level API and security model for devices to consume and expose services and manage the connections with other devices. Careful attention has been paid to the API ergonomics to make sure experiences that span multiple devices align with Web principles and developers knowledege and expectations.

## H2 Exposing and consuming services
Web clients consume and servers expose services using HTTP as the transport layer. We want to keep this model for multi device Web applications. We just provide a mechanism for user agents to expose HTTP services to other user agents.

## H2 Managing connections (security model)
We want to provide a secure mecanism to facilitate the management of cross device relationships. It has to be clear and easy to use for both users and developers.

## H2 Advertising and discovering services
Asuming that all devices are connected to the same network we need a way to find and advertise services to other devices.

## H2 Network setup
In many cases we can't assume that a network where all the devices are connected exists. We need a way to create networks that devices can freely join and leave without interrupting the connections of the devices already connected