![Docker Grunn](full_6650351.jpeg)

# Docker Grunn meetup, woensdag 27 mei

## Vooraf zelf te installeren!!

Zorg dat je op zijn minst de beschikking hebt over een laptop (met volle accu) of VPS met

- een **werkende** installatie van *Docker* (Linux) of *boot2docker* (OS X / Windows).  

De voorkeur gaat uit naar Linux en OS X; een installatie-handleiding staat [hier](http://docs.docker.com/compose/install/#install-docker).  
*boot2docker* is op zijn beurt weer afhankelijk van VirtualBox.  

Referenties:  
- [Guide to Docker on OS X](http://blog.tutum.co/2015/05/19/guide-to-docker-on-os-x/)  
- [How to use Docker on Windows](http://blog.tutum.co/2014/11/05/how-to-use-docker-on-windows/)

Of Docker juist werkt, kan je vervolgens testen met `docker run --rm hello-world`.

- en **Docker Compose ** ([link](http://docs.docker.com/compose/install/)):  

Alleen beschikbaar voor Linux & OS X!  
De installatie werkt zo:

```
$ curl -L https://github.com/docker/compose/releases/download/1.2.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```

### **DRINGEND ADVIES**

De WiFi-bandbreedte bij TFE is niet om te juichen, helaas.  Zie [deze afbeelding](TFE_wifi_2015-05-22-161244.png)!  
Dus... om snel aan de slag te kunnen i.p.v. lang te wachten (omdat iedereen tegelijkertijd over WiFi wil downloaden), is het advies om alvast de Docker-images voor WordPress en MySQL te downloaden:

- `$ wget https://uploads.tfe.nl/dckrgrnn/mysql.tar.gz` (size: 142 MB)
- `$ wget https://uploads.tfe.nl/dckrgrnn/wordpress.tar.gz` (size: 163 MB)

Deze images zijn afkomstig uit de officiele Docker Registry, *saved to a tar archive*, *gzipped* en op een host van TFE geplaatst. Tijdens de *hands-on* kan je deze importeren en zo relatief snel aan de slag!


## WordPress & MySQL dual container setup

Meer tijdens de *hands-on* op de #dockermeetup !

---
Henk Bokhoven, 05/2015
