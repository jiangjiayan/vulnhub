# Configuration de machines vulnérables VulnHub sur MacBook M1 (configuration d'images x86 ova sur Mac ARM)

Utilisation de la machine virtuelle UTM (uniquement pour Mac)   

L’address :  https://mac.getutm.app/

1.Téléchargez l'image du laboratoire sur Vulnhub. Après le téléchargement, vous obtiendrez un fichier avec l'extension .ova. Prenons l'exemple de Earth.ova.

2.Ensuite, placez cette image dans un système Linux pour un traitement ultérieur (vous pouvez également le faire directement sur Mac en utilisant brew pour installer qemu-img).


```bash
tar -xvf Earth.ova
```
3.Après la décompression, vous obtiendrez un disque virtuel au format vmdk. Ensuite, vous devez utiliser l'outil qemu pour le convertir au format qcow2.

```bash
# Debian ou Ubuntu
sudo apt-get install qemu-utils

# CentOS, Fedora ou RHEL
sudo yum install qemu-img

qemu-img convert -O qcow2 Earth_dev-disk001.vmdk Earth.qcow2
```

4.Après quelques instants, le disque de l'image sera converti.
