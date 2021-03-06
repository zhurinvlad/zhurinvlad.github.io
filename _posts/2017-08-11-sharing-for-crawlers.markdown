---
layout: post
title:  "Sharing for Crawlers"
date:   2017-08-11 12:53:16 +0300
tags:
- js
- DevOps
- Crawlers
thumbnail_path: blog/sharing-for-crawlers/workflow.png
categories: software
---

## Проблема ##
Роботы и краулеры не умеют выполнять JavaScript, поэтому если это не "статика", которая отдается с сервера, а, например, AngularJS, контент не будет загружен. Чтобы crawlers увидел необходимую информацию требуется дополнительная настройка веб-сервер, который проверяет `User-Agent`. В случае идентификации краулера, вместо того, чтобы показывать простой файл шаблона, например, AngularJS, веб-сервер перенаправляет его на страницу, созданную сервером приложения, которая будет содержать желаемые метатеги, заполненные правильной информацией. 
{% include figure.html path=page.thumbnail_path alt="Схема работы такого подхода" url="https://github.com/DerekCuevas/friend-list"%}

#### Несколько полезных ссылок для шаринга информации с помощью crawlers (краулеров) ####

1. Блог одного хорошего [разработчика][michaelbromley] из Манчестера, в котором все подробно изложено.
2. Документация cтандарта [OpenGraph][sharing-open-graph]
3. Документация по шарингу в [Twitter][sharing-twitter], т.к. он использует свои meta-тэги.
4. Документация по настройке [nginx][nginx-doc]

**Example in etc/nginx/sites-avaliable/site:**
{% highlight powershell %}
location = /url/:id
{
    if ($http_user_agent ~ vkShare|facebookexternalhit|Twitterbot|Facebot|Pinterest|Google.*snippet) {
        proxy_pass http://localhost:port/url/$arg_id;
    }
    try_files $uri/index.html $uri @app;
}
$arg_id -  если с параметром ?id=1
{% endhighlight %}
**Example html:**
{% highlight html %}
<meta name="twitter:card" content="summary" />
<meta name="twitter:site" content="@name" />
<meta name="twitter:creator" content="@name" />
<meta property="og:description" content="<%=strip_tags(@ob[:content])%>" />
<meta property="og:image" content="<%=@ob.image.url%>" />
<meta property="og:title" content="<%=@ob[:title]%>"/>
<meta property="og:site_name" content="site.com" />
{% endhighlight %}


[michaelbromley]: https://www.michaelbromley.co.uk/blog/enable-rich-social-sharing-in-your-angularjs-app/
[sharing-twitter]: https://dev.twitter.com/cards/getting-started
[sharing-open-graph]: https://yandex.ru/support/webmaster/open-graph/intro-open-graph.xml
[nginx-doc]: https://nginx.org/ru/docs/