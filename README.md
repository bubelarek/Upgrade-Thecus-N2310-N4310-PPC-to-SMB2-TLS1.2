# Upgrades Thecus N2310 and possibly also N4310

Some updates to this nice fast and power efficient server.

## Install SAMBA with SMB2 support

This install new version of Samba with SMB2 support

``` bash
export URL=http://dl.fedoraproject.org/pub/archive/fedora-secondary/updates/17/ppc
export SUFIX=-3.6.12-1.fc17.1.ppc.rpm
wget ${URL}/libwbclient${SUFIX} -O libwbclient.rpm --no-check-certificate
wget ${URL}/samba${SUFIX} -O samba.rpm --no-check-certificate
wget ${URL}/samba-client${SUFIX} -O samba-client.rpm --no-check-certificate
wget ${URL}/samba-common${SUFIX} -O samba-common.rpm --no-check-certificate
wget ${URL}/samba-winbind${SUFIX} -O samba-winbind.rpm --no-check-certificate
wget ${URL}/samba-winbind-clients${SUFIX} -O samba-winbind-clients.rpm --no-check-certificate
yum erase libwbclient samba samba-client samba-common samba-winbind samba-winbind-clients  samba-winbind-32bit
yum localinstall libwbclient.rpm samba.rpm samba-client.rpm samba-common.rpm samba-winbind.rpm samba-winbind-clients.rpm

## this adjust bash script which generate Samba config each reboot / time there is changes in NAS www setting panel
sed -i '/## block size/c\## SMB 2 \n echo "max protocol = SMB2" >> ${Confile} \n## block size' /img/bin/smbdb.sh
```

## Add Fedora repositories

This will fix yum package manager and let install any software we like. It will be used to update samba , openssl.
Some packages can be downloads from Fedora 18, 19, 20 updates channel but adding this to repos will make some problems, so better to download rpm files and localinstall them.

``` bash

rm /etc/yum.repos.d/*.repo -f

cat > /etc/yum.repos.d/release_16.repo<< EOF
[release_16]
name= release fedora 16
baseurl=http://dl.fedoraproject.org/pub/archive/fedora-secondary/releases/16/Fedora/ppc/os
gpgcheck=0
enabled=1
EOF

cat > /etc/yum.repos.d/updates_16.repo<< EOF
[updates_16]
name= release fedora 16
baseurl=http://dl.fedoraproject.org/pub/archive/fedora-secondary/updates/16/ppc/
gpgcheck=0
nagle=1
EOF	
```

## Fix ability to used HTTPs 
To use HTTPS/AWS API operating system need use TLS 1.2 and have updated CA certificate 

### Fix expired root CA certificate 
``` bash
# Clean old certificates
rm /etc/pki/tls/certs/ca-bundle.crt -f
rm /etc/pki/CA/certs/ca-bundle.crt -f
rm /etc/ssl/certs/ca-bundle.crt -f
# Download new root certificate
wget https://raw.githubusercontent.com/bagder/ca-bundle/refs/heads/master/ca-bundle.crt -O /etc/pki/tls/certs/ca-bundle.crt --no-check-certificate
# copy to location used by other liblaries 
cp  /etc/pki/tls/certs/ca-bundle.crt  /etc/pki/CA/certs/ca-bundle.crt
cp  /etc/pki/tls/certs/ca-bundle.crt   /etc/ssl/certs/ca-bundle.crt

# verify
openssl s_client -connect www.google.com:443 -tls1_2

 ```

### Add support for TLS 1.2 

``` bash
export URL=https://dl.fedoraproject.org/pub/archive/fedora-secondary/updates/18/ppc
export SUFIX=.fc18.ppc.rpm
wget ${URL}/openssl-libs-1.0.1e-36${SUFIX} -O openssl-libs-1.0.1e-36.rpm --no-check-certificate
wget ${URL}/openssl-1.0.1e-36${SUFIX} -O openssl-1.0.1e-36.rpm --no-check-certificate
wget ${URL}/make-3.82-15${SUFIX}  -O make-3.82-15.rpm --no-check-certificate
yum localinstall openssl-libs-1.0.1e-36.rpm openssl-1.0.1e-36.rpm make-3.82-15.rpm
# verify
openssl s_client -connect www.google.com:443 -tls1_2
```

### Add twonky media server 

``` bash
mkdir /usr/bin/twonky
cd  /usr/bin/twonky
wget http://download.twonky.com/8.5.2/twonky-powerpc-glibc-2.2.5-8.5.2.zip  --no-check-certificate
unzip twonky-powerpc-glibc-2.2.5-8.5.2.zip


cat >> /usr/bin/mediaserver<< EOF
cd  /usr/bin/twonky
/usr/bin/twonky/mediaserver
EOF


chmox +x /usr/bin/mediaserver

cat >> /img/bin/check_twonky<< EOF
echo "Twonky instaled"
EOF

chmod +x /img/bin/check_twonky

sed -i '/twonky=0/c\twonky=1' /img/bin/conf/sysconf.N2310.txt
sed -i '/media=0/c\media=1' /img/bin/conf/sysconf.N2310.txt

```
