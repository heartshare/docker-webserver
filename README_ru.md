# Что это то за образ?

Образ представляет собой набор для быстрой и удобной разработки сайтов на предустановленых
`nginx (1.6.2) + php-fpm (5.6.5) + mysql (5.6.22)`. Контейнер можно использовать как замену vagrant 
(или как замену провайдера virtualbox на docker).

## Особенности:

1. Контейнер можно использовать сразу для всех ваших сайтов: nginx настроен на автоматическую
обработку всех доменов `*.dev`. очень удобно: создаешь в `DOCUMENT_ROOT` каталог `test-app` и сайт
уже доступен по ссылке `http://test-app.dev/` (ВАЖНО: нужна предварительная подготовка, смотри дополнительный софт).

2. Уже установлены `composer` (установлен assets plugin), `git`, `phpmyadmin` (автоматический вход `root` без пароля).

3. Ваш код и данные mysql расположены у вас же на локальной машине. вы можете пересоздавать, удалять контейнеры,
но всегда данные будут в порядке.

4. Php process запущен от пользователя с UID 1000 (по умолчанию, вы на локальной машине работаете от такого же пользователя),
поэтому ему доступны все файлы, а созданные сервером файлы доступны вам - можно изменять и удалять

# Как пользоваться контейнером

## дополнительный софт

Для конфортной работы, я советую использовать [dnsmasq](https://ru.wikipedia.org/wiki/Dnsmasq) для автоматического 
резолвинга доменов вида `http://*.dev/`. Вы можете установить его в Ubuntu так (запускать от рута `root`):

    apt-get install dnsmasq
    echo "address=/.dev/127.0.0.1" >> /etc/dnsmasq.conf

По умолчанию контейнер использует `80` порт на локальном адресе `127.0.0.1:80`. Если вы используете другие настройки,
измените команду выше в соответствии с ними.

Другой вариант - это ручное обновление `/etc/hosts`, каждый раз нужно будет добавить туда что то на подобии этого:

    127.0.0.1   my-app.dev

## как запустить контейнер

Предварительно нужно установить [Docker](https://www.docker.com/) у себя на машине. Все команды нужно запускать от 
`root` или добавить своего пользователя в группу docker. 

    docker run --name dev-server -p 127.0.0.1:80:80 -v /opt/mysql:/var/lib/mysql -v /var/www:/web -d metalguardian/php-web-server

`--name dev-server` устанавливает имя контейнеру, с помощью него в будущем можно будет управлять контенером, 
можно стартовать, останавливать, перезапускать контейнер по его имени: `docker start dev-server`

`-p 127.0.0.1:80:80` прокидывает порт `80` контейнера на порт `80` локальной машины по адресу 127.0.0.1

`-v /opt/mysql:/var/lib/mysql` использовать директорию машины (`/opt/mysql`) как директорию (`/var/lib/mysql`) на контейнере

`-v /var/www:/web` использовать директорию машины (`/var/www`) как дерикторию (`/web`) на контейнере

`-d` запускает контейнер в бекграунде

## как получить доступ в контейнер

Достаточно выполнить в консоли:

    docker exec -ti dev-server su docker

Эта команда подключится к контейнеру под специальным пользователем `docker`, который имеет доступ к коду и консольным 
командам.

Если же нужны права `root`, выполняем так:

    docker exec -ti dev-server bash

Это подключится от рута `root`

## установка новых пакетов

Прежде чем устанавливать пакеты, нужно выполнить от `root`:

    apt-get update

Потому что во время сборки образа был очищен кеш пакетов

## как еще использовать

Вы можете создать простой `Dockerfile` что бы получить новый образ включающий дополнительные конфиги:

    FROM metalguardian/php-web-server
    COPY specific-nginx.conf /etc/nginx/conf.d/other-vhost.conf

Разместите его в каталоге и выполните: `docker build -t some-other-server .`, теперь можно запускать новый
контейнер:

    docker run --name other-server -d some-other-server

# Поддержка версий Docker

Официально поддерживается версия 1.4.1.

Поддержка более старых версий (вплоть до 1.0) осуществляется по мере возможностей.

# Обратная связь

## Баг-трекер

Если у вас возникли проблемы с образом, пишите мне тут: [GitHub issue](https://github.com/metalguardian/docker-webserver/issues).
