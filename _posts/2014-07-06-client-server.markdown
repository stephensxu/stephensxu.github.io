---
layout: post
title:  Client-Server Model
date:   2014-07-06
categories: technical
---

So how does the internet work? Well, obviously this is a huge topic that takes thousands of pages if we want to go in details. But here, I would like to describe the most high level understanding of the inner working of the internet, with the most commonly-understandable language.

A huge chunk of internet works off something called "Client-server model" (there are other type of architectures such as P2P, but the client-server is the most common model in present days). The internet is basically a huge network of clients and servers passing round commands and informations. Activities on the internet usually involves a client "talking" with the server, the server process the client request, then sends back response to the client. Informations are stored in databases, which are usually hosted somewhere on the server side.

<img src="{{ site.url }}/images/client-server.png" height="400" width="600" alt="">

The clients, such as our own laptops and mobile phones, sends request to the servers via Internet Protocols such as HTTP and HTTPS. Protocols are rules ensuring every client and server speaks the same language so there is no misunderstanding. There are many types of request that are being passed back and forth between server and clients, they usually involve information being exchanged between clients and the database. The server is just another computer with specialized software(eg.web applications) installed and host databases that can be accessed by clients or other servers.

The most common four types of requests are "GET", "POST", "PUT", "DELETE". "GET" request retrieve information from the database; "POST" request create new entry to the database. "PUT" request modifies an entry in the database, "DELETE" request removes an entry in the database.

So that's basically a very high level overview of how client-server model works. You can think of it as a huge network of computers "talking" with each-other, passing around commands and sharing information. Of course the internet is actually much more complex than described above, and it has had monumental impact on human society since its birth.

Questions? Email me at stephensxub@gmail.com.

<a href="{{ site.url }}">Back Home</a>