Traccar con MySQL 
===========

<p align="center">
	<img src="https://github.com/ginppian/Traccar4/blob/master/imgs/traccar.png" width="682" height="331">
</p>

## Definición

<p align="justify">
	Vamos a hacer la instalación de Traccar con una Base de Datos en MySQL.
</p>

## Requisitos

* Cuenta en <a href="https://m.do.co/c/ccd8a4dcf484">DigitalOcean</a>

## Desarrollo

1. Damos de alta nuestro servidor. 

Si no sabemos cómo hacer lo podemos consultar <a href="https://github.com/ginppian/DigitalOcean-New_Drop_UpWith_Filezilla">aquí</a>.

En mi caso yo elegí:

* Ubuntu 16.04.3 x64

* $10/mo (1GB RAM, 30GB SSD disk, 1 CPU, 2 TB transfer)

* San Francisco, Data Center

* Sin Opciones Adicionales

2. Creamos un nuevo usuario en nuestro VPS (Servidor Virtual Personal)

```
ssh root@ip
```

* creamos un nuevo usuario:

```
adduser ginppian
```

nos pedirá una *password* y después unos datos personales, estos los podemos dejar vacios y precionar solamente enter.

* cambiamos el modo para que nuestro nuevo usuario tenga permisos de administrador (add Group sudoers):

```
usermod -aG sudo ginppian
```

* agregamos la llave *ssh*, esta la podemos obtener de nuestra máquina (cd ~, cd .ssh, cat id_rsa.pub) o desde nuestro VPS:

```
cd ~
mkdir .ssh
sudo cat /root/.ssh/authorized_keys > ~/.ssh/authorized_keys
```

nos salimos 

```
exit
exit
```

y volvemos a entrar

```
ssh ginppian@ip
```

y ya podremos acceder con nuestro nuevo usuario.

3. Instalamos Apache, MySQL, y PHP

*Consultar Fuente 1*

### Apache

```
sudo apt-get update
sudo apt-get install apache2
```

Agregamos un ServerName

```
sudo nano /etc/apache2/apache2.conf
```

Escribimos la siguiente linea al final del archivo y guardamos

```
ServerName server_domain_or_IP
```

donde server_domain_or_IP es nuestra ip. Tipeamos el siguiente comando y si nos regresa el mensaje: *Syntax OK* todo va bien.

```
sudo apache2ctl configtest
```

Reseteamos Apache para implementar los cambiamos

```
sudo systemctl restart apache2
```

**FireWall**

```
sudo ufw app list
```

*If you look at the Apache Full profile, it should show that it enables traffic to ports 80 and 443:* podemos verificar que efectivamente están abiertos:

```
sudo ufw app info "Apache Full"
```

Permitimos la entrada de traffico a estos puertos:

```
sudo ufw allow in "Apache Full"
```

checamos en un navegador:

http://your_server_IP_address

y podemos ver una imágen así:

<p align="center">
	<img src="https://github.com/ginppian/Traccar4/blob/master/imgs/apache_default.png" width="372" height="517">
</p>

*If you see this page, then your web server is now correctly installed and accessible through your firewall*

### MySQL

Nos pedirá una contraseña.

```
sudo apt-get install mysql-server
```

Instalamos MySQL de forma segura con el siguiente comando: 

```
mysql_secure_installation
```

* Nivel Password
* Cambiar password para usuario Root
* Remover usuarios anonimos
* Desabilitar el login de Root de manera remota
* Remover Base de Datos de pruebas
* Refrescar los privilegios de las tablas ahora

Yo en todos le doy *Y* (YES) y la contraseña le dejo la misma. Si le das en una contraseña muy segura, tal vez tengas que cambiar la.

### PHP

```
sudo apt-get install php libapache2-mod-php php-mcrypt php-mysql
```

*In most cases, we'll want to modify the way that Apache serves files when a directory is requested. Currently, if a user requests a directory from the server, Apache will first look for a file called index.html. We want to tell our web server to prefer PHP files, so we'll make Apache look for an index.php file first.*

Editamos el siguiente archivo:

```
sudo nano /etc/apache2/mods-enabled/dir.conf
```

y veremos algo como:

```
<IfModule mod_dir.c>
    DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
</IfModule>
```

permutamos *index.php* al lugar de *index.html*`. De tal manera que nos quede así:

```
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

guardamos los cambios.

