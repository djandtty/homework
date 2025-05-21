Определите разницу между контейнером и образом  
Разница между контейнером и образом в том, что образ - это статичный файл, содержащий все необходимое для запуска приложения, а контейнер - это инстанс образа, созданный для работы приложения в изолированном окружении  

Ответьте на вопрос: Можно ли в контейнере собрать ядро?  
Не вижу смысла, для какой задачи это может потребоваться, т.е. логичней для сборки ядра использовать виртуальную машину или физический сервер, где есть полный контроль над системой  
Но фактически можно использовать привилегированный контейнер, с флагом --privileged при запуске контейнера - для получения доступа к хостовой системе и для сборки ядра.  
Ко всему прочему эти действия могут быть небезопасными, так как предоставляет полный доступ к хостовой системе и может нарушиться изоляция контейнера  
Т.е. ответ **да, в контейнере можно собрать ядро**  
  
Команды:  
```
  394  mkdir Les_Docker
  395  cd Les_Docker/
  396  nano Dockerfile

cat Dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

  397  nano index.html

cat index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TEST TEST TEST !</title>
</head>
<body>
    <h1>h1 TEST !</h1>
    <p>p TESTTESTETESTSET</p>
</body>
</html>

  398  docker build -t custom-nginx .
  399  docker ps
  400  docker ps -a
  401  docker run -d -p 8080:80 custom-nginx
  402  docker ps
  403  docker login
  405  docker tag custom-nginx djandtty/custom-nginx
  406  docker push djandtty/custom-nginx
  407  docker ps
  408  docker stop focused_feynman
```

Задание со звездочкой:  
docker-compose.yml  
```
cat docker-compose.yml
version: '3.8'

services:
  redmine:
    build:
      context: ./redmine
    image: custom-redmine:latest
    ports:
      - "3000:3000"
    depends_on:
      - db
    volumes:
      - redmine_data:/usr/src/redmine/files
      - redmine_themes:/usr/src/redmine/public/themes
    environment:
      REDMINE_DB_MYSQL: db
      REDMINE_DB_DATABASE: redmine
      REDMINE_DB_USERNAME: redmine
      REDMINE_DB_PASSWORD: secret

  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: redmine
      MYSQL_USER: redmine
      MYSQL_PASSWORD: secret
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
  redmine_data:
  redmine_themes:
```
Dockerfile:
```
cat Dockerfile
FROM redmine:5.1

COPY themes/farend_bleuclair /usr/src/redmine/public/themes/farend_bleuclair

RUN chown -R redmine:redmine /usr/src/redmine/public/themes/farend_bleuclair
```
  
Команды:
```
  410  mkdir Les_Docker_2
  411  cd Les_Docker_2/
  413  nano docker-compose.yml

  420  ls
  421  mkdir -p redmine/themes
  425  cd redmine/
  430  nano Dockerfile

  434  cd themes/
  435  git clone https://github.com/farend/redmine_theme_farend_bleuclair.git farend_bleuclair
  436  ls
  437  cd farend_bleuclair/
  438  ls
  439  pwd
  445  tree
  448  docker compose build
  449  docker compose up -d
  450  docker ps
  452  docker logs les_docker_2-redmine-1
  453  docker compose restart redmine
  454  docker ps
  455  docker logs les_docker_2-redmine-1
  456* docker exec -it les_docker_2-redmine-1 b
  457  docker ps
  458  curl http://localhost:3000
  459  curl localhost 3000
  460  sudo netstat -tuln | grep 3000
  461  sudo ss -tuln | grep 3000
  462  curl http://localhost:3000
  463  docker compose down
  464  docker ps
  465  docker compose up -d
  466  docker ps
```
