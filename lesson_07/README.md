Домашнее задание  

1) Создать свой RPM пакет (можно взять свое приложение, либо собрать, например,
Apache с определенными опциями).  
2) Создать свой репозиторий и разместить там ранее собранный RPM.  

Установил требуемые пакеты:  
`yum install -y wget rpmdevtools rpm-build createrepo yum-utils cmake gcc git nano`  

Загрузил исходник nginx:  

`mkdir rpm`
'cd rpm'
`yumdownloader --source nginx`

Скачал исходный код модуля ngx_brotli:
cd /root
git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli
ls
cd ngx_brotli/
ls
cd deps/
ls
cd brotli/
ls
mkdir out && cd out

Собираем модуль ngx_brotli: 
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DCMAKE_C_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_CXX_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_INSTALL_PREFIX=./installed ..

-- Build files have been written to: /root/ngx_brotli/deps/brotli/out

cmake --build . --config Release -j 2 --target brotlienc

Редактирую файл spec
cd ~/rpmbuild/SPECS/
ls
vi nginx.spec
в секции %build
--add-module=/root/ngx_brotli \

Сборка:  
rpmbuild -ba nginx.spec -D 'debug_package %{nil}'

Выдет ошибку по требуемым зависимостям
        ```
        error: Failed build dependencies:
        gd-devel is needed by nginx-2:1.20.1-20.el9.alma.1.x86_64
        libxslt-devel is needed by nginx-2:1.20.1-20.el9.alma.1.x86_64
        pcre-devel is needed by nginx-2:1.20.1-20.el9.alma.1.x86_64
        perl-generators is needed by nginx-2:1.20.1-20.el9.alma.1.x86_64
        ```
  
Установил все зависимости для сборки пакета nginx:  
ls
rpm -Uvh nginx-1.20.1-20.el9.alma.1.src.rpm
yum-builddep nginx

Сюорка, теперь успешно:  
rpmbuild -ba nginx.spec -D 'debug_package %{nil}'

Можно проверить созданные пакеты:  
cd /root/
ll rpmbuild/RPMS/x86_64/
nginx-1.20.1-20.el9.alma.1.x86_64.rpm

Копирую пакеты в общий каталог:  
cp ~/rpmbuild/RPMS/noarch/* ~/rpmbuild/RPMS/x86_64/
ls -alh ~/rpmbuild/RPMS/x86_64/

Устанавливаю nginx из собранного пакета, запускаю его и проверяю его статус:
cd ~/rpmbuild/RPMS/x86_64
yum localinstall *.rpm
systemctl start nginx
systemctl status nginx

Создаю каталог для репозитория:
mkdir /usr/share/nginx/html/repo

Копирую в директорию репозитория собранные пакеты:
cp ~/rpmbuild/RPMS/x86_64/*.rpm /usr/share/nginx/html/repo/

Делаю инициализацию репозитория
createrepo /usr/share/nginx/html/repo/

NGINX выдает 403 Forbidden - правлю конфигурацию и перезагружаю сервис:
vi /etc/nginx/nginx.conf

  index index.html index.htm;
	autoindex on;
nginx -t
nginx -s reload

Добавляю репозиторий в /etc/yum.repos.d:
vi /etc/yum.repos.d/otus.repo
[otus]
name=otus-linux
baseurl=http://localhost/repo
gpgcheck=0
enabled=1

Добавляю пакет в наш репозиторий:
cd /usr/share/nginx/html/repo/
wget -4 https://repo.percona.com/yum/percona-release-latest.noarch.rpm

Обвновляю список пакетов в репозитории:
createrepo /usr/share/nginx/html/repo/
yum makecache

И могу устанавливать:
yum install -y percona-release.noarch
