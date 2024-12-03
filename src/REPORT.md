# Simple Docker

## Contents

1. [Part 1. Готовый докер](#chapter_1)
2. [Part 2. Операции с контейнером](#chapter_2)
3. [Part 3. Мини веб-сервер](#chapter_3)
4. [Part 4. Свой докер](#chapter_4)
5. [Part 5. Dockle](#chapter_5)
6. [Part 6. Базовый Docker Compose](#chapter_6)


## Part 1. Готовый докер<a name="chapter_1"></a>

* Возьмём официальный докер-образ с nginx и выкачаем его при помощи docker pull.

* Для этого выполним команду `docker login` и перейдем во вкладку браузера

  * Создадим аккаунт *docker*

    ![Регистрация в докере](../misc/screenshots/docker_registration.png)

* Прописываем команду `docker pull nginx`

  ![Вывод команды docker pull](../misc/screenshots/docker_pull_nginx.png)

* Запустим докер-образ через команду `docker run -d [image_id|repository]`

  ![Запуск докер-образа](../misc/screenshots/docker_images.png)

* Проверим, что образ запустился через `docker ps`

    ![Вывод команды docker ps](../misc/screenshots/docker_ps.png)

* Посмотрим информацию о контейнере через `docker inspect [container_id|container_name]`

  ![docker inspect вывод (начало)](../misc/screenshots/docker_inspect_start.png)

  ![docker inspect вывод (конец)](../misc/screenshots/docker_inspect_end.png)

* Определим размер контейнера, список замапленных портов и ip контейнера с помощью следующих команд

```
docker inspect -sf '{{ .SizeRootFs }}' fde5806f0343
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' fde5806f0343
docker inspect --format='{{range $p, $conf := .NetworkSettings.Ports}}{{$p}}{{if $conf}}: {{(index $conf 0).HostPort}}{{else}}: not mapped{{end}}{{"\n"}}{{end}}' fde5806f0343
```

  ![Информация о контейнере](../misc/screenshots/docker_container_info.png)

  1) Размер контейнера: **187708004 bytes**

  2) IP контейнера: **172.17.0.2**

  3) Список замапленных портов: **80/tcp: not mapped**

* Остановим докер контейнер через `docker stop [container_id|container_name]`

  ![Остановка докер контейнера](../misc/screenshots/docker_stop.png)

* Проверим, что контейнер остановился через `docker ps`

  ![Проверка остановки контейнера](../misc/screenshots/docker_ps_check.png)

* Запустим докер с портами 80 и 443 в контейнере, замапленными на такие же порты на локальной машине, через команду `run`

  ![Запуск контейнера с портами](../misc/screenshots/docker_run_with_ports.png)

* Проверим, что в браузере по адресу localhost:80 доступна стартовая страница nginx

  ![Страница nginx по адресу localhost:80 в браузере](../misc/screenshots/browser_nginx_check.png)

* Перезапустим докер контейнер через `docker restart [container_id|container_name]`

  ![Перезапуск контейнера](../misc/screenshots/docker_restart.png)

## Part 2. Операции с контейнером<a name="chapter_2"></a>

* Прочитаем конфигурационный файл *nginx.conf* внутри докер контейнера через команду *exec*

  ![Конфигурационный файл nginx.conf внутри контейнера](../misc/screenshots/docker_nginx_conf_file.png)

* Создадим на локальной машине файл *nginx.conf*. Настроим в нем по пути /status отдачу страницы статуса сервера nginx.

  ![Настройка файла nginx.conf на локальной машине](../misc/screenshots/part2_nginx_conf_file.png)

* Скопируем созданный файл nginx.conf внутрь докер-образа через команду `docker cp`

  ![Копирование файла из локальной машины в контейнер](../misc/screenshots/part2_nginx_cp_conf_file.png)

* Перезапустим nginx внутри докер-образа через команду `exec`

  ![Перезагрузка nginx в контейнере](../misc/screenshots/part2_nginx_reload.png)

* Проверим, что по адресу `localhost:80/status` отдается страничка со статусом сервера `nginx`

  ![Проверка в браузере](../misc/screenshots/part2_nginx_new_confix_check.png)

* Экспортируем контейнер в файл *container.tar* через команду *export*.

  ![Экспорт контейнера](../misc/screenshots/part2_container_export.png)

* Останавливаем контейнер

  ![Останавливаем контейнер](../misc/screenshots/part2_docker_stop.png)

* Удаляем образ через `docker rmi [image_id|repository]`, не удаляя перед этим контейнер

  ![Удаление образа контейнера](../misc/screenshots/part2_rmi_force.png)

* Удаляем остановленный контейнер

  ![Удаление контейнера](../misc/screenshots/part2_docker_rm_containers.png)

* Импортируем контейнер обратно через команду `import`

  ![Импорт контейнера](../misc/screenshots/part2_import_image.png)

  > При импорте контейнера через *docker import*, информация о CMD может быть потеряна, если контейнер был экспортирован с помощью *docker export*. Это происходит потому, что *docker export* сохраняет только файловую систему контейнера, а не его метаданные или команду запуска. Поэтому мы явно указываем команду по умолчанию при импортировании контейнера

* Запустим импортированный контейнер

  ![Запуск импортированного контейнера](../misc/screenshots/part2_myContainer_run.png)

* Проверим, что по адресу *localhost:80/status* отдается страничка со статусом сервера *nginx*

  ![Проверка статуса сервера из импортированного контейнера](../misc/screenshots/part2_chec_import_status.png)

## Part 3. Мини веб-сервер<a name="chapter_3"></a>

* Напишем мини-сервер на **C** и **FastCgi**, который будет возвращать простейшую страничку с надписью `Hello World!`

  ![Код мини сервера](../misc/screenshots/part3_mini_server.png)

* Запустим написанный мини-сервер через *spawn-fcgi* на порту *8080*

  * Для этого создадим новый контейнер из образа *nginx* командой:
  
    ```
    docker run -d --name part_3 -p 81:81 nginx
    ```

    ![Создание контейнера part_3](../misc/screenshots/part3_create_container.png)
  
  * Перейдем в терминал созданного контейнера командой `docker exec -it [CONTAINER ID | NAME] bash`

  * Установим на нем `gcc`, `libfcgi-dev` и `spawn-fcgi`

  * С помощью команды `docker cp` переместим в контейнер исходный код мини сервера

    ![Копируем необходимые файлы в контейнер](../misc/screenshots/part3_docker_cp.png)

  * Скомпилируем исходный код и запустим мини сервер

  ![Компиляция и запуск сервера](../misc/screenshots/part3_spawn_server.png)

* Напишем свой nginx.conf, который будет проксировать все запросы с *81* порта на *127.0.0.1:8080* и скопируем его в контейнер по пути *./nginx/nginx.conf*

  ![Созданный nginx.conf](../misc/screenshots/part3_nginx_config.png)

* Перезагружаем сервер

  ![Перезагрузка сервера nginx](../misc/screenshots/part3_reload_nginx.png)

* Проверим, что в браузере по *localhost:81* отдается написанная нами страничка

  ![Страница в браузере](../misc/screenshots/part3_browser_check.png)

## Part 4. Свой докер<a name="chapter_4"></a>

* Напишем свой докер-образ, который:

  * собирает исходники мини сервера на FastCgi из Части 3;

  * запускает его на 8080 порту;

  * копирует внутрь образа написанный *./nginx/nginx.conf*;

  * запускает nginx.

  ![Написанный Dockerfile](../misc/screenshots/Part4_dockerfile.png)

* Собираем написанный докер-образ через `docker build` при этом указав имя и тег

  ![Вывод команды docker build](../misc/screenshots/part4_dockerfile_build.png)

* Проверим через `docker images`, что все собралось корректно

  ![Проверка всех образов докер](../misc/screenshots/part4_docker_images.png)

* Запустим собранный докер-образ с маппингом 81 порта на 80 на локальной машине и маппингом папки *./nginx* внутрь контейнера по адресу, где лежат конфигурационные файлы nginx'а

  > Для этого сначала вытащим из другого контейнера с *nginx* папку */etc/nginx* командой `sudo docker export 96f27863b481 -o tmp.tar`

  ```
  sudo docker run -d --name part_4 -p 80:81 -v $(pwd)/nginx:/etc/nginx my_server:s21
  ```

  ![Запуск нашего контейнера с маппингом порта и папки nginx](../misc/screenshots/part4_docker_container.png)

* Проверим, что по localhost:80 доступна страничка написанного мини сервера

  ![Проверка контейнера через браузер](../misc/screenshots/part4_check_container.png)

* Допишем в *./nginx/nginx.conf* проксирование странички */status*, по которой надо отдавать статус сервера *nginx*

  ![Редактирование файла nginx.conf на хост-машине](../misc/screenshots/part4_edit_nginx_conf.png)

* Перезапустим докер-контейнер

  ![Перезапуск контейнера](../misc/screenshots/part4_restart_container.png)

* Проверим, что теперь по *localhost:80/status* отдается страничка со статусом *nginx*

  ![Проверка страницы "status"](../misc/screenshots/part4_check_status_page.png)

## Part 5. Dockle<a name="chapter_5"></a>

* Просканируем образ из предыдущего задания через `dockle [image_id|repository]`

  ![Вывод команды dockle](../misc/screenshots/part5_dockle_output.png)

* Исправим образ так, чтобы при проверке через dockle не было ошибок и предупреждений

  ![Исправленный dockerfile](../misc/screenshots/part5_dockerfile_edit.png)

  * Собираем новый образ через редактированный докерфайл

  ![Билд нового образа docker](../misc/screenshots/part5_new_image.png)

  * Проверим через dockle

  ![Проверка dockle](../misc/screenshots/part5_dockle_on_new_image.png)

  > *INFO CIS-DI-0005* указывает на то, что у нашего образа отсутствует цифровая подпись. Чтобы добавить подпись нужно сгенерировать ее на локальной хост машине и на сервере, в который можно выгрузить этот образ (например dockerhub)

## Part 6. Базовый Docker Compose<a name="chapter_6"></a>

* Напишем файл docker-compose.yml, с помощью которого:

  * Поднимем докер-контейнер из Части 5 (он должен работать в локальной сети, т. е. не нужно использовать инструкцию EXPOSE и мапить порты на локальную машину).

  * Поднимем докер-контейнер с nginx, который будет проксировать все запросы с 8080 порта на 8081 порт первого контейнера.

  ![Написанный docker-compose файл](../misc/screenshots/part6_docker-compose.png)

* Сделаем для второго контейнера файл *nginx.conf*

  ![Файл nginx.conf для второго контейнера с прокси](../misc/screenshots/part6_nginx-conf.png)

* Остановим все запущенные контейнеры

  ![Вывод команды docker ps](../misc/screenshots/part6_stop_containers.png)

* Соберем и запустим проект с помощью команд `docker compose build` и `docker compose up`

  ![Билд и запуск контейнеров](../misc/screenshots/part6_build-and-up.png)

* Проверим, что контейнеры работают

  ![Проверка работы контейнеров](../misc/screenshots/part6_docker_ps_check.png)

* Проверим, что в браузере по localhost:80 отдается написанная нами страничка, как и ранее.

  ![Проверка в браузере по localhost](../misc/screenshots/part6_localhost.png)

  ![Проверка в браузере по localhost/status](../misc/screenshots/part6_localhost-status.png)