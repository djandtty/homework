Определите разницу между контейнером и образом  
Разница между контейнером и образом в том, что образ - это статичный файл, содержащий все необходимое для запуска приложения, а контейнер - это инстанс образа, созданный для работы приложения в изолированном окружении  

Ответьте на вопрос: Можно ли в контейнере собрать ядро?  
Не вижу смысла, для какой задачи это может потребоваться, т.е. логичней для сборки ядра использовать виртуальную машину или физический сервер, где есть полный контроль над системой  
Но фактически можно использовать привилегированный контейнер, с флагом --privileged при запуске контейнера - для получения доступа к хостовой системе и для сборки ядра.  
Ко всему прочему эти действия могут быть небезопасными, так как предоставляет полный доступ к хостовой системе и может нарушиться изоляция контейнера  
Т.е. ответ **да, в контейнере можно собрать ядро**  
  
Команды:  
```
mkdir Les_Docker
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
