+++ 
title = "my take on rails environment specific constants"
date = "2010-01-23"
slug = "2010/01/23/my-take-on-rails-environment-specific-constants"
tags =["Rails"]
+++

<p>
It’s funny how every Rails application I - and possibly you - work on ends up needing some sort of per-environment global constants.<br><br>Examples may include the application url - It might be used in account activation emails and thus should be different between the development and production environments.<br><br>Or perhaps your application depends on external services that, depending on the environment, are available in different URIs.<br><br>There are a couple solutions out there but my needs were simple and straightforward, thus I developed a small rails plugin that is the simplest thing that could possibly work: <strong><a href="http://github.com/leonardoborges/app_constants">AppConstants</a></strong>.<br><br>It's been useful for my current project and I hope it can be useful to someone else too ;)
</p>

