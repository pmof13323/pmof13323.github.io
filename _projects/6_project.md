---
layout: page
title: Weather Data Distributed System
description: client/server system
img: assets/img/yaba.jpg
importance: 1
category: engineering
---

In this project, which has been my first introduction into Distributed Systems, I created a distributed weather data system to gain an understanding of what is required to build a client/server system, and to build a system that aggregates and distributes weather data in JSON format using a RESTful API. This project was also my first venture with java, where I learned about and implemented sockets and multi-threaded systems.

Here is a short summary of implemented functionalities:
- Text sending
- client, server and content server processes start up and communicate
- PUT operation works for one content server
- GET operation works for many read clients
- Aggregation server expunging expired data after 30 seconds
- Retry on errors (server not available etc)
- Lamport clocks are implemented
- Critical error codes are implemented
- Content servers are replicated and fault tolerant

Failure management has been applied in many parts of the program, here are (just a few of the many) measures taken as a starting point:
- if incorrect id is inputted in a client it is handled safely and user is asked again in correct form
- if server is not available client and content servers retry
- if there is no data correct error codes are sent to relevant components indicating the issue safely
- if data storage files (between all components) are not found, the system creates them
- if user does not specify ID and the specific headers designed (to maintain lamport clocks) are not sent, default parameters are used such that they still receive correct data (browser connections for example) hence implementing a successful RESTful API
- in case of system failure, data is stored in an intermediate text file so data is not lost

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/yaba.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    A short code snippet from the aggregation server, the intermediary server in charge of running requests and maintaining the true lamport clock value.
</div>

Lamport Clock implementation:
The aggregation server, content server and client all hold, and are capable of maintaining their own lamport clock. Lamport clocks are ticked upon each request sent, and each request received. The lamport clock request ordering system inside of my program works as follows:
When a content server and/or a client are opened, they make a get request to the aggregation server, receiving the lamport clock value of the aggregation server and setting their lamport clocks accordingly. Upon sending a request (from a client or a content server) to the aggregation server, the aggregation server ticks its own lamport clock, if necessary, corrects incoming timestamp values to match the current correct clock/request order. Note that timestamps are sent to the aggregation server from clients and content servers via request headers. The response sent back to the client or server sending a request includes a timestamp header, which the respective server or client uses to update its own lamport clock prior to the next request. These requests once arriving into the aggregation server are turned into a request abstract data type that holds the request as well as it's timestamp (timestamp of the client and/or content server). These requests, with their timestamp are added to a priority queue which is fundamentally responsible for maintining a correct request order. A request processing thread is always running when an aggregation server is on, which (if the priority queue is not empty) polls the next request in order of lowest timestamp, and runs it accordingly, hence ordering system requests appropriately.

Testing information:

A variety of effective and versatile testing systems have been developed to ensure system performance.
- Error catching: Automated error catching can be found throughout the various files.
- Integration testing: Integration testing has been/is done through user interaction with the system. 
- Aggregation server unit testing, Content server unit testing, Client unit testing and Lamport Clock unit testing.

Testing Harness for Multiple Distributed Entities:

- A lamport clock system (refer to lamport clock section) has been used in order to maintin order and synchronisation between the interactions of the distributed entities
- Multithreading is used throughout the program, successfully simulating the interactions between multiple clients and servers
- Concurrent put and get requests were tested

Synchronization and fault testing:

A lamport clock system (refer to lamport clock section) has been used in order to maintain order and synchronisation between the interactions of the distributed entities
Request timestamps are dealt with appropriately (refer to lamport clock section). Incorrect order request runs are caught by the system, and the user is informed in the terminal running the aggregation server that an incorrected order has been detected gracefully
Edge cases.
