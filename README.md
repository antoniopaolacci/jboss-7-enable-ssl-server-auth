# Obtain SSL certificate

[https://gethttpsforfree.com/](https://gethttpsforfree.com/)

#Common file related to jboss and ssl certificate

- account.key 
- domain.key
- intermediate.crt
- domain.crt
- domain.p12
- domain.jks

NOTE: 																						<br>
<i>The password of the P12-Certifcate and the password of the Keystore has to be the same.  <br>
To avoid: Unable to start service Caused by: java.security.UnrecoverableKeyException: Cannot recover key </i>

# Create PKCS12 keystore from private key and public certificate.
```openssl pkcs12 -export -name domain -in domain.crt -inkey domain.key -out domain.p12```

Enter Export Password: ... 				<br>
Verifying - Enter Export Password: ...  <br>

# Convert PKCS12 keystore into a JKS keystore
```keytool -importkeystore -srckeystore domain.p12 -srcstoretype pkcs12 -destkeystore domain.jks -deststoretype JKS```

Immettere la password del keystore di destinazione: ...  					  <br>
Immettere nuovamente la nuova password: ...  								  <br>
Immettere la password del keystore di origine: ...  						  <br>
La voce dell' alias domain è stata importata.  								  <br>
Importazione completata:  1 voci importate, 0 voci non importate o annullate  <br>

# Import intermediate certificate
```keytool -import -trustcacerts -alias intermediate -file intermediate.crt -keystore domain.jks```

Immettere la password del keystore:  				   <br>
Considerare attendibile questo certificato? [no]:  si  <br>
Il certificato è stato aggiunto al keystore  		   <br>

# Import domain certificate
```keytool -import -trustcacerts -alias domain -file domain.crt -keystore domain.jks```

Immettere la password del keystore:  					<br>
Considerare attendibile questo certificato? [no]:  si   <br>
Il certificato è stato aggiunto al keystore  			<br>

#Verify the contents of the JKS
```keytool -list -v -keystore domain.jks```

Immettere la password del keystore:  <br>

Tipo keystore: JKS 				<br>
Provider keystore: SUN 			<br>

Il keystore contiene 2 entry 	<br>

#Editing configuration/standalone.xml

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

#Adding listener for https by editing standalone.xml

```
<subsystem xmlns="urn:jboss:domain:undertow:1.2">
	<buffer-cache name="default"/>
	<server name="default-server">
		<http-listener name="default" socket-binding="http"/>
		<https-listener name="default-ssl" socket-binding="https" security-realm="SslRealm"/>
		...
```

#Verify https port binding

```
<socket-binding-group name="standard-sockets" default-interface="public" port-offset="${jboss.socket.binding.port-offset:0}">
    <socket-binding name="https" port="${jboss.https.port:8443}"/>
	...
```

NOTE:																			<br>
<i>Add a unix redirect, because port 80 is open only by root user: 				<br>
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080 <br>
sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8443</i>