# Void-Linux-Manual-Gnome
Instalación manual de Void Linux con entorno de escritorio gnome

## Información de los discos
```
lsblk
```
  
## Particionado
+ Suponemos que el disco a utilizar es vda
  
```
cfdisk /dev/vda
```
+ Ejemplo vda1 para boot/efi, vda2 para swap, vda3 para sistema


## Formateo de particiones
```
mkfs.vfat /dev/vda1
mkswap /dev/vda2
mkfs.ext4 /dev/vda3
```

## Montado de particiones
```
mount /dev/vda3 /mnt
mount /dev/vda1 /mnt/boot/efi --mkdir
swapon /dev/vda2
```
## Instalar sistema base en /mnt
```
xbps-install -S -r /mnt -R https://repo-de.voidlinux.org/current base-system grub-x86_64-efi nano
```
## Generar archivo fstab
```
xgenfstab -U /mnt > /mnt/etc/fstab
```
## Hacer chroot a /mnt
```
xchroot /mnt /bin/bash
```
### Configurar sistema
+ Contraseña y shell para root:
```
passwd
``` 
```
chsh -s /bin/bash
```
+ Teclado y locales
```
nano /etc/rc.conf
```
+ Descomentar la línea **KEYMAP=es**
   
```
nano /etc/locale.conf
```
+ Modificar la línea _LANG= ..._ En mi caso queda como **LANG=es_ES.UTF-8**
```
nano /etc/default/libc-locales
```
+ Descomentar la línea **es_ES.UTF-8 UTF-8**

+ Nombre para la máquina
```
nano /etc/hostname
```
+ Establecer zona horaria a **Europa/Madrid**
```
ln -s /usr/share/zoneinfo/Europe/Madrid /etc/localtime
```
+ Crear nuevo usuario y su contraseña
```
useradd -m -s /bin/bash username
passwd username
```
+ Grupos para el usuario, **wheel** para que tenga permisos de root

```
usermod -aG wheel,audio,video,lp,storage,users username
```
+ Editar /etc/sudoers para permitir acciones de root al grupo wheel
```
visudo
```
+ Descomentar la línea  **%wheel ALL=(ALL:ALL) ALL**

**Instalar grub**
```
grub-install /dev/vda1
```
**Último paso: configurar el sistema**
```
xbps-reconfigure -fa
```
**Salir de chroot**
```
exit
```
**Desmontar el sistema de ficheros y reiniciar**
```
umount -R /mnt
reboot
```

## Instalar entorno gráfico Gnome (audio y red)
```
xbps-install -S xorg gnome pipewire wireplumber
```
+ **Configurar pipewire y wireplumber**
```
mkdir -p /etc/pipewire/pipewire.conf.d
ln -s /usr/share/examples/wireplumber/10-wireplumber.conf /etc/pipewire/pipewire.conf.d			
ln -s /usr/share/examples/pipewire/20-pipewire-pulse.conf /etc/pipewire/pipewire.conf.d
```
+ Archivo para autoarranque de pipewire
+ Crear archivo pipewire.desktop en /etc/xdg/autostart
```
cd /etc/xdg/autostart
nano pipewire.desktop
```
+ Añadir lo siguiente:
```
[Desktop Entry]
Type=Application
Name=Pipewire
Exec=pipewire
```
+ Eliminar servicio dhcpcd
```
rm /var/service/dhcpcd
```
+ Activar servicios necesarios para gnome
```
ln -s /etc/sv/dbus /var/service
ln -s /etc/sv/NetworkManager /var/service
ln -s /etc/sc/gdm /var/service
```
+ A los pocos segundos arranca el entorno de escritorio
+ Si todo ha ido bien, ya aparece el símbolo de red y volumen.. (NetworkManager y pipewire están funcionando)
+ Configurar resolución de pantalla (1920x1080 16:9)
+ Configurar teclado español en las opciones de Configuración
+ Cerrar y abrir sesión para que se apliquen los cambios del teclado		
+ Comprobación de pipewire/wireplumber en terminal
```
pactl info
```
```
wpctl status
```
+ Directorios de usuario (Descargas, Documentos ...)
```
sudo xbps-install xdg-user-dirs 
```
```
xdg-user-dirs-update
```
+ Repositorios privativos desde terminal
```
sudo xbps-install -Su void-repo-nonfree void-repo-multilib void-repo-multilib-nonfree
```
+ Actualizar la lista de paquetes
```
sudo xbps-install -Su
```

+ Instalar navegador, suite ofimática y un gestor gráfico de paquetes desde terminal (es solo a modo de ejemplo) 
```
sudo xbps-install -Su firefox libreoffice octoxbps
```

**Referencias consultadas:**
+ [Documentación oficial de Void Linux](https://docs.voidlinux.org/)

