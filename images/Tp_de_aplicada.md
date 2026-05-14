Bueno, arranqué por **blanquear el root**, porque la contraseña inicial era desconocida. 

![[Blanqueo de root.webm]]
En el video se aprecia cómo hice el proceso y después configuré la contraseña pedida:
`palermo`

Después configuré el **hostname** de la máquina como:
![[Hostnname.png|379]]
`TPServer`

Luego, al momento de actualizar el sistema, me encontré con algunos problemas. La VM venía con **Debian 11**, y cuando intenté actualizar directamente tuve inconvenientes con la resolución de DNS y con los repositorios configurados en `/etc/apt/sources.list`, así que primero tuve que corregir eso.

Por ese motivo, primero realicé una **actualización completa de Debian 11** para dejar el sistema estable y consistente. Acá el proceso de instalación
![[En el camino tuve que actualizar debian 11.png]]Y acá ya con debian 11 completamente actualizado

![[Debian 11 completamente actualizado 1.png|596]]

Una vez resuelto eso, modifiqué los repositorios para apuntar a **Debian 12 (Bookworm)** y recién ahí inicié la actualización de Debian 11 a Debian 12. (evidencia: _debian 11 a debian 12.png_)!![[debian 11 a debian 12.png|449]]![[proceso de debian 12 instalandose.png|475]]

A continuación la reinstalación del grub:
![[Pasted image 20260512181748.png]]
Volviendo ahora a la terminal como la conocemos, desde el root, lo que toca por hacer según la consigna es installar ssh. 

Pero primero instalo nano para trabajar más comodamente. (Ejecuto entonces apt-get install nano)

![[Pasted image 20260512184638.png|695]]Hacemos un systemctl restart ssh

Lo habilitamos con "systemctl enable ssh" para que arranque solo al iniciar

![[Pasted image 20260512185149.png]]

Con todo esto creado sumado al ssh-keygen.
Procedemos a instalar Apache y PHP

![[Pasted image 20260512185652.png]]
Hecho.

Descomprimimos la carpeta: Material_Adicional_TPVMCA para acceder a los archivos con los que vamos a trabajar

tar -xzvf /root/Material_Adicional_TPVMCA.tar.gz -C /root/

![[Pasted image 20260512190426.png]]
Extra:

A esta altura habilite con el disco optico la posibilidad de copiar y pegar en la terminal.

Resulta que funciona para gui y no para tty que sería solo terminal.

![[Pasted image 20260512202101.png]]
Por fin se instaló mariadb

![[Pasted image 20260512202155.png]]

Y ya con todo esto pasamos a la consigna 3

![[Pasted image 20260512202451.png]]

Con estos datos ya podemos abrir el: nano /etc/network/interfaces

Porque tenemos: 
- **Interfaz:** `enp0s3`
- **IP actual:** `192.168.0.176` (usamos esta misma como estática)
- **Gateway:** `192.168.0.1`
- **Netmask:** `255.255.255.0`
- ![[Pasted image 20260512202711.png]]

Parte 4 de la consigna.

![[Agregando el nuevo disco 1.mp4]]

![[Pasted image 20260512203754.png]]

Y acá las respectivas particiones que se piden en la consigna

Damos el formato ext4

![[Pasted image 20260512203901.png]]

Creamos las carpetas correspondientes y montamos, esto es crear el directorio que nos pide la consigna.
![[Pasted image 20260512204040.png]]

_"4) Configurar el directorio /www_dir para que se monte automáticamente al iniciar el sistema operativo."_ _"5) Configurar el directorio /backup_dir para que se monte automáticamente al iniciar el sistema operativo."_

El archivo `/etc/fstab` es una lista que Linux lee **cada vez que arranca**, y monta automáticamente los discos que están listados ahí. Sin esto, las particiones `/www_dir` y `/backup_dir` existirían pero quedarían vacías y sin conectar al sistema cada vez que reiniciás.

Lo que hicimos fue:

- Agregar las dos líneas con el UUID de cada partición → así Linux las identifica aunque cambien de nombre
- El `mount -a` recarga el fstab sin reiniciar, para verificar que está bien escrito
- El `reboot` es para confirmar que arrancan solas al inicio

![[Pasted image 20260513131724.png]]


Así quedó despues del reboot:
![[Pasted image 20260513131907.png]]
Seguimos entonces con la consigna 4.3 que es: mover los archivos web a `/www_dir` y actualizar Apache

![[Pasted image 20260513133646.png]]Archivos movidos y actualizado apache![[Pasted image 20260513133724.png]]

Hecho lo último del 4
![[Pasted image 20260513133827.png]]

Vamos con el 5

Ejecutamos:

mkdir /opt/scripts
Para la primer consigna

y ahi nos quedamos porque no aprendimos nada más

nano /opt/scripts/backup_full.sh
El código con las configuraciones
![[Pasted image 20260513160909.png|489]]
chmod +x /opt/scripts/backup_full.sh
![[Pasted image 20260513161039.png]]

Ahora abrimos el crontab con crontab -e
![[Pasted image 20260513161243.png]]

Y así nos quedaría entonces
![[Pasted image 20260513161423.png]]