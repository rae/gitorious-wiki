# Soon

* atom feed of wiki page history
* opt out of wiki project setting
* Better wiki css
* Filtering of events on the project index page ("don't show wiki edits" etc)
* be able to clone the wiki repository
* be able to push to the wiki repository (project admins only?)
* Add multiple emails to a user account so user will be recognized when pushing

# Later

* xapian (or other) based search
  So we can index the non-sql resources we have, such as blobs and wiki pages
* Use Delayed::Job for queueing, instead of the home-rolled  Task setup
  Allows for always running a daemon, instead of cronjob. Along with nicer features and easier to support queuing more things. Along with not requiring any extra dependencies 
* Better dashboard
* More consistent UI

