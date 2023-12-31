# Deploy nexus registry on ubuntu 22.04

- create directory
```
mkdir /etc/nexus
mkdir /data-nexus
```

- update package and install package tambahan
```
apt-get update && apt-get upgrade -y --with-new-pkgs
apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release git software-properties-common sysstat htop net-tools bash-completion openjdk-8-jdk
```

- download binary & install nexus
> untuk version terbaru dari nexus bisa dilihat [disini](https://help.sonatype.com/repomanager3/product-information/download)

```
wget https://download.sonatype.com/nexus/3/nexus-3.63.0-01-unix.tar.gz
tar xzf nexus-3.63.0-01-unix.tar.gz -C /etc/nexus/ --strip-components 1 && mv /etc/nexus/nexus3/* /data-nexus && rm -rf /etc/nexus/nexus3 && rm nexus-3.63.0-01-unix.tar.gz
```

- create user
> user ini yang akan digunakan untuk nexus
```
useradd --system --no-create-home nexus
chown -R nexus:nexus /data-nexus
chown -R nexus:nexus /etc/nexus
```

- create systemd to nexus
```
cat << 'EOF' > /etc/systemd/system/nexus.service
[Unit]
Description=nexus service
After=network.target
  
[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/etc/nexus/bin/nexus start
ExecStop=/etc/nexus/bin/nexus stop
User=nexus
Restart=on-abort
TimeoutSec=600
  
[Install]
WantedBy=multi-user.target
EOF
```

- edit file /etc/nexus/bin/nexus.vmoptions

```
nano /etc/nexus/bin/nexus.vmoptions
```

```
-Xms2703m
-Xmx2703m
-XX:MaxDirectMemorySize=2703m
-XX:+UnlockDiagnosticVMOptions
-XX:+LogVMOutput
-XX:LogFile=/data/nexus/log/jvm.log
-XX:-OmitStackTraceInFastThrow
-Djava.net.preferIPv4Stack=true
-Dkaraf.home=.
-Dkaraf.base=.
-Dkaraf.etc=etc/karaf
-Djava.util.logging.config.file=etc/karaf/java.util.logging.properties
-Dkaraf.data=/data/nexus
-Dkaraf.log=/data/nexus/log
-Djava.io.tmpdir=/data/nexus/tmp
-Dkaraf.startLocalConsole=false
-Djdk.tls.ephemeralDHKeySize=2048
-Djava.util.prefs.userRoot=/etc/nexus/lock
#
# additional vmoptions needed for Java9+
#
# --add-reads=java.xml=java.logging
# --add-exports=java.base/org.apache.karaf.specs.locator=java.xml,ALL-UNNAMED
# --patch-module java.base=${KARAF_HOME}/lib/endorsed/org.apache.karaf.specs.locator-4.3.6.jar
# --patch-module java.xml=${KARAF_HOME}/lib/endorsed/org.apache.karaf.specs.java.xml-4.3.6.jar
# --add-opens java.base/java.security=ALL-UNNAMED
# --add-opens java.base/java.net=ALL-UNNAMED
# --add-opens java.base/java.lang=ALL-UNNAMED
# --add-opens java.base/java.util=ALL-UNNAMED
# --add-opens java.naming/javax.naming.spi=ALL-UNNAMED
# --add-opens java.rmi/sun.rmi.transport.tcp=ALL-UNNAMED
# --add-exports=java.base/sun.net.www.protocol.http=ALL-UNNAMED
# --add-exports=java.base/sun.net.www.protocol.https=ALL-UNNAMED
# --add-exports=java.base/sun.net.www.protocol.jar=ALL-UNNAMED
# --add-exports=jdk.xml.dom/org.w3c.dom.html=ALL-UNNAMED
# --add-exports=jdk.naming.rmi/com.sun.jndi.url.rmi=ALL-UNNAMED
# --add-exports java.security.sasl/com.sun.security.sasl=ALL-UNNAMED
#
# comment out this vmoption when using Java9+
#
-Djava.endorsed.dirs=lib/endorsed
```
![image](https://github.com/galihtw04/nexus-registry/assets/96242740/d0ab3d6a-b848-41cd-8776-6c19052234aa)

- create new directory
```
mkdir -p /etc/nexus/lock
```

- start nexuus
```
systemctl start nexus
systemctl enable nexus
systemctl status nexus
```
![image](https://github.com/galihtw04/nexus-registry/assets/96242740/d49844ee-e5ed-4042-a683-65464228675e)

# # enabling https nexus

- create certificate

```
mkdir ~/cert-nexus && cd ~/cert-nexus
```

- create certificate root
```
openssl genrsa -out root.key 2048
openssl req -x509 -new -nodes -key root.key -out root.crt -sha256 -days 3650
```
![image](https://github.com/galihtw04/nexus-registry/assets/96242740/5caa1836-05b7-4893-8469-8870a53219ec)

- create certificate server
```
openssl genrsa -out nexus.key 2048
openssl req -new -key nexus.key -out nexus.csr 
```
![image](https://github.com/galihtw04/nexus-registry/assets/96242740/a44ad23d-71f7-4825-aa0f-6d49ac90eed8)

- create extension
```
cat << 'EOF' > nexus.txt
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = registry-nexus.cloud
IP.1 = 192.168.210.2
EOF
```
- signature certificate server
```
# tanda tangan sertifikat server oleh sertificate root
openssl x509 -req -in nexus.csr -CA root.crt -CAkey root.key -CAcreateserial -out nexus.crt -days 365 -sha256 -extfile nexus.txt
```

- check certificate root and server
```
openssl x509 -in root.crt -text -noout
openssl x509 -in root.crt -enddate -noout
# server
openssl x509 -in nexus.crt -text -noout
openssl x509 -in nexus.crt -enddate -noout
```
![image](https://github.com/galihtw04/nexus-registry/assets/96242740/85d8840c-37fe-4652-9ad1-53a754741e13)
![image](https://github.com/galihtw04/nexus-registry/assets/96242740/4cdacb03-8300-4633-8aa7-f1a2703aa76e)

![image](https://github.com/galihtw04/nexus-registry/assets/96242740/ddf88e59-2dea-490b-b33b-bab586229fb1)
![image](https://github.com/galihtw04/nexus-registry/assets/96242740/7435c4fc-5fdf-4ee1-8869-2630607ba3ec)

convert certificate server to PKCS12
```
keytool -importkeystore -srckeystore nexus.p12 -srcstoretype PKCS12 \ 
-srcstorepass password -alias admin -deststorepass password \ 
-destkeypass password -destkeystore /etc/nexus/etc/ssl/keystore.jks 
 
keytool -importkeystore -srckeystore /etc/nexus/etc/ssl/keystore.jks \ 
-srcstorepass password -destkeystore /etc/nexus/etc/ssl/keystore.jks \ 
-deststoretype pkcs12
```
![image](https://github.com/galihtw04/nexus-registry/assets/96242740/8a994e06-152e-4cf2-b23d-94bf542ba72f)

```
chown -R nexus:nexus /etc/nexus/
```

- edit file /etc/nexus/etc/nexus-default.properties to enable https

```
nano /etc/nexus/etc/nexus-default.properties
```

```
---
application-port-ssl=8443
nexus-args=${jetty.etc}/jetty.xml,${jetty.etc}/jetty-http.xml,${jetty.etc}/jetty-https.xml,${jetty.etc}/jetty-requestlog.xml
ssl.etc=/etc/nexus/etc/ssl
---

```
![image](https://github.com/galihtw04/nexus-registry/assets/96242740/6d7d3913-7fb0-42b9-b65b-0e32de6f5a81)


- restart neuxs
```
systemctl restart nexus
```

access nexus
![image](https://github.com/galihtw04/nexus-registry/assets/96242740/29f6a1ed-7da2-4a46-826b-f54a5edaafd0)
> untuk sigin user admin password bisa di cek /data-nexus/admin.password
