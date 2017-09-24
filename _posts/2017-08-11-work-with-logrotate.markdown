---
layout: post
title:  "Работа с Logrotate"
date:   2017-08-11 11:53:16 +0300
thumbnail_path: blog/logrotate/stock.jpg
tags:
- Software
- DevOps
categories: software
---

После запуска Rails-приложений полезно настроить автоматическую архивацию (ротацию) файлов журналов (лог-файлов), так как они имеют обыкновение крайне быстро разрастаться до весьма объемных размеров:

{% highlight powershell %}
$ ls -lh log/production.log
  	-rw-rw-r-- 1 www-data www-data 193,2M feb 20 17:49 production.log
{% endhighlight %}
Найти что-либо в 200-мегабайтном файле не очень просто ;)

Для ротации логов можно использовать стандартный интструмент [logrotate][logrotate-docs]. Если его нет в системе, то в случае Debian или Ubuntu его можно просто установить командой:

{% highlight powershell %}
sudo apt-get install logrotate
{% endhighlight %}

В планировщике задач (cron) ежедневно выполняется запуск `logrotate 
/usr/sbin/logrotate /etc/logrotate.conf` с указанием файла конфига 
В конфиге настраиваются глобальные параметры, которые будут применяться по умолчанию, и как правило подключается директория
`include /etc/logrotate.d` откуда будут подружатся файлы с описанием правил (секции) для конкретных лог файлов. Пример глобального конфига:
{% highlight powershell %}
create
compress

include /etc/logrotate.d
{% endhighlight %}
*При ротации логов - текущий log-файл с которым работает программа - удаляется или перемещается, поэтому после ротации лога, будет правильным перезапустить программу/сервис, чей log-файл был удален. Нужно это, что бы программе был сообщен новый дискриптор файла. Хотя это ситуация может разрулится автоматически и без перезагрузки - если такую ситуацию предусмотрели разработчики.*

