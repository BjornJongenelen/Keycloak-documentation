## How to set up keycloak

first go to the [keycloak](https://www.keycloak.org/downloads) site to donwload the required packages. Once you finished the download you transfer the file to the environment you want to run it on. 

## basic installation of keycloak 
Keycloak uses Java VM, so we need to install a java package preferably Java 11. By running the next command you'll get a list with the available packages. 
```bash 
dnf search java
 ```
 install the packages 
 ```bash 
 dnf -y install java-17-openjdk-headles
 ```
```bash
  dnf clean all 
```
```bash
  dnf java version
```
After installment this should be the result.

![javainstallation](../images/java_Installation.png)
<!-- !!!!!!!!!!!!!!!!KAN EVENTUEEL MEER FOTO'S GERBUIKEN VAN MASTERCLASS OEF. -->

## Deploy keycloak 
Since we already installed Java JDK, we just need to deploy keycloak you can get the latset version form the keycloak website as prevously mentioned. But since we installed Java we kan also download it with the following command. 

```bash
mkdir -p /opt/keycloak
curl https://github.com/keycloak/keycloak/releases/download/19.0.1/keycloak-19.0.1.tar.gz -L | tar --strip-components=1 -zxf - -C /opt/keycloak;
 ```

Create an extra user just in case if you dont want to run keycloak as the root user of your linux system. en dont forget the permissions in an bash commmand it will look like this:

```bash
useradd --shell /bin/sh --uid 1001 keycloak; 
```
```bash
chown keycloak: /opt/keycloak -R; 
```
```bash
 ll /opt/keycloak/ 
```

before we runkeycloak we still have to expose it to the network we're currently using to fix this issue we'll expose the common used ports.
```bash
firewall-cmd --add-port=8080/tcp --permanent; 
firewall-cmd --add-port=8443/tcp --permanent; 
firewall-cmd --add-port=80/tcp --permanent; 
firewall-cmd --add-port=443/tcp --permanent; 
firewall-cmd --reload;
```

start keycloak with the following command, you can always stop it with ```CTR + C ``` : 
```bash
sudo -u keycloak bin/kc.sh start-dev;
```
![keycloak run](../images/keycloak%20run.png)


## Admin console 
After installing and starting keycloak you should be able to navigate to the machine's IP address wich is on the defautl port 8080 in this case it is http://192.168.254.66:8080

![admin console](../images/admin%20console.png)







## generate self signed certificate to enbale https

you can do it with the following command: 
```bash
 keytool -genkeypair -storepass password -storepass passworde -storetype PKCS12 -keyalg RSA -keysize 2048 -dname "CN=server" -alias server -ext "SAN:c=DNS:localhost,IP:127.0. 
 ```

Or you can use: [XCA](https://hohnstaedt.de/xca)
This application is intended for creating and managing X.509 certificates, certificate requests, RSA, DSA and EC private keys, Smartcards and CRLs.
Everything that is needed for a CA is implemented.
All CAs can sign sub-CAs recursively. These certificate chains are shown clearly.
For an easy company-wide use there are customiseable templates that can be used for certificate or request generation.




