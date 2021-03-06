+++ 
title = "monit thinking sphinx and rvm"
date = "2010-01-13"
slug = "2010/01/13/monit-thinking-sphinx-and-rvm"
tags =["Rails", "Sphinx"]
+++

<p>
In one of my Rails projects I'm using <a href="http://www.sphinxsearch.com/" target="_blank">Sphinx</a> to provide full-text search capabilities. To integrate both worlds I chose <a href="http://freelancing-god.github.com/ts/en/" target="_blank">Thinking Sphinx</a>, which is just great and so far has met all of my expectations.<br><br>Also, <a href="http://www.leonardoborges.com/writings/2010/01/02/managing-multiple-ruby-versions/" target="_blank">as I previously mentioned</a>, I'm using <a href="http://rvm.beginrescueend.com/">RVM</a> to manage my ruby installations on both my development and production machines and this setup is what motivated this post.<br><br>I use <a href="http://mmonit.com/monit/" target="_blank">Monit</a> to monitor the services running on my production server - nginx, mysql, php - and as of the first deploy of this application, it only made sense to also monitor Sphinx.<br><br>In order to create an initialization script, I would need at least a way to start and stop sphinx from the command line, which, using Thinking Sphinx, can be done using these rake tasks:<br><pre lang="bash"><br>$ rake thinking_sphinx:stop<br>$ rake thinking_sphinx:start<br></pre><br>It's worth mentioning now that I don't run sphinx as root. I run it with the same user my rails application uses. For the purpose of this post, let's call it <em>deploy</em>.<br><br>When I tried using my script I got errors such as these:<br><br><pre lang="bash"><br>Missing the Rails 2.3.5 gem. Please `gem install -v=2.3.5 rails`, update your RAILS_GEM_VERSION setting in config/environment.rb for the Rails version you do have installed, or comment out RAILS_GEM_VERSION to use the latest version installed.<br></pre><br><br>After some digging around I found that the GEM_HOME environment variable for my deploy user wasn't being set correctly - something to do with rvm, but not quite sure - and to fix this, well, I just had to set it, leaving the final working version of my script somewhat like this - simplified :


``` bash
#! /bin/sh

### BEGIN INIT INFO
### END INIT INFO

set_path="cd /your/rails/app/root; export GEM_HOME=/path/to/your/gems; RAILS_ENV=production;"

case "$1" in
  start)
        echo -n "Starting sphinx: "
                su - deploy -c "$set_path rake ts:start" >> /var/log/sphinx.log 2>&1
        echo "done."
        ;;
  stop)
        echo -n "Stopping sphinx: "
                su - deploy -c "$set_path rake ts:stop" >> /var/log/sphinx.log 2>&1
        echo "done."
        ;;
      *)
            N=/etc/init.d/sphinx
            echo "Usage: $N {start|stop}" >&2
            exit 1
            ;;
esac

exit 0
```

There are 2 things happening here. First, regardless of the user I'm running this init script as, I drop privileges in order to execute the rake task with my deploy user. Second, I set the GEM_HOME environment variable to stop getting the 'gem not found' errors.<br><br>After that, monit was able not only to monitor my sphinx instance, but also (re)start/stop it with the correct user.<br><br>I'm no Linux wizard so if you wanna suggest improvements to this script, feel free to do so!
</p>

