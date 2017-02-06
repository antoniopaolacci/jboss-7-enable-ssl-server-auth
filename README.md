# jboss-7-enable-ssl-server-auth

How to enable server authentication through SSL certificate in JBoss AS 7.1.1.Final. This configuration will make sure that web server 
with certificate is trusted by the client who connects to the application.

Let us create the keys required for JBoss under /usr/save/keystore

<pre>
	<code>
		mkdir /usr/save/keystore
	</code>
</pre> 
 
Server keystore is used by the server to establish a secure connection (through HTTPS protocol)
Use the java keytool, provided by java kit, and the genkey command to create the RSA keypair and/or a self-signed certificate as shown below.

<pre>
	<code>
		/usr/java/jdk1.6.0_31/bin/keytool -genkey -keyalg RSA -alias jbosskeys -keystore jbosskeys.jks -validity 365 -keysize 2048
	</code>
</pre> 

For self-signed certificate use:

<pre>
	<code>
		/usr/java/jdk1.6.0_31/bin/keytool -v -genkey -alias jbosskeys -keyalg RSA -keysize 1024 -keystore jbosskeys.jks -keypass SecretPwd -storepass SecretPwd -validity 365 -dname "CN=localhost"
	</code>
</pre> 

Found/Buy a valid certificate. Import the crt file to the keystore.

<pre>
	<code>
		/usr/java/jdk1.6.0_31/bin/keytool -v -import -keypass SecretPwd -noprompt -trustcacerts -alias domain -file localfile.crt -keystore cacerts.jks -storepass SecretPwd
	</code>
</pre> 


Certificate was added to keystore. Now modify standalone.conf

Modify the <i>jboss-as-7.1.1.Final/bin/standalone.conf</i> file and add the following <i>JAVA_OPTS</i> parameters.

<i>JAVA_OPTS="$JAVA_OPTS \-Djavax.net.ssl.keyStorePassword=SecretPwd"</i>
<i>JAVA_OPTS="$JAVA_OPTS \-Djavax.net.ssl.trustStorePassword=SecretPwd"</i>
<i>JAVA_OPTS="$JAVA_OPTS \-Djavax.net.ssl.keyStoreType=JKS"</i>
<i>JAVA_OPTS="$JAVA_OPTS \-Djavax.net.ssl.trustStoreType=JKS"</i>
<i>JAVA_OPTS="$JAVA_OPTS \-DCLIENT_KEY_ALIAS=jbosskeys"</i>
<i>JAVA_OPTS="$JAVA_OPTS \-Djavax.net.ssl.keyStore=/usr/save/keystore/jbosskeys.jks"</i>
<i>JAVA_OPTS="$JAVA_OPTS \-Djavax.net.ssl.trustStore=/usr/save/keystore/cacerts.jks"</i>

In the standalone.xml file add the following SSL connector information, after this line: <connector name="http" protocol="HTTP/1.1" scheme="http" socket-binding="http"/>


<pre>
	<code>	
		<connector name="https" protocol="HTTP/1.1" scheme="https" socket-binding="connect" secure="true">
			  <ssl name="ssl"
			  protocol="TLSv1"
			  password="SecretPwd"
			  certificate-key-file="/usr/save/keystore/jbosskeys.jks"
			  ca-certificate-file="/usr/save/keystore/cacerts.jks"
			  verify-client="true" />
		</connector>
	</code>
</pre> 


