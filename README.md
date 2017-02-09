#Common file related to jboss and ssl certificate

- account.key 
- domain.key
- intermediate.crt
- domain.crt
- domain.p12
- domain.jks

# Create PKCS12 keystore from private key and public certificate.
```openssl pkcs12 -export -name myservercert -in domain.crt -inkey domain.key -out domain.p12```

Enter Export Password: ...
Verifying - Enter Export Password: ...

# Convert PKCS12 keystore into a JKS keystore
```keytool -importkeystore -srckeystore domain.p12 -srcstoretype pkcs12 -destkeystore domain.jks -deststoretype JKS```

Immettere la password del keystore di destinazione: ...
Immettere nuovamente la nuova password: ...
Immettere la password del keystore di origine: ...
La voce dell'alias myservercert è stata importata.
Importazione completata:  1 voci importate, 0 voci non importate o annullate

# Import intermediate certificate
```keytool -import -trustcacerts -alias intermediate -file intermediate.crt -keystore domain.jks```

Immettere la password del keystore: ...
Considerare attendibile questo certificato? [no]:  si

# Import domain certificate
```keytool -import -trustcacerts -alias domain -file domain.crt -keystore domain.jks```

Immettere la password del keystore: ...
Considerare attendibile questo certificato? [no]:  si

#Verify the contents of the JKS:
```keytool -list -v -keystore domain.jks```

Immettere la password del keystore: ...


