+++ 
title = "a couple of things from here"
date = "2008-05-10"
slug = "2008/05/10/a-couple-of-things-from-here"
tags = ["Rails", "Ruby", "World"]
+++

<p>
It's been some time since my last post but here I am! Where? In Spain, of course! Having a great time, I must say.<br><br>I arrived last week in Madrid and the past 2 weeks before that I spent basically packing my stuff. There is still some paperwork going on but everything is flowing well.<br><br>Besides this little feedback, I was reading this week's issue of the excellent series <a href="http://antoniocangiano.com/2008/05/05/this-week-in-ruby-may-5-2008/">This Week In Ruby</a>, from my friend <a href="http://antoniocangiano.com/">Antonio Cangiano</a>. I found something quite interesting, a plugin called <a href="http://hobocentral.net/hobofields/">HoboFields</a>.<br><br>One of the things that bothers me in rails is the fact that by looking at your model classes, you can't tell the fields you have there. Sure, you can look at the migration script. Yeah, you can also load the development environment and inspect the object. It's a pain in the @zz! But this is the way ActiveRecord works...<br><br>Other ORM solutions like <a href="http://datamapper.org/">DataMapper</a>, allows you to define the fields directly in the class. It's a much cleaner and clear way to maintain your models. And you get to know what properties you have just by looking at your classes.<br><br>That's exactly what HoboFields adds to ActiveRecord.<br><br>You define your properties and its types straight into your model class, and the plugin creates the migration scripts for you. Coming from a java world my self, I find it rather interesting, useful and it also reminds me of the way <a href="http://hibernate.org">Hibernate</a> works. You define your mappings with anotations in your class and hibernate just generate the schemas from there.<br><br>It's worth a try.
</p>

