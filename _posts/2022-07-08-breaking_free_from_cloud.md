---
layout: article
title: Cloud, meet Desktop - a system design case study
tags: system design
mode: immersive
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: '#203028'
  background_image:
    src: https://uploads-ssl.webflow.com/5e1f17bab0dc6527c1ecc801/5e835dd88d500a6603ab9a0c_computing-2-p-1600.jpeg
---


🧑‍💻 System Design Challenge
--------------------------

Let's look at an interesting system design challenge that I recently came across. Given an ordinary cloud system comprised of multiple services. These services communicate over HTTP using RESTful APIs. Your mission - [should you choose to accept it](https://youtu.be/KlyLtJd-ArY?t=173) - is to integrate the services of an external provider with the system. The provider had two offerings to integrate with. A modern RESTful API over HTTP and a desktop app providing a socket API.

It goes without question that from the perspective of a cloud system, the desktop app presents challenges that make the REST API route be the one without much consideration. Or so I thought... 

📦 SDK vs API
----------

As an online service provider, one has a handful of options to cater for customers who wish to interact with your service in a programmatic fashion. Two common approaches are an application programming interface (API) or a software development kit (SDK). Both has benefits and tradeoffs. Let's take a look at them one by one.

The benefits of an API is that you only have to implement it in a single programming language. Therefore, it is usually the more economical choice. The downside, on the other hand, is that the clients wishing to interact with the API will have to implement the client side part for themselves. Quality documentation as well as a well tested implementation is essential. Otherwise, your help desk will be flooded with questions or worse your API users will look for another provider.

The benefits of an SDK is that your clients will _thank you_. It is the white glove service of the internet. You implement and give client side software to the users of your API or services that turn complex network I/O, authentication, data representation and transfer into simple function calls. Coupled with decent documentation, you will be lightyears ahead of your competition that only provides an API but no SDK. The downside, is that it takes more effort. You have to select the client side software languages you want to support and implement an SDK for _all_ of them. 

🖥️ Desktop API
-----------

...continuing my story of the integration with the external provider. For reasons outside my reach it turned out that the integration via the RESTful API is a dead end. So what do you do when the task at hand suddenly demands to integrate the services of a cloud provider with your own cloud system via a _desktop app_?! Since your whole system runs in the cloud, the first thought would be to: 
- deploy a virtual server with a desktop OS on it in the cloud
- install the desktop app on it
- open the right ports 
- and call it a day. 

However, what if a user has to interact with the desktop app, say regularly log in? Remote desktop, VPN and all the extra hoops and complexity. Can we keep do something simpler?

🆚 Desktop vs. Server
------------------

Desktop and server are both just fancy terms for computers. There is usually one crucial difference between the two. When you think about your laptop as a desktop, you run around with it all day long. Connecting to the internet from a myriad of locations: home, office, coffee shop. In all these locations, chances are your laptop is assigned a new IP address. Even if your computer is a true desktop, changes are your home internet provider did not assign you a permanent IP address. A server, on the other hand, usually earns the title _server_ by having a permanent IP address assigned to it. If you know the address, you can connect to that server.

Why do I call out this difference? If we can deploy a piece of software on the desktop, it can initiate a connection to the server on one side. On the other side, it can establish a local socket connection with the desktop app of the service provider we are integrating with. This piece of software acts as a simple proxy between our two components. 

🔁 Bi-directional Communication
----------------------------

The only remaining question at hand is: what type of connection should the proxy app use to connect to the cloud system? As usual, it depends. And the question we have to answer is: How will data flow between the service provider (A) and our cloud system (B)? Since this is analogous with a cable there are really only two options. From A to B or from B to A. If the service provider sends data to the proxy app we can forward that data via simple HTTP requests. This makes sense since we mentioned that our cloud system is already using a RESTful communication. 
The more interesting scenario is the reverse one. What if we want to send data from our cloud system to the service provider? One approach is HTTP [long polling](https://en.wikipedia.org/wiki/Push_technology#Long_polling), but that is kind of an emulation. A more flexible approach is establishing a websocket connection. This gives us the option to communicate in both directions. The proxy app can initiate the connection to the cloud server and once the connection is established, the cloud server can send data to the proxy on demand. Here is a quick sketch of the setup:

![WebSocket Proxy](/assets/websocket_proxy.png)

🎁 Wrap up
----------

In hindsight this design seems trivial, but at the time it did not feel like one - when you code cloud server components all day long, the idea of a desktop app can seem far fetched - and definitely required some outside the box thinking. My favorite kind of thinking. Happy coding!
