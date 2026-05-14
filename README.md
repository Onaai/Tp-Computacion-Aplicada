# 🖥️ TP Integrador — Computación Aplicada

**Universidad de Palermo**

---

## 👥 Integrantes del grupo

| Nombre |
|--------|
| Javier Valdez |
| Mateo Poggio |
| Emiliano Campos |

---

## 📋 Descripción del proyecto

Trabajo Práctico Integrador grupal de la materia **Computación Aplicada**. Consiste en la configuración completa de un servidor GNU/Linux Debian en una máquina virtual VirtualBox, abarcando servicios de red, web, base de datos, almacenamiento y automatización de backups.

---

## ✅ Consigna 1 — Configuración del entorno

### 1.2 — Blanqueo y cambio de contraseña root

La VM venía con contraseña de root desconocida. Se accedió al modo recovery desde el GRUB para blanquearla y luego se estableció la contraseña requerida por la consigna: `palermo`

> 📹 **Video:** `Blanqueo de root.webm` — se aprecia el proceso completo de blanqueo y cambio de contraseña.

### 1.3 — Hostname

Se configuró el nombre del servidor con:

```bash
hostnamectl set-hostname TPServer
hostname
```

![Configuración del hostname](images/Hostnname.png)

---

## ✅ Consigna 2 — Servicios

### 2.1 — Actualización del SO a Debian 12

La VM venía con **Debian 11 (Bullseye)**. Antes de actualizar a Debian 12, fue necesario corregir problemas de resolución DNS y repositorios en `/etc/apt/sources.list`. Se realizó primero una actualización completa de Debian 11 para dejar el sistema estable:

```bash
apt update
apt upgrade -y
apt autoremove
```

![Actualización de Debian 11](images/En_el_camino_tuve_que_actualizar_debian_11.png)
![Debian 11 completamente actualizado](images/Debian_11_completamente_actualizado_1.png)

Luego se modificaron los repositorios para apuntar a **Debian 12 (Bookworm)** y se inició la migración:

```bash
nano /etc/apt/sources.list
apt update
apt full-upgrade -y
reboot
```

![apt update con repos de Debian 12](images/Pasted_image_20260512181733.png)
![Proceso de instalación de Debian 12](images/proceso_de_debian_12_instalandose.png)
![Debian 12 - full-upgrade completado](images/debian_11_a_debian_12.png)

Durante la actualización se reinstalaron componentes del sistema, incluyendo el gestor de arranque GRUB:

![Reinstalación del GRUB](images/Pasted_image_20260512181748.png)

### 2.2 — SSH

Se instaló y configuró el servicio SSH. Se editó `/etc/ssh/sshd_config` habilitando el acceso root por clave pública:

```bash
apt-get install openssh-server -y
apt-get install nano -y
```

![Configuración de sshd_config](images/Pasted_image_20260512184638.png)

```bash
systemctl restart ssh
systemctl enable ssh
ssh-keygen
cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys
```

![SSH keygen y authorized_keys](images/Pasted_image_20260512185149.png)

### 2.3 — Servidor Web (Apache + PHP)

```bash
apt-get install apache2 php libapache2-mod-php -y
systemctl start apache2
systemctl enable apache2
```

![Instalación de Apache y PHP](images/Pasted_image_20260512185652.png)

Los archivos `index.php` y `logo.png` se obtuvieron descomprimiendo el material adicional provisto por la cátedra:

```bash
tar -xzvf /root/Material_Adicional_TPVMCA.tar.gz -C /root/
```

![Descompresión del material adicional](images/Pasted_image_20260512190426.png)

### 2.4 — Base de datos (MariaDB)

```bash
apt-get install mariadb-server -y
systemctl start mariadb
systemctl enable mariadb
mysql -u root < /root/Material_Adicional_TPVMCA/db.sql
mysql -u root -e "SHOW DATABASES;"
```

![Instalación de MariaDB](images/Pasted_image_20260512202101.png)
![Verificación de bases de datos](images/Pasted_image_20260512202155.png)

---

## ✅ Consigna 3 — Configuración de red

Se obtuvo la información de red con `ip a` e `ip route`:

![Datos de red](images/Pasted_image_20260512202451.png)

Con esos datos se configuró una IP estática editando `/etc/network/interfaces`:

```
auto enp0s3
iface enp0s3 inet static
    address 192.168.0.176
    netmask 255.255.255.0
    gateway 192.168.0.1
```

![Configuración de interfaces](images/Pasted_image_20260512202710.png)

```bash
systemctl restart networking
```

---

