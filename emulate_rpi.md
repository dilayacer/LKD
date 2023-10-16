İmaj dosyasının içeriği farklı partitionlara ayrılabilir.
Raspberry Pi'de Raspbian 2 tane partition içerir. boot, rootfs

Boot: Rp'nin önyükleme dosyalarını içerir. Kernel, config dosyaları bulunur.
FAT dosya sistemi kullanır.

Rootfs:Rp'nin işletim sistemini ve kullanıcı verilerini içerir.
 
İmaj dosyası indirilir -> https://www.raspberrypi.com/software/
2023-05-03-raspios-bullseye-arm64-lite.img
İndirilen dosya açılır.
```
command:
xz -d 2023-05-03-raspios-bullseye-arm64-lite.img.xz 
```

İndirilen imaj dosyasının disk bölümlerini incelemek, görüntülemek için "fdisk -l" komutu kullanılır.
```
fdisk -l 2023-05-03-raspios-bullseye-arm64-lite.img 

output:
Disk 2023-05-03-raspios-bullseye-arm64-lite.img: 16 GiB, 17179869184 bytes, 33554432 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x544c6228

Device                                      Boot  Start     End Sectors  Size Id Type
2023-05-03-raspios-bullseye-arm64-lite.img1        8192  532479  524288  256M  c W95 
2023-05-03-raspios-bullseye-arm64-lite.img2      532480 4104191 3571712  1,7G 83 Linu
```
Output'a bakıldığında boot ve rootfs dosyaları görüntülenir.
Start: Bölümün başladığı fiziksel sektör numarası
End: Bölümün bittiği fiziksel sektör numarası

Start numaraları offset eklemek için önemlidir. 1 sektör 512 byte olarak görülür.
offset= start number * 512 bytes

İmaj bölümlerinin alt bölümlerinde değişiklik yapabilmek için losetup komutu kullanılır.

```
sudo losetup -Pf 2023-05-03-raspios-bullseye-arm64-lite.img 
```
Alt bölümlerde değişiklik yapmak için mount işlemi yapacağız. 
Bunun için /mnt altına loop oluşturulur. 
Hangi loopta olduğunu görüntülemek için aşağıdaki komut kullanılır.

```
lsblk

```
mount komutu ile bu dosya bağlanır.

```
sudo mkdir /mnt/boot
sudo mount /dev/loop11p1 /mnt/boot
sudo mkdir /mnt/rootfs
sudo mount -o loop11p2 /mnt/rootfs
```
boot dosyası içinden kernel8.img ve bcm2710-rpi-3-b-plus.dtb dosyaları kopyalanır.

```
cp  kernel8.img  bcm2710-rpi-3-b-plus.dtb /home/disk imajının olduğu komum
```
cmdline.txt dosyasının içine gerekli ip configleri yazılır.

```
ip=42.42.42.8::42.42.42.1:255.255.255.0::eth0:off
```
Gerekli işlemler yapıldıktan sonra umount ile bağlar kaldırılır.

İmaj dosyasının bulunduğu dizine qemu da çalıştırmak için boot.sh dosyası oluşturulur.

```
touch boot.sh
nano boot.sh
```

boot.sh içine aşağıdaki script yazılır.
```
qemu-system-aarch64 \
    -m 1024 \
    -M raspi3b \
    -kernel kernel8.img \
    -dtb bcm2710-rpi-3-b-plus.dtb \
    -sd 2023-05-03-raspios-bullseye-arm64-lite.img \
    -append "console=ttyAMA0 root=/dev/mmcblk0p2 rw rootwait rootfstype=ext4" \
    -device usb-net,netdev=net0 \
    -netdev user,id=net0,hostfwd=tcp::2222-:22 \
    -usb -device usb-mouse -device usb-kbd \
    -nographic \
```
```
./boot.sh
```
 komutu ile çalıştırılır.

:)
