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

* "REST" was coined by Roy Fielding in his 2000 doctoral dissertation.
Includes set of design principles and best practices for designing systems meant to
be "RESTful".

Note: 
HTTP has become the de-facto way to make your systems available to others over
the public Internet





## REST and HTTP

HTTP is just a messaging protocol. Happens to be the one the Internet is based
on. 

RESTful systems use this protocol to their advantage e.g. caching resources

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

...So we need to write a server capable of accepting HTTP requests, acting on
them, and returning HTTP responses.




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

Note: Resources are the nouns of a system. If we were twitter, "tweet" would be a
resource, as would "user". If that sounds similar to *models* in an ORM that
because they're basically the same thing. REST resources are usually represented
on the backed by a database table.




## Let's Play Pretend

Imagine we're Twitter weeks after launch. Ashton Kutcher seems to be able to use
our service, but what about *other systems*?

That's right, we'll need to create an API. Being an internet company, we'll
build a REST API service. For now, we'll focus on two
resources:

* user
* tweet





## Unique IDs and URL structures

All resources must be identified by a unique address at which
they can be reached, their URI. This usually requires each resource
contain a unique ID, usually a monotonically increasing integer or
UUID.

Our pattern for building URLs will be `/resource_name[/resource_id[/resource_attribute]]`.

Note: The last two are optional. It will start with something like `/tweet`,
then might have `2348` as the tweet ID.





## Let's See Some Code!

Here we define our resources is a file called `models.py`:


```python
class User(db.Model, SerializableModel):
    __tablename__ = 'user'

    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String)


class Tweet(db.Model, SerializableModel):
    __tablename__ = 'tweet'

    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.String)
    posted_at = db.Column(db.DateTime)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    user = db.relationship(User)
```

Note: We're using SQLAlchemy as our ORM. It's by far the most popular Python ORM
and plays well with Flask





## Smarter Resources Through Magic

```python
class SerializableModel(object):
    """A SQLAlchemy model mixin class that can serialize itself as JSON."""

    def to_dict(self):
        """Return dict representation of class by iterating over database
        columns."""
        value = {}
        for column in self.__table__.columns:
            attribute = getattr(self, column.name)
            if isinstance(attribute, datetime.datetime):
                attribute = str(attribute)
            value[column.name] = attribute
        return value
```

Note: Here we're adding a mixin class that will allow our resources to return a
dictionary representation of themselves, which can easily be transformed to
JSON. This uses database reflection, a technique that will become a lot more
important later on.





## Show Me a GET

Here's the code that handles retreiving a single tweet and returning it as JSON:

```python
from models import Tweet, User

@app.route('/tweets/<int:tweet_id>', methods=['GET'])
def get_tweet(tweet_id):
    tweet = Tweet.query.get(tweet_id)
    if tweet is None:
        response = jsonify({'result': 'error'})
        response.status_code = 404
        return response
    else:
        return jsonify({'tweet': tweet.to_dict()})
```

Note: `jsonify` is provided by Flask and turns a dictionary into a JSON
response object. Up to us to use proper HTTP status codes (404 means the
resource was not found, which is where the notion of 404 HTML pages originates
from).




## OK, But Does it Blend?

Let's `curl` our new API (preloaded with a single tweet):

```bash
$ curl localhost:5000/tweets/1
{
  "tweet": {
    "content": "This is awesome",
    "id": 1,
    "posted_at": "2014-07-05 12:00:00",
    "user_id": 1
  }
}
```




## How Do We Add a Tweet?

```python
from models import Tweet, User

@app.route('/tweets/', methods=['POST'])
def create_tweet():
    """Create a new tweet object based on the JSON data sent
    in the request."""
    if not all(('content', 'posted_at', 'user_id' in request.json)):
        response = jsonify({'result': 'ERROR'})
        response.status_code = 400  # HTTP 400: BAD REQUEST
        return response
    else:
        tweet = Tweet(
            content=request.json['content'],
            posted_at=datetime.datetime.strptime(
                request.json['posted_at'], '%Y-%m-%d %H:%M:%S'),
            user_id=request.json['user_id'])
        db.session.add(tweet)
        db.session.commit()
        return jsonify(tweet.to_dict())
```

