---
layout: post
title:  "Rails 5 предотвращает нечаянное уничтожение данных на production database"
date:   2017-09-15 09:27:16 +0300
thumbnail_path: blog/rails_5_prevents_destructive_action/old_rails.jpg
tags:
- Rails
categories: software
---

Бывают такие ошибки, которые повторяются только на production. Для того чтобы ее отладить некоторые разработчики загружают настроки production на локальную машину для отладки этого бага, а иногда разработчики и вовсе ошибочно выполняют команды, запущенные не в том окружении. Все этого может уничножить данные на production среде! Очень мого пользователей жаловались на это и Richard Schneeman стал героем, который решил эту проблему.

Rails 5 предотвращают уничножение данных на production базе данных.

В Rails 5 [добавлена](https://github.com/rails/rails/pull/22967) новая таблица **`ar_internal_metadata`**, содержащая версию `environment`, которая использовалась во время миграции базы данных.

Когда в первый раз мы запускаем `db:migrate`, то в таблицу сохраняется значение **production**. Теперь, когда мы попытаемся загрузить [database schema](https://github.com/rails/rails/pull/24399) или [database structure](https://github.com/rails/rails/pull/24484) запуская `rake db:schema:load` или `rake db:structure:load` соответственно. Rails проверит environment в котором запущено Rails. Если environment не равняется **production**, то Rails вызовет исключение и, таким образом, предотвратит уничтожение данных.

Чтобы пропустить эту проверку среды, мы можем вручную передать  `DISABLE_DATABASE_ENVIRONMENT_CHECK=1` в качестве аргумента с командой `db:schema:load`/`db:structure:load`.

Вот пример запуска `db:schema:load` на production database.

{% highlight powershell %}
$ rake db:schema:load

rake aborted!
ActiveRecord::ProtectedEnvironmentError: 
You are attempting to run a destructive action against 
your 'production' database. If you are sure you want to continue, 
run the same command with the environment variable:
DISABLE_DATABASE_ENVIRONMENT_CHECK=1
{% endhighlight %}

Как мы видим, Rails предотвратил уничтожение данных на production. А вот команда `RAILS_ENV=production rake db:schema:load` отработает корректно.