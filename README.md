# Designing Domain Logic with PHP in 2018

Early in 2017 I was helping a friend of mine to come up with a proof of concept for one of
his countless ideas. The project was fairly simple - I was implementing an API server for a mobile app.
The was already a prototype written in php which used JSON as the data standard. The API was growing fast, so 
to avoid reinventing the wheel, I decided to stick to some sort of a standard. Naturally, I stumbled upon the
[JSON API](http://jsonapi.org/) standard. By that time a bunch of implementation had been created and available through
[Composer](https://packagist.org/), but the two I considered the most promising and cleanly written had one small issue
which prevented me from going ahead and using them. The way the libraries implemented the idea of serializers broke 
the letter "L" in [SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)). 
An [issue](https://github.com/tobscure/json-api/issues/115) was open on GitHub,
and the question of how would I implement that logic arose. Trying to come up with a pull request with a "right" solution
I started plying with the implementation model and matching it with the JSON API standard. Pretty soon I realised
there was too many things I wanted to change so I got sucked into the process and when I woke up I realised I had gone
too deep in the woods so that turning this work into a pull request would mean rewriting the entire thing. Yes, a clear
case of [14 competing standards](https://xkcd.com/927/), I know. But enough time had been invested, so something complete
had to be made out of this. In the following parts I'd like to share the lessons learned and show how the Domain rules
can be expressed with means of modern php.

## Setting the expectations
Rewriting the whole library did not make any sense, I focused on the object model of JSON API. To myself, I defined that
I wanted the create an object model which would serve just one purpose - to implement the rules and restrictions of JSON API
defined in the standard. The following framework was set.

### Test-first development only
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
