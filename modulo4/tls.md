# Postfix y el cifrado de correos con TLS

## Cifrado de correos entre MTA

Los MTA utilzan el puerto 25/tcp y el procolo smtp para el envío y recepción de correo. Un MTA tiene dos funciones:

* Como **servidor** (smtpd), cuando otros MTA se conectan a él para enviarles un correo. En este caso nuestro MTA recibe correos.
* Como **cliente** (smtp), cunado nuestro MTA se conecta a otro MTA para enviarle un correo.

Los parámetros por defecto del cifrado cuando nuestro MTA funciona como MTA, es decir, cuando recibe correos es:

```
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
smtpd_tls_security_level=may
```

* Como vemos, por defecto se utiliza un certificado autofirmado. Por lo tanto garantizamos que un MTA que se conecta al nuestro para enviarle un correo, lo podrá mandar de forma cifrada, pero no podrá confiar en nuestro MTA. La desconfianza en nuestro servidor puede hacer que el cliente smtp no envíe el correo (actualmente los servidores no están configurado tan estrictamente). Si queremos que los clientes confíen en nuestro MTA necesitaríamos usar un certificado firmado por un CA de confianza, por ejemplo Lets Encrypt.
* `smtpd_tls_security_level`: Configura el nivel de seguridad TLS para las conexiones entrantes al servidor SMTP (smtpd). 
    * La opción `may` ofrecerá TLS a los clientes SMTP, pero no lo requerirá. Es decir, los clientes pueden enviar correos con o sin cifrado. 
    * Si le ponemos el valor `encrypt`, sl cliente sólo nos enviará un correo si la conexión es cifrada. Tenemos que tener en cuenta que algunos servidores de correo (especialmente antiguos) no soportan TLS. Con `encrypt`, no podrás enviarles correos.

Los parámetros por defecto cuando nuestro MTA funciona como cliente y se conecta a otro MTA para enviarle un correo son:

```
smtp_tls_CApath=/etc/ssl/certs
smtp_tls_security_level=may
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
```

* Donde se indica los certificados de las CA para verificar los certificados enviados por el servidor.
* Se utiliza el parámetro `smtp_tls_security_level` para configurar la política de cifrado:
    * Con el valor `may`: Postfix intentará usar TLS al enviar correos, pero si el servidor remoto no lo soporta, enviará el correo sin cifrado.
    * Con el valor `encrypt`: Requiere que el servidor destino soporte TLS; si no lo hace, el correo no se envía.
    * Con el valor `secure`: Exige TLS y además requiere que el certificado esté validado por una CA confiable. Si falla la validación, el correo no se envía.
* Y se define la base de datos donde se almacena la caché de sesiones TLS para conexiones salientes.

## Cifrado de correos con IMAP

Cuando configuras IMAP con cifrado en un servidor de correo, puedes elegir entre **IMAP con STARTTLS** o **IMAPS (IMAP sobre SSL/TLS)**. Veamos las diferencias, ventajas y configuración en Postfix y Dovecot.  



| Modo   | Puerto | Cifrado | Descripción |
|--------|--------|---------|-------------|
| **IMAP con STARTTLS** | 143 | Opcional | Comienza en texto plano y negocia TLS si el cliente lo solicita. |
| **IMAPS (IMAP sobre SSL/TLS)** | 993 | Obligatorio | La conexión está cifrada desde el inicio, como HTTPS. |

🔹 **STARTTLS** es útil si quieres compatibilidad con clientes que pueden usar **cifrado opcional**.  
🔹 **IMAPS (993)** es más seguro porque el cifrado **siempre es obligatorio** desde el inicio.  

Algunas consideraciones:

* Usar solo IMAPS (993): Más seguro, el cliente no puede enviar credenciales sin cifrado.
* Habilitar IMAP con STARTTLS (143) + IMAPS (993): Máxima compatibilidad con clientes antiguos.
* Deshabilitar IMAP en 143 sin STARTTLS: Si quieres obligar a todos los clientes a usar IMAPS.


## Cifrado de correos con SMTP

Cuando se configura SMTP con cifrado en un servidor de correo como **Postfix**, se puede elegir entre **ESMTP con STARTTLS** y **SMTPS (SMTP sobre SSL/TLS directo en el puerto 465)**.  


| Modo   | Puerto | Cifrado | Descripción |
|--------|--------|---------|-------------|
| **SMTP con STARTTLS** | 25, 587 | Opcional | Inicia en texto plano y luego negocia TLS si el cliente lo soporta. |
| **SMTPS (SMTP sobre SSL/TLS)** | 465 | Obligatorio | La conexión está cifrada desde el inicio, como HTTPS. |

🔹 **STARTTLS en el puerto 587** es el estándar moderno para envío de correos autenticados.  
🔹 **SMTPS (465)** se usaba antes, pero fue reemplazado por STARTTLS; sin embargo, algunos clientes aún lo requieren.  

Alguna consideraciones:

* Usar solo SMTP con STARTTLS en el puerto 587: Es el estándar moderno recomendado.  
* Habilitar SMTPS (465) solo si es necesario: Algunos clientes antiguos aún lo requieren.  
* Obligar cifrado con `smtpd_tls_security_level=encrypt`: Evita que los correos viajen en texto plano.  

