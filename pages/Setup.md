## how to set up forgerock

first go to the [keycloak](https://www.keycloak.org/downloads) site to donwload the required packages. Once you finished the download you transfer the file to the environment you want to run it on. 

- tranfser file 
with ifle zilla 
- unzip de file in
- generate self signed certificate to enbale https

you can do it with the following command: 
```bash
 keytool -genkeypair -storepass password -storepass passworde -storetype PKCS12 -keyalg RSA -keysize 2048 -dname "CN=server" -alias server -ext "SAN:c=DNS:localhost,IP:127.0. 
 ```

or you can use: CXA

