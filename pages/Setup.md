## How to set up keycloak

first go to the [keycloak](https://www.keycloak.org/downloads) site to donwload the required packages. Once you finished the download you transfer the file to the environment you want to run it on. 

## basic installation of keycloak 
Keycloak uses Java VM, so we need to install a java package preferably Java 11. By running the next command you'll get a list with the available packages. 
```shell
dnf search java
 ```
 install the packages 
 ```shell 
 dnf -y install java-17-openjdk-headles
 ```
```shell
  dnf clean all 
```
```shell
  dnf java version
```
After installment this should be the result.

![javainstallation](../images/java_Installation.png)
<!-- !!!!!!!!!!!!!!!!KAN EVENTUEEL MEER FOTO'S GERBUIKEN VAN MASTERCLASS OEF. -->

## Database PostgreSQL
PostgreSQL is een vrije relationele databaseserver, uitgegeven onder de PostgreSQL licence, gelijkwaardig aan de flexibele BSD-licentie. Het biedt een alternatief voor zowel opensource-databasemanagementsystemen, zoals MariaDB en Firebird, als voor propriëtaire systemen, zoals Oracle, Oracle MySQL, DB2 en Microsoft SQL Server. PostgreSQL wordt niet beheerd of gecontroleerd door één enkel bedrijf, maar steunt op een wereldwijde gemeenschap van ontwikkelaars en bedrijven. De lijst van medewerkers, van voltijds-ontwikkelaars tot ad-hoc-testers, is al jarenlang ongeveer 300-400 namen lang. 

You can also follow the tutorial of [postgreSQL](https://www.postgresql.org/download/linux/redhat/) itself o install from repositories provided by PostgreSQL team  itself, but we’ll ignore that for now.

!> since we need a database so we can succesfully launch keycloak, first we will configure PostgreSQL, we'll start with the listening address:  

```shell
dnf install postgresql-server postgresql
```
```shell
dnf clean all 
```
begofore configuringthe database. We're are going to launch the PostgreSQL database using the following command: 
```shell
/usr/bin/postgresql-setup --initdb 
```
Which should give the following output. Which confirms that the database is ready to be configured.

```bash 
* Initializing database in '/var/lib/pgsql/data' 
* Initialized, logs are in /var/lib/pgsql/initdb_postgresql.log
```

```shell
sed "s/^#listen_addresses = 'localhost'/listen_addresses = 'localhost,192.168.254.66'/g" 
-i /var/lib/pgsql/data/postgresql.conf; 
``` 

Now that we configured the listen address , we also need to allow remote connections on that servcice, execute the following command for this: 

```shell 
 cat << EOF | sudo tee /var/lib/pgsql/data/pg_hba.conf 
# TYPE  DATABASE        USER            ADDRESS                 METHOD 
 
# "local" is for Unix domain socket connections only 
local   all             all                                     peer 
# IPv4 local connections: 
host    all             all             127.0.0.1/32            md5 
# IPv6 local connections: 
host    all             all             ::1/128                 md5 
 
# Remote Connections 
host    all             all             192.168.254.0/24           md5 
 
# Allow replication connections from localhost, by a user with the 
# replication privilege. 
local   replication     all                                     peer 
host    replication     all             127.0.0.1/32            ident 
host    replication     all             ::1/128                 ident 
EOF
``` 

Ensure that your network range is included in the “Remote Connections” settings. Remember this configuration 
file as you may need to reconfigure it depending on your network setup (routing vs NATing
Finally, we will start PostgreSQL and ensure it will automatically start when we boot our system.We can verify whether PostgreSQL is listening on the *:5432 port using the “show sockets” command: 

```shell
systemctl enable postgresql --now
```
```shell
ss -tlpn; 
```
```shell
systemctl status postgresql; 
```
Which should give the following output: 

![postgres stat](../images/postgresql%20status.png)

now that the postgrSQL is configured we can now acces the shell with: 

```shell
sudo -u postgres psql;
```
When running previous command this should output the following:

```
could not change directory to "/root": Permission denied 
psql (13.7) 
Type "help" for help. 
 
postgres=#
```
next, we'll check out the exisiting schema's 
!> a schema is a namespace that contains named database objects such as tables, views, indexes, data types, functions, stored procedures and operators

```shell
SELECT schema_name FROM information_schema.schemata;
```
Which again should give the following output:

```
    schema_name 
-------------------- 
 pg_toast 
 pg_catalog 
 public 
 information_schema 
(4 rows)
```

If you do not want table layout, we can use expanded layout in “psql” with: 
```shell 
\x 
SELECT schema_name FROM information_schema.schemata; 
```

```
-[ RECORD 1 ]------------------- 
schema_name | pg_toast 
-[ RECORD 2 ]------------------- 
schema_name | pg_catalog 
-[ RECORD 3 ]------------------- 
schema_name | public 
-[ RECORD 4 ]------------------- 
schema_name | information_schema 
```

### firewall 
now it is not necessary, we can configure the firewall so that you can access it remotely 

Although it is not necessary, we can configure the firewall as well: 
firewall-cmd --permanent --add-port=5432/tcp; 
firewall-cmd --reload; 
success 
You will now be able to access it remotely. 
  


<!-- ------------------------------------------------------------------------------- -->

## Deploy keycloak 
Since we already installed Java JDK, we just need to deploy keycloak you can get the latset version form the keycloak website as prevously mentioned. But since we installed Java we kan also download it with the following command. 

```shell
mkdir -p /opt/keycloak
curl https://github.com/keycloak/keycloak/releases/download/19.0.1/keycloak-19.0.1.tar.gz -L | tar --strip-components=1 -zxf - -C /opt/keycloak;
 ```

Create an extra user just in case if you dont want to run keycloak as the root user of your linux system. en dont forget the permissions in an bash commmand it will look like this:

```shell
useradd --shell /bin/sh --uid 1001 keycloak; 
```
```shell
chown keycloak: /opt/keycloak -R; 
```
```shell
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
```shell
sudo -u keycloak bin/kc.sh start-dev;
```
![keycloak run](../images/keycloak%20run.png)

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


## Certificates
### Root CA
Another important part of a network-facing service is the engineer’s capability to understand and configure 
certificates. In this case we will set up a “server certificate” which is used to authenticate our Keycloak network 
service and encrypt all traffic in transit. 
The  authentication portion is  based on  the already-established trust in your system’s keystore (or Mozilla 
Firefox’ keystores in case you’re using that browser). Where the certificate of the server is signed by a party 
(Certificate Authority) that our system, or we, trust. Based on this delegated trust, we can verify the authenticity 
of the service presenting itself. 
The encryption part happens after the client, our browser, has verified the server’s identity and establishes 
session keys to encrypt further communication. For an in-depth explanation how this protocol works, refer to 
an online article, e.g.: Cloudflare. 
For now, we will generate a certificate authority using XCA. Download and install the program. 


