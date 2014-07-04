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

Note:
Example: "I used Twitter's API to build a tool to see if anyone is bad-mouthing
me online."

Translation: I wrote a program that interacts with a program on Twitter's side
built to *interface* with Twitter's information systems.





## REST: Representational State Transfer

Way to interact with APIs over HTTP (the communication protocol the Internet is built on).

* "REST" was coined by Roy Fielding in his 2000 doctoral dissertation
Includes set of design principles and best practices for designing APIs meant to
be "RESTful".

Note: 
HTTP has become the de-facto way to make your systems available to others over
the public Internet





## HTTP: Hypertext Transfer Protocol

HTTP is just a messaging protocol 

Note: built on the *request/response* pattern.
Each request has an associated *method* that indicates the type of request the
client is making. 
It may also include data sent by the client (e.g. from a
filled in web form).




## Some HTTP Methods

* GET
* POST
* PUT
* PATCH
* DELETE

Note: GET: get a resource (page, image, video, etc); POST: create a new
resource; PUT/PATCH: create/update resources DELETE: delete resource




## Let's Build a RESTful API Service!

A REST API allows *clients* to send *HTTP requests* to manipulate *resources*.

...So we need to write a server capable of accepting HTTP requests and acting on
them.




## Hey! That Describes Every Web Application!

Yep. A REST API Service is just a web application and, as such, is built using
the same set of tools.

Note: All major web frameworks can be used to create a REST API service, with
various degrees of difficulty. Django has the django-rest-framework, for
example. We're going to talk about Flask as it's as closes to pure Python as we
can get.





## What is a Resource?

Earlier we said a REST API allows clients to manipulate *resources* via HTTP.
What is a resource?

Note: Resources are the nouns of a system. If we were twitter, "user" would be a
resource, as would "tweet". If that sounds similar to *models* in an ORM that
because they're basically the same thing. REST resources are usually represented
on the backed by a database table.




## Let's Play Pretend

Imagine we're Twitter weeks after launch. Ashton Kutcher seems to be able to use
it, but what about *other systems*?

That's right, we'll need to create a REST API. For now, we'll focus on two
resources:

* user
* tweet

In our system (and in REST APIs), resources each have their own unique ID. Our
pattern for building URLs will be `/resource_name[/resource_id[/resource_attribute]]`.

Note: The last two are optional. It will start with something like `/tweet`,
then might have `2348` as the tweet ID.

## Let's See Some Code!

    from twitter.models import Tweet, User

    @app.route('/tweets/<int:tweet_id>', methods=['GET'])
    def get_tweet(tweet_id):
        """Return the tweet with ID *tweet_id* as JSON."""
        tweet = Twitter.query.get(tweet_id)
        if tweet is None:
            response = jsonify({'result': 'ERROR'})
            response.status_code = 404
            return response
        else:
            return jsonify({
                'content': tweet.content,
                'posted_at': tweet.created_at,
                'posted_by': tweet.user.username})

Note: includes some ORM code (Twitter.query.get). That's SQLAlchemy, the main
Python ORM. `jsonify` is provided by Flask and turns a dictionary into a JSON
response object. Up to us to use proper HTTP status codes (404 means the
resource was not found, which is where the notion of 404 HTML pages originates
from).





## Let's See a POST!

    from twitter.models import Tweet, User

    @app.route('/tweets/', methods=['POST'])
    def create_tweet():
        """Create a new tweet object based on the JSON data sent in the request."""
        if not all(('content', 'posted_at') in request.json)a:
            response = jsonify({'result': 'ERROR'})
            response.status_code = 400
            return response
        else:
            tweet = Tweet(
                content=request.json['content'],
                posted_at=request.json['posted_at'],
                user=current_user)
            db.session.add(tweet)
            db.session.commit()
            return jsonify({
                'content': tweet.content,
                'posted_at': tweet.created_at,
                'posted_by': tweet.user.username})

Note: Here, we're creating a new Tweet. We create the ORM object then save it to
the database. If a field is missing from the JSON message, the request is
malformed.





## What About Seeing All Tweets?

In REST APIs, a group of resources is called a *collection*. REST APIs are
heavily built on the notion of resources and collections. In our case, a list of
all tweets is a *collection* of tweets. It's accessed by the URL for a resource
without any ID following it, i.e. `/tweets/`.





## How Do We Write That Code?

    from twitter.models import Tweet, User

    @app.route('/tweets/', methods=['GET'])
    def get_tweet_collection():
        """Return all tweets as JSON."""
        all_tweets = []
        for tweet in Tweet.query.all():
            all_tweets.append({'content': tweet.content, 'posted_at':
            tweet.posted_at, 'posted_by': tweet.user.username})

