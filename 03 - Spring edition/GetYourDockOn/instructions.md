![Docker Grunn](http://photos3.meetupstatic.com/photos/theme_head/c/4/a/f/full_6650351.jpeg)

# Docker Grunn meetup, woensdag 27 mei

## Vooraf installeren!!

Zorg dat je op zijn minst de beschikking hebt over

- een **werkende** installatie van *Docker* (Linux) of *boot2docker* (OS X / Windows).  
 
De installatie staat [hier](http://docs.docker.com/compose/install/#install-docker) beschreven.  
*boot2docker* is op zijn beurt weer afhankelijk van VirtualBox.  

Referentie: [Guide to Docker on OS X](http://blog.tutum.co/2015/05/19/guide-to-docker-on-os-x/)

Of Docker juist werkt, kan je vervolgens testen met `docker run --rm hello-world`.

- **Docker Compose ** ([link](http://docs.docker.com/compose/install/)):  

De installatie werkt zo:

```
$ curl -L https://github.com/docker/compose/releases/download/1.2.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```

## WordPress & MySQL dual container setup

### Docker images downloaden

Twee methodes zijn voorhanden.  
Om bandbreedte te besparen heeft de eerste methode de voorkeur!  

**1. Tarball van saved images downloaden en inladen**

Dit zijn images uit de officiele Docker Registry, *saved to a tar archive*, gzipped en op een host geplaatst, die relatief dichtbij is.

- `$ wget https://uploads.tfe.nl/dckrgrnn/mysql.tar.gz` (size: 142 MB)  
- `$ wget https://uploads.tfe.nl/dckrgrnn/wordpress.tar.gz` (size: 163 MB)

Importeer deze images:

- `$ docker load < mysql.tar.gz`  
- `$ docker load < wordpress.tar.gz`

**2. Pull de images direct uit de officiele Docker Registry**

- `$ docker pull mysql` (size: 282.9 MB)
- `$ docker pull wordpress` (size: 460.3 MB)

Controleer welke images er nu lokaal beschikbaar zijn:

```
$ docker images
REPOSITORY            TAG                  IMAGE ID            CREATED             VIRTUAL SIZE
wordpress             latest               5ff368875b77        4 days ago          460.3 MB
mysql                 latest               56f320bd6adc        3 weeks ago         282.9 MB
```
### "Met de hand"

Als eerste start je een MySQL-container op basis van het *mysql:latest*-image.  
We gebruiken "p4ssw0rd" als root password en geven dat met "-e" mee als omgevings-variabele aan de container:

`$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=p4ssw0rd -d mysql`

Vervolgens start je een WordPress-container.  
Deze geef je een **link** naar de MySQL-container en laat je poort 80 in de container forwarden naar poort 8080 op je Docker-host:

`$ docker run --name some-wordpress --link some-mysql:mysql -p 8080:80 -d wordpress`

*That was quick!*  

Ga in een webbrowser naar  
- `http://localhost:8080/` (Linux)  
- `http://192.168.59.103:8080/ ` (boot2docker)  
en configureer WordPress verder af.

### Onder de motorkap [1]

Wat draait er nu?

```
$ docker ps

CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                  NAMES
5c2537c15ba9        wordpress:latest    "/entrypoint.sh apac   About an hour ago   Up About an hour    0.0.0.0:8080->80/tcp   some-wordpress
11ddc1271814        mysql:latest        "/entrypoint.sh mysq   About an hour ago   Up About an hour    3306/tcp               some-mysql
```

Om bijvoorbeeld IN je MySQL-container te checken wat er in de database staat, kan je met `docker exec -ti` een shell opstarten en in die container commandos uitvoeren:

```
$ docker exec -ti some-mysql bash

root@11ddc1271814:/# mysql -u root -p
Enter password: p4ssw0rd
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 15
Server version: 5.6.24 MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| wordpress          |
+--------------------+
4 rows in set (0.00 sec)

mysql> use wordpress;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-----------------------+
| Tables_in_wordpress   |
+-----------------------+
| wp_commentmeta        |
| wp_comments           |
| wp_links              |
| wp_options            |
| wp_postmeta           |
| wp_posts              |
| wp_term_relationships |
| wp_term_taxonomy      |
| wp_terms              |
| wp_usermeta           |
| wp_users              |
+-----------------------+
11 rows in set (0.00 sec)

mysql> exit
Bye
root@11ddc1271814:/# exit
```

### Opruimen

Het commando `docker kill` is minder netjes dan `docker stop`, maar sneller als je toch besloten hebt de containers te verwijderen.

```
$ docker kill 5c2537c15ba9 11ddc1271814
5c2537c15ba9
11ddc1271814

$ docker rm 5c2537c15ba9 11ddc1271814
5c2537c15ba9
11ddc1271814

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

## "Met docker-compose"

Nu doen we hetzelfde, maar met behulp van *docker-compose*. 

### Bestand docker-compose.yml

De waardes die je hierboven op de command-line hebt gegeven, kan je in `docker-compose.yml` combineren. Een YAML-bestand, dat er dan als volgt uit komt te zien:

```
db:
  image: mysql:latest
  environment:
    - MYSQL_ROOT_PASSWORD=p4ssw0rd

wordpress:
  image: wordpress:latest
  links:
    - db:mysql
  ports:
    - "8080:80"
```
Start de stack in detached mode:

```
$ docker-compose up -d
Creating wordpressmysql_db_1...
Creating wordpressmysql_wordpress_1...
```

### Onder de motorkap [2]

Wat draait er nu?

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                CREATED          STATUS              PORTS                  NAMES
fe9eead88a32        wordpress:latest    "/entrypoint.sh apac   3 minutes ago    Up 3 minutes        0.0.0.0:8080->80/tcp   wordpressmysql_wordpress_1
6a2392af4d87        mysql:latest        "/entrypoint.sh mysq   3 minutes ago    Up 3 minutes        3306/tcp               wordpressmysql_db_1

$ docker-compose ps
           Name                         Command               State          Ports
------------------------------------------------------------------------------------------
wordpressmysql_db_1          /entrypoint.sh mysqld            Up      3306/tcp
wordpressmysql_wordpress_1   /entrypoint.sh apache2-for ...   Up      0.0.0.0:8080->80/tcp
```
WordPress weer opzetten middels de hierboven genoemde link naar "localhost".

En na ge- / misbruik weer alles netjes achterlaten:

```
$ docker-compose kill
Killing wordpressmysql_wordpress_1...
Killing wordpressmysql_db_1...

$ docker-compose rm --force
Going to remove wordpressmysql_wordpress_1, wordpressmysql_db_1
Removing wordpressmysql_db_1...
Removing wordpressmysql_wordpress_1...
```

## Data volume container

De data die door WordPress wordt gegenereerd (accounts, posts, etc.) kan worden opgeslagen buiten de MySQL-container in een zgn. *data volume container*. Die kan je op de CLI zo aanmaken:

`$ docker run -v /var/lib/mysql --name=dbdata -d mysql /bin/true`

De container is op basis van het MySQL-image, want die hebben we immers al binnen.  
Een data volume container hoeft niet "up" te zijn om gebruikt te kunnen worden. Je ziet 'm dan ook alleen met het `docker ps -a` command:

```
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS                     PORTS               NAMES
c2aa204baf71        mysql:latest        "/entrypoint.sh /bin   4 minutes ago       Exited (0) 4 minutes ago                       dbdata
```

Om dit met docker-compose te doen, voegen we deze container toe aan de YAML-configuratie.  
Het bestand ziet er dan zo uit:

```
dbdata:
  image: mysql
  volumes:
    - /var/lib/mysql
  command:
    - /bin/true

db:
  image: mysql
  volumes_from:
    - dbdata
  environment:
    - MYSQL_ROOT_PASSWORD=p4ssw0rd

wordpress:
  image: wordpress
  links:
    - db:mysql
  ports:
    - "8080:80"
```

Start weer een nieuwe stack met `docker-compose up -d` en controleer wat er draait:

```
$ docker-compose ps
           Name                         Command               State           Ports
-------------------------------------------------------------------------------------------
wordpressmysql_db_1          /entrypoint.sh mysqld            Up       3306/tcp
wordpressmysql_dbdata_1      /entrypoint.sh /bin/true         Exit 0
wordpressmysql_wordpress_1   /entrypoint.sh apache2-for ...   Up       0.0.0.0:8080->80/tcp
```

Het bewijs dat beide containers dezelfde "layer" gebruiken voor /var/lib/mysql kan geleverd worden middels `docker inspect`:

```
$ docker inspect dbdata | grep "var/lib/mysql" | grep vfs
        "/mnt/sda1/var/lib/docker/vfs/dir/9acd80fd069e82a89ac04a9141cc518ce8ee176bd7e4cc92c972dd3df50dffcc"

$ docker inspect wordpressmysql_db_1 | grep "var/lib/mysql" | grep vfs
        "/mnt/sda1/var/lib/docker/vfs/dir/9acd80fd069e82a89ac04a9141cc518ce8ee176bd7e4cc92c972dd3df50dffcc"
```

In boot2docker zie je dat als volgt - in die directory staat de database-data:

```
docker@boot2docker:~$ sudo ls -altrF /mnt/sda1/var/lib/docker/vfs/dir/9acd80fd069e82a89ac04a9141cc518ce8ee176bd7e4cc92c972dd3df50dffcc/
total 176164
drwx------    2 999      999           4096 May 14 10:59 performance_schema/
drwx------    2 999      999           4096 May 14 10:59 mysql/
-rw-rw----    1 999      999       50331648 May 14 10:59 ib_logfile1
-rw-rw----    1 999      999             56 May 14 10:59 auto.cnf
drwx------    2 999      999           4096 May 14 11:02 wordpress/
drwx------  126 root     root         16384 May 14 11:07 ../
drwxr-xr-x    5 999      999           4096 May 14 11:07 ./
-rw-rw----    1 999      999       79691776 May 14 11:07 ibdata1
-rw-rw----    1 999      999       50331648 May 14 11:07 ib_logfile0
```

### Backup en restore

Eén manier van backuppen van de database is de volgende:

```
$ docker exec -ti wordpressmysql_db_1 bash -c \
 'mysqldump --user=root --password="$MYSQL_ROOT_PASSWORD" --all-databases' \
  > MySQL_full_backUp.sql
```

Restoren kan middels volume-sharing en een gelinkte container ("db" in het onderstaande geval):

```
$ docker run -ti --rm -v $(pwd):/tmp --link wordpressmysql_db_1:db mysql bash

root@b2252da7520e:/# cd /tmp/
root@b2252da7520e:/tmp# ls -al

...

-rw-r--r--  1 1000 staff   1075329 May 18 14:36 MySQL_full_backUp.sql

...

root@b2252da7520e:/tmp# mysql -u root -p -h $DB_PORT_3306_TCP_ADDR
Enter password:

mysql> source MySQL_full_backUp.sql

```

Referentie: YouTube - [Docker Tutorial: Backing Up and Restoring MySQL and WordPress in Docker](https://www.youtube.com/watch?v=2ZX3F-aFOxQ)


'n Handigere tool? [docker-backup](https://github.com/docker-infra/docker-backup). Wel even compileren vanuit Go en niet om tegen *boot2docker* te gebruiken...

*That's all Folks!*

---
Henk Bokhoven, 05/2015
