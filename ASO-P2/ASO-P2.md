# Práctica 2: Máquina Virtual 2

## Instalar los SO

[:exclamation:] Habilitar EFI antes de arrancar la máquina.

### Ubuntu Server

- Particionado
    - Disco principal [32 GB]
        - Partición */* (12 GB)
        - Partición *swap* (2 GB)
        - Partición */var* (2 GB)
    - Disco secundario [6 GB]
        - Partición */home* (1.5 GB)

```
FILE SYSTEM SUMMARY

  MOUNT POINT   SIZE    TYPE        DEVICE TYPE
[ /            12.000G  new ext4    new partition of local disk ► ]
[ /boot/efi     1.049G  new fat32   new partition of local disk ► ]
[ /home         1.500G  new ext4    new partition of local disk ► ]
[ /var          2.000G  new ext4    new partition of local disk ► ]
[ SWAP          2.000G  new swap    new partition of local disk ► ]


AVAILABLE DEVICES

  DEVICE                                                            TYPE           SIZE
[ Disco principal                                                   local disk    32.000G ► ]
  free space                                                                      14.948G ►

[ Disco secundario                                                  local disk     6.000G ► ]
  free space                                                                       4.498G ►


USED DEVICES

  DEVICE                                                            TYPE           SIZE
[ Disco principal                                                   local disk    32.000G ► ]
  partition 1 new, primary ESP, to be formatted as fat32, mounted at /boot/efi     1.049G ►
  partition 2 new, to be formatted as ext4, mounted at /                          12.000G ►
  partition 3 new, to be formatted as swap                                         2.000G ►
  partition 4 new, to be formatted as ext4, mounted at /var                        2.000G ►

[ Disco secundario                                                  local disk     6.000G ► ]
  partition 1 new, to be formatted as ext4, mounted at /home                       1.500G ►
```

- Perfil
    - Your name: `maquina2` (user full name)
    - Your server's name: `ubuntuserver` (hostname)
    - Pick a username: `usuario`

### Fedora MATE

- Teclado: *Spanish (Windows)*
- Particionado
    - Disco principal (sda) [32 GB]
        - Editar sda1 (Set Mountpoint: /boot/efi)
        - Añadir partición */* (12 GB)
        - Añadir partición */var* (2 GB)
    - Disco secundario (sdb) [6 GB]
        - Añadir partición */home* (1.5 GB)

### Ambos S.O. compartirán *swap*

Si hemos seguido los pasos, Fedora usará automáticamente el *swap* de Ubuntu.

### Crear dos *menuentry* MUY SIMPLES en cada uno de los dos grubs (`/etc/grub.d/40_custom`) que haga chainload al otro grub y que arranque directamente el otro operativo

#### Ubuntu Server

```
menuentry "GRUB Fedora MATE" {
    set root=(hd0,1)
    chainloader /EFI/fedora/grubx64.efi
}
```

- Actualizamos el GRUB: `grub-mkconfig -o /boot/grub/grub.cfg`

#### Fedora MATE

```
menuentry "GRUB Ubuntu Server" {
    set root=(hd0,1)
    chainloader /EFI/ubuntu/grubx64.efi
}
```

- Actualizamos el GRUB2: `grub2-mkconfig -o /boot/grub2/grub.cfg`

## Instalar el cargador syslinux-efi y configurarlo para que posibilite elegir que operativo arranca al iniciar la máquina

