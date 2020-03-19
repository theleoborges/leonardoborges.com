+++ 
title = "a few more thoughts on final classes"
date = "2009-10-07"
slug = "2009/10/07/a-few-more-thoughts-on-final-classes"
tags =["Architecture", "Java"]
+++

<p>
I <a href="http://www.leonardoborges.com/writings/2009/03/17/final-classes-are-evil/" target="_blank">said final classes are evil</a> and that post got some attention with interesting comments. Maybe because of the title and the tone I wrote it, a few comments didn't get my real intention and perhaps I should have been more explicit about it. Go ahead and <a href="http://www.leonardoborges.com/writings/2009/03/17/final-classes-are-evil/" target="_blank">read it</a>. I'll wait. :)<br><br>Anyway, I thought I'd expand a little more on that subject, explaining my motivation to write that post and going through the topics I think were raised by my dear readers.<br><br>First off, final classes <strong>are evil for testing</strong>. And that's what it was all about in my previous post.<br><br>If you depend on a final class, your code will be harder to test. Unless the final class provides an interface that captures its intent - or you wrap that dependency.<br><br>But this affirmation has some implications that were pointed out by a few comments, some of which I agree with - others, not so much. So let's start!<br><br><strong>- Immutability </strong><br><br><span style="background-color: #ffffff;">Someone said "Why make a class final ? To make it immutable". This is not entirely true. Only by marking it final you do not ensure immutability. There is no point in doing that if you provide mutators - e.g. setters. - and don't declare your members private and final.</span><br><br>I think it's important to make this clear and understand that the immutability part you achieve by marking a class final is the one of preventing inheritance. Subclasses could possibly contain malicious/careless code and change the internal state of the class.<br><br><span style="background-color: #ffffff;">But there is another way of preventing subclassing without marking the parent final: declare its constructor private and provide a <a href="http://www.javapractices.com/topic/TopicAction.do?Id=21" target="_blank">static factory</a>.</span><br><br><strong>- Designing for extensibility</strong><br><br>This is hard. It basically means that if you don't mark a class final, you should document it for inheritance.<br><br>And this is why inheritance is, in general, a bad OO practice. By documenting the class you basically break encapsulation since you tell the world about your internals.<br><br>Therefore, the recommendation is to mark a class final if you're not sure if it's safe to subclass it - or if you just don't wanna bother writing documentation and thinking too much about your "client" subclasses.<br><br><strong>- Coding against interfaces</strong><br><br>This one is simple but yet often forgotten. Do not code to concrete classes. Always choose interfaces where possible.<br>It roughly means to do this:<br><pre lang="java">  List args = new ArrayList();</pre><br>instead of this:<br><pre lang="java">  ArrayList args = new ArrayList();</pre><br>By doing so you have the flexibility to not care about the implementation you're working with, as long as it obeys the interface. That way, the implementations can be swapped at any time without breaking any client's code.<br><br><strong>- The problem with testing</strong><br><br><strong></strong>All items listed here so far are widely regarded as best practices and the bullet I raised about hard testing usually happens when you "violate" some of them.<br><br>Specifically, if you decide not to design a class for inheritance and mark it as final, it's wise - in my opinion - to try and capture the class's intent through an interface.<br><br>That way you can safely mark your class final and users of your class can easily use the interface to extend it - by favoring <a href="http://www.artima.com/lejava/articles/designprinciples4.html" target="_blank">composition over inheritance</a> - or by providing it to mocking frameworks for easy testing.<br><br><strong>- Conclusion</strong><br><br>I don't really think there is a rule of thumb here. Java's standard library shows many examples of both approaches and some of them are now considered bad practices but yet are there for backward compatibility. Nevertheless, these are points to consider when designing your classes.<br><br>As pointed out by Josua Bloch in his awesome book <a href="http://www.shelfari.com/books/purchase?EditionId=1523265&amp;AssociateId=leonaborge-20&amp;WidgetId=111594" target="_blank">Effective Java</a>, "If a concrete class does not implement a standard interface, then you may inconvenience some programmers by prohibiting inheritance".<br><br>As usual, comments are more than welcome :)
</p>