## ✅ Consigna 4 — Almacenamiento

### 4.1 — Nuevo disco

Se agregó un disco de **10 GB** desde la configuración de VirtualBox (Puerto SATA 3 → `sdc`).

> 📹 **Video:** `Agregando el nuevo disco 1.mp4` — muestra el proceso de agregar el disco desde VirtualBox.

### 4.2 — Particiones

Se crearon dos particiones estándar (tipo 83 - Linux) usando `fdisk /dev/sdc`:

| Partición | Tamaño | Directorio |
|-----------|--------|------------|
| `/dev/sdc1` | 3 GB | `/www_dir` |
| `/dev/sdc2` | 6 GB | `/backup_dir` |

![fdisk - creación de particiones](images/Pasted_image_20260512203754.png)

Se formatearon con ext4:

```bash
mkfs.ext4 /dev/sdc1
mkfs.ext4 /dev/sdc2
```

![Formato ext4 de las particiones](images/Pasted_image_20260512203901.png)

Se crearon los directorios y se montaron:

```bash
mkdir /www_dir
mkdir /backup_dir
mount /dev/sdc1 /www_dir
mount /dev/sdc2 /backup_dir
```

![Creación de directorios y montaje](images/Pasted_image_20260512204040.png)

### 4.3 — Archivos web en `/www_dir`

Se copiaron `index.php` y `logo.png` a la nueva partición y se actualizó la configuración de Apache apuntando el `DocumentRoot` a `/www_dir`:

```bash
cp /root/Material_Adicional_TPVMCA/index.php /www_dir/
cp /root/Material_Adicional_TPVMCA/logo.png /www_dir/
```

`/etc/apache2/sites-available/000-default.conf`:
```apache
<VirtualHost *:80>
    DocumentRoot /www_dir
</VirtualHost>
```

En `/etc/apache2/apache2.conf` se agregaron permisos:
```apache
<Directory /www_dir>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

```bash
systemctl restart apache2
```

![Apache reiniciado correctamente](images/Pasted_image_20260513133646.png)
![Archivos en /www_dir](images/Pasted_image_20260513133724.png)

### 4.4 y 4.5 — Montaje automático (fstab)

El archivo `/etc/fstab` es una lista que Linux lee **cada vez que arranca** y monta automáticamente los discos listados. Se agregaron las entradas usando los UUID de cada partición:

```bash
echo "UUID=$(blkid -s UUID -o value /dev/sdc1)  /www_dir  ext4  defaults  0  2" >> /etc/fstab
echo "UUID=$(blkid -s UUID -o value /dev/sdc2)  /backup_dir  ext4  defaults  0  2" >> /etc/fstab
mount -a
reboot
```

![fstab con UUIDs correctos](images/Pasted_image_20260513131724.png)

Resultado tras el reboot:

![df -h tras el reboot](images/Pasted_image_20260513131907.png)

### Nota — Archivo de particiones

```bash
cat /proc/partitions > /opt/particion
```

![/opt/particion](images/Pasted_image_20260513133827.png)

---

## ✅ Consigna 5 — Backup

### 5.1 a 5.6 — Script `backup_full.sh`

```bash
mkdir /opt/scripts
nano /opt/scripts/backup_full.sh
chmod +x /opt/scripts/backup_full.sh
```

![Script backup_full.sh](images/Pasted_image_20260513160909.png)

El script:
- Acepta argumentos `<origen>` y `<destino>`
- Incluye opción `-help`
- Valida que los directorios existan antes de ejecutar
- Genera archivos con la fecha en formato ANSI (`YYYYMMDD`), por ejemplo: `log_bkp_20260513.tar.gz`

Prueba de ejecución:

![Ejecución del script](images/Pasted_image_20260513161039.png)

### 5.7 — Cron

```bash
crontab -e
```

![crontab -e](images/Pasted_image_20260513161243.png)

Tareas configuradas:

```
# Todos los días a las 00:00 → backup de /var/log
0 0 * * * /opt/scripts/backup_full.sh /var/log /backup_dir

# Lunes, miércoles y viernes a las 23:00 → backup de /www_dir
0 23 * * 1,3,5 /opt/scripts/backup_full.sh /www_dir /backup_dir
```

![crontab -l verificación](images/Pasted_image_20260513161423.png)

---

## 📁 Estructura del repositorio

```
/
├── README.md
├── images/               ← capturas de evidencia
├── root.tar.gz
├── etc.tar.gz
├── opt.tar.gz
├── www_dir.tar.gz
├── backup_dir.tar.gz
└── var/                  ← spliteado en partes pequeñas
```
