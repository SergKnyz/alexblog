---
layout: post
title: "Использование Docker для кросскомпиляции проектов"
permalink: docker-cpp
tags: C++ docker
---

<img src='{{ site.baseurl }}assets/2017-05-15-docker-c++/cplus-docker.jpg' style="width: 720px;">
Как не проходить квест по настройке окружения каждый раз.

---

## Содержание
- [Проблема](#problem)
- [Постановка задачи](#formulation)
- [Решение](#solution)
    - [Docker](#docker)
    - [Docker Registry](#registry)
- [Заключение](#resume)

## Проблема    {#problem}
Сколько времени должен потратить новый сотрудник на то, чтобы собрать не знакомый ему проект? Обычно на это отвечают подобным образом: суммарное время необходимое на следующие действия:

 - выкачать/склонировать репозиторий проекта;
 - установить/настроить средства разработки(тулчейн, IDE, библиотеки);
 - разложить все по правильным путям, сконфигурировать make/cmake/qmake-файл/project_build.sh для сборки;
 - запустить саму сборку. 

По дороге может выясниться что где-то чего-то не хватает в зависимостях, неправильно прописан путь к либе/утилите/тулчейну, не та версия библиотеки стоит в системе. И пока решишь все эти мелкие проблемы - пройдет значительный кусок времени. Плюс еще по любому будут привлекаться сотрудники, которые уже давно работают с проектом.

Обычно для того что бы каждый раз не набивать шишки - пишется некий README, где прописываются *точные* инструкции, по выполнению которых можно добиться воспроизводимого результата. За пару человек эти инструкции отлаживаются и наступает счастье.

В идеале хочется чтобы сборка проекта выглядела подобным образом:  

{% highlight bash %}
    install all required software, SDKs, libs
    git clone git@project.git
    cd project_dir
    make/cmake/build_project.sh
{% endhighlight %}

Дальнейший шаг - инструкции из README автоматизируются скриптом.

Ок, подобный подход можно реализовать, если проект собирается под одну платформу/ОС.
Но представим следующую ситуацию:

 - есть n-ное количество целевых платформ, под который проект собирается, соответственно разные тулчейны для кросскомпиляции
 - под каждый конкретный процессор идет собственное SDK от чипмейкера. Есть различные версии одного и того же SDK.
 - в SDK находятся баги, их патчат, соответственно SDK необходимо пересобрать, бинари/хидера расшарить между разработчиками
 - то же самое касается еще ряда библиотек, которые работают поверх SDK, например, Qt
 - необходимо свести к минимуму время на развертывание окружения для разработки под конкретную платформу
 - большинству разработчиков вообще не надо или не интересно заморачиваться окружением, им бы сразу попасть в готовое, где можно писать уже непосредственно код
 - весь зоопарк платформ надо как-то тестировать, желательно автоматически, хотя бы на предмет того что код проекта успешно компилируется под все платформы.

Какие варианты решений я встречал/проходил:

 - каждый сам себе все настраивает: долго, не воспроизводимо(проблема ["у меня на компьютере все работает"](http://lurkmore.to/%D0%A3%D0%9C%D0%92%D0%A0), так как у каждого разработчика разное окружение), подвержено ошибкам конфигураций. При выпуске новой версии SDK - прохождение квеста каждым разработчиком заново. Автоматизация сборки всего shell скриптом может частично решить проблему;
 - единое окружение у всех разработчиков, варианты обеспечения:
    - единожды настроенная виртуальная машина билмастером, все ее себе копируют и работают локально. Новое SDK/библиотека/что-то пропатчено/какое-то важное изменение - новая виртуалка. Тяжело, много места, неудобно;
    - настроенные сервера, у каждого есть свой аккаунт, все туда логинятся по ssh, у каждого своя копия исходников, которая примонтирована на локальную машину разработчика по samba/nfs/sshfs/whatever, и все собираются в уже готовом и настроенном окружении. Сеть/сервер упали - разработчики ковыряются в носу. Качество работы определяется качеством работы сети, в целом невысокая производительность работы IDE(парсинг проекта) так как сеть в любом случае проигрывает в скорости диску. Очень осторожно надо менять настройки окружения, ибо если накосячить случайно - зацепит всех, не классно;
    - [linux контейнеры](https://ru.wikipedia.org/wiki/Виртуализация_на_уровне_операционной_системы) - об этом и поговорим далее.

## Постановка задачи    {#formulation}

Вкратце задача звучит так: хочется иметь контейнер, который содержит внутри себя все необходимое для разработки под конкретную платформу. Чтобы его мог скопировать себе на машину любой разработчик и тут же начать работать. Ведь несомненно проще скопировать и запустить, чем проходить квест по настройке окружения в соответствии с README проекта. N платформ - N контейнеров. Также было бы классно, чтобы разработчики не бегали с флешками/ходили по сети к билдмастеру, а как-то имели возможность централизовано получать интересующие их контейнеры. Из всех вариантов(LXC, OpenVZ, и т.д.), которые были доступны на данный момент, лишь Docker подошел полностью.

## Решение  {#solution}

### Docker  {#docker}

Docker simplifies the packaging, distribution, installation and execution of (complex) applications.

In this context, applications are:

blogging platforms like Wordpress or Ghost
tools for software collaboration like Gitlab or Gogs
file synchronization platforms like OwnCloud or Seafile
These kinds of applications usually consist of many components which need to be installed and configured. This is often a time consuming and frustrating experience for users.

Есть четыре класса задач, для которых Docker подходит, если не идеально, то лучше любого другого инструмента. Это:

 - упрощение процесса разворачивания/сопровождения проектов. Docker позволяет разбить проект на небольшие независимые, удобные в сопровождении компоненты, работать с которыми гораздо комфортнее, чем с реальными сущностями вроде Apache 2.4.12, установленного на хосте 1.2.3.4, работающем под управлением CentOS 6.
 - Continous development и zero-downtime deployment. Каждый образ Docker — вещь в себе, включающая сервис (или набор сервисов), окружение для его запуска и необходимые настройки. Поэтому контейнеры можно передавать между членами команды в ходе цикла «разработка -> тестирование -> внедрение» и быстро внeдрять изменения, просто переключая настройки на новые контейнеры.
 - IaaS/PaaS. Благодаря легковесности контейнеров Docker можно использовать в качестве движка виртуализации в IaaS, а благодаря простоте миграции Docker становится идеальным решением для запуска сервисов в PaaS.
 - Запуск небезопасного кода. Docker позволяет запустить любой, в том числе графический софт внутри изолированного контейнера с помощью одной простой команды. Поэтому он идеально подходит для запуска разного рода недовереннoго или просто небезопасного кода.


У Docker следующие преимущества перед вышеперечисленными вариантами:

 - виртуализация на уровне операционной системы, а это значит минимальные потери производительности, легковесность в сравнении с полноценной виртуалкой;
 - позволяет «упаковать» приложение со всем его окружением и зависимостями в контейнер, который может быть перенесён на любую Linux-систему с поддержкой [cgroups](https://ru.wikipedia.org/wiki/Cgroups) в [ядре](https://ru.wikipedia.org/wiki/Ядро_Linux)
 - версионирование
 - изоляция. В системе одна версия библиотеки, в контейнере другая. В принципе можно на Ubuntu запустить CentOS/Gentoo/etc.
 - вообще круто использовать docker для экспериментов - позволяет не засорять систему в процессе разработки/тестирования
 - дистрибуция контейнеров - сервер [Docker Registry](#registry), у других контейнеров не нашел такой удобной фишки.


Контейнеры идеальны для автоматического тестирования. Каждый тест можно в своей песочнице запускать


### Docker Registry {#registry}

Централизованная доставка образов. Можно скидывать образы на какой-то ftp/web-сервер, облако. Тогда каждый разработчик сам сознательно должен скачать на свою машину, что-то вроде следующих действий:

    wget https://cloud.company.com/path/to/dockers/specific_img_version.tar
    docker load specific_img_version.tar
    rm -f specific_img_version.tar

Где specific_img_version.tar - название экспортированного docker образа.

А можно поднять свой Docker Registry, откуда сам docker клиент будет тянуть заданный образ. А еще круче автоматизировать использование последнего образа bash скриптом

[Docker Hub](hub.docker.com) - аналог Github в мире docker контейнеров. Там можно размещать неограниченное количество публичных образов и только один приватный. Большее количество приватных образов можно добавлять за отдельную плату. Если нет ничего секретного - заводим аккаунт, выкладываем образы, которые публичны и доступны всем. Можно купить подписку и размещать приватные образа, т.е. публично не доступные. Эти два варианты клевые тем, что сразу снимается вопрос поддержки, за нас это делают админы Docker Hub.
Если хочеться приватности(например, внутри контейнера размещены компоненты третьих сторон, NDA которых запрещает хранение за пределами инфрастуктуры компании-пользователя) - подымаем [Docker Registry](https://docs.docker.com/registry/). Docker Registry это не тоже самое что Docker Hub, т.е. мы не получим хранилище контейнеров с красивым веб-интерфейсом.

Docker Registry - это хранилище с версионированными docker образами. Регистр удобно использовать как единную точку хранения и раздачи контейнеров внутри компании.
По умолчанию Docker всегда "смотрит" на Docker Hub. Для того что бы он мог работать с другим хранилищем, надо его адрес добавить в /etc/docker/daemon.json:
{% highlight bash %}
    docker push registry_ip:registry_port/image_name:image_version
{% endhighlight %}

[Docker Registry](https://docs.docker.com/registry/)

прописываем адрес Docker Registry в /etc/docker/daemon.json


Схема работы такова:

 - на рабочем компьютере билдмастера собирается образ, который содержит все необходимое для разработки под конкретную платформу, и отправляется в регистр:
{% highlight bash %}
    docker push registry_ip:registry_port/image_name:image_version
{% endhighlight %}
 - все желающие забирают/пулят образ из регистра и работают в готовом окружении:
{% highlight bash %}
    docker pull registry_ip:registry_port/image_name:image_version
{% endhighlight %}


запуск:
{% highlight bash %}
    docker run -it --rm -v /path/to/project/sources:/path/of/sources/inside/container registry_ip:registry_port/image_name:image_version bash
{% endhighlight %}
В результате получаем обычную командную строку(в данном случае bash), в которой можно перейти в папку с проектом и вызвать make/cmake/build.sh
Можно запустить сразу билд, что бы ручками не ходить куда-то:

{% highlight bash %}
    docker run -it --rm -v /path/to/project/sources:/path/of/sources/inside/container registry_ip:registry_port/image_name:image_version bash -c "cd /path/of/sources/inside/container; make/cmake/build.sh;"
{% endhighlight %}
Команду выше можно использовать для интеграции с IDE: в каждой приличной IDE есть что-то вроде custom build steps, куда приведенную команду можно прописать.






Копипасты выводов:
{% highlight bash %}
alex@alex-pc:~$ docker images
REPOSITORY                                          TAG                    IMAGE ID            CREATED             SIZE
broadcom/platform351                                16.4_v0                e90ae3da554e        2 days ago          5.87 GB
hub.docker.company.com:5000/chipmaker/platform351   16.4_v3                e90ae3da554e        2 days ago          5.87 GB
hub.docker.company.com:5000/chipmaker/platform351   latest                 e90ae3da554e        2 days ago          5.87 GB
hub.docker.company.com:5000/chipmaker/platform324   latest                 b363901a6dda        2 days ago          5.18 GB
v43_hi310_4.4                                       latest                 a2c5423f0ccc        2 weeks ago         8.38 GB
v43_hi320_4.4                                       latest                 c3b9dbe0b582        2 weeks ago         7.9 GB
chipmaker/platform420                               base                   1f58ee5af66c        3 weeks ago         413 MB
hitool                                              1                      7846efda5be6        3 weeks ago         866 MB
chipmaker/platform420                               V100R005C00SPC040_v1   23e9f56aa5be        3 weeks ago         9.98 GB
ubuntu                                              14.04.5                302fa07d8117        2 months ago        188 MB
nate/dockviz                                        latest                 4ea294712533        6 months ago        7.33 MB
starefossen/github-pages                            41                     78ef097875de        17 months ago       859 MB
starefossen/github-pages                            40                     84156a8a47a9        18 months ago       849 MB
cmaohuang/firefox-java                              latest                 e31b29d62bab        21 months ago       635 MB
alex@alex-pc:~$ docker ps -a
CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS              PORTS                              NAMES
9b9b5a7d3821        starefossen/github-pages:41   "jekyll serve --dr..."   2 hours ago         Up 2 hours          0.0.0.0:4000->4000/tcp, 4100/tcp   zealous_snyder
8e2801d70d9f        chipmaker/platform420:base    "bash"                   3 days ago          Up 2 days                                              Hi3798MV200_41
0e5890960206        chipmaker/platform420:base    "bash"                   2 weeks ago         Up 2 weeks                                             v43_310_1024M
ed2988615670        chipmaker/platform420:base    "--name v43_310_10..."   2 weeks ago         Created                                                romantic_murdock
8e6f24831527        v43_hi310_4.4                 "bash"                   2 weeks ago         Up 2 weeks                                             qt_softfp
{% endhighlight %}


скрипт запуска:
{% highlight bash %}
#!/bin/bash

DOCKER_IMG_NAME="docker_image_name:latest"

set -e

docker pull ${DOCKER_IMG_NAME}

self_script_name="$(basename "$(test -L "$0" && readlink "$0" || echo "$0")")"

src_dir=${1?Usage: ${self_script_name} path_to_sources}
resolved_dir=$(readlink -f $src_dir)

echo "Will mount host path \"${resolved_dir}\" to container path \"/src\""

hint_msg=$(cat <<EOF
#################################################
#Some infomation tips                           #
#################################################
EOF
)
bash_cmd=$(cat <<EOF
echo '${hint_msg}' && bash
EOF
)

docker run -it --rm -v ${resolved_dir}:/src -e LOCAL_USER_ID=`id -u ${USER}` ${DOCKER_IMG_NAME} bash -c "${bash_cmd}"
{% endhighlight %}



Security

As the registry is the hinge pin of our build and deployment process, securing it is vital. By default, everybody who can access the registry server has full permissions for reading and writing images. The registry offers two options for securing its content: HTTP basic auth and a custom token-based authentication protocol. Basic auth is simple to set up and use, but does not allow for any kind of permission management: all authorized users have full access to the registry. The second option is more complicated, but offers more way more flexibility.


Docker Registry не позволяет сделать разграничение по правам доступа. Типичный сценарий: анонимный pull и авторизированный push. Соответствующее [обсуждение](https://github.com/docker/distribution/issues/1028) на гитхабе.

## Заключение   {#resume}