Reseteamos Apache:

```
sudo systemctl restart apache2
```

Verificamos que este en *active (running)* con:

```
sudo systemctl status apache2
```

Para instalar módulos adicionales podemos consultar con este comando:

```
apt-cache search php- | less
```

para instalar los simplemente con *apt-get install package-name*

### Test PHP Server

```
sudo nano /var/www/html/info.php
```

Copiamos y pegamos esto:

```
<?php
phpinfo();
?>
```

Guardamos los cambios.

Podemos consultar que todo salió bien en la siguiente liga:

http://your_server_IP_address/info.php

Si vemos algo como esta imagen todo SALIO BIEN.

<p align="center">
	<img src="https://github.com/ginppian/Traccar4/blob/master/imgs/php_info.png" width="372" height="320">
</p>

4. Instalamos PHPMyAdmin

*Consulte la Fuente 2*

Instalamos *PHPMyAdmin*

```
sudo apt-get install phpmyadmin apache2-utils
```

* Cuando nos pregunte dejamos seleccionado *Apache2*.

* Después nos pedirá la contraseña de nuestra Base de Datos.

* Después nos pedirá la contraseña que queremos usar para PHPMyAdmin.

Después que la instalación se ha completado agregamos phpMyAdmin a la configuración de Apache.

Editamos el siguiente archiovo:

```
sudo nano /etc/apache2/apache2.conf
```

Pegamos lo siguiente al final del archivo:

```
Include /etc/phpmyadmin/apache.conf
```

Reseteamos Apache

```
sudo service apache2 restart
```

Ahora podemos provar que efectivamente funciona

Entrando a la liga:


