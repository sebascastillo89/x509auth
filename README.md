# x509auth in Spring Boot

Based on this Baeldung tutorial:
* https://www.baeldung.com/x-509-authentication-in-spring-security


To implement X.509 authentication in a Spring application, we’ll first create a keystore in the Java Key-Store (JKS) format.

For creating a new keystore with a certificate authority, we can run make as follows 

	> Generate a certificate authority (BAST CA)
	```
	keytool -genkey -alias ca -ext BC=ca:true -keyalg RSA -keysize 4096 -sigalg SHA512withRSA -keypass changeit -validity 3650 -keystore keystore.jks -storepass changeit
	```

Now, we will add a certificate for our development host to this created keystore and sign it by our certificate authority:

	> Generate a host certificate (localhost)
	```
	keytool -genkey -alias localhost -keyalg RSA -keysize 4096 -sigalg SHA512withRSA -keypass changeit -validity 3650 -keystore keystore.jks -storepass changeit
	```
	> Generate a host certificate signing request
	```
	keytool -certreq -alias localhost -ext BC=ca:true -keyalg RSA -keysize 4096 -sigalg SHA512withRSA -validity 3650 -file "localhost.csr" -keystore keystore.jks -storepass changeit
	```
	> Generate signed certificate with the certificate authority
	```
	keytool -gencert -alias ca -validity 3650 -sigalg SHA512withRSA -infile localhost.csr -outfile "localhost.crt" -rfc -keystore keystore.jks -storepass changeit
	```
	> Import signed certificate into the keystore
	```
	keytool -import -trustcacerts -alias localhost -file "localhost.crt" -keystore keystore.jks -storepass changeit
	```
	
To allow client authentication, we also need a keystore called “truststore”. This truststore has to contain valid certificates of our certificate authority and all of the allowed clients. For reference on using keytool, please look into the Makefile at the following given sections:

	> Export certificate authority
	```
	keytool -export -alias ca -file ca.crt -rfc -keystore keystore.jks -storepass changeit
	```
	
	> Import certificate authority into a new truststore
	```
	keytool -import -trustcacerts -noprompt -alias ca -file ca.crt -keystore truststore.jks -storepass changeit
	```
	
	> Generate client certificate
	```
	keytool -genkey -alias cid -keyalg RSA -keysize 4096 -sigalg SHA512withRSA -keypass changeit -validity 3650 -keystore truststore.jks -storepass changeit
	```
	
	> Generate a host certificate signing request
	```
	keytool -certreq -alias cid -ext BC=ca:true -keyalg RSA -keysize 4096 -sigalg SHA512withRSA -validity 3650 -file "cid.csr" -keystore truststore.jks -storepass changeit
	```
	
	> Generate signed certificate with the certificate authority
	```
	keytool -gencert -alias ca -validity 3650 -sigalg SHA512withRSA -infile "cid.csr" -outfile "cid.crt" -rfc -keystore keystore.jks -storepass changeit
	```
	
	> Import signed certificate into the truststore
	```
	keytool -import -trustcacerts -alias cid -file cid.crt -keystore truststore.jks -storepass changeit
	```
	
	> Export private certificate for importing into a browser
	```
	keytool -importkeystore -srcalias cid -srckeystore truststore.jks -srcstorepass changeit -destkeystore "cid.p12" -deststorepass changeit -deststoretype PKCS12
	```
