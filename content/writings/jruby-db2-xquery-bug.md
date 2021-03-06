+++ 
title = "jruby db2 xquery bug"
date = "2008-04-07"
slug = "2008/04/07/jruby-db2-xquery-bug"
tags = ["JRuby", "Rails", "Ruby"]
+++

<p>
<strong>Update</strong>: Follow up link to this issue on JRuby's Jira, <a href="http://jira.codehaus.org/browse/JRUBY-2430">here</a><br><br>As I told in my last <a href="http://leonardoborges.com/writings/2008/04/04/qcon-2008-slides-available/">post</a>, it was time to give JRuby a serious try. So I took one of our rails projects at work and decided to migrate it to JRuby and see what happens.<br><br>We heavily use the XML capabilities of DB2 and this was a huge problem. Every query would work just fine through the activerecord-jdbc-adapter - part of the <a href="http://rubyforge.org/projects/jruby-extras">JRuby Extras</a> . But every <strong>X</strong>query would gracefully <strong>fail</strong>!<br><br>After some debugging I got stuck and decided to get JRuby and activerecord-jdbc-adapter's source to see what was happening.<br><br>As I could see, it has a bug -in my opinion - at the java part of the code. The jdbc-adapter is a bridge to allow Active Record to talk with databases through native JDBC drivers, so it's normal that we do have a java part here. At this point, what the code does is to inspect the sql statement sent from ruby and decide if it's a select, update or insert.<br><br>I fixed the problem and submitted a <a href="http://rubyforge.org/tracker/index.php?func=detail&amp;aid=19340&amp;group_id=2014&amp;atid=7859">patch</a> to rubyforge. I'm not sure if it's the best solution or not, but now I got the xQueries working just fine.<br><br>I'd love to hear from people with similar environments whether this patch works for you or not. I'm sure I didn't try every possibility.<br><br>If you wanna try it, just drop me a message (e-mail in the <a href="http://leonardoborges.com/writings/about/">About</a> page) and I can send the pre-compiled jar file - for activerecord-jdbc-0.8<br><br>You can also just check out the code and compile yourself. ;)
</p>