```
`http://my_ip_sever/phpmyadmin
```

User: root
Pass: ****


5. Instalamos Java

Para más información podemos consultar un tutorial anterior <a href="https://github.com/ginppian/Traccar2.0">aquí</a>

```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
sudo apt-get install oracle-java8-set-default
```

6. Instalamos Traccar

Para consultar todas las versiones de Traccar <a href="https://github.com/tananaev/traccar/releases">aquí</a>

Modificamos la hora:

```
sudo timedatectl set-timezone America/Mexico_City
```

sino sabemos nuestra hora podemos consultar tipeando: *timedatectl list-timezones*

```
cd /opt
sudo wget "https://github.com/tananaev/traccar/releases/download/v3.13/traccar-linux-3.13.zip"
sudo apt-get install unzip
sudo unzip traccar-linux-3.13.zip
sudo chmod +x traccar.run
sudo ./traccar.run
```

Si queremos modificarle la cantidad máxima de memoria a usar podemos modificar este archivo:

```
sudo nano /opt/traccar/conf/wrapper.conf
```

y agregamos la siguiente linea, podemos modificar *80* por el pocentaje que quiéramos:

```
wrapper.java.maxmemory.percent=80
```

Empezamos Traccar:

```
sudo /opt/traccar/bin/startDaemon.sh
```

Comprobamos que se ejecuta correctamente en:

http://ip:8082

### MySQL en Traccar

*Consultar la fuente 3*

Detenemos Traccar:

```
sudo /opt/traccar/bin/stopDaemon.sh
```

abrimos *phpMyAdmin* y nos logueamos para crear una nueva Base de Datos:

* Nueva
* Ponemos nombre a la Base de Datos
* En mi caso uso utf8_spanish_ci como cotejamiento

movemos y editamos el archivo de configuración:

```
cd /opt/traccar/conf/
sudo nano traccar.xml
```

borramos estas lineas:

```
<entry key='database.driver'>org.h2.Driver</entry>
<entry key='database.url'>jdbc:h2:/home/user/Documents/traccar/target/database</entry>
<entry key='database.user'>sa</entry>
<entry key='database.password'></entry>
```

pegamos estas en el mismo lugar donde estaban las otras:

```
<entry key='database.driver'>com.mysql.jdbc.Driver</entry> 
<entry key='database.url'>jdbc:mysql://[HOST]:3306/[DATABASE]?allowMultiQueries=true&amp;autoReconnect=true&amp;useUnicode=yes&amp;characterEncoding=UTF-8&amp;sessionVariables=sql_mode=''</entry>
<entry key='database.user'>[USER]</entry> 
<entry key='database.password'>[PASSWORD]</entry>
```

[HOST]:3306 ponemos localhost
[DATABASE] ponemos el nombre de la Base de Datos que acabamos de crear
[USER] ponemos root
[PASSWORD] ponemos la contraseña de nuestra Base de Datos

reiniciamos

```
sudo /opt/traccar/bin/startDaemon.sh
```

### Mapas de Google

Necesitamos entrar a Traccar en modo *admin* como no sabemos la contraseña tenemos que modificarla directamente desde la Base de Datos, podemos:

* Abrimos la consola de mysql

```
mysql --user=root --password=****
```

Usamos nuestra Base de Datos:

```
USE mydatabase;
```

Cambiamos la contraseña sea cual sea en nuestra tabla *users* para que tenga por *password* como viene cifrada usa un ciframiento *SALT*, este cifrado con este *SALT* corresponden a *password*:

```
ef38a22ac8e75f7f3a6212dbfe05273365333ef53e34c14c
000000000000000000000000000000000000000000000000
```

ejecutamos:

```
UPDATE users SET hashedpassword='ef38a22ac8e75f7f3a6212dbfe05273365333ef53e34c14c' WHERE id=1;
UPDATE users SET salt='000000000000000000000000000000000000000000000000' WHERE id=1;
```

o podemos hacer lo directo desde *PHPMyAdmin*, modificando directamente la tabla *users* dandole en editar.

* En el ícono de gradiente (arriba a la derecha) o nuestras preferencias damos clic
* En el cuadro de texto de hasta arriba llamado *Capa de Mapa* seleccionamos *Mapa Personalizado*
* En el cuadro de texto *Mapa Personalizado* pegamos la siguiente *URL*:

```
http://mt0.google.com/vt/lyrs=m&hl=en&x={x}&y={y}&z={z}&s=Ga
```

y refrescamos el navegador. Podemos hacer *log out* e iniciar sesión con otra cuenta.


7. Aplicación

El código que usamos en el momento de que se vea este tutorial estará antiguo de todas formas lo anexamos en el proyecto, si se quiere descargar el nuevo código podremos hacer lo desde <a href="https://github.com/tananaev/traccar-client-android">aquí</a>.

* Visible para el usuario. 

Buscamos este sección de código:

```java
if (BuildConfig.HIDDEN_APP) {
	removeLauncherIcon();
}
```

y lo comentamos:

```java
if (BuildConfig.HIDDEN_APP) {
	//removeLauncherIcon();
}
```

de esta manera permitiremos que la aplicación sea visible para el usuario.

* Habilitar/Deshabilitar Edición

Para que el usuario de la aplicación pueda editar las opciones, Traccar usa esta función:

```java
    private void setPreferencesEnabled(boolean enabled) {
        findPreference(KEY_DEVICE).setEnabled(enabled);
        findPreference(KEY_URL).setEnabled(enabled);
        findPreference(KEY_INTERVAL).setEnabled(enabled);
        findPreference(KEY_DISTANCE).setEnabled(enabled);
        findPreference(KEY_ANGLE).setEnabled(enabled);
        findPreference(KEY_PROVIDER).setEnabled(enabled);
    }
```

entonces si queremos nos otros configurar las y después que el usuario ya no pueda configurarlas, tenemos que modificar esa función.

En el método *onCreate* que es el primero que se carga, hasta el final agregamos la siguiente función. Esta función pregunt, si es la primera vez que se abre el app. Si es la primera vez nos otros como administradores podremos editar la con los campos correspondientes a nuestro servidor, sino es la primera vez que se ejecuta, deshabilitaremos la edición para el usuario:

```
        final String PREFS_NAME = "MyPrefsFile";

        SharedPreferences settings = getSharedPreferences(PREFS_NAME, 0);

        if (settings.getBoolean("my_first_time", true)) {
            //the app is being launched for first time, do something
            Log.d("Comments", "First time");

            // first time task

            // record the fact that the app has been started at least once
            settings.edit().putBoolean("my_first_time", false).commit();
        } else {

            setPreferencesEnabled(false);
        }
