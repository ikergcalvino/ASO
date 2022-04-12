# Práctica 1: Máquina Virtual 1

## 0. Instalar los SO

### Solaris 10
- Instalación
    - [x] Networked
    - [x] DHCP
    - [ ] IPv6
    - [ ] Kerberos
    - Name Service : none
- Particionado
    - Partición de 18454 MB (18 GB + 22 MB de redondeo)
    - Modify VTOC

| Disk/File System | Size(MB) | Size(Cyls) |
| :--------------- | -------: | ---------: |
| c0t0d0           |
| /                | 13312 MB |       1697 |
| swap             |  2048 MB |        261 |
| /var             |  2048 MB |        261 |
| /export/home     |  1024 MB |        131 |

### FreeBSD 13
- Instalación

```
[*] base-dbg    Base system (Debugging)
[*] kernel-dbg  Kernel (Debugging)
[*] lib32-dbg   32-bit compatibility libraries (Debugging)
[*] lib32       32-bit compatibility libraries
[*] ports       Ports tree
[*] src         System source tree
[*] tests       Test suite
```

- Particionado
    - Auto (UFS) -> Partition

```
ada0            64 GB   MBR
  ada0s1        18 GB   !191
  ada0s2        16 GB   BSD
    ada0s2a     11 GB   freebsd-ufs     /
    ada0s2b     2.0 GB  freebsd-swap    none
    ada0s2d     2.0 GB  freebsd-ufs     /var
    ada0s2e     1.0 GB  freebsd-ufs     /home
```

```
[*] local_unbound   Local caching validating resolver
[*] sshd            Secure shell daemon
[*] moused          PS/2 mouse pointer on console
[*] ntpdate         Synchronize system and network time at bootime
[*] ntpd            Synchronize system and network time
[*] powerd          Adjust CPU frequency dynamically if supported
[*] dumpdev         Enable kernal crash dumps to /var/crash
```

```
[*] 0 hide_uids       Hide processes running as other users
[*] 1 hide_gids       Hide processes running as other groups
[*] 2 hide_jail       Hide processes running in jails
[ ] 3 read_msgbuf     Disable reading kernel message buffer for unprivil
[ ] 4 proc_debug      Disable process debugging facilities for unprivile
[ ] 5 random_pid      Randomize the PID of newly created processes
[*] 6 clear_tmp       Clean the /tmp filesystem on system startup
[*] 7 disable_syslogd Disable opening Syslogd network socket (disables r
[*] 8 disable_sendmailDisable Sendmail service
[*] 9 secure_console  Enable console password prompt
[ ] 10 disable_ddtraceDisallow DTrace destructive-mode
```

### OpenBSD 7.0
- Instalación
    - Instalación por defecto + X Window System
- Particionado
    - Editamos las particiones (`fdisk`)
    - Editamos la partición 3 con `e 2`
        - ID: A6 (OpenBSD)
        - Start: 4443 [inicio del cilindro] (final de la anterior partición + 1)
        - End: 6545 [final del cilindro] (nº de cilindros aprox.)
    - Custom layout

|  #  | size | mount      |
| :-- | ---: | :--------- |
|  a  | 9 GB |  /         |
|  b  | 2 GB | *swap*     |
|  c  |  -   | *(unused)* |
|  d  | 2 GB |  /var      |
|  e  | 1 GB |  /home     |

### Devuan Linux 4.0
- Instalación
    - Instalación por defecto
- Particionado
    - Partición manual
        - Partición 4: primaria y entera (≈14.9 GB)

```
[*] Devuan desktop enviroment
[*] ... Xfce
[ ] ... MATE
[ ] ... Cinnamon
[ ] ... KDE
[ ] ... LXQt
[ ] ... GNOME Flashback
[ ] web server
[*] Console productivity
[*] SSH server
[*] standard system utilities
```

Init system to install: sysvinit

**[:exclamation:] ¡ATENCIÓN! [:exclamation:]**

Instalación del GRUB en `/dev/sda4`

## 1. Comprobar que cambiando la partición activa cambia el SO que arranca

### Solaris

``` shell
format  # 0 (Specify disk)
fdisk   # 2 (Specify the active partition) + n (Select partition)
    # n = nº de partición que queremos que sea activa
reboot
```

### FreeBSD

``` shell
gpart show ada0
gpart set -a active -i n ada0
    # n = nº de partición que queremos que sea activa
reboot
```

### OpenBSD

``` shell
fdisk -e sd0
flag n  # n = nº de partición que queremos que sea activa
reboot
```

### Devuan

``` shell
fdisk -l
fdisk /dev/sda  # usamos 'a' para activar y desactivar la bootable flag (*) + n
    # n = nº de partición que queremos que sea activa
reboot
```

## 2. Comprobar que si un cargador está en el MasterBoot arranca ese cargador independientemente de la partición activa

### Comprobarlo instalando el GRUB de Solaris y/o el GRUB de Linux en el MasterBoot. Después volverlo a poner al inicio de la partición

- Ambos GRUBs ya han sido instalados en el apartado 0

### Solaris

- Ejecutamos `bootadm list-menu` para mostrar la ubicación del menú de GRUB activo y las entradas del menú de GRUB actuales
- Modificamos el archivo `/boot/grub/menu.lst`

```
title FreeBSD 13
    rootnoverify (hd0,1)
    chainloader +1

title OpenBSD 7.0
    rootnoverify (hd0,2)
    chainloader +1

title Devuan Linux 4.0
    rootnoverify (hd0,3)
    chainloader +1
```

### Devuan

- Modificamos el archivo `/etc/grub.d/40_custom`

```
menuentry "Solaris 10" {
    set root=(hd0,1)
    chainloader +1
}

menuentry "FreeBSD 13" {
    set root=(hd0,2)
    chainloader +1
}

menuentry "OpenBSD 7.0" {
    set root=(hd0,3)
    chainloader +1
}
```

- Ejecutamos `update-grub` para actualizar las entradas del GRUB

## 3. Instalar *boot0* (desde FreeBSD) para poder seleccionar que SO arranca al iniciar la máquina

``` shell
boot0cfg -B boot0
```

## 4. Si no se ha creado usuario durante la instalación del SO, añadir un usuario a Solaris 10 usando *Solaris Managment Console* (`smc`)

``` shell
useradd -c 'usuario' -m -d /export/home/usuario -s /bin/bash usuario
passwd usuario
```

## 5. Añadir swap (2GB) a Devuan Linux en un fichero

- Ejecutamos `swapon –s` o `free –m` para comprobar la información de la memoria swap
- Creamos un archivo swap
    - Método tradicional: `dd if=/dev/zero of=/swapfile bs=1G count=2`
    - Método rápido: `fallocate –l 2G /swapfile`
    - Podemos comprobar que se ha creado bien con: `ls -lh /swapfile`
- Habilitamos el archivo swap

``` shell
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

- Lo hacemos permanente modificando el archivo `/etc/fstab`

```
/swapfile    none    swap    sw    0    0
```
