---
layout: post
title:  "Rails 5 prevents destructive action on production database"
date:   2017-09-15 09:27:16 +0300
thumbnail_path: blog/rails_5_prevents_destructive_action/old_rails.jpg
tags:
- Rails
categories: software
---

This blog is part of our Rails 5 series.

Sometimes while debugging production issue mistakenly developers execute commands like `RAILS_ENV=production rake db:schema:load`. This wipes out data in production.

Users of heroku download all the config variables to local machine to debug production problem and sometimes developers mistakenly execute commands which wipes out production data. This has happened enough number of times to heroku users that Richard Schneeman of heroku decided to do something about this issue.

Rails 5 prevents destructive action on production database

Rails 5 [has added](https://github.com/rails/rails/pull/22967) a new table **`ar_internal_metadata`** to store `environment` version which is used at the time of migrating the database.

When the first time rake `db:migrate` is executed then new table stores the value **production**. Now whenever we load [database schema](https://github.com/rails/rails/pull/24399) or [database structure](https://github.com/rails/rails/pull/24484) by running rake `db:schema:load` or rake `db:structure:load` Rails will check if Rails environment is “production” or not. If not then Rails will raise an exception and thus preventing the data wipeout.

To skip this environment check we can manually pass `DISABLE_DATABASE_ENVIRONMENT_CHECK=1` as an argument with load schema/structure db command.

Here is an example of running rake `db:schema:load` when development db is pointing to production database.

{% highlight powershell %}
$ rake db:schema:load

rake aborted!
ActiveRecord::ProtectedEnvironmentError: You are attempting to run a destructive action against your 'production' database.
If you are sure you want to continue, run the same command with the environment variable:
DISABLE_DATABASE_ENVIRONMENT_CHECK=1
{% endhighlight %}

As we can see Rails prevented data wipeout in production.

This is one of those features which hopefully you won’t notice. However if you happen to do something destructive to your production database then this feature will come in handy.