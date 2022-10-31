# ayakurume

# 別手順
パッチ済kernelcacheをpreboot内に配置して、iBSS/iBootを送信して起動する方法もあります。  


## [追加手順] sshrdで必要なものをセットアップ
- macos side
```
./img4 -i kernelcache.release.n71 -o kernelcachd -P kc.bpatch -M apticket.der
scp -P {port} kernelcachd root@localhost:/mnt6/{UUID}/System/Library/Caches/com.apple.kernelcaches/kernelcachd
```

## 初回起動前準備
- macos side
```
./gaster pwn
./gaster decrypt iBSS.n71.RELEASE.im4p iBSS.n71.RELEASE.dec
./gaster decrypt iBoot.n71.RELEASE.im4p iBoot.n71.RELEASE.dec
bspatch iBSS.n71.RELEASE.dec pwniBSS.dec iBSS.n71.RELEASE.patch
bspatch iBoot.n71.RELEASE.dec pwniBoot.dec iBoot.n71.RELEASE.patch
./img4 -i pwniBSS.dec -o iBSS.img4 -M apticket.der -A -T ibss
./img4 -i pwniBoot.dec -o iBoot.img4 -M apticket.der -A -T ibec
```
*iBootのboot-argsは`rd=disk0s1s8 serial=3`に設定されています。(`rd=disk0s1s8`は必須、それ以外は必要に応じて改変ok)

## 初回起動
- macos side
```
./gaster pwn
irecovery -f iBSS.img4
irecovery -f iBoot.img4
```

dropbearの起動を確認後
- macos side
```
iproxy {port} 44
```
```
ssh root@localhost -p {port}
scp -P {port} bootstrap-ssh.tar root@localhost:/var/root 
scp -P {port} org.swift.libswift_5.0-electra2_iphoneos-arm.deb root@localhost:/var/root 
scp -P {port} com.ex.substitute_2.3.1_iphoneos-arm.deb root@localhost:/var/root 
scp -P {port} com.saurik.substrate.safemode_0.9.6005_iphoneos-arm.deb root@localhost:/var/root 
```

- ios side
```
mount -uw /
cd /var/root
tar --preserve-permissions --no-overwrite-dir -xvf bootstrap-ssh.tar -C /
/prep_bootstrap.sh
apt update
apt upgrade -y
apt install org.coolstar.sileo
dpkg -i *.deb
rm *.deb
rm bootstrap-ssh.tar
touch /.installed_ayakurume
reboot
```

## 起動
- macos side
```
./gaster pwn
irecovery -f iBSS.img4
irecovery -f iBoot.img4
```
