# null_site

Landing page para nulldevs.tech dockerizada :whale: :whale2:

`wrote by: n0b0dy-su`

Lo escrito a continuación describe el proceso de instlación de docker y 
docker compse mas la configuración de paache que utilicé para setear la 
landing, pero de igual manera es util para cualquier app dockerizada del 
mismo estilo.

## Instalacion de Docker y Docker compose

En este caso la distribución del servidor es :heart: debian 11 :heart:, por lo
que para la instalación de docker realicé los siguentes pasos, basado en 
el post [Install Docker CE and Docker Compose on Debian 11/10](https://computingforgeeks.com/install-docker-and-docker-compose-on-debian/) 

**NOTA:** Todos los comandos son ejecutados desde el root por lo que no llevan 
sudo

### Docker

1. Para la instalación de dokcer y docker compose es necesario primero instalar
algunas dependencias con los siguientes comandos
```bash
apt update
apt -y install apt-transport-https ca-certificates curl gnupg2 software-properties-common
```

2. Importar y agregar la llave GPG para instalr docker
```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
```

3. Agregar el repositorio a debian
```bash
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
```

4. Actualizar los repositorios co nel agregado reciente mente
```bash
apt update -y
```

5. Instalar docker community edition
```bash
apt -y install docker-ce docker-ce-cli containerd.io
```

6. Habiliatr el servicio de docker
```bash
systemctl enable --now docker
```

**OPCIONAL:** SI se esta utilizando un usuario diferente a root se debe agregar
al grupo de docker para que este pueda ejecutarlo

```bash
sudo usermod -aG docker $USER
newgrp docker
```

### Docker compose

Estos pasos estan basado en el post [How To Install Docker Compose on Linux](https://computingforgeeks.com/how-to-install-latest-docker-compose-on-linux/)

1. Descragar la ultima versión desde github
```bash
curl -s https://api.github.com/repos/docker/compose/releases/latest | grep browser_download_url  | grep docker-compose-linux-x86_64 | cut -d '"' -f 4 | wget -qi -
```

2. Dar permisos al ejecutable
```bash
chmod +x docker-compose-linux-x86_64
```

3. Mover el archivo al path de los binarios para ser ejecutado simplemente 
```bash
mv docker-compose-linux-x86_64 /usr/local/bin/docker-compose
```

## Configuración de apache

Esto tambien se debe hacer para cualquier aplicacion dockerizada en este caso 
una web, excepto los pasos **1** y **2** que solo se realizan la primera vez,
si ya se realizaron estos pasos solo se realiza apartir del **3**

1. Cree el directorio **/etc/apache2/own-conf/**, para las configuraciones 
propias de las apps dockerizadas para usarla en la configuracion de apache,las
cuales son importadas al archivo de la configuración principal de apache que 
para este caso es el archivo **/etc/apache2/sites-enabled/000-default-le-ssl.conf**

2. Para importar la configuracion de los proxys de cada contenedor en el 
archivo **/etc/apache2/sites-enabled/000-default-le-ssl.conf** agregué las
siguientes lineas dentro de la configuracion de virtulhost *:443, con esto
se inicializa la configuración del proxy de apache y se importan las 
configuraciones de los demas proxys para los contendores, ya que la 
configuración de los proxys es la misma solo la inicialicé en el archivo de 
apache y solo se importan los archivos del directrio **/own-conf**

```bash
	<Proxy *>
      Require all granted
    </Proxy>

	ProxyPreserveHost On
	ProxyRequests Off	
    SSLProxyEngine On

    Include /etc/apache2/own-conf/*.conf
```

3. Dentro del directorio **/etc/apache2/own-conf/** cree el archivo 
correspondiente al proxy del contendor. Por la manera en que el **Include** 
esta escrito en la configuracion de apache los archivos dentro del directorio 
**own-conf** deben tener la extencion **.conf**  seran ignorados por el include,
el contenido de dichos archivos debe ser el siguiente

```bash
  ProxyPass / http://localhost:81/
  ProxyPassReverse / http://localhost:81/
```

**NOTA:** Para este caso el contenedor hace un mapeo de su puerto 80 al puerto 
81 del host `0.0.0.0:81->80/tcp, :::81->80/tcp`, por lo lo que por ejemplo si 
el contendor que se va ausar ahce un mapeo al puerto 8080 del host la 
configuración del host de quedar de la sigueinte manera

```bash
  ProxyPass / http://localhost:8080/
  ProxyPassReverse / http://localhost:8080/
```

**NOTA:** El `/` del final en `http://localhost:8080/` debe estar ya que si no
se causa un error ya que no se pueden encontrar los recursos como por ejemplo 
estilos de css.

4. Despues de realizar los pasos anteriores se debe reiniciar apache para
aplicar la configuración, si los modulos de proxy no estan habilitados se debe
reiniciar con el comando 
`a2enmod proxy && a2enmod proxy_http && systemctl restart apache2`, este 
comando esta ejecutado desde el usuario root, sin oesta en un usuario root
agregar el sudo, en caso de que ya se esten habilitados los modulos de proxy 
solo reiniciar con `systemctl restart apache`

`n0b0dy-su was here`