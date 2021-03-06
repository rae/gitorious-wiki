Components
==========

When installing Gitorious on their own most people feel overwhelmed by the number of moving parts. First of all: don't worry! Gitorious isn't just a web application; it also speaks several other protocols (like Git and SSH), and performs some complex tasks in Git. 

This page describes the various parts of Gitorious, and how they interact.

RAILS_ENV
---------

Being a Ruby on Rails application, Gitorious supports different environments. Most Rails applications will have three different environments:

* development is the environment used when developing the application. This environment is very practical on your local machine; classes are automatically reloaded on each request etc., but not suited in a production setting. Use this environment when you're hacking Gitorious itself.
* test is the environment used when running the test suites in Gitorious. The test environment's database will be flushed whenever you run the tests, so make sure to use a separate database for your test environment, or your production data will be wiped. 
* production is the environment intended for your production environment, ie. where you use Gitorious to share repositories and projects. Unless you're hacking on Gitorious, the production environment is what you'll be using.

Keep in mind that each Rails environment has its own settings (like database connections and Gitorious settings), you need to make sure you use the same environment in all the parts you run. Setting the environment variable RAILS_ENV before/when calling a script will take care of this for you:

     env RAILS_ENV=production script/poller start


The messaging server
--------------------

Some operations Gitorious performs take too long to run for the user to wait for it to complete before a page is rendered. Cloning a large Git repository, for example, can take several minutes to complete. 

Gitorious solves this by creating a message describing the operation that should be performed, and sending this off to a separate process. This is achieved using [Resque](https://github.com/resque/resque).

Furthermore, you'll probably want to make sure it hasn't died or takes up too much memory. Have a look at [Monit](http://mmonit.com/monit/) if you don't already have a tool to monitor long running processes on your server. 


The git daemon
--------------

In order for users to be able to use the Git protocol for cloning, the git daemon needs to be running on a port that's accessible through the firewall. The git daemon is a script that ships with Gitorious, and is located in script/git-daemon. Like the poller script, this can be daemonized, and is started and stopped with the same syntax. The same goes for monitoring.

HTTP cloning
------------

The problem with the Git and SSH protocols is that people working from behind a firewall may not be able to access the repositories. For this purpose, Gitorious supports cloning repositories over HTTP - which should be open for most users - even through a proxy. HTTP is the least efficient of the three protocols (Git and SSH being the two others), but in some cases it's the only possibility.

When cloning over HTTP, all requests are handled before the request reaches the Rails stack, by a separate part of Gitorious. This recognizes requests to the "git" subdomain and treat these as Git requests (over HTTP), unless the hide_http_clone_urls setting in config/gitorious.yml is set to true.

To get this working, you'll need the [mod_xsendfile Apache module](http://tn123.ath.cx/mod_xsendfile/) installed and configured.

Git repository archives (.tgz etc)
------------

Just like for HTTP cloning, downloading archives requires mod_xsendfile to be installed and configured correctly.

The sphinx daemon
-----------------

Gitorious uses Sphinx for full text searching. This is a separate daemon that must be installed, have a look in the doc/recipes folder in the source code for more information. Again, make sure this daemon is running.

The sphinx rake task
--------------------

The full text index needs to be updated regularly; Gitorious ships with a rake task for this. 


    * */1 * * * cd <gitorious_root> && <rake> ultrasphinx:index RAILS_ENV=production


in your cron tab will make sure the index is updated regularly. Substitute <gitorious_root> with the full path to you Gitorious installation, and <rake> with the full path to your rake binary.


    which rake


will tell you where Rake resides on your system. 

Frontend web server
-------------------

All Rails applications can be run using the script/server command. This is fine while you're hacking Gitorious, but not usable in a production environment. 


There are a lot of alternatives to choose from when deploying Rails applications, most of which spawn a number of Rails processes which are shared by a front end web server like Apache or Nginx. Gitorious will run fine with any of these.


These days, most people will use either Nginx or Apache in combination with the excellent [passenger, a.k.a mod rails module](http://modrails.com/). Passenger takes care of starting the Ruby processes for you.
