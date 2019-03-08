# Obtain SSL certificate

[https://gethttpsforfree.com/](https://gethttpsforfree.com/)

# Or self signed SSL certificate


# Common file related certificate gethttpsforfree.com

<i>If you will have the following <b>PEM</b>-encoded files:</i>

- cert.pem:   Your domain's certificate
- chain.pem:   The Let's Encrypt chain certificate
- fullchain.pem:   cert.pem and chain.pem combined
- privkey.pem:   Your certificate's private key


# Common file related to jboss and ssl certificate

- account.key 
- domain.key
- intermediate.crt
- domain.crt
- domain.p12
- domain.jks

NOTE: 																							   <br>
<i><b>The password of the P12-Certifcate and the password of the Keystore has to be the same.</b>  <br>
To avoid: Unable to start service Caused by: java.security.UnrecoverableKeyException: Cannot recover key </i>

# Create PKCS12 keystore from private key and public certificate.
```openssl pkcs12 -export -name domain -in domain.crt -inkey domain.key -out domain.p12```

Enter Export Password: ... 				<br>
Verifying - Enter Export Password: ...  <br>

# Convert PKCS12 keystore into a JKS keystore
```keytool -importkeystore -srckeystore domain.p12 -srcstoretype pkcs12 -destkeystore domain.jks -deststoretype JKS```

<i>Immettere la password del keystore di destinazione: ...  				  <br>
Immettere nuovamente la nuova password: ...  								  <br>
Immettere la password del keystore di origine: ...  						  <br>
La voce dell' alias domain è stata importata.  								  <br>
Importazione completata:  1 voci importate, 0 voci non importate o annullate  <br></i>

# Import intermediate certificate
```keytool -import -trustcacerts -alias intermediate -file intermediate.crt -keystore domain.jks```

<i>Immettere la password del keystore:  			   <br>
Considerare attendibile questo certificato? [no]:  si  <br>
Il certificato è stato aggiunto al keystore  		   <br></i>

# Import domain certificate
```keytool -import -trustcacerts -alias domain -file domain.crt -keystore domain.jks```

<i>Immettere la password del keystore:  				<br>
Considerare attendibile questo certificato? [no]:  si   <br>
Il certificato è stato aggiunto al keystore  			<br></i>

# Verify the contents of the JKS
```keytool -list -v -keystore domain.jks```

<i>Immettere la password del keystore:  <br>

Tipo keystore: JKS 				<br>
Provider keystore: SUN 			<br>
Il keystore contiene 2 entry 	<br></i>

# Editing configuration/standalone.xml

```
 <security-realms>
	<security-realm name="SslRealm">
	  <server-identities>
		<ssl>
		  <keystore path="keystore.jks" relative-to="jboss.server.config.dir" keystore-password="changeme"/>
		</ssl>
	  </server-identities>
	</security-realm>
</security-realms>
```

# Adding listener for https by editing standalone.xml

```
<subsystem xmlns="urn:jboss:domain:undertow:1.2">
	<buffer-cache name="default"/>
	<server name="default-server">
		<http-listener name="default" socket-binding="http"/>
		<https-listener name="default-ssl" socket-binding="https" security-realm="SslRealm"/>
		...
```

# Verify https port binding

```
<socket-binding-group name="standard-sockets" default-interface="public" port-offset="${jboss.socket.binding.port-offset:0}">
    <socket-binding name="https" port="${jboss.https.port:8443}"/>
	...
```

NOTE:																			<br>
<i><b>Add a unix redirect, because port 80 is open only by root user</b> 		<br>
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080 <br>
sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8443</i>


# Self signed certificate

Generate self-signed ssl cert with keytool:

```
keytool -keystore ssl-server.jks -genkey -alias testssl -keyalg RSA
```

Enter keystore password: <b>--the keystore password--</b>  
Re-enter new password:
What is your first and last name?
  [Unknown]:  Marco Rossi
What is the name of your organizational unit?
  [Unknown]:  Net    
What is the name of your organization?
  [Unknown]:  ACN
What is the name of your City or Locality?
  [Unknown]:  Rome
What is the name of your State or Province?
  [Unknown]:  RM
What is the two-letter country code for this unit?
  [Unknown]:  IT
Is CN=Marco Rossi, OU=Net, O=ACN, L=Rome, ST=RM, C=IT correct?
  [no]:  y

For example, use it on spring-boot project: 

 - copy the file on <i>/src/main/resources</i> classpath directory of a java project
 - write on <i>application.properties</i> file the following configuration parameters
 
```

server.ssl.enabled=false
server.ssl.key-store=classpath:ssl-server.jks
server.ssl.key-store-password=\<b>--the keystore password--\</b>
server.ssl.key-store-provider=SUN
server.ssl.key-store-type=JKS
```


