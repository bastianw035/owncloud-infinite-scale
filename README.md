# owncloud-infinite-scale
installation of owncloud infinite scale (ocis) in clean debian container (pve)

> Alles ausgeführt als root


## Schritt 1: System vorbereiten:
```
apt update && apt upgrade -y && timedatectl set-timezone Europe/Berlin
# Add Docker's official GPG key:
apt-get update
apt-get install ca-certificates curl
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update


apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

reboot
```



OCIS Docker image herunterladen:
```
docker pull owncloud/ocis
```

Benötigte Ordner anlegen:
```
mkdir -p $HOME/ocis/ocis-config
```
```
mkdir -p $HOME/ocis/ocis-data
```

Den gerade erstellten Ordner die richtigen Rechte geben, damit der User im Container auch auf diese zugreifen kann:
```
sudo chown -Rfv 1000:1000 $HOME/ocis/
```

## Schritt 2: Erstes Starten des Containers
Beim ersten Starten des Containers werden die Konfigurationsdateien für ownCloud erstellt. Hierbei stellen wir auch das "certificate checking" aus (also Antwort mit 'yes'):
```
docker run --rm -it \
    --mount type=bind,source=$HOME/ocis/ocis-config,target=/etc/ocis \
    owncloud/ocis init
```

Nun sollte man einen Output wie diesen sehen:
```
Do you want to configure Infinite Scale with certificate checking disabled?
 This is not recommended for public instances! [yes | no = default]

=========================================
 generated OCIS Config
=========================================
 configpath : /etc/ocis/ocis.yaml
 user       : admin
 password   : <removed for documentation>
```

Diesen Container kann man nun auch wieder stoppen. Um ownCloud richtig zu starten benutzen wir folgenden Befehl:
```
docker run -d --restart=always --name ocis -it -p 9200:9200 --mount type=bind,source=$HOME/ocis/ocis-config,target=/etc/ocis --mount type=bind,source=$HOME/ocis/ocis-data,target=/var/lib/ocis -e OCIS_INSECURE=true -e PROXY_HTTP_ADDR=0.0.0.0:9200 -e OCIS_URL=https://<deine-domäne> -e OCIS_DEFAULT_LANGUAGE=german owncloud/ocis
```