Note: Here, we're creating a new Tweet. We create the ORM object then save it to
the database. If a field is missing from the JSON message, the request is
malformed (that's what the `if not all` part does).





## What About Seeing All Tweets?

In REST APIs, a group of resources is called a *collection*. REST APIs are
heavily built on the notion of resources and collections. In our case, 
the *collection* of tweets is a list of all tweets in the system. 

The `tweet` collection is accessed by the following URL (according to our rules,
described earlier): `/tweets/`. 

Note: Looks like a folder on a filesystem. Not a coincidence.





## How Do We Write That Code?

```python
    from twitter.models import Tweet, User

    @app.route('/tweets/', methods=['GET'])
    def get_tweet_collection():
        """Return all tweets as JSON."""
        all_tweets = []
        for tweet in Tweet.query.all():
            all_tweets.append({'content': tweet.content, 'posted_at':
            tweet.posted_at, 'posted_by': tweet.user.username})
```





## This Code Is Boring

All the code thus far has been pretty much boilerplate. Every REST API you write
in Flask (modulo business logic) will look identical. How can we use that to our
advantage?

Note: While different applications may need to add a bit of business logic to
the GET and POST code, it's usually pretty minimal and ends up looking almost
identical to what we have here.





## It's 2014...

We have self-driving cars and delivery drones, why can't we build REST APIs
automatically?

Note: I don't like boring code. I'm of the opinion that boring code should write itself.
Once you've defined your resources, it should be possible to just hit a
button and BLAM!, RESTful API service created. 





## Whenever Possible, Code Should Write Itself

This allows one to work at a higher level of abstraction. Solve the problem once
in a general way and let code generation solve each individual instance of the
problem.

* Part of *The Unix Philosophy*




## Enter `Flask-Sandboy`

Third party Flask extension written by the dashing Jeff Knupp.
Define your models. Hit a button. BAM! RESTful API service that *just works*.

(The name will make more sense in a few minutes)

Note: This isn't meant to create the REST API for Twitter or GitHub, both of
which are complex and expertly designed. This is meant to create the REST API
for XYZ corp who just needs to get it out the door quickly.





## How Doew It Work?

Generalizes REST resource handling into notion of a *Service* (e.g. the "Tweet
Service" handles all tweet-related actions).

```python
class Service(MethodView):
    """Base class for all resources."""

    __model__ = None
    __db__ = None

    def get(self, resource_id=None):
        """Return response to HTTP GET request."""
        if resource_id is None:
            return self._all_resources()
        else:
            resource = self._resource(resource_id)
            if not resource:
                raise NotFoundException
            return jsonify(resource.to_dict())
```

Note: take note of the __model__ and __db__ class attributes. The latter is just
a handle to the database. __model__ is the interesting part and will be defined
later.




## How Does It Work (continued)

```python
def _all_resources(self):
    """Return all resources of this type as a JSON list."""
    if not 'page' in request.args:
        resources = self.__db__.session.query(self.__model__).all()
    else:
        resources = self.__model__.query.paginate(
            int(request.args['page'])).items
    return jsonify(
        {'resources': [resource.to_dict() for resource in resources]})
```

Note: `page` is used for pagination, or limiting the number of results returned
and providing a `next` and `previous` link to get the next or previous set of
results.





## How Does It Work (continued)

Here's how POST works. Notice the `verify_fields` decorator and use
of `**request.json` magic...

```python
    @verify_fields
    def post(self):
        """Return response to HTTP POST request."""
        resource = self.__model__.query.filter_by(**request.json).first()
        if resource:
            return self._no_content_response()
        instance = self.__model__(**request.json)
        self.__db__.session.add(instance)
        self.__db__.session.commit()
        return self._created_response(instance.to_dict())
```

Note: verify_fields is a decorator I wrote that introspects the model (much like
to_dict which we saw earlier) and makes sure that all required fields are
present. Uses database introspection to determine which fields are required.

**request.json abuses two facts: SQLAlchemy models can be constructed using
keywords for each field/value and request.json is a dict. End result is we've
called the constructor and explicitly listed out the resource's fields and
values without even knowing them in advance. verify_fields makes sure we have
all the ones we need.





## So How Are Model Definitions Turned Into Services?

We have our models defined. How do we take advantage of the generic `Service`
class and create services from our models?

```python
def register(self, cls_list):
    """Register a class to be given a REST API."""
    for cls in cls_list:
        serializable_model = type(
            cls.__name__ + 'Serializable',
            (cls, SerializableModel),
            {})
        new_endpoint = type(
            cls.__name__ + 'Endpoint',
            (Service,),
            {'__model__': serializable_model,
                '__db__': self.db})
        view_func = new_endpoint.as_view(
            new_endpoint.__model__.__tablename__)
        self.blueprint.add_url_rule(
            '/' + new_endpoint.__model__.__tablename__,
            view_func=view_func)
        self.blueprint.add_url_rule(
            '/{resource}/<resource_id>'.format(
                resource=new_endpoint.__model__.__tablename__),
            view_func=view_func, methods=[
                'GET',
                'PUT',
                'DELETE',
                'PATCH',
                'OPTIONS'])
```

Note: __model__ is set here to the class sent in augmented with the
SerializableModel we saw before. This lets the service keep track of the model
class associated with it. We add endpoints based on the class's name.



## Dynamic Class Generation Using `type`

In Python, `type` with one argument returns a variable's type. With three
arguments, *it creates a new type with the given name, base classes, and
attributes*.





## Using `type` to Create Services

```python
serializable_model = type(cls.__name__ + 'Serializable', (cls, SerializableModel), {})

new_endpoint = type(cls.__name__ + 'Endpoint', (Service,),
    {'__model__': serializable_model,
        '__db__': self.db})
```

Note: SerializableModel is created from the class sent in as an argument. We
then create the Service class that derives from `Service` and sets its __model__
and __db__ attributes. Once we add the endpoints a few lines down, the service
is totally registered with Flask and ready to be used.




## Let's Use This Puppy!

```python
from flask.ext.sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class Cloud(db.Model):
    __tablename__ = 'cloud'

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String, nullable=False)
    description = db.Column(db.String, nullable=False)

    def __str__(self):
        return self.name

class Machine(db.Model):
    __tablename__ = 'machine'

    id = db.Column(db.Integer, primary_key=True)
    hostname = db.Column(db.String)
    operating_system = db.Column(db.String)
    description = db.Column(db.String)
    cloud_id = db.Column(db.Integer, db.ForeignKey('cloud.id'))
    cloud = db.relationship('Cloud')
    is_running = db.Column(db.Boolean, default=False)

    def __str__(self):
        return self.hostname
```

## The Application Code

```python
from flask import Flask
from flask.ext.sandboy import Sandboy
from models import Machine, Cloud, db

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///db.sqlite3'
db.init_app(app)
with app.app_context():
    db.create_all()
sandboy = Sandboy(app, db, [Machine, Cloud])
app.run(debug=True)
```

## The Application Code (continued)

That's it. Those 7 lines create a RESTful API service from our models.





## That's Pretty Easy

In cases where we're building a REST API from scratch, this is pretty easy. But
what if:

* We have an existing database
* We want to create a RESTful API for it
* It has 200 tables





## I'm Not Writing 200 Model Classes

Only downside of `Flask-Sandboy` is you have to define your model classes
explicitly. If you have a lot of models, this would be tedious. 

...I don't do tedious





## It's 2014...

We have private companies building rocket ships and electric cars. Why can't we
have a tool that you point at a database and hit a button, then, BLAM! RESTful
API service.





## Enter `sandman`

`sandman` creates a RESTful API service for *legacy databases* with **no code
required**.





## Prove It

Here's how you run `sandman` against a mysql database:

```bash
$ sandmanctl mysql+mysqlconnector://localhost/Chinook
 * Running on http://0.0.0.0:8080/
 * Restarting with reloader
```





## And Here's How You Use It:

```bash
$ curl -v localhost:8080/artists?Name=AC/DC"
GET /artists/1 HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate, compress
Host: localhost:8080
User-Agent: HTTPie/0.8.0



HTTP/1.0 200 OK
Content-Length: 143
Content-Type: application/json
Date: Sun, 06 Jul 2014 15:55:21 GMT
ETag: "cea5dfbb05362bd56c14d0701cedb5a7"
Link: </artists/1>; rel="self"
Server: Werkzeug/0.9.6 Python/2.7.6

{
    "ArtistId": 1,
    "Name": "AC/DC",
    "links": [
        {
            "rel": "self",
            "uri": "/artists/1"
        }
    ],
    "self": "/artists/1"
}
```




## Things To Note

* ETag set correctly, allowing for caching responses
* Link Header set to let clients discover links to other resources
* Search enabled by sending in an attribute name and value
    * Wildcard searching supported




## How Do We Explore?

We can `curl` `/` and get a list of all available services and their URLs. We
can hit `<resource>/meta` to get meta-info about the service.

Example (the "artist" service):

```bash
$ curl -v localhost:8080/artists/meta
GET /artists/meta HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate, compress
Host: localhost:8080
User-Agent: HTTPie/0.8.0



HTTP/1.0 200 OK
Content-Length: 80
Content-Type: application/json
Date: Sun, 06 Jul 2014 16:04:25 GMT
ETag: "872ea9f2c6635aa3775dc45aa6bc4975"
Server: Werkzeug/0.9.6 Python/2.7.6

{
    "Artist": {
        "ArtistId": "integer(11)",
        "Name": "varchar(120)"
    }
}
```




## Exploring the API

"Real" REST APIs enable clients to use the API using only the information
returned from HTTP requests. `sandman` tries to be as "RESTful" as possible
without requiring any code from the user.




## But Wait, There's More

Would be nice to be able to visualize your data in addition to interacting with it via
REST API.




## Enter The Admin Page

<img src=admin_tracks_improved.jpg>

Note: When `sandmanctl` is run, it automatically opens your browser to this
admin page. All without requiring any code.





## How Is All Of This Possible?

1. Code generation
1. Database introspection
1. Lots of magic

Note: Sandman can do things like create primary keys on the fly for tables that
don't have them. While it doesn't require any code, you can extend it very
simply to add method specific validation, behavior, and authentication.



## Sandman and Flask-Sandboy

`sandman` came first. Has been number one Python project on GitHub twice and
is downloaded 25,000 a month. Flask-Sandboy is `sandman`'s little brother...

Note: Hopefully now the name makes sense




## So What's The Point of All This?

* The fact that the end result is a REST API is not especially interesting
* More important are the concepts underpinning `sandman` and Flask-Sandboy





## Prefer Code Generation To Hand-written Code

* Work at higher level of abstraction
* Solve a problem once in a generic manner
* Reduces errors, improves performance

In general: **automate everything**




## Compose Well-Made Tools To Create New Ones

* `sandman` = Flask + SQLAlchemy + Lots of Glue
* Requires you know the capabilities of your tools
* Part of the UNIX Philosophy





## Be Lazy

The best programming advice I ever got was to "be lazy"

* Sandman exists because I was too lazy to write boilerplate ORM code for an
    existing database
* Flask-Sandboy exists because I was too lazy to write the same API services
    over and over
* Being lazy forces you to learn your tools and make heavy use of them
* **Don't re-solve solved problems**




## Questions?

Contact me at:

* `jeff@jeffknupp.com`
* `@jeffknupp` on Twitter
* http://www.jeffknupp.com on the tubes