- Descargamos el siguiente archivo: [x86-64 binary tarball of SYSLINUX 6.03+dfsg-14](https://www.rodsbooks.com/efi-bootloaders/syslinux-6.0.3+dfsg-14.tgz)
- Creamos el directorio `kernel/` tanto para Ubuntu como para Fedora en sus respectivas carpetas del EFI (sda1) (partición montada en `/boot/efi/`)
    - `/boot/efi/EFI/ubuntu/kernel/`
    - `/boot/efi/EFI/fedora/kernel/`
- Copiamos los kernel y los initrd de los 2 sistemas operativos en sus respectivas carpetas:
    - Ubuntu Server
        - Kernel: `vmlinuz`
        - Kernel (Generic): `vmlinuz-5.4.0-107-generic` > `vmlinuz-generic`
        - Initrd: `initrd.img`
        - Initrd (Generic): `initrd.img-5.4.0-107-generic` > `initrd.img-generic`
    - Fedora MATE
        - Kernel: `vmlinuz-5.14.10-300.fc35.x86_64` > `vmlinuz`
        - Kernel (Rescue Mode): `vmlinuz-0-rescue-*` > `vmlinuz-rescue`
        - Initrd: `initramfs-5.14.10-300.fc35.x86_64.img` > `initramfs.img`
        - Initrd (Rescue Mode): `initramfs-0-rescue-*` > `initramfs-rescue.img`
- Editamos el archivo `syslinux.cfg` que acabamos de descargar

```
DEFAULT ubuntu
TIMEOUT 50
UI menu.c32

LABEL ubuntu
    MENU LABEL Ubuntu Server
    LINUX /EFI/ubuntu/kernel/vmlinuz
    APPEND root=/dev/sda2 ro quiet splash
    INITRD /EFI/ubuntu/kernel/initrd.img

LABEL ubuntu-generic
    MENU LABEL Ubuntu Server (Generic)
    LINUX /EFI/ubuntu/kernel/vmlinuz-generic
    APPEND root=/dev/sda2 ro quiet splash
    INITRD /EFI/ubuntu/kernel/initrd.img-generic

LABEL fedora
    MENU LABEL Fedora MATE
    LINUX /EFI/fedora/kernel/vmlinuz
    APPEND root=/dev/sda5 ro quiet splash
    INITRD /EFI/fedora/kernel/initramfs.img

LABEL fedora-rescue
    MENU LABEL Fedora MATE (Rescue Mode)
    LINUX /EFI/fedora/kernel/vmlinuz-rescue
    APPEND root=/dev/sda5 ro quiet splash
    INITRD /EFI/fedora/kernel/initramfs-rescue.img
```

- Copiamos el directorio `syslinux/` en `/boot/efi/EFI/`
- Ejecutamos `efibootmgr -c -l '/EFI/syslinux/syslinux.efi' -L SYSLINUX -p 1` para utilizar syslinux-efi como cargador en el arranque

## Instalar el cargador rEFInd y configurarlo para que permita elegir que operativo arranca al iniciar la máquina, o hacer chainload a otro cargador (y mediante un submenú elegir qué cargador)

- Descargamos el siguiente archivo: [refind-0.13.3.1-1.x86_64.rpm](https://sourceforge.net/projects/refind/files/0.13.3.1/refind-0.13.3.1-1.x86_64.rpm/download)
- Realizamos la instalación con `rpm -Uvh refind-0.13.3.1-1.x86_64.rpm`
- Editamos el archivo `/boot/efi/EFI/refind/refind.conf`

```
enable_mouse

scan_all_linux_kernels false

menuentry "Ubuntu Server" {
    icon /EFI/refind/icons/os_ubuntu.png
    loader /EFI/ubuntu/kernel/vmlinuz
    initrd /EFI/ubuntu/kernel/initrd.img
    options "root=/dev/sda2 ro quiet splash"
    submenuentry "Boot Generic Mode" {
        loader /EFI/ubuntu/kernel/vmlinuz-generic
        initrd /EFI/ubuntu/kernel/initrd.img-generic
    }
}

menuentry "Fedora MATE" {
    icon /EFI/refind/icons/os_fedora.png
    loader /EFI/fedora/kernel/vmlinuz
    initrd /EFI/fedora/kernel/initramfs.img
    options "root=/dev/sda5 ro quiet splash"
    submenuentry "Boot Rescue Mode" {
        loader /EFI/fedora/kernel/vmlinuz-rescue
        initrd /EFI/fedora/kernel/initramfs-rescue.img
    }
}

menuentry "Loaders" {
    loader /EFI/refind/refind_x64.efi
    submenuentry "SYSLINUX" {
        loader /EFI/syslinux/syslinux.efi
    }
    submenuentry "GRUB Ubuntu Server" {
        loader /EFI/ubuntu/grubx64.efi
    }
    submenuentry "GRUB Fedora MATE" {
        loader /EFI/fedora/grubx64.efi
    }
}
```

- Eliminar entrada rEFInd: `Del` o `Supr`
- Abrir submenú: `F2`
