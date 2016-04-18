---
title: Security in Web Development & modern PHP frameworks
categories:
  - Talks
tags:
  - PHP
  - Security
  - Symfony2
  - Zend Framework
---

After the recent PHP User Group Finland meeting I started thinking the presentation I gave at PHPUG Finland a few years ago and decided to post the slides here since I didn't have a blog at the time to post them in. So, here are the slides from my PHP User Group Finland talk 24.11.2011 (in Finnish). The talk is titled Tietoturva Web -kehityksessä &amp; Zend Frameworkissä (Security in Web Development &amp; Zend Framework) and it was a two-part session between me first covering few of the most critical security threats in web applications and how to prevent them in Zend Framework and after that Matti Suominen demonstrated how these threats can be exploited in practice.

The topics covered in the presentation are SQL injection, Cross-site scripting (XSS), Cross-site request forgery (CSRF) and password hashing. After about 1,5 years, the points made and the tools presented in the talk are still mostly relevant, but Zend Framework 1 has since been surpassed by newer frameworks, namely Symfony2 and Zend Framework 2. So I decided to revisit these topics a little bit in this post in regard to todays standards.

<!--more-->

Here are the slides of the talk (in Finnish):
<iframe style="border: 1px solid #CCC; border-width: 1px 1px 0; margin-bottom: 5px;" src="http://www.slideshare.net/slideshow/embed_code/15055214" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" width="427" height="356"></iframe>
<div style="margin-bottom: 5px;"><strong> <a title="Tietoturva web-kehityksessä &amp; Zend Frameworkissä" href="http://www.slideshare.net/cvuorinen/tietoturva-webkehityksess-zend-frameworkiss" target="_blank">Tietoturva web-kehityksessä &amp; Zend Frameworkissä</a> </strong> from <strong><a href="http://www.slideshare.net/cvuorinen" target="_blank">cvuorinen</a></strong></div>
And here is a video of my part of the presentation split into two parts (in Finnish):<br>
[Part 1](http://www.youtube.com/watch?v=EE45mBY929Y&amp;list=UUKS_noOuXiwljOTWyeIbJOQ)<br>
[Part 2](http://www.youtube.com/watch?v=aty-YE_6B88&amp;list=UUKS_noOuXiwljOTWyeIbJOQ)

You can also find the second part by Matti from the playlist in YouTube if you open the above links.

## SQL injection

Things have not changed that much regarding SQL injection. Different frameworks have their own way of abstracting the raw SQL queries away from the casual developer, like Doctrine and Zend\Db. That makes things easier for the developer in most situations, but it can also lead to a false sense of security. If a developer never has to worry about SQL injections since the actual queries are abstracted away, then they are less likely to pay attention to them when they actually do need to write some SQL. Maybe it's for tuning the performance of some heavy query or the abstraction layer doesn't support some feature or special case, there are many situations when you still need to write SQL queries by hand and then it's better to be aware what security implications there might be. The points made in the presentation are still valid today, when you need to write SQL queries by hand, the ways to protect against SQL injection are variable escaping, input filtering &amp; validation and prepared statements.

## Cross-site scripting (XSS)

Regarding XSS protection, there have been some developments after the presentation. Symfony's template engine Twig has automatic escaping, so that the developer does not have to call an escape method on each variable value that they intend to output. This makes things easier and most of the time more secure since the developer can't forget to escape something. But the thing is, that is not enough since the proper way to escape depends on the context. Pádraic Brady has [a good post](http://blog.astrumfutura.com/2012/06/automatic-output-escaping-in-php-and-the-real-future-of-preventing-cross-site-scripting-xss/) on the subject so you can read more about this topic if interested.

And if you need to output user submitted HTML, HTMLPurifier is still the way to go, as was at the time of the presentation. Again I quote Pádraic Brady "there is NO other secure HTML sanitizer in PHP!". That quote is from the README of Pádraic's [SecurityMultiTool project](https://github.com/padraic/SecurityMultiTool), that you should definitely check out and keep an eye on since it offers many security related libraries and best practices so that you don't have to implement them yourself. It also has a secure HTML escaper class that can be used to securely escape in different contexts (HTML, HTML attribute, JS, CSS, URL).

## Cross-site request forgery (CSRF)

CSRF protection is also pretty much the same today as it was at the time of the presentation. Hidden unique token that is submitted with the form is the way to go, and most frameworks have something built-in for this in their form component, like the csrf field type in Symfony2 and Zend\Form\Element\Csrf in Zend Framework 2.

## Password hashing

Regarding password hashing, the computers are of course always becoming more powerful and faster so brute forcing passwords is also faster. But the currently recommended best practice of hashing with bcrypt is still the same as 1,5 years ago. With bcrypt, the thing is that it was designed to be future proof. It has a cost parameter that can be used to make it slower and slower as computers become faster and faster over time. So you should test it with the hardware that you are going to use for production and modify the cost parameter accordingly. If you are interested in the topic, Anthony Ferrara has some numbers regarding brute forcing on modern hardware and some background on how password hashing has evolved etc. in the [slides of a recent talk](http://www.slideshare.net/ircmaxell/password-storage-and-attacking-in-php) he gave at the PHPBenelux 2013 conference.

The password hashing library PHPASS that I promoted at the time of the presentation is still a solid option today, but since then Anthony Ferrara has developed an even easier to use library, that will actually be in the PHP core since version 5.5. Until then, you can use the library [password_compat](https://github.com/ircmaxell/password_compat) that provides forward compatibility for the password hashing functions that will be in the PHP core, meaning that you can start using the password hashing functions in exactly the same way now as when they will be in PHP core and when you upgrade PHP to a version that has them you don't need to change anything, you can just remove the library dependency. Symfony2 and Zend Framework 2 both also have their own password hashing components that are secure and use bcrypt (or at least give the option to use bcrypt). Just make sure you are using bcrypt and remember to check and adjust the cost parameter!

## Conclusion

Mostly the security threats concerning web applications have not changed that much in the past 1,5 years and also the means to protect applications against them are pretty much the same. Modern frameworks have good security practices and provide many security related features built-in that have been left for the developer to worry about in the past.

If you do PHP development and are interested in security (you should be) then I highly recommend you follow the work of [Pádraic Brady](https://twitter.com/padraicb) and [Anthony Ferrara](https://twitter.com/ircmaxell). They both have great stuff in their GitHub repos, they tweet interesting things and write great blog posts.

As a final note, I want to mention one more security related tool that has recently been published. The [Security Advisories Checker](https://security.sensiolabs.org/) by Fabien Potencier and SensioLabs is a tool that can be used to check if any of your Composer dependencies have known security vulnerabilities. It's so easy to use that there really is no excuse not to use it.