### Параметры настройки ### 
Вот список всех параметров на текущий момент:
<table class= "table-content">
   <thead>
      <tr>
         <th>Параметр </th>
         <th>Описание </th>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td> <strong>rotate &lt;число&gt;</strong> </td>
         <td> Количество хранимых файлов </td>
      </tr>
      <tr>
         <td> <strong>daily</strong> <br>
            <strong>weekly</strong> <br>
            <strong>monthly</strong> 
         </td>
         <td> Производить ротацию раз в день/неделю/месяц  </td>
      </tr>
      <tr>
         <td> <strong>size &lt;байт&gt;</strong> <br>
            <em>size 1000 <br>
            size 100k <br>
            size 1M </em> 
         </td>
         <td> Производить ротацию если log-файл превысил указанный размер <br>
            байт <br>
            Кбайт <br>
            Мбайт 
         </td>
      </tr>
      <tr>
         <td> <strong>start &lt;число&gt;</strong> </td>
         <td> число с которого начнётся нумерация файлов </td>
      </tr>
      <tr>
         <td> <strong>compress</strong> </td>
         <td> Архивировать файлы (по умолчанию gzip) </td>
      </tr>
      <tr>
         <td> <strong>nocompress</strong> </td>
         <td> Отключает <em>compress</em> </td>
      </tr>
      <tr>
         <td> <strong>delaycompress</strong> </td>
         <td> Не сжимать 'свеже' созданный архив. Например access.log. не будет зжат. <br>
            Используется с <em>compress</em> 
         </td>
      </tr>
      <tr>
         <td> <strong>create  &lt;права&gt;&lt;владелец&gt;&lt;группа&gt;</strong> <br>
            <em>create  root root</em> 
         </td>
         <td> После ротации создать пустой log-файл. Любые из этих атрибутов могут быть опущены, <br>
            в этом случае вместо них для нового файла будут использованы атрибуты, <br>
            имеющие те же значения, что и первоначальный log-файл 
         </td>
      </tr>
      <tr>
         <td> <strong>nocreate</strong> </td>
         <td> Не создавать файл </td>
      </tr>
      <tr>
         <td> <strong>copy</strong> </td>
         <td> Создать копию оригинального log-файла, не изменяя его. Исключает <em>create</em> </td>
      </tr>
      <tr>
         <td> <strong>nocopy</strong> </td>
         <td> Отключает <em>copy</em> </td>
      </tr>
      <tr>
         <td> <strong>copytruncate</strong> </td>
         <td> Создать копию оригинального log-файла, а потом его 'обнулить'. <br>
            Таким образом сам файл <strong><em>не</em></strong> удаляется. <br>
            Исключает <em>copy, create</em>
         </td>
      </tr>
      <tr>
         <td> <strong>ifempty</strong> </td>
         <td> Архивирует даже пустой файл (используется по умолчанию) </td>
      </tr>
      <tr>
         <td> <strong>notifempty</strong> </td>
         <td> Не архивировать пустые файлы </td>
      </tr>
      <tr>
         <td> <strong>missingok</strong> </td>
         <td> В случае отсутствия оригинального log-файла не вызовет ошибку </td>
      </tr>
      <tr>
         <td> <strong>nomissingok</strong> </td>
         <td> В случае отсутствия оригинального log-файла вызовет ошибку </td>
      </tr>
      <tr>
         <td> <strong>postrotate</strong> <br>
            <em>&lt;команды&gt;</em> <br>
            <strong>endscript</strong> 
         </td>
         <td> Строки, находящиеся между <em>postrotate</em> и <em>endscript</em> <br>
            будут выполнены как sh скрипт после архивирования log-файла 
         </td>
      </tr>
      <tr>
         <td> <strong>prerotate</strong>  <br>
            <em>&lt;команды&gt;</em> <br>
            <strong>endscript</strong> 
         </td>
         <td> Аналогично <em>postrotate</em>, только действия будут выполнены до начала архивирования </td>
      </tr>
      <tr>
         <td> <strong>sharedscripts</strong> </td>
         <td> Скрипты <em>postrotate</em> и <em>prerotate</em> будут выполнены только один раз в рамках своей секции. </td>
      </tr>
      <tr>
         <td> <strong>nosharedscripts</strong> </td>
         <td> Отключает <em>sharedscripts</em>. <br>
            Скрипты будут выполняются при ротации <em>каждого</em> log-файла, <br>
            при определение <em>/var/log/apache2/*.log</em> скрипт будет выполнен столько раз  <br>
            сколько уникальных log-файлов будет находится в данной директории 
         </td>
      </tr>
      <tr>
         <td> <strong>olddir &lt;путь&gt;</strong> <br>
            <em>olddir /home/logs</em>
         </td>
         <td> Перемещать архивные файлы в указанную директорию </td>
      </tr>
      <tr>
         <td> <strong>noolddir</strong> </td>
         <td> Отключает <em>olddir</em> </td>
      </tr>
      <tr>
         <td> <strong>dateext</strong> </td>
         <td> К имени файлов журналов добавляется дата (%Y%m%d), вместо номера </td>
      </tr>
      <tr>
         <td> <strong>su &lt;user&gt; &lt;group&gt;</strong> </td>
         <td> Выполняется с правами указанного пользователя. Необходимо если ошибка: "because parent directory has insecure permissions", т.е. на директорию с логами, есть право на запись кроме root'a </td>
      </tr>
   </tbody>
</table>


### Пример настройки logrotate ###
Создаём файл конфигурации logrotate для Rails-приложения `/etc/logrotate.d/rails_example_com` (настройки для нескольких приложений можно хранить в одном файле, но более правильно использовать по одному файлу на приложение).
Содержание файла должно быть примерно такое:
{% highlight powershell %}
/path/to/rails/app/log/*.log {
  daily
  missingok
  rotate 7
  compress
  delaycompress
  notifempty
  copytruncate
}
{% endhighlight %}
## Запуск ##
#### Параметры запуска ####
<table>
  <thead>
    <tr>
      <th>Параметр </th>
      <th>Описание </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td> <strong>-d</strong> </td>
      <td> debug - отладочный режим. <br>
        В режиме отладки не будут производиться изменения в log-файле и файле состояния 
      </td>
    </tr>
    <tr>
      <td> <strong>-f <br>
        --force</strong> 
      </td>
      <td> Принудительно произвести ротацию, даже если в данный момент она не требуется </td>
    </tr>
    <tr>
      <td> <strong>-m &lt;команда&gt; <br>
        --mail &lt;команда&gt;</strong> 
      </td>
      <td> Указать команду для отправки почты. <br>
        Команда должна принимать два аргумента: тема сообщения и адрес получателя. <br>
        Текст письма передается стандартным вводом (stdin). <br>
        По умолчанию  <em>/usr/bin/mail -s</em> <br>
      </td>
    </tr>
    <tr>
      <td> <strong>-s &lt;файл&gt; <br>
        --state &lt;файл&gt;</strong> 
      </td>
      <td> Указать куда записать файл состояния. <br>
        Что - то типа лога, показывает последнюю дату когда производилась ротация <br>
        (если создания архива не требуется правилами, то дата все ровно обновляется) <br>
        По умолчанию <em>/var/lib/logrotate/status</em> 
      </td>
    </tr>
    <tr>

    </tr>
  </tbody>
</table>
Для проверки работы можно запустить logrotate с параметром -d (отладка), благодаря которому logrotate покажет, как он будет работать, но ничего не изменит:
{% highlight powershell %}
sudo logrotate -d /etc/logrotate.d/rails_example_com
{% endhighlight %}
Если logrotate в тестовом режиме отработал без ошибок, то можно ждать, когда он отработает автоматически на следующие сутки или запусить его вручную:
{% highlight powershell %}
sudo logrotate -f /etc/logrotate.d/rails_example_com
{% endhighlight %}
Также, можно второй раз вручную запустить logrotate, чтобы убедиться, что опция delaycompress работает, результатом двух запусков logrotate должно быть следующее:
{% highlight powershell %}
ls
production.log production.log.1 production.log.2.gz
{% endhighlight %}

Что мы видим: 
- *production.log* — остался на месте, пуст и доступен для записи для приложения; 
- *production.log.1* — копия логов между первым и вторым запуском logrotate;
- *production.log.2.gz* — сжатый лог за всё предыдущее время.

С указанными настройками каждый день будет проходит архивация лог-файла, будут храниться логи за последние 7 дней и каждый день будет удаляться самый старый лог.
Каждому архивному файлу присваивается номер, чем больше номер тем 'старее' архив. 

## Варианты настроек ##
В предложенном ниже файле настройки для хранения журналов за последний год с еженедельной архивацией:
{% highlight powershell %}
/path/to/rails.example.com/tmp/log/*.log {
    weekly
    missingok
    rotate 52
    compress
    delaycompress
    notifempty
    copytruncate
}
{% endhighlight %}

Ротация логов Апача:
{% highlight powershell %}
/var/log/apache2/*.log {
        weekly
        missingok
        rotate 52
        compress
        delaycompress
        notifempty
        create 640 root adm
        sharedscripts
        postrotate
                # Скрипт на перезагрузку Апача
                if [ -f "`. /etc/apache2/envvars ; echo ${APACHE_PID_FILE:-/var/run/apache2.pid}`" ]; then
                        /etc/init.d/apache2 reload > /dev/null
                fi
        endscript
}
{% endhighlight %}

[logrotate-docs]: https://linux.die.net/man/8/logrotate