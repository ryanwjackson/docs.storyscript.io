# Storyscript

> A **Goal-oriented Cloud Native Programming Language** that choreographs microservices. By removing unnecessary complexity applications can be written with pure business logic that is intuitive, transparent and syntax-light.

# Table of Contents
[[toc]]

## About Storyscript

Storyscript focuses on the **application logic** rather than all the *tape and glue* that bind applications together. The underlining services have a standard for **logs, metrics, fail-over, rate-limiting, tracebacks and scaling** which eliminates the need to write it within the application. This cultivates a development environment primed for rapid application development in a production-ready platform.

Let's build a quick application for example. Our goals are to upload, analyse, compress and archive a video. A non-trivial application but in a **couple lines of Storyscript** we made it.

```coffeescript
# Registers with Asyncy Server as an endpoint
when http server listen method:'post' path:'/upload' as client
    client write content:'Success! Processing asynchronously.'
    client set_status code:201
    client finish

    # At this we are running asynchronously

    # generate a unique id for this upload
    id = uuid uuid4

    video = client.files['my_uploaded_video']

    # using https://machinebox.io find the video topics
    topics = machinebox/videobox find_topics content:video

    # save record in mongodb
    mongodb insert db:'uploads' data:{'id': id, 'topics': topics}

    # using https://github.com/xiph/daala let's compress it to h264
    video = xiph/daala compress video:video codex:'h264'

    # upload to AWS S3
    aws/s3 put target:'/video/{id}.mp4' data:video
```

In comparison, the same application would likely take **hundreds of lines of code**, not to mention that each service above includes metrics, logging and scaling out-of-the-box.

