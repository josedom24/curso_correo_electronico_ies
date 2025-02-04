# ¿Qué tenemos que hacer para que nuestro correo "llegue a buen puerto"?

## IPs estáticas y limpias

La técnica más importante para asegurar el envío de nuestro correo y luchar contra el spam son las listas negras. Podemos configurar nuestro servidor de correo para que consulte en las distintas listas que existen la dirección de ip de origen [How To Block Spam Before It Enters The Server (Postfix)](https://www.howtoforge.com/block_spam_at_mta_level_postfix). Tenemos muchos sitios para ver si nuestra ip está en una lista negra:

* [dnsbl.info](https://www.dnsbl.info/dnsbl-database-check.php)
* [mxtoolbox ](http://mxtoolbox.com/blacklists.aspx)
* [Spamhaus Block List ](http://www.spamhaus.org/sbl/index.lasso)

## SPF

En principio cualquier máquina puede enviar mensajes de correo a cualquier destino utilizando cualquier remitente, esto es así en principio para permitir que podamos enviar mensajes con diferentes remitentes usando un mismo servidor de correos, pero esta característica del correo ha sido masivamente usada para inundar de mensajes no deseados los servidores de correo y para hacer suplantación del remitente (email spoofing), por lo que se ha generalizado la implementación de medidas complementarias para reducir al máximo este problema y hoy en día la mayoría de servidores de correo o rechazan o mandan a la bandeja de spam los mensajes que lleguen desde servidores que no implementen algún mecanismo adicional de autenticación.

**Sender Policy Framework (SPF)** es un mecanismo de autenticación que mediante un registro DNS de tipo TXT describe las direcciones IPs y nombres DNS autorizados a enviar correo desde un determinado dominio. Actualmente muchos servidores de correo exigen como mínimo tener un registro SPF para tu correo o en caso contrario los mensajes provenientes de tu servidor se clasifican como spam o se descartan directamente.

Ejemplo de registro SPF:

    DOMINIO.    600 IN  TXT "v=spf1 a mx ip4:X.X.X.X/32 ip6:XXXX:XXXX:XXXX::XXXX:XXXX/128 a:nombre_máquina -all"

En definitiva es una lista de direcciones IP que autorizamos el envío del correo de nuestro dominio. Los parámetros significan los siguiente:

* **a**: Las direcciones IP declaradas  en cualquier registro A de la zona DNS del dominio.
* **mx**: IP de la máquina a la que apunta el registro MX del DNS del dominio.
* **ptr**: Las direcciones IP declaradas en los registros PTR de nuestra zona de resolución inversa.
* **ip4**:  Direcciones IPv4.
* **ip6**:  Direcciones IPv6.

Por ejemplo, en nuestro caso en que el servidor de correo es `satelite.gonzalonazareno.org` que tiene la dirección IP `5.39.73.79` está declarada de la siguiente forma:

```
dig txt gonzalonazareno.org
...
gonzalonazareno.org.	0	IN	TXT	"v=spf1 ip4:5.39.73.79  ~all"
```

Es importante comentar el signo que aparece antes de `all`, ya que podemos indicarle a los otros servidores lo que deben hacer si reciben correo desde otra dirección o máquina diferente a las que aparecen en el registro anterior:

* `-`: Descartar el mensaje.
* `~`: Clasificarlo como spam.
* `?`: Aceptar el mensaje (sería como no usar SPF).

De esta forma el correo que enviemos desde nuestra máquina, pasará los filtros SPF en destino y la mayoría de nuestros correos llegarán a destino con poca probabilidad de que se clasifiquen como spam. 

Para más información puedes leer el documento: [Sender Policy Framework (SPF)](https://github.com/josedom24/serviciosgs_doc/raw/master/correo/doc/SPF.pdf)


## DKIM

**DomainKeys Identified Mail o DKIM** es un método de autenticación pensado principalmente para reducir la suplantación de remitente. DKIM consiste en que se publica a través de DNS la clave pública del servidor de correos y se firman con la correspondiente clave privada todos los mensajes emitidos, así el receptor puede verificar cada correo emitido utilizando la clave pública.

Para configurar DKIM en nuestro servidor instalaremos los paquetes necesarios:

```
apt install opendkim opendkim-tools
```
A continuación vamos a crear el par de claves, para ello ejecutamos la siguiente instrucción:

```
sudo -u opendkim opendkim-genkey -D /etc/dkimkeys -d midominio.algo -s miclave
```

Es decir con el usuario `opendkim` se generan un par de claves que se guardan en el directorio `/etc/dkimkeys` (`-D`), del dominio `midominio.algo` y le ponemos un nombre al selector, en nuestro caso `miclave`. El selector es el nombre que se va a utilizar para identificar las claves.

En este caso en el directorio `/etc/dkimkeys` se han creado dos ficheros:

* `miclave.private`: Donde se guarda la clave privada con lo que se va a firmar los correos.
* `miclave.txt`: Donde se guarda la clave pública.

A continuación vamos a configurar opendkim, para ello en el fichero `/etc/opendkim.conf` descomentamos y ponemos los siguiente valores en los siguientes parámetros:

```
Domain                  midominio.algo
KeyFile                 /etc/dkimkeys/miclave.private
Selector                miclave
Socket                  inet:8892@localhost
```

Indicamos nuestro dominio, el fichero donde está la clave privada, el selector que hemos utilizado al crear las claves y por último configuramos el opendkim para que escuche peticiones en el puerto 8892/tcp de localhost (para que postfix se conecte a él).

Para integra postfix con opendkim, añadimos al fichero `/etc/postfix/main.cf` las siguientes líneas:

```
smtpd_milters = inet:localhost:8892
non_smtpd_milters = $smtpd_milters
```

Por último reiniciamos los dos servicios:

```
systemctl restart opendkim
systemctl restart postfix
```

Por último tenemos que publicar nuestra clave pública en un registro TXT del DNS de nuestro dominio, para ello visualizamos el contenido de la clave pública:

```
cat miclave.txt
miclave._domainkey	IN	TXT	( "v=DKIM1; h=sha256; k=rsa; "
	  "p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAx2PSS+Z1bCqnUGay+rbr/0jjdBjlQ5SRdzX237NGv6YaeK7DqVFfWBY83Nk6QWCLjxg9Qg588AqjjnlLLDmVNNNPRpzytpFnBdIge6P5kUyJI8VPqw+c6uaNJ2yfG6awfWZvgvDGmqjO6ZFQX+vDV2yR7N0uejJd+WPvSMVN9fYGdBFWWnX+JZ8VVb49Cn9L4tbsMqhiDLY/4L"
	  "/3pLsJMzLOAVuzUac8p0CGPL/nJOKhaXDGdxyehxZW/FbT7ZYx/fYzSvG9OdEVHcTBxQkvE3hYWv/dPc617dJrO6YrB0AeJxOWmPJgeMbYehZYELUIMOGgIHt7z6/eR6du+27mYQIDAQAB" )  ; ----- DKIM key miclave for dominio.algo
```

Tenemos que crear un registro TXT en nuestro DNS con el nombre `miclave._domainkey.dominio.algo` con el contenido desde **v=DKIM1** hasta la última comilla, quitando las comillas y poniendo todo en una misma línea.

Por ejemplo, para el registro de `gonzalonazareno.org` sería:

![dkim](img/dkim_dns.png)

    
Verificamos que la configuración del registro de DKIM es correcta utilizando alguna [herramienta externa](https://mxtoolbox.com/dkim.aspx), que da los resultados de forma fácilmente interpretable:

![dkim](img/dkim.png)

## DMARC

**DMARC** (Domain-based Message Authentication, Reporting, and Conformance) es el último mecanismo de autenticación que vamos a configurar, realmente lo hace DMARC es ampliar el funcionamiento de SPF y DKIM, mediante la publicación en DNS de la política del sitio, en el que decimos si usamos, SPF, DKIM o ambos,y que se debe hacer coon nuestro correo si no cumple con los protocolos anteriores.

La configuración de DMARC para el correo saliente es sencilla, consiste en un registro DNS TXT en el que se especifica si se está usando SPF y/o DKIM y qué hacer con el correo que no cumpla con los mecanismos de autenticación que estén habilitados, como en este caso están ambos, creamos un registro como el siguiente:

    _dmarc.DOMINIO. 3600    IN  TXT "v=DMARC1; p=quarantine;adkim=r;aspf=r; rua=mailto:correo@DOMINIO"

Veamos algunos parámetros:

* **p**, se indica la política de correos:
    * `p=none`, que permite que los correos que suspenden sigan pasando.
    * `p=quarantine` indica que los servidores de correo electrónico deben "poner en cuarentena" los correos electrónicos que no superan DKIM y SPF, considerándolos potencialmente spam.
    * `p=reject`, que ordena a los servidores de correo electrónico que bloqueen los correos que suspenden.
* **adkim**: Indica cómo son la comprobaciones DKIM. `adkim=s` significa que las comprobaciones DKIM son "estrictas" (el dominio del from debe coincidir con tu dominio). Esto también se puede configurar como "relajado" (el dominio del from puede ser un subdominio de tu dominio) si se cambia por `adkim=r`.
* **aspf**: Lo mismo que el anterior pero para SPF.
* **rua**: Se utiliza para designar una o varias direcciones de correo electrónico a las que enviar los informes DMARC Aggregate Feedback. 

El único parámetro obligatorio es el **p**.

Se pueden ver los detalles del formato en [What is a DMARC DNS Record?](https://mxtoolbox.com/dmarc/details/what-is-a-dmarc-record).

Por ejemplo, el registro DMARC de gonzalonazareno.org es:

```
dig txt _dmarc.gonzalonazareno.org
...
_dmarc.gonzalonazareno.org. 0	IN	TXT	"v=DMARC1; p=quarantine; adkim=r; aspf=r;"
```

