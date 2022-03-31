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

## Instalar el cargador syslinux-efi y configurarlo para que posibilite elegir que operativo arranca al iniciar la máquina



## Instalar el cargador rEFInd y configurarlo para que permita elegir que operativo arranca al iniciar la máquina, o hacer chainload a otro cargador (y mediante un submenú elegir qué cargador)