::: tip Blog
[Storytelling your first application](https://asyncy.com/blog/story-telling/)
&mdash; Build a file.io or WeTransfer clone in 10 minutes.
:::


## Syntax Overview

```coffeescript
###
Meet Storyscript
###

# Strings
my_string = "Hello"
"Say {my_string}!"  # string formatting
# >>> Say Hello!

# Numbers
one = 1
onethree = 1.3

# Boolean
foo = true
bar = false

# List
letters = ['a', 'b', 'c']
letters[0]
# >>> 1

# Object
fruit = {'apple': 'red', 'banana': 'yellow'}
fruit['banana']
# >>> yellow

# Regexp
pattern = /^foobar/
'foobar' like pattern
# >>> true

# Null
empty = null

# Conditions
if one > 1
    # then do this
else if one == 1
    # then do this
else
    # do this

# Loops
foreach my_list as item
    # more stuff here

while foobar
    # more stuff here

# Services
val = service action key:value
val = owner/repo action key:value

# Event-based service
when slack bot hears pattern:/hello/ as msg
    msg reply message:'world'

# Functions
function walk distance:number returns string
    # more stuff here
    return "Ok, walked {distance}km!"
walk distance:10
# >>> Ok, walked 10km!

# Chaining calls
my_service action foo:(my_string split by:',')
                  bar:(my_object find key:(my_list random))
'abc' uppercase split
# >>> ['A', 'B', 'C']

# try and catch
try
  # more stuff here
catch as error
  # more stuff here
  retry  # try the block again
  # -or-
  raise  # bubble it up
```

## Semantics

### Procedure

```coffeescript
output = do_third foo:(do_second (do_first ...)) bar:(do_second ...)
```

Parentheses MUST be used to produce inline procedures. The innermost Parentheses will be executed first moving out to the outermost.

Same level parentheses MAY be called at the same time which done by parallel processing in new threads.

First set of parentheses when assigning variables is optional. E.g., `a = my_list length` is the same as `a = (my_list length)`.

### Attributes, methods and entries

Objects may have attributes and methods. A Map can *only* have entries.

1. `Object` attributes can **only** be accessed using a period character (`.`) as in `user.name`.
1. `Object` methods can **only** be accessed using a space character (` `) as in `cart add item:...`.
1. `Map` entries can **only** be accessed using brackets and strings (`['key']`) as in `some_map['key']`

The example below illustrates how the object `tweet` has attributes and methods.

```coffee
tweet = twitter tweet text:'Hello world'
id = tweet.id  # returns the tweet id 123456 of type int.
tweet like     # calls "like", a method of tweet
```

### Mutations
Mutations refer to running operations on the built in data types, such as strings, arrays, maps, and numbers.
```coffeescript
# Strings
# Note: None of the string operations below change the original string in any form
str = ''
str length  # returns the number of UTF-8 characters
str replace pattern: 'ab' by: 'AB'  # returns a string by replacing 'ab' with 'AB'
str replace pattern: /ab/ by: 'AB'  # same as above
str split by: '.'  # returns an array by splitting the string with the delimiter
str uppercase  # returns a string where all characters are upper cased
str lowercase  # returns a string where all characters are lower cased
str capitalize  # returns a string where the first letter of each word is capitalized (eg: 'jane smith' becomes 'Jane Smith')
str in list:['foo', 'bar']  # returns true or false if the string is found in the list

# Numbers
num = 10
num is_odd  # returns false
num is_even  # returns true
num absolute  # returns the absolute value (if num was -1, this would return 1)
num increment  # returns 11. Note that num is not changed
num decrement  # returns 9. Note that num is not changed

# Arrays
arr = [1, 2, 3, 4, 5]
arr index of: 5  # returns the index of an element, 4 in this case
arr length  # returns the length of the array, 5 in this case
arr append item: 6  # adds 6 to the end of the array
arr prepend item: 1  # adds 1 to the start of the array
arr random  # returns a random element from this array
arr reverse  # reverses the array in-place
arr sort  # sorts the array in an ascending fashion
arr min  # returns the lowest of the elements in this array (if it contains numbers)
arr max  # returns the largest of the elements in this array (if it contains numbers)
arr sum  # returns the sum of all the elements in this array (if it contains numbers)
arr unique  # reduces the array to contain only unique items
arr contains item: 3  # returns true if 3 is present, false otherwise
arr remove item: 3  # removes the item specified from the array

# Maps
m = {'a': 1, 'b': 2}
m size  # returns the size of the map, 2 in this case
m keys  # returns an array of all keys
m values  # returns an array of all values
m flatten  # returns a list of key/value pairs (eg: [['a', 1], ['b', 2]])
m pop key: 'a'  # removes and returns the value for key 'a'
m get key: 'b'  # returns the value for the key 'b'
m contains key: 'c'  # returns true if the key 'c' exists in the map, false otherwise
```

#### Chaining Mutations

Mutations can be chained to help reduce compleixty.

```coffeescript
'abc' uppercase split
# >>> ['A', 'B', 'C']

'abc' uppercase
      then split
# >>> ['A', 'B', 'C']
```

### Variable Scope

Variables are not global.

```coffeescript
n = 1

every minutes:3
  n increment
  log n
```

```
+0  INFO 2
+3m INFO 3
+6m INFO 4
```

Functions do not have access to local variables. All variables must be provided as arguments.

```coffeescript
n = 1

function incr
  n increment
```

```
Syntax error. Variable "n" is not defined at line 4.
```

### Compiling

Storyscript is a dynamically compiled language. Type checking is performed at compile time, but not in a traditional way. From the perspective of the developer the following steps are performed during compile time.

Compile time consists of four primary processes:

1. **Linting** is performed to check syntax and grammar.
1. **Translation** is performed which translates the Storyscripts into event-logic tree.
1. **Dependancy** checks are performed to ensure command and arguments exists.
1. **Type-Checking** is performed on the Stories the ensure data integrity.

The type-checking includes the following checks:

1. Type mutation method exists.
2. Arguments are of the expected type.


### Environment Variables

Environment variables are stored in a restricted keyword `app.environment`.

```yaml
# asyncy.yml
environment:
  food: cake
  FOO: bar
```

```coffeescript
if app.environment.food == 'cake':
    ...

if app.environment.foo == 'bar':
    ...
```

::: tip Note

Variables are **ALWAYS** exposed as lower-case attributes.

:::

### Execution Model

Storyscripts are executed by an interpretation engine (not compiled to C or Java).

#### Deployment

1. All dependancies are gathered and prepared for execution.
1. The Asyncy Engine is prepared with the Stories as first-class assets for swift execution.
1. Every Storyscript is executed allowing them to register with the gateway, cron, etc.

#### Execution

A story may [execute in many ways](/faq/#how-are-storyscripts-started).

1. The Engine received notice to start a Story with or without starting arguments.
1. The Story is executed in a single thread.
1. When a service is called the Engine will communicate with the service passing necessary data to and from the service back into the primary thread.
1. Asynchronous commands may generate new threads and execute in the same pattern above.

```coffeescript
translated = service_a translate:my_string to:'spanish'
parts = translated split ' '
first_word = service_b name:parts[0]
```

The Story above is would perform the following operations:

1. Translate a string to Spanish
1. Split the translated string by whitespace
1. Assign `first_word` to the first word in the `parts` array


## Strings

```coffeescript
data = "foobar"

long_string = "Hi Friend,
This is a lo\
ng string."
# Hi Friend, This is a long string.

more_data = """
    The quick brown fox
    jumps over the lazy dog.
"""
# The quick brown fox\njumps over the lazy dog.

where = "Earth"
data_formatted = "Hello, {where}"
# Hello, Earth
```

::: v-pre
Like many traditional programming languages, Storyscript supports strings as delimited by the `"` or `'` characters.
Storyscript also supports string interpolation within "-quoted strings, using `{ variable }`.
Single-quoted strings are literal. You may even use interpolation in object keys.
:::

Multiline strings are allowed in Storyscript.
Lines are joined by a single space unless they end with a backslash.
Indentation is ignored.

Block strings, delimited by `"""` or `'''`, can be used to hold formatted or indentation-sensitive text (or, if you just don’t feel like escaping quotes and apostrophes).
The indentation level that begins the block is maintained throughout, so you can keep it all aligned with the body of your code.

Double-quoted block strings, like other double-quoted strings, allow interpolation.

## Numbers

```coffeescript
int = 1
number = 1.2
```

## Comments

```coffeescript
###
Large
  Comment
Block
###

# Inline comment

foo = "bar"  # end of line comment
```

In Storyscript, comments are denoted by the `#` character to the end of a line,
or from `###` to the next appearance of `###`.
Comments are ignored by the compiler, though the compiler makes its best effort at reinserting your comments into the output JavaScript after compilation.


## Boolean

```coffeescript
happy = true
sad = false
```

## Lists

```coffeescript
list_inline = ["string", 1, 2]
list_multiline = [
  "string",
  1,
  2
]
```

## Objects

```coffeescript
object_inline = {'foo': 'bar', 'apples': 'oranges'}
object_multiline = {
  'foo': 'bar',
  'apples': 'oranges'
}
```

## Conditions

```coffeescript
if foo == bar
  # more stuff here
else if foo > bar
  # more stuff here
else
  # more stuff here

if (foo > 0 or cat is not dog) or foobar like /regexp/
  # more stuff here
```

`if`/`else` statements can be written without the use of parentheses and curly brackets. As with functions and other block expressions, multi-line conditionals are delimited by indentation.

## Looping

Looping through index and/or object keys.

```coffeescript
foreach my_list as item
    # ...

foreach my_list as index, item
    # ...

foreach my_object as key
    # ...

foreach my_object as key, value
    # ...

while foobar
    # ...
```

Loops have reserved keywords for ending and continuing loops.

```coffeescript
foreach my_list as item
    # more stuff here
    if do_end_loop
        end
    if do_skip_to_next_item
        next
    # ...
```


## Functions

```coffeescript
function get_user id:int returns map
    someone = (psql exec query:'select * from users where id={id} limit 1;')[0]
    someone['contact'] = fullcontact person email:someone['email']
    return someone

user_a = get_user id:7
user_b = get_user id:10
```

The example above is a function what queries the database and also downloads their FullContact profile.

Function must define their inputs and outputs which help with transparency, autocomplete and type checking during the Asyncy CI process.

Functions **MAY** declare one output and identify it's type. If an output type is used the function **MUST** use `return` and return that type.

```coffeescript
function add this:int that:int returns int
   return this + that
```

Functions that do not have an output may not use `return`. An error is thrown if a return is used.

```coffeescript
function do_this
    # ...

function do_that
    return 1
>>> ERROR: Function must set type of return.
```


## Services

A service is a containerize microservice that is registered in the [Asyncy Hub](https://hub.asyncy.com). Discover hundreds of services in the Hub or build your own in any language, submit to the Asyncy Hub and call it in your Storyscript like this:

```coffeescript
# Call a service with a command and all arguments named
service cmd key:value foo:bar
org/repo cmd key:value foo:bar

# Service output assigned to variable
foobar = service cmd key:value

# Arguments may by indented under the service
service cmd key:value
            foo:bar
```

In Storyscript, the syntax to run a service appears natural and arguments are named for transparency.

```coffeescript
tweet = "hello"
twitter tweet message:tweet
# would result in ```twitter tweet message:"hello"```
```

Containers, commands and argument names are **static grammar** and **interpreted literally**.

## Event-Based Services

Services may publish events which run a new block of logic.

```coffeescript
service cmd foo:bar as client
    when client event foo:bar as data
        # ...
```

A good example of this is streaming Tweets by hashtag.

```coffeescript
twitter stream as client
    when client tweet hashtag:'asyncy' as tweet
        res = machinebox/language data:tweet.message
        if res.tone == 'good'
            tweet reply message:'Thank you!'
            tweet retweet
            tweet like
```

Every new tweet will be passed into the block below in the variable `tweet`.
Then machine learning will determine if the tone of the tweet's message is good or bad. The streaming service will wait for new tweets until the story is ended.

You can also use a shorthand syntax for single event streaming.

```coffeescript
when slack bot hears pattern:/hello/ as message
    message reply content:'world'
```

## Operations

```coffeescript{2}
if something_went_wrong
    end story
```

Use `end story` to stop the story and exit now.


## Exception Handling

```coffeescript
try
  # more stuff here
catch as error
  # more stuff here
finally
  # more stuff here
```

In Storyscript, the `try` expressions catch exceptions and pass the error to the `catch` block.

The `finally` block is **always** entered regardless of an exception being raised or not, use it for cleanup commands.

You may omit both the `catch` and `finally`.

```coffeescript
try
  # more stuff here
catch as error
  # more stuff here
  raise
```

Use the `raise` keyword to raise the exception, bubbling up to the next try block or stopping the story.


## Regular Expressions

```coffeescript
pattern = /^foo/i
```

Regular expressions are supported without any special characters of escaping necessary.

### Regular Expressions Methods

```coffeescript
pattern = /(?P<key>\w):(?P<value>\w)/
```

::: tip Info
Learn the methods you can call with regular expression in the [Asyncy Hub](https://hub.asyncy.com)
:::

## Types

```coffeescript
1 type
# int

true type
# bool

"" type
# string

[] type
# list

{} type
# object

null type
# null

(date now) type
# date

(interval days:1) type
# interval

(range from:foo to:bar) type
# range

/^foobar/ type
# regexp

function foobar returns int
    return 1

function foobar
    # ... no return allowed

foobar type
# function
```

Use the method `type` to get the type of a variable as a string.

```coffeescript
(1 is int) and (true is bool) and ("" is string)
# true

([] is list) and ({} is object)
# true

(1 is number) and (1.2 is number)
# true

{} is string
# false
```

Type checking can be applied to any type.