```

También la ocultaremos cuando se detiene el servicio (stopTrackingService) y cuando empieza el servicio (startTrackingService):

```
    private void startTrackingService(boolean checkPermission, boolean permission) {
        if (checkPermission) {
            Set<String> missingPermissions = new HashSet<>();
            if (!hasPermission(Manifest.permission.ACCESS_FINE_LOCATION)) {
                missingPermissions.add(Manifest.permission.ACCESS_FINE_LOCATION);
            }
            if (!hasPermission(Manifest.permission.ACCESS_COARSE_LOCATION)) {
                missingPermissions.add(Manifest.permission.ACCESS_COARSE_LOCATION);
            }
            if (missingPermissions.isEmpty()) {
                permission = true;
            } else {
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                    requestPermissions(missingPermissions.toArray(new String[missingPermissions.size()]),
                            PERMISSIONS_REQUEST_LOCATION);
                }
                return;
            }
        }

        if (permission) {
            //ginppian
            setPreferencesEnabled(false);
            startService(new Intent(this, TrackingService.class));
            alarmManager.setInexactRepeating(AlarmManager.ELAPSED_REALTIME_WAKEUP,
                    15000, 15000, alarmIntent);
        } else {
            //ginppian
            setPreferencesEnabled(false);
            sharedPreferences.edit().putBoolean(KEY_STATUS, false).commit();
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.ICE_CREAM_SANDWICH) {
                TwoStatePreference preference = (TwoStatePreference) findPreference(KEY_STATUS);
                preference.setChecked(false);
            } else {
                CheckBoxPreference preference = (CheckBoxPreference) findPreference(KEY_STATUS);
                preference.setChecked(false);
            }
        }
    }

    private void stopTrackingService() {
        alarmManager.cancel(alarmIntent);
        stopService(new Intent(this, TrackingService.class));
        //ginppian
        //setPreferencesEnabled(true);
        setPreferencesEnabled(false);

    }
```

De esta menera nos otros únicamente podremos editar las configuraciones la primera vez.

* Vista

Cambiamos el nombre del app:

```
En la carpeta res>values>strings>mx, 
	app_name: myName
	hidden_app_name: myName
```

Cambiar los íconos con unos que nos provee *Android Studio*:

```
Android Studio Help > Find Action > image > Image Assets 
```

Editamos nuestra imagen y al guardar le damos:

```
Res Directory > hidden
```

Así sobre escribirá los íconos. Hidden es el nombre de nuestra carpeta si es diferente deberiamos escoger el correspondiente.

Para cambiar los colores:

```
res > values > colors.xml
```

modificamos los valores:

```
    <color name="primary">#E91E63</color>
    <color name="primary_dark">#C2185B</color>
    <color name="accent">#E040FB</color>
```

si queremos una referencia de colores usar, podemos echar un vistazo en <a href="https://www.materialpalette.com/">material palette</a>.

## Fuentes

1. <a href="https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-16-04">How To Install Linux, Apache, MySQL, PHP (LAMP) stack on Ubuntu 16.04</a>

2. <a href="https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-ubuntu-12-04">How To Install and Secure phpMyAdmin on Ubuntu 12.04</a>

3. <a href="https://www.traccar.org/mysql/">Traccar MySQL Database</a>

4. <a href="https://wrapper.tanukisoftware.com/doc/spanish/prop-java-maxmemory.html">wrapper.java.maxmemory</a>

5. <a href="https://www.traccar.org/forums/topic/my-service-stops-java-heap/">Traccar - My service stops - JAVA Heap</a>

6. <a href="https://github.com/tananaev/traccar/issues/1293#issuecomment-121156068">Traccar - Update from 2.x to 3.0 migrate all data</a>

7. <a href="https://www.traccar.org/map-layers/">Offical Traccar Map Layers</a>

8. <a href="https://www.traccar.org/forums/topic/google-map-layer/">Traccar - Google Map Layer</a>

9. <a href="https://www.traccar.org/forums/topic/google-maps-road-satellite/">Google Maps Road + Satellite</a>

10. <a href="https://developer.android.com/studio/write/image-asset-studio.html?hl=es-419#access">Ejecución de Image Asset Studio</a>

11. <a href="https://stackoverflow.com/questions/4636141/determine-if-android-app-is-the-first-time-used">Determine if android app is the first time used</a>



## Otros

* <a href="https://www.youtube.com/watch?v=vnpRx0pkSsc">Where to change the google map api key? </a>
* <a href="https://www.traccar.org/forums/topic/custom-web-interface/">Traccar - Custom Web Interface</a>
* <a href="http://traccar.litvak.su/installation.html">Fork Traccar</a>
* <a href="https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-16-04">DigitalOcean - How To Install MySQL on Ubuntu 16.04</a>





