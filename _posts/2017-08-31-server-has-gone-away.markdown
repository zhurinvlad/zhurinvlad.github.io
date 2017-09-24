---
layout: post
title:  "MySQL server has gone away. How to fix?"
date:   2017-08-31 09:27:16 +0300
thumbnail_path: blog/mysql_server_has_gone_away/has_gone_away.png
tags:
- Rails
- MySql
categories: software
---

Ошибка MySQL Server Has Gone Away (error 2006) может возникнуть в двух случаях.

{% include media-image.html url="mysql-gone-away.jpg" caption="Схема работы" link="https://ruhighload.com/post/%D0%9A%D0%B0%D0%BA+%D0%B8%D1%81%D0%BF%D1%80%D0%B0%D0%B2%D0%B8%D1%82%D1%8C+Mysql+Server+Has+Gone+Away" width="500" %}

## Таймаут соединения

Наиболее распространенная проблема: таймаут соединения, в результате чего сервер его закрывает. Решение весьма тривиальное — увеличение лимита времени `wait_timeout` в файле конфигурации [`my.cnf`](https://ruhighload.com/post/my.cnf). Для этого в Debian нужно выполнить:
<figure>
    <figcaption># Открытие файла настроек MySQL</figcaption>
    {% highlight powershell %}
    sudo nano 
    /etc/mysql/my.cnf
    {% endhighlight %}
</figure>

Затем установить тайм-аут ожидания:

<figure>
    <figcaption># Время ожидания в секундах, можно установить вплоть до 28800 с (8 часов)</figcaption>
    {% highlight powershell %}
    wait_timeout = 600
   {% endhighlight %}
</figure>

Не забудьте перезагрузить базу:

<figure>
    <figcaption># Перезагрузка базы данных MySQL</figcaption>
    {% highlight powershell %}
    sudo /etc/init.d/mysql restart 
   {% endhighlight %}
</figure>

Иногда, при выполнении длительных запланированных задач, также может появиться ошибка MySQL Server Has Gone Away все из-за того же таймаута соединения. При этом лимит времени не получится существенно увеличить (максимум до нескольких часов), так как это может привести к заполнению буфера ожидающими соединениями. Поэтому лучше проверить соединение и, при необходимости, переподключиться.

## Большой или некорректный пакет

Вторая распространенная проблема: сервер получает большой или некорректный пакет и отклоняет его. В этом случае сервер считает, что проблема на стороне клиента и закрывает соединение. Так что для решения нужно увеличить лимит на максимальный размер пакета все в том же файле конфигурации:
<figure>
    <figcaption># Увеличение лимита размера входящего пакета, в МБ</figcaption>
    {% highlight powershell %}
    [mysqld]
    ...
    max_allowed_packet = 64M
    …
   {% endhighlight %}
</figure>
Также не забудьте перезагрузить базу данных.

##### Самое главное

После того, как устраните ошибку MySQL Server Has Gone Away, поиграйтесь с параметрами `wait_timeout` и `max_allowed_packet` для получения оптимальных лимитов. Стандартные конфигурации MySQL под различные размеры оперативной памяти можно посмотреть на сайте [ruhighload](https://ruhighload.com/post/my.cnf). 

Подробнее об ошибке можно почитать в документации на [русском](http://www.mysql.ru/docs/man/Gone_away.html) и [английском](https://dev.mysql.com/doc/refman/5.7/en/gone-away.html)


## Reconnect в Rails, каков он?

 Каково было мое удивление, когда однажды сервис на **Rails** прислал 1,5к сообщение о том что соединения потеряно, и при этом даже не попробовал повторно подключиться!!!
 
 Как бы странно это не было, но раньше возможности автоматического переподключения не было... Пакеты просто бились об закрытое соединения, заполняя логи и бесконечно уведомляя разработчиков. Приходилось либо в ручную делать MonkeyPatch, либо каждый раз, при возникновении ошибки, перезапускать приложение(особо хардкорный путь).

MySQL поддерживает флаг повторного подключения в своих соединения. И в случае, если установлено значение true, клиент будет пытаться повторно подключиться к серверу, прежде чем отказаться выкинуть ошибку в случаее потерянного соединения. 

И вот, начиная с версии Rails 2.3, у нас появился параметр `reconnect`. Теперь, чтобы получуть такое поведение из приложений Rails, достаточно установить флаг `reconnect: true` для подключений mysql в файле **database.yml**.

Однако по умалчанию этот флаг равен `false`. Команда Rails cделала это затем, чтобы
не нарушать работу текущих приложений, т.к. у данной опции есть существенный минус - **ЭТО НЕ БЕЗОПАСНО ДЛЯ ВЫПОЛНЕНИЯ ТРАНЗАКЦИЙ** . И в документации mysql об этом явно сказано,

``` Any active transactions are rolled back and autocommit mode is reset ```
 
 а также указаны другие [минусы](https://dev.mysql.com/doc/refman/5.7/en/c-api-auto-reconnect.html). Опираясь на это руководство *ActiveRecord* с установленным флагом попытается подключиться всего  ***один раз***!
 
```
The MySQL client library can perform an automatic reconnection to the server if
it finds that the connection is down when you attempt to send a statement to the server to be executed. 
If auto-reconnect is enabled, the library tries once to reconnect to the server and send the statement again.
```

Такое поведение далеко не самое лучшее, ведь возможны случаи, когда нам потребуется больше одной попытки подключения. Например неставить работа сервера при репликации master-slave. И на некоторое время мы должны не потерять соедениение, чтобы обеспечить надежность сервиса и не потерять запрос.

Один из способов сделать это - добавить патч к *AR*, который будет выполнять автоматический reconnect нужно количество раз через нужные нам интервалы.

{% highlight ruby linenos %}
module Mysql2AdapterPatch
  def execute(*args)
    # During `reconnect!`, `Mysql2Adapter` first disconnect and set the
    # @connection to nil, and then tries to connect. When connect fails,
    # @connection will be left as nil value which will cause issues later.
    connect if @connection.nil?

    begin
      super(*args)
    rescue ActiveRecord::StatementInvalid => e
      if e.message =~ /server has gone away/i
        in_transaction = transaction_manager.current_transaction.open?
        try_reconnect
        in_transaction ? raise : retry
      else
        raise
      end
    end
  end

  private
  def try_reconnect
    sleep_times = [0.1, 0.5, 1, 2, 4, 8]

    begin
      reconnect!
    rescue Mysql2::Error => e
      sleep_time = sleep_times.shift
      if sleep_time && e.message =~ /can't connect/i
        warn "Server timed out, retrying in #{sleep_time} sec."
        sleep sleep_time
        retry
      else
        raise
      end
    end
  end
end

require 'active_record/connection_adapters/mysql2_adapter'
ActiveRecord::ConnectionAdapters::Mysql2Adapter.prepend Mysql2AdapterPatch
{% endhighlight %}

В случае, если соединение будет потеряно, он будет пытаться повторить запрос и завершиться успешно когда сервер БД поднимется.

{% highlight powershell %}
>> Post.count
   (0.6ms)  SELECT COUNT(*) FROM `posts`
Server timed out, retrying in 0.1 sec.
Server timed out, retrying in 0.5 sec.
Server timed out, retrying in 1 sec.
Server timed out, retrying in 2 sec.
Server timed out, retrying in 4 sec.
   (1.1ms)  SELECT COUNT(*) FROM `posts`
=> 0
{% endhighlight %}

Стоит отметить, что если разрыв соединения произошел в блоке транзакции, и затем восстановилось, то будут продолжаться выполняться следующие запросы, а все предыдущие запросы от начала транзакции до момента, когда соединение было отключено, будут проигнорированы. Вот почему с этим патчем в блоке *transaction* безопаснее повторно вызвать ошибку подключения.

Рассмотрим пример:
{% highlight ruby %}
Post.transaction do
  Post.create
  sleep 5
  Post.count
end
{% endhighlight %}

В данном случае, если при `sleep 5` произошло успешное переподключение, то все равно будет вызвана ошибка подключения, т.к. в соответствии с документацией MySQL, описанной выше, Post.create выполнено не будет.

{% highlight powershell %}
   (0.3ms)  BEGIN
  SQL (0.2ms)  INSERT INTO `posts` (`created_at`, `updated_at`) VALUES ('2017-01-18 20:18:14', '2017-01-18 20:18:14')
   (0.2ms)  SELECT COUNT(*) FROM `posts`
Server timed out, retrying in 0.1 sec.
Server timed out, retrying in 0.5 sec.
Server timed out, retrying in 1 sec.
Server timed out, retrying in 2 sec.
   (0.1ms)  ROLLBACK
ActiveRecord::StatementInvalid: Mysql2::Error: MySQL server has gone away: SELECT COUNT(*) FROM `posts`
{% endhighlight %}