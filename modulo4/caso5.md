# Caso 5: Envío de correo electrónico usando nuestro servidor de correos

## Protocolo SMTP



### Configuración de Postfix

Hay que recordar que en la configuración de Postfix tenemos la siguiente directiva:

* `mynetworks`: Con `mynetworks` se indican las IPs desde las que pueden enviarse mensajes. Por defecto: `127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128`.

Por defecto, sólo se permite el envío de correos desde el propio servidor, por lo tanto para permitir el uso de un cliente de correo externo tenemos que indicar en este parámetro de configuración la dirección IP o la red donde van a estar los clientes de correo. Si los clientes de correo van a estar en internet (no en una red completa) tenemos que configurar este parámetro con el valor `0.0.0.0/0` (a esta configuración se le llama abrir el relay) y en en esta circunstancia cualquier cliente de correo podría usar nuestro servidor para eviar correo.

**Si abrimos el relay (`mynetworks = 0.0.0.0/0`) es totalmente necesario que la conexión entre el cliente y el servidor sea autentificada y estar cifrada.**

## Auntentificación y cifrado de la conexión para el envío del correo

En primer lugar hay que tener en cuenta que para el envío de correos entre MTA se utiliza el puerto 25/tcp.

Sin embargo si vamos a usar un cliente de correos para el envío de correos, este cliente, en la actualidad no usa el puerto 25/tcp para conectarse al servidor (ya que no estaría ni autentificada, ni cifrada). Para autentificar y cifrar la conexión, tenemos dos opciones:

* **ESMTP + STARTTLS**: esmtp (Enhanced Simple Mail Transfer Protocol): En este caso se usa el puerto 587/tcp. Este protocolo tiene nuevas extensiones: como smtp-auth y STARTTLS (STARTTLS transforma una conexión insegura en una segura mediante el uso de SSL/TLS).

	El puerto 587/tcp se conoce como puerto de submission o presentación. Al abrir este puero postfix esta funcionando de MSA (mail submission agent) que recibe mensajes de correo electrónico desde un Mail User Agent (MUA) y coopera con un Mail Transport Agent (MTA) para entregar el correo.

	Tenemos que conseguir que la comunicación que se establece desde el cliente con el servidor sea autentificada para ella usamos **SASL** (Simple Authentication and Security Layer) que es un framework para autenticación y autorización en protocolos de Internet. Para realizar la autenticación vamos a usar **dovecot** (que ya tiene un mecanismo de autenticación).

	Además tenemos que conseguir que la comunicación sea cifrada, para ello vamos a usar **STARTTLS** que nos permite que utilizando el mismo puerto (587/tcp) la conexión este cifrada.

* **SMTPS**: Simple Mail Transfer Protocol Secure: Con este protocolo conseguimos el cifrado de la comunicación entre el cliente y el servidor. Utiliza un puerto no estándar 465/tcp. No es una extensión de smtp. Es muy parecido a HTTPS.

Como comentábamos en el apartado anterior nosotros vamos a usar un certificado firmado por LetsEncrypt para cifrar la comunicación.

## Enlaces interesantes

Os dejo dos entradas que os pueden ayudar para la configuración de estos dos últimos apartados:

* [ISP Mail Tutorial for Debian 10 (Buster)](https://123qwe.com/tutorial-debian-10/)
* [How to secure Postfix using Let’s Encrypt](https://upcloud.com/community/tutorials/secure-postfix-using-lets-encrypt/)
