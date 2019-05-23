# project-manager


# Небольшой справочник по Docker

`docker build` - создание образа.<br>
`docker run` - запуск контейнера на основе какого-либо образа.

1. Запуск контейнера на основе образа из docker hub:
    - 
    ```
    docker run --rm -v ${PWD}/manager:/app --workdir=/app php:7.2-cli php bin/app.php 
    ```
    - `--rm` - удалить контейнер после запуска.
    - `-v ${PWD}/manager:/app` -v это монтирование хостовой папки manager на папку app внутри контейнера. 
    Если папка app не существует, будет создана.
    - `--workdir=/app` - указываем рабочую директория внутри контейнера.
    - `php:7.2-cli` - образ из docker hub на основе которого нужно создать контейнер.
    - `php bin/app.php` - команда которую необходимо выполнить внутри контейнера.

2. Создание собственного образа на основе Dockerfile:<br>
    -
    Содержимое Dockerfile, manager/docker/production/php-cli.docker
    ```
    FROM php:7.2-cli
    
    WORKDIR /app
    
    COPY ./ ./ 
    ```
    
    Команда для создания образа:

    ```
    docker build --file=manager/docker/production/php-cli.docker --tag manager-php-cli manager 
    ```
    - `--file=manager/docker/production/php-cli.docker` - Dockerfile на основе которого нужно создать образ.
    - `--tag manager-php-cli` - название образа.
    - `manager` - директория которая будет доступна при создании образа.<br>
     
    Команда для запуска контейнера на основе образа `manager-php-cli`
    ```
    docker run --rm manager-php-cli php bin/app.php
    ```
    - `--rm` - удалить контейнер после запуска.
    - `manager-php-cli` - название образа на основе которого нужно создать контейнер.
    - `php bin/app.php` - команда которую необходимо выполнить внутри контейнера.
    
3. Работа с контейнером:
    -
    - `docker network create app` - создание сети для взаимодействия между контейнерами. Название сети `app`
    
    Запуска контейнера на основе образа `manager-nginx`
    ```
    docker run -d --name manager-nginx-container -v ${PWD}/manager:/app -p 8080:80 --network=app manager-nginx
    ```
    - `-d` - запустить в фоне.
    - `--name manager-nginx-container` - задаем название контейнера `manager-nginx-container`.
    - `-v ${PWD}/manager:/app` -v это монтирование хостовой папки manager на папку app внутри контейнера.
    - `-p 8080:80` - прокидываем порты, 8080 хостовой на 80 внутри контейнера.
    - `--network=app` - контейнер доступен внутри сети `app`
    - `manager-nginx` - образ на основе которого нужно запустить контейнер.
    
4. Работа с docker-compose:
    -
    ```
    version: '3'
    services:
        manager-nginx:
            build:
                context: ./manager/docker/development
                dockerfile: nginx.docker
            volumes:
                - ./manager:/app
            depends_on:
                - manager-php-fpm
            ports:
                - "8080:80"
        manager-php-fpm:
            build:
                context: ./manager/docker/development
                dockerfile: php-fpm.docker
            volumes:
                - ./manager:/app
        manager-php-cli:
            build:
                context: ./manager/docker/development
                dockerfile: php-cli.docker
            volumes:
                - ./manager:/app
    ```
    - `docker-compose build` - собрать образы, сервисы.
    - `docker-compose up -d` - запустить контейнеры, `-d` в фоне.
    - `docker-compose down --remove-orphans` - останавливает, удаляет эти контейнеры и сеть между ними,
    `--remove-orphans` - означает удаление тех контейнеров, которые были удалены из блока `services` в запущеном режиме, 
    т.е чтобы остановить и удалить все контейнеры.    