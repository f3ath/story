# Designing Domain Logic with PHP in 2018

Early in 2017 I was helping a friend of mine to come up with a proof of concept for one of
his countless ideas. The project was fairly simple - I was implementing an API server for a mobile app.
The was already a prototype written in php which used JSON as the data standard. The API was growing fast, so 
to avoid reinventing the wheel, I decided to stick to some sort of a standard. Naturally, I stumbled upon the
[JSON API](http://jsonapi.org/) standard. By that time a bunch of implementation had been created and available through
[Composer](https://packagist.org/), but the two I considered the most promising and cleanly written had issues
which prevented me from going ahead and using them. E.g. the way the libraries implemented the idea of serializers broke 
the letter "L" in [SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)). 
An [issue](https://github.com/tobscure/json-api/issues/115) was open on GitHub,
and the question of how would I implement that logic arose. Trying to come up with a pull request with a "right" solution
I started plying with the implementation model and matching it with the JSON API standard. Pretty soon I realised
there was too many things I wanted to change so I got sucked into the process and when I woke up I realised I had gone
too deep in the woods so that turning this work into a pull request would mean rewriting the entire thing. Yes, a clear
case of [14 competing standards](https://xkcd.com/927/), I know. However enough time had been invested, so something complete
had to be made out of this. In the following parts I'd like to tell about the process of converting Domain rules
into object design with means of modern php and the lessons learned. Despite of being an
[accidental language](https://motherboard.vice.com/en_us/article/pgkbey/know-your-language-the-wither-of-php), having
a poor type system, and a bunch of bad parts PHP has its own patterns which may be useful and I want to demonstrate 
a few of them.

## Setting the expectations
Rewriting the whole library did not make any sense, I focused on the object model of JSON API. To myself, I decided that
I wanted the create an object model which would serve just one purpose - to implement the rules and restrictions of JSON API
defined in the standard. The following framework was set.

### Test-first development
Was it TDD? Probably not. I tried to follow the [Three Rules](http://butunclebob.com/ArticleS.UncleBob.TheThreeRulesOfTdd) 
when possible. But here is the thing. One of the reasons why TDD is considered useful is that it allows you not to think
about code design immediately. The design is supposed to evolve as a sub-product. This idea is sometimes expressed by
changing the meaning the last "D" - Test-Driven Design instead of Development. In my case the primary source of design
decisions was the Domain. Important disclaimer: that was not DDD either. I did not (and neither do) consider myself a
DDD expert to make such a bold statement. Though, some ideas from the [Big Blue Book](https://domainlanguage.com/ddd/)
were possibly (but not necessarily correctly) implemented.

### Each test should reflect one of the standard's requirements
The procedure was simple: 
- take one business-rule from the standard
- write a test case for it
- make it pass
- refactor
- repeat

### Tests should serve as the documentation
Another approach I wanted to try out was 
[tests as executable specification](http://agiledata.org/essays/tdd.html#Documentation). Perhaps the real reason was
me being too lazy to create good docs, but at that moment that's how I explained it to myself.

## The first step
So I started with the [Document Structure](http://jsonapi.org/format/#document-structure) and that gave me the simplest
case possible: the `Document` class should exist and contain the required media type somewhere:
```php
<?php
class Document {
    const MEDIA_TYPE = 'application/vnd.api+json';
}
```

## Object instantiation and invalid state
You may note that the previous step broke the test-first rule. Yeah, perhaps. But since there was no actual logic to test,
it's probably not that big of a sin. Anyway, we're going to fix it right now. The next domain rule [said]:(http://jsonapi.org/format/#document-top-level)
> A document MUST contain at least one of the following top-level members:
> - data: the document’s “primary data”
> - errors: an array of error objects
> - meta: a meta object that contains non-standard meta-information.

> The members data and errors MUST NOT coexist in the same document.

So it's time to write a Document creation test case. So what could it look like? Clearly, those were `Document`'s 
dependencies, and the a naive implementation would be something like
```php
<?php
class Document {
    function __construct($data, $errors, $meta) {}
}
```

This approach had two major drawbacks: any two dependencies can be missing and `data` and `errors` could not coexist.
Violating these rules would leave the newly created object in an invalid state.
### Optional dependencies
What do we usually do with an optional dependency in PHP? We can introduce a default value, e.g. `null` or `false`
(what a horrible choice from the type system's point of view, but I've seen that used a lot).
In the OOP world this is called the [Null Object](https://sourcemaking.com/design_patterns/null_object). So let them
be optional?
```php
<?php
class Document {
    function __construct($data = null, $errors = null, $meta = null) {}
}
```
Well, but now I would have to have some ugly `if`s in the constructor checking that at least one of those has been
provided and throw an exception otherwise to prevent an invalid state. It would get even uglier if we added the 
data/error coexistence check. Luckily, this was easy to solve.

## Named Constructors
[Method overloading](https://en.wikipedia.org/wiki/Function_overloading) in PHP is far from perfect. But in case of
constructors there is a nice workaround called Named Constructors:
- make `__construct()` private
- create public static methods with meaningful names and call `__construct()` from there

```php
<?php
class Document {
    private $data, $meta, $errors;
    private function __construct() {}
    static function fromData($data): Document {
        $doc = new self;
        $doc->data = $data;
        return $doc;
    }
    static function fromErrors($data): Document { /* similar logic */ }
    static function fromMeta($data): Document { /* similar logic */ }
    function setMeta($meta) {
        $this->meta = $meta;
    }
}
```

This solved the issue. A Document would inevitably have at least one of the dependencies. There was no way to create a 
Document containing both data and errors. Meta could be combined with data and with errors (via `setMeta()`) or could 
be used standalone (via `fromMeta()`). Another advantage was that we did not have to remember the order of the arguments
in the constructor.



