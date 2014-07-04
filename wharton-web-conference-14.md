## Automated Building of RESTful API Services in Python

* Jeff Knupp
* @jeffknupp
* jeff@jeffknupp.com
* Wharton Web Conference 2014




## Jeff Knupp?

* Author of “Writing Idiomatic Python”
* Full-time Python developer @ AppNexus
* Blogger at jeffknupp.com
* Creator of the “sandman” Python library




## Let's Talk About REST APIs

Seven letters. Two acronyms. Buy what does it mean?




## API: Application Programming Interface

Programatic way of interact with a third-party system.

Example: "I used Twitter's API to build a tool to see if anyone is bad-mouthing
me online."

Translation: I wrote a program that interacts with a program on Twitter's side
built to *interface* with Twitter's information systems.





## REST: Representational State Transfer

Way to interact with APIs over HTTP, the communication protocol the Internet is built on.

HTTP has become the de-facto way to make your systems available to others.

* "REST" was coined by Roy Fielding in his 2000 doctoral dissertation




## HTTP: Hypertext Transfer Protocol

HTTP is just a messaging protocol built on the *request/response* pattern.
Each request has an associated *method* that indicates the type of request the
client is making. It may also include data sent by the client (e.g. from a
filled in web form).




## Some HTTP Methods

* GET: retrieve a resource (e.g. a web page, an image, a video)
* POST: send a new resource to the system (i.e. creating a new account by
    filling out a web form)

#### Lesser Used Methods

* PUT: send full snapshot for a resource update (might be new, might update an exisitng
    resource)
* PATCH: send only changed fields for existing resource
* DELETE: delete a resource
