# vsftpd M08-UF2

Para empezar tendremos que tener instalada una maquina virutal de Ubuntu Desktop(que es la que he elegido para hacer la actividad)

Empezaremos instalando el servicio vsftpd en Ubuntu con el comando: sudo apt install vsftpd

Al tenerlo instalado tendremos que comprobar que este instalado y funcionando con el comando sudo service vsftpd restart


Al verificar que este todo correcto, iremos a editar el archivo de configuración de vsftpd que esta dentro del directorio /etc/vsftpd.conf

Y tendremos que cambiar los siguientes valores:

anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES

Guardamos el archivo y reiniciamos el servicio con sudo service vsftpd restart

Ahora pasaremos a crear el usuario admin
Usaremos el comando sudo adduser admin
Y nos creara el usuario, y para establecer la contraseña usaremos el comando: sudo passwd admin
y le pondremos la contraseña "admin"

Ahora para hacer la conexión de la maquina virtual a la física, usaremos la herramienta filezilla, en el ordenador físico.
El cual tendremos que abrirlo, gestionar un nuevo sitio, y configurar el sitio con los siguientes valores:

Host: La dirección IP o el nombre de dominio de tu servidor FTP.
Protocolo: FTP - Protocolo de transferencia de archivos.
Cifrado: "Solo FTP"

Despues en la sección de inicio de sesión, se tendra que introducir lo siguiente:

Tipo de inicio de sesión: Normal.
Usuario: admin.
Contraseña: admin.
Puerto: 21 (puerto predeterminado para FTP).

Y ahora en conectar, y ya podremos comprobar que podemos conectar-nos.

Ahora para que los usuarios no deban poder subir de su directorio personal, hay que tener en cuenta que dentro del archivo de configuración de vsftpd
/etc/vsftpd.conf

Con esta opción que hemos marcado anteriormente sirve para que los usuarios se encuentren restringidos a su directorio personal:
chroot_local_user=YES

Y para permitir a los usuarios que tengan acceso de escritura de su propio directorio añadiremos lo siguiente:
write_enable=YES
Y restableceremos el servicio

Para configurar el rango de puertos para el modo pasivo, tendremos que entrar en el archivo de configuración de vsftpd otra vez, y añadir lo siguiente:

pasv_enable=YES //Habilita el modo pasivo
pasv_min_port=62000 
pasv_max_port=63000
pasv_address=<IP_DEL_SERVIDOR>

El pasv_min_port y el pasv_max_port especifican el rango de puertos que se usaran para las conexiones en modo pasivo.

Y en el pasv_address debe reemplazarse por la dirección IP pública o la dirección IP local de tu servidor

Ahora tendremos que reiniciar el servicio

Volveremos a entrar al archivo de configuración de vsftpd.conf
Y añadiremos lo siguiente:

max_clients=30 // Para especificar el numero maximo total de conexiones suimultaneas permitidas
max_per_ip=5 // Para especificar el numero maximo de conexiones simultaneas permitidas desde una misma dirección IP. En ese caso se establece en 5 para permitir 5 conexiones anonimas simultaneas

Y volvemos a reiniciar el servicio para que se guarde

Ahora para configurar al usuario anonimo para que pueda descargar archivos existentes en el directorio /srv/public tendremos que crear ese directorio
sudo mkdir .p /srv/public
Y estableceremos en ese directorio permisos para que sean accesibles por el usuario anonimo

sudo chmod 755 /srv/public

Al establecer los permisos, tendremos que volver al archivo de configuración de vsftpd y añadir las lineas sigueintes y descomentarlas:

anonymous_enable=YES // Habilita el acceso anonimo
anon_root=/srv/public // Establece el directorio raíz para el usuario anonimo como /srv/public

Despues de guardar la configuración reiniciamos el servicio otra vez.

Ahora para agregar ui certificado autofirmado en vsftpd, tendremos que usar el siguiente comando:

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout vsftpd.pem -out vsftpd.pem

Una vez hemos generado los archivos de clave privada y el certificado (que es el vsftpd.pem) los copiaremos en la ubicación adecuada.

sudo cp vsftpd.pem /etc/vsftpd/

Y entraremos en el archivo de configuración vsftpd.conf

Asegurarnos de que las lineas estan descomentadas y cambiandolas por lo siguiente:

rsa_cert_file=/etc/vsftpd/vsftpd.pem
rsa_private_key_file=/etc/vsftpd/vsftpd.pem

Y guardaremos los cambios y reiniciaremos el servicio.

Para acabar tendremos que lograr acceder desde la maquina del usuario ubuntu al usuario admin, en otra maquina sin contraseña usando certificados autogenerados.

En la maquina donde queramos crear el usuario admin, generaremos varias claves ssh

ssh-keygen -t rsa -b 4096 -f admin_key

Se generara dos archivos: admin_key (clave privada) y admin_key.pub (clave publica)

Copiamos la clave publica en el archivo  "~/.ssh/authorized_keys" del usuario admin en la maquina destino, y para ello podemos hacerlo con este comando:

ssh-copy-id -i admin_key.pub admin@192.168.1.98
(La IP es en mi caso)

Lo que hace es reemplazar la IP que hemos puesto por la dirección IP de la maquina destino

Nos aseguraremos con el comando: chmod 600 ~/.ssh/authorized_keys
Que teng alos permisos adecuados.

En la maquina del usuario ubuntu, crearemos dos claves SSH (publica y privada)
Y seguiremos los mismos pasos que hemos hecho antes.
ssh-copy-id -i ubuntu_key.pub admin@192.168.1.97

Despues tendremos que abrir el archivo sshd_config ubicado en /etc/ssh para verificar que en la línea "PasswordAuthentication" este configurado como "no"

Ahora reiniciaremos el servicio ssh: sudo service ssh restart

Y por ultimo tendremos que probar el acceso desde la maquina ubunu al usuario admin, abriremos la terminal del suuario ubuntu y ejecutamos:

ssh -i ubuntu_key admin@192.168.1.98

Y estableceremos conexión ssh sin necesidad de contraseña.
