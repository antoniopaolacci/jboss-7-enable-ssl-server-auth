#Common file related to jboss and ssl certificate

- account.key 
- domain.key
- intermediate.crt
- domain.crt
- domain.p12
- domain.jks

NOTE:
<i>The password of the P12-Certifcate and the password of the Keystore has to be the same.</i>

# Create PKCS12 keystore from private key and public certificate.
```openssl pkcs12 -export -name domain -in domain.crt -inkey domain.key -out domain.p12```

Enter Export Password: ... <br>
Verifying - Enter Export Password: ...  <br>

# Convert PKCS12 keystore into a JKS keystore
```keytool -importkeystore -srckeystore domain.p12 -srcstoretype pkcs12 -destkeystore domain.jks -deststoretype JKS```

Immettere la password del keystore di destinazione: ...  <br>
Immettere nuovamente la nuova password: ...  <br>
Immettere la password del keystore di origine: ...  <br>
La voce dell' alias domain è stata importata.  <br>
Importazione completata:  1 voci importate, 0 voci non importate o annullate  <br>

# Import intermediate certificate
```keytool -import -trustcacerts -alias intermediate -file intermediate.crt -keystore domain.jks```

Immettere la password del keystore:  <br>
Considerare attendibile questo certificato? [no]:  si  <br>
Il certificato è stato aggiunto al keystore  <br>

# Import domain certificate
```keytool -import -trustcacerts -alias domain -file domain.crt -keystore domain.jks```

Immettere la password del keystore:  <br>
Considerare attendibile questo certificato? [no]:  si  <br>
Il certificato è stato aggiunto al keystore  <br>

#Verify the contents of the JKS:
```keytool -list -v -keystore domain.jks```

Immettere la password del keystore:  <br>


