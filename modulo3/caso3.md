# Caso 3: Recibir correos desde internet a usuarios del servidor

## Desde el aula

Vamos a tener un correo de la forma `usuario@dominio_de_cada_alumno`, para nuestro ejemplos pongamos `jose@josedom.gonzalonazareno.org`.

Tenemos que tener en cuenta los siguientes aspectos:

1. Si queremos recibir correos desde internet a nuestro servidor, todos nuestros dominios tienen que apuntar a nuestra ip pública `5.39.73.79` (IP de `satelite.gonzalonazareno.org`), sin embargo no hay que tocar el DNS de cdmon ya que tenemos un registro genérico que envía a `5.39.73.79 ` cualquier cosa de .gonzalonazareno.org que no tenga un registro tipo ADDRESS. Prueba a hacer un `dig loquesea.gonzalonazareno.org`. Esto se hace con el registro DNS:

		* IN CNAME satelite.gonzalonazareno.org.

	Por lo tanto cuando se recibe un correo en esa dirección pública, lo recibe el servidor de correo que tenemos en `satelite.gonzalonazareno.org`.

2. Tenemos que configurar el servidor de correos de `satelite.gonzalonazareno.org` para que haga relay con los correos cuyo destinos sean nuestros dominios, es decir el correo que vaya a `josedom.gonzalonazareno.org` lo tiene que enviar al servidor de correos de ese dominio, para ello:
    * Añadimos en la directiva `relay_domains`, del servidor de correos de `satelite.gonzalonazareno.org`, cada uno de los nombres de dominios a los que queremos reenviar los mensajes.
	* `satelite.gonzalonazareno.org` tiene como primer DNS a nuestro dns de la red local (**macaco**), por lo que puede preguntarlo por nuestro registro MX, ya que tiene delegada todos nuestros subdominios.
    * Para que `satelite.gonzalonazareno.org` conozca la IP de nuestro servidor de correo tendremos que crear un registro MX en nuestro servidor DNS  para realizar la resolución.

3. Con la configuración que tenemos en el servidor de correo de nuestra máquina debe ser suficiente para recibir el correo. Recuerda mandar un mensaje a un usuario que exista en el servidor.

##  Desde tu VPS

En este caso vamos vais a utilizar vuestro nombre de dominio. Tenemos que tener en cuenta los siguientes aspectos:

1. Configura el servidor postfix en tu servidor, teniendo en cuenta que en el fichero `/etc/mailname` este tu nombre de dominio.
2. Configura tu DNS para que el registro MX apunte a un nombre de ordenador que está definido como un registro A a tu dirección IP pública. Ejemplo:

		# dig mx tudominio.org
		; <<>> DiG 9.7.3 <<>> -t mx tudominio.org
		;; global options: +cmd
		;; Got answer:
		;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9147
		;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 3, ADDITIONAL: 4		

		;; QUESTION SECTION:
		;tudominio.org.        IN    MX		

		;; ANSWER SECTION:
		tudominio.org.    900    IN    MX    10 tumaquina.tudominio.org.		

		;; ADDITIONAL SECTION:
		tumaquina.ieshnXX.es.    900    IN    A    XX.XX.XX.XX (tu ip pública)

3. Recuerda que el correo se guardará en el buzón del usuario, que tendrá que leerlo desde el servidor con la utilidad `mail`.

## ¿Los correos que estamos recibiendo son legítimos?

Con la configuración que hemos explicado estaremos en disposición de que nuestro servidor de correo reciba correos de otros MTA. Pero, ¿podemos estar seguros de qué los correos recibidos no son spam?

En el apartado: [Soluciones al problema del spam](https://github.com/josedom24/curso_correo_electronico_ies/blob/main/modulo3/spam.md) explicaremos como configurar nuestros servidores de correo para que determinen si el correo recibido es válido y legítimo. Para ello, tendremos que comprobar el registro SPF, DKIM y DMARC del emisor.
