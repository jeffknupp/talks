## Writing Idiomatic Python:

**Towards Comprehensible and Maintainable Code**

PyConPH 2017

* Jeff Knupp
* @jeffknupp
* jeff@jeffknupp.com




## Jeff Knupp?

* Author of “Writing Idiomatic Python”
* Principal Engineer @ Enigma (enigma.io)
* Creator of the “sandman(2)” Python library
* Blogger at jeffknupp.com




## What's the format for this talk?


Was going to be a dynamic, interactive, question and answer style *conversation*
that naturally led to us discussing interesting things

<p class="fragment roll-in">Then I realized you guys would probably not ask the exact questions I need you to</p>

<p class="fragment roll-in">So I took the liberty of asking the questions for you</p>




## What’s this about, anyway?	

<p class="fragment roll-in">It's about writing idiomatic Python, of course!</p>



## OK, well then what's the point of all this?

Python is only getting *more* popular. That's great. But learning to write code
in an idiomatic fashion is *vital* both to your success *and* the language's.

<p class="fragment roll-in">If you want Python's popularity to continue to grow, **write Pythonic code that newcomers can follow.**</p>

<p class="fragment roll-in">If you don't want to every Python job for the next 20 years to be maintaining legacy code no one understands, **teach
newcomers to write idiomatic Python.**</p>



## I have some questions...

* What does 'idiomatic Python' mean?
* How do I write it?
* Why should I care?



## What is it?

"Idiomatic Python" is another way of saying "Pythonic code"




## Oh, of course… wait, what’s THAT?

**Py\*thon\*ic**

*Adjective*

> Code written in the way the Python community has agreed it should be written.

Example Usage: “The way you use a for loop to find a dictionary entry by comparing each key in sequence isn’t very Pythonic”

Synonyms: **idiomatic**




## Who decides all of this?


<p class="fragment roll-in">ME</p>




## Who *really* decides all of this?

Python developers, through the code they write, share, and criticize




## Who *else* really decides all of this?

Occasionally, BDFL or PEP authors…




## Why should I care about "idiomatic code"?

Three reasons:
 
* Readability
* Maintainability
* Correctness


<p class="fragment roll-in">Bonus reason:

People will stop laughing at your code</p>

Note: The point of writing idiomatic Python is not to show how clever you are or how many language constructs you can make use of. Rather, there are three very important benefits that idiomatic code provides. I don't care what language you program in; if there's some magic way of writing code that increases readability, maintainability, *and* correctness, you should definitely take notice.




## Intermezzo: Common Objections to Being Told to Write Idiomatic Code



## "But I’m not a programmer, I’m a [noun|adjective] scientist!"

Even more important!

Want your research to be peer reviewed? You better not write eye-bleedingly bad code, or you will get peer ignored.
Note: Gret influx of Python programers from the sciences, which is fantastic.
You don't get a pass, though, if you're not a programmer by trade. Making your
code readable and maintainble is just as much a part of your job as is
publishing results. Would a journal accept a paper where the description of you
research was total gibberish.



## "But I’m not a programmer, I just write “scripts” "

“I’m not a thief! I just steal small things!”

When you *write a program in Python*, you are **assuming the role of a programmer and must operate in that realm.**

<p class="fragment roll-in">If you call your Python programs “scripts” and use that as justification as to why you don’t have to write thoughtful, readable code, I will punch you.</p> 

<p class="fragment roll-in">I’m 6’3”. 230 lbs. That punch is going to hurt.</p>



## "I’m coming from Java. Can I use classes for everything, ignore list comprehensions, and add “factory” to all my class names?"

No! That’s not Python! (Maybe call it “Jython”? Look into securing this name later…)

<p class="fragment roll-in">If you write Python as Java, other Python programmers won’t be able to maintain your code</p>

<p class="fragment roll-in">*Idioms* are what make Python, *Python*</p>




## OK. Fine. But how does it help readability?

Glad you asked!

Idiomatic Python is the agreed upon correct approach, so readers of your code will have no problem following along

Problems following along = “cognitive burden”




## What’s “cognitive burden” and how is it related to readability?

Cognitive burden is the increased mental effort required to keep track of what’s going on.

I’m pretty stupid and can only remember like two things at a time. Don’t write code that makes me ask questions or remember things.

Cognitive burden = best measure of readability

Note: Readability is not merely concision. Could be in reading a program,
reading a Moby Dick, watching an episode of Law and Order.




## That’s great, but you haven’t quoted Knuth yet…

> Instead of imagining that our main task is to 
> instruct a computer what to do, let us concentrate rather 
> on explaining to human beings what we want a computer to do

-Knuth




## You said it helps with maintainability. Who cares?

Well, let's put it this way:

    

> “Always code as if the guy who ends up maintaining your code will be a violent psychopath who knows where you live.”

-John Woods, comp.lang.c++




## OK, I'm sufficiently scared. Now how does idiomatic Python help?

Well, if you're the only one who can read your code, correctness is irrelevant

Bad, but working code is not a victim-less crime




## So how do I make sure others can read my code?

Idiomatic code always makes the author's intention clear. *They may not have
come up with the correct solution*, but it's clear what they were doing.




## Give me an example of how non-idiomatic code is harder to maintain!

Well, if you write this:

```python
index = 0
while index < len(my_list):
    do_something_with(my_list)
    index += 1
```

...you instantly make me suspicious. "Why did they iterate over a loop like
that? Surely I'm missing something. I must inspect this further...". And it ends
up with me a quivering ball of mush all because you don't know the correct way
to iterate over a damn list.




## Moar examples!

Quick, what does the following code do?

```python

def do_calculation(my_list):
    """Return the results"""
    a = 0
    b = 0
    for score in my_list:
        a += score
        b += 1
 
    return a / b
```

I swear to God I've seen code like this, with variable names like `a`,
**`aa`(!)**, `ab`. I became that violent psychopath.




## Now better the thing!

Quick, what does the following code do?

```python
def calculate_mean(tests):
    """Return the mean score of a series of tests."""
    return sum((test for test in tests)) / len(tests)
```

Three things working together: *Names*, *Documentation*, and *Code Itself*




## Wait! I can do better!

Yep, in Python 3.4, this becomes:

```python
import statistics

calculate_mean = statistics.mean()
```

Part of writing idiomatic code is staying up to date with the changes in the
language!




## OK, I can see how idiomatic code helps maintainability. But correctness?

Regardless of how badly written code is, automated tools will find issues.

The same is *not* true of humans.

Logically:

```python
if (It's clear what you're trying to do) and (how you do it is clear):
    It is easier for others to detect flaws in your thinking
```




## Whatever. We do code reviews here. *They* catch everything.

Fine, but if your code is poor:

* How likely is a reviewer to spot a bug?
* How likely is it that I'll give your code the extra attention it requires?
* How likely am I to care about reviewing your code in the future?

Checkmate




## I'm paid to write code, not read it.

Reading and writing code are two sides of the same coin.

**You** are the most frequent reader of code you write.

Note: I guarantee everyone has done the following:
* come back to some code your wrote a month ago
* have no idea what it does or why you wrote it that way
* wish you could slap the past version of yourself in the mouth




## How does idiomatic code make code more correct?

It doesn't. It makes *flaws* in the approach more obvious. If you write
idiomatic code, small errors in logic are glaringly obvious to reviewers.

This is because they can focus on *what* the code does, not *how* it does it.

**How** has already been established by the community.
**What** is the reason the program was written.




## How else does it help?

If there were fifty ways to iterate over a list, code reviews would be
impossible.

When you write non-idiomatic code, now I have to check if your solution to *this already solved problem* is correct.

I don't want to spend any time determining if you iterated over a list correctly.




## Give me the code!

*Harmful*

```python
my_container = ['Larry', 'Moe', 'Curly']
index = 0
for element in my_container:
    print('{} {}'.format(index, element))
    index += 1
```

*Idiomatic*

```python
my_container = ['Larry', 'Moe', 'Curly']
for index, element in enumerate(my_container):
    print('{} {}'.format(index, element))
```




## More!

*Harmful*

```python
def get_both_popular_and_active_users():
    # Assume the following two functions each return a
    # list of user names
    most_popular_users = get_list_of_most_popular_users()
    most_active_users = get_list_of_most_active_users()
    popular_and_active_users = []
    for user in most_active_users:
        if user in most_popular_users: popular_and_active_users.append(user)
    return popular_and_active_users
```

*Idiomatic*

```python
def get_both_popular_and_active_users():
    # Assume the following two functions each return a 
    # list of user names
    return(set(get_list_of_most_active_users()) & set(
            get_list_of_most_popular_users()))
```




## Even Moar!

*Harmful*

```python
is_generic_name = False
name = 'Tom'
if name == 'Tom' or name == 'Dick' or name == 'Harry':
        is_generic_name = True
```

*Idiomatic*

```python
name = Tom
is_generic_name = name in ('Tom', 'Dick', 'Harry')
```








# I hope it's over now!

Yep. That's it. Questions? Comments?




