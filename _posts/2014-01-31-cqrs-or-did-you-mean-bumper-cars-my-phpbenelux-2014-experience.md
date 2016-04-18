---
title: CQRS? Or did you mean (bumper) cars? – My PHPBenelux 2014 experience
categories:
  - Conferences
tags:
  - Community
  - Conference
  - PHP
  - PHPBenelux
---

<img class="alignright" src="/assets/images/2014/01/phpbnl14.png" alt="" />
I recently attended the [PHPBenelux 2014](http://conference.phpbenelux.eu/2014/) conference with a co-worker of mine. It was the 5th anniversary edition of PHPBenelux and the second time I attended the conference. It was held January 24th &amp; 25th at hotel Ter Elst in Antwerp, Belgium.

In this post I will recap my experience at the conference and highlight what I found most interesting, fun or otherwise noteworthy.

<!--more-->

## Model Storming

Friday started with half day workshops. We attended a work shop called [Model Storming](http://conference.phpbenelux.eu/2014/sessions/#model-storming-workshop) by Mathias Verraes. It was a very interesting workshop that focused on building a domain model in small teams. We didn't use any digital tools for this, but instead consumed massive amounts of Post-It notes of various colours. We received some good advice what kind of things you should think about and ask from customers (or "domain expert") when designing software and some nice insight into Domain-driven design.

Mathias also presented a quick introduction to a concept called CQRS towards the end of the workshop. This is a very interesting concept. CQRS stands for *Command Query Responsibility Segregation*. Basically it means that there is a separate model for writing things and a separate model for reading things. CQRS combined with a concepts like *Event Sourcing* and *Task-based UI* provides some interesting capabilities where the UI sends commands to the write model, the write model turns them into events that get stored into an event store. The read model can then create all kinds of different representations of the data, even afterwards because the events can be replayed. CQRS seems interesting and I intend to learn more about it, in case you are interested you can check out some links [here](http://cqrs.wordpress.com/). The slides Mathias showed us can also be found [here](http://verraes.net/2013/12/fighting-bottlenecks-with-cqrs/).

## Conference time

After the workshop, the "main" conference began. It featured only one keynote (at the beginning) and after that 3 tracks of presentations for the friday afternoon and whole saturday. This year there was also an [UnConference](http://conference.phpbenelux.eu/2014/want-unconference/), which turned out to be quite popular and I think almost, if not all, of the available UnCon slots were taken.

The keynote was called [Mentoring Developers](http://conference.phpbenelux.eu/2014/sessions/#keynote-mentoring-developers) by Elizabeth M Smith. Mentoring is a quite interesting topic and Elizabeth was clearly passionate about the topic. The talk was an interesting mix of the social and professional aspects of mentoring and stories from her personal experience. I enjoyed the talk a lot and it served as a good reminder that I really should get into this mentoring business myself.

## CQRS? Or did you mean cars?

Stijn Vannieuwenhuyse (who works with Mathias Verraes and was also helping out at the workshop) did an UnCon talk titled [CQRS? Or did you mean cars?](http://conference.phpbenelux.eu/2014/sessions/#sign-venue-present). The story behind the title is that some time ago when they first heard about CQRS it was so new that when they Googled for it, Google asked "Did you mean cars?". This talk recapped most of the information about CQRS that Mathias presented in the morning, but offered some more in-depth code examples so it was very interesting.

## Functional Design Patterns Under the Hood

We spent the rest of the afternoon learning [Functional Application Design in PHP](http://conference.phpbenelux.eu/2014/sessions/#functional-application-design-php) by Michael John Burgess, [Refactoring with Design Patterns](http://conference.phpbenelux.eu/2014/sessions/#refactoring-design-patterns) by Benjamin Eberlei and [PHP Performance: Under The Hood](http://conference.phpbenelux.eu/2014/sessions/#php-performance-hood) by Davey Shafik. All of these talks were interesting and well presented and we learned a lot.

## Bumper cars!

After the last sessions of the day, it was time for the Conference Social. This year the organizers had really out done themselves and the social was a real success. There were the famous Belgian fries and always tasty Belgian beer and fun and games of various kind, including pinball, air-hockey, arcade games, mechanical bull rodeo and to top it all, bumper cars! The bumper cars were so much fun, but the most important thing in any social event are the people. It was great to meet so many friends and make new friends in such a relaxed and fun atmosphere. Really enjoyed it a lot.

<a href="/assets/images/2014/01/2014-01-25-22.21.401.jpg"><img class="alignnone" src="/assets/images/2014/01/2014-01-25-22.21.401-300x166.jpg" alt="" width="300" /></a>

## Day two: Hemoglobin and Hobgoblins

In a multi track conference there will almost always be at least few slots where you really want to see more than one session. The organizers really wanted people to wake up early for day two, since all three of the first sessions were really interesting talks with great speakers. We decided to go see [Models and Service Layers; Hemoglobin and Hobgoblins](http://conference.phpbenelux.eu/2014/sessions/#models-service-layers-hemoglobin-hobgoblins) by Ross Tuck. It was a great talk with some very good points, but you really had to pay attention to keep up. The [first slide](http://www.slideshare.net/rosstuck/models-and-service-layers-hemoglobin-and-hobgoblins) really says it all. Ross also talked a little about CQRS towards the end of the presentation, are we starting to see a pattern here? There was also a link to a nice [PHP library for CQRS](https://github.com/beberlei/litecqrs-php) by Benjamin Eberlei.

After recovering from the first talk of the day, we went to see [The seven deadly sins of Dependency Injection](http://conference.phpbenelux.eu/2014/sessions/#seven-deadly-sins-dependency-injection) by Stephan Hochdörfer and after that [Application monitoring with Heka and statsd](http://conference.phpbenelux.eu/2014/sessions/#application-monitoring-heka-statsd) by Jordi Boggiano.

## Clean Code at the UnCon

After lunch it was time for my UnConference talk [Clean Code](http://joind.in/10482). The room was full (I like to think it was all me, but the fact that Matthew Weier O'Phinney was right after me might also have something to do with it) and I think it went quite well. I really wish I would have had a little more time to prepare the talk and the slides, but I got some good feedback so I will keep working on it and maybe submit as a real talk to another conference someday. After my session Matthew demoed [Apigility](http://www.apigility.org/) which seems great for building good APIs.

## Extracting and Closing

Last sessions we went to see were [Social human architecture for beginners](http://conference.phpbenelux.eu/2014/sessions/#social-human-architecture-beginners) by Sebastian Schürmann which concentrated more on the human factor and [Extract Till You Drop](http://conference.phpbenelux.eu/2014/sessions/#extract-till-drop) by Mathias Verraes which was a really nice live coding demo of refactoring legacy code.

After the last sessions it was time for the closing remarks, raffles and post-conference social. The organizing team gave us some insight what it is like to create such an event and also thanked the sponsors. There were some great prizes in the raffles or competitions like a 3D printer, a quadro-copter with video camera, few Raspberry Pis and also some elePHPants as is accustomed. Was not my lucky day as I did not win anything.

After the closing it was time for another social and some more bumper cars! Bumper cars were still fun and the BBQ food was really good. Enjoyed it a lot and made some new friends, so again it was a success.

## Retrospective

All in all, the trip was a great success in my mind and I really enjoyed it. There were great presentations and I learned a lot, I got some speaking experience and some good feedback from my UnCon talk and had lots of fun with existing and new friends. What more can you ask for?

I have only mentioned the talks I went to see, but there were many more. You can find all of the talks and slides for most of them through [joind.in](http://joind.in/event/view/1509). I think some of the talks were also filmed, so keep an eye out for videos later. Or you could also try googling for previous recordings as some of the talks have been presented before.

If you want to be a part of the PHP community, I can't recommend enough that you attend the next PHP Benelux conference (and why not some others also if you haven't already). The PHP community is so welcoming and there are so many great people it is really worth the time, money and effort. Already waiting for the next one, hope to see you there.<br>
-Carl.
