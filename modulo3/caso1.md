# Caso 1: Envío local, entre usuarios del mismo servidor"

Esta situación podemos aprovecharla para el envío de correos entre usuarios de la máquina, disponiendo de correo interno. 

## Envío de correos

Para el envío de correo vamos a usar la utilidad `mail` (se encuentra en el paquete `bsd-mailx`). Y para enviar un correo desde el usuario `debian` al usuario `root`, simplemente ejecutamos:

```
debian@maquina:~$ mail root@DOMINIO
Subject: Hola
Esto es una prueba
Cc: 
```

**Recuerda que para terminar de escribir el cuerpo del mensaje hay que ponerse en una nueva línea e introducir CTRL+D.**

Para leer el usuario root simplemente ejecuta la instrucción `mail`:

```
root@maquina:~#  mail
Mail version 8.1.2 01/15/2001.  Type ? for help.
"/var/mail/root": 1 message 1 unread
>U  1 debian@DOMINIO  Sat Jan 16 18:22   79/3597  Re: hola
& 1
```

Introducimos el número de correo para leerlo.

Como vemos el buzón del usuario está en `/var/mail/root`. Los correos leídos se guardan en `~/mbox`.


Podemos comprobar el log `/var/log/mail.log` para comprobar que se ha mandado el mensaje:


	Feb 6 18:10:05 vostro postfix/smtpd[3660]: DE3232C16A: client=localhost[127.0.0.1]
	Feb 6 18:11:07 vostro postfix/cleanup[3907]: DE3232C16A: message-id=<20120206171005.DE3232C16A@mail2.josedomingo.org>
	Feb 6 18:11:07 vostro postfix/qmgr[3531]: DE3232C16A: from=<jose@josedomingo.org>, size=400, nrcpt=1 (queue active)
	Feb 6 18:11:08 vostro postfix/local[3908]: DE3232C16A: to=<usuario@josedomingo.org>, relay=local, delay=75, delays=74/0/0/1, dsn=2.0.0, status=sent (delivered to command: procmail -a "$EXTENSION")
	Feb 6 18:11:08 vostro postfix/qmgr[3531]: DE3232C16A: removed
	Feb 6 18:11:09 vostro postfix/smtpd[3660]: disconnect from localhost[127.0.0.1]
