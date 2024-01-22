# Caso 2: Envío de correo desde usuarios del servidor a correos de internet

## Desde el aula

En el caso de la configuración de nuestra red del instituto, sólo puede enviar correo el servidor de correo de `satelite.gonzalonazareno.org`. Por lo tanto tenemos que configurar nuestro servidor para que utilice este servidor como relay para enviar nuestros correos, para que sea más sencillo hemos creado un alias llamado `mail.gonzalonazareno.org`. Para ello, modificamos la siguiente directiva en el fichero de configuración:

	relayhost = mail.gonzalonazareno.org


## Desde tu servidor VPS

¿Qué necesitamos para que desde nuestro VPS podamos mandar correos al exterior?:

* Con la configuración básica del servidor, seríamos capaces de enviar el correo (no necesitamos un relay).
* Quizás no sea necesario para el envío de correos, pero estaría muy bien que ya tengamos configurado en nuestro DNS el registro MX apuntando a nuestra máquina.
* Necesitamos implementar varias técnicas para asegurar que el servidor de correo al que mandamos nuestro correo confíe en nuestro servidor de correos.
* Necesitamos que nuestra IP esté "limpia", es posible que si mandamos el correo desde un servidor que utilice una ip dinámica, seguramente gmail /hotmail/yahoo lo rechaza, por estar en una lista de bloqueo, al intentar enviar un correo nos salen registro de este tipo:
 un correo nos salen registro de este tipo:

	![postfix6](img/postfix4.jpg)

En el siguiente apartado profundizaremos en los mecanismos necesarios para asegurar el envío de nuestro correos.

## Asegurar que el correo llegue

Es posible que la configuración anterior que hemos explicado no sea suficiente para que el MTA donde enviamos el correo, lo acepte como válido. Existen tres mecanismos que tenemos que configurar que permiten que los servidores de correo confíen en los correos que mandamos.

En concreto, hemos recibido un correo de CDMON con la siguiente información:

*Google y Yahoo exigirán que todos los correos cumplan con los estándares SPF, DKIM y DMARC a partir del 1 de febrero de 2024. SPF autoriza al servidor que envía tus correos, DKIM garantiza la integridad del mensaje y DMARC indica al receptor qué hacer si alguno de los anteriores no coincide (por ejemplo, rechazarlo o enviarlo a carpeta de spam).*

*Estos registros agregan capas adicionales de seguridad al verificar la legitimidad de los correos electrónicos enviados desde tu Hosting. Esto no solo aumentan la seguridad del correo electrónico, sino que también reducen la probabilidad de que tus correos sean marcados como spam o rechazados por otros servidores.*

En resumen, los tres mecanismos de seguridad que tenemos que configurar son:

* **SPF**: autoriza al servidor que envía tus correos.
* **DKIM** garantiza la integridad del mensaje.
* **DMARC** indica al receptor qué hacer si alguno de los anteriores no coincide (por ejemplo, rechazarlo o enviarlo a carpeta de spam).

La configuración de estos métodos lo vamos a estudiar en el apartado: [¿Qué tenemos que hacer para que nuestro correo "llegue a buen puerto"?](https://github.com/josedom24/curso_correo_electronico_ies/blob/main/modulo3/asegurar_envio_correo.md).