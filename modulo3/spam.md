# Soluciones al problema del spam

## SMTPd restrictions

Con las restricciones postfix podemos limitar el uso de nuestro servidor como cliente para enviar correos y como servidor a la hora de recibir correos. 

Tenemos varias directivas que nos permiten restringuir tanto el envío como la recepción de correos:

* `smtpd_helo_restrictions`: Podemos configurar nuestro servidor de correos para que filtre a los clientes que intentan usarlo por medio de algún parámetro del protocolo: HELO, MAIL FROM, RCPT TO,..., 
* `smtpd_client_restrictions`: Podemos filtrar los dominios de envíos, campo MAIL FROM.
* `smtpd_relay_restrictions`: Podemos filtrar quien tiene permiso para utilizar nuestro servidor como relay.
* `smtpd_recipient_restrictions`: Podemos filtrar los clientes desde los que nos llegan correos. Por ejemplo, podemos hacer una búsqueda en las listas de cloqueo de SPAM (RBL).

Para más información:

* [SMTPd restrictions, SPF, DKIM and greylisting ](https://workaround.org/ispmail/wheezy/smtpd-restrictions-spf-dkim-and-greylisting)
* [Postfix restrictions](https://wiki.centos.org/HowTos/postfix_restrictions)


## SPF

Podríamos hacer que nuestro servidor de correos hiciera la comprobación del registro SPF del dominio origen del correo.

Para realizar la configuración debemos modificar postfix para que haga la verificación de SPF de los correos recibidos, para lo que instalamos el paquete `postfix-policyd-spf-python` y añadimos la siguiente línea a `/etc/postfix/master.cf`:

    policyd-spf  unix  -       n       n       -       0       spawn     user=policyd-spf argv=/usr/bin/policyd-spf

Se ejecutará un proceso en un socket UNIX para realizar el análisis de SPF, le decimos a postfix que acepte los correos que se validen con SPF realizando la siguiente modificación en /`etc/postfix/main.cf`:
	
    policyd-spf_time_limit = 3600
    smtpd_recipient_restrictions =
    ...
        check_policy_service unix:private/policyd-spf

Probamos a enviar un correo desde otro origen con destino a nuestra máquina y comprobaremos que se realiza la comprobación de spf:

    postfix/smtpd[28324]: connect from X.red-X-X-X.staticip.rima-tde.net[X.X.X.X]
    policyd-spf[28329]: prepend Received-SPF: Pass (mailfrom) identity=mailfrom; client-ip=X.X.X.X; helo=X.gonzalonazareno.org; envelope-from=redmine@gonzalonazareno.org; receiver= 
    postfix/smtpd[28324]: E6ECD143: client=X.red-X-X-X.staticip.rima-tde.net[X.X.X.X]
    postfix/cleanup[28330]: E6ECD143: message-id=
    postfix/smtpd[28324]: disconnect from X.red-X-X-X.staticip.rima-tde.net[X.X.X.X] ehlo=1 mail=1 rcpt=1 data=1 quit=1 commands=5
    postfix/qmgr[28052]: E6ECD143: from=, size=3432, nrcpt=1 (queue active)
    postfix/local[28331]: E6ECD143: to=, relay=local, delay=1.9, delays=1.8/0.03/0/0, dsn=2.0.0, status=sent (delivered to maildir)
    postfix/qmgr[28052]: E6ECD143: removed


## DKIM

Podemos configurar nuestro postfix, para que verifique el registro DKIM y valide la firma que trae el correo. Ya lo hemos configurado al poner en la configuración de `opendkim` el modo de funcionamiento a `v`:

```
Mode	sv
```


## DMARC

* [Debian: Postfix + DMARC](https://blog.cyberfront.org/index.php/2021/10/28/debian-postfix-dmarc/)

## Antivirus y antispam

### Antivirus

Normalmente usamos el antovirus [**ClamAV**](https://www.clamav.net/). Es un motor de antivirus de código abierto, diseñado para detectar y eliminar **virus, malware y otros tipos de software malicioso**. 

ClamAV se utiliza principalmente para:  
* **Detección de virus** en los correos electrónicos.  
* **Bloquear archivos adjuntos peligrosos** que contengan malware, troyanos, etc.  
* **Escaneo de correos entrantes y salientes** para asegurar que no contienen código malicioso.

ClamAV **no analiza el contenido del mensaje** en busca de características típicas de spam (como lo hace SpamAssassin), sino que **busca patrones de malware** en los archivos adjuntos y los correos electrónicos en general.

Para usar ClamAV junto con Postfix, normalmente se integra mediante [**Amavis**](https://www.ijs.si/software/amavisd/). **Amavis** actúa como un **intermediario** que se encarga de escanear los correos antes de que lleguen a su destino final.

El esquema de funcionamiento es:

```
                                      [SpamAssassin]
                                            ^
                                            |
      Email --> [Postfix] --> [(10024) amavisd-new] --> [(10025) Postfix] --> Mailbox
                                            |
                                            v
                                         [ClamAV]
```

1. El correo llega a Posfix.
2. Postfix lo pasa a amavis (puerto 10024).
3. amavis lo pasa a ClamAV (también lo puede pasar a SpamAssassin). Determina qué hacer con el correo según los resultados del escaneo de ClamAV y otras reglas de filtrado (como las de SpamAssassin).
    Dependiendo del resultado del escaneo de ClamAV y otras reglas:
        * Si el correo contiene un virus o un archivo malicioso, Amavis puede rechazar el correo o marcarlo con una etiqueta de aviso (por ejemplo, agregando una cabecera como `X-Virus-Status`: Virus Detected o enviando un correo de aviso al administrador).
        * Si el correo es limpio (sin virus), Amavis lo pasa a Postfix para su entrega final, añadiendo cabeceras o realizando más análisis (por ejemplo, de spam).
4. El correo se devuelve a Postfix (puerto 10025), que determinará que hacer con el correo:
    * Rechazar el correo
    * Marcar correo como sospechoso pero no rechazarlo
    * Reenvío a una dirección de alerta

Para probar el antiviruas puedes mandar un correo con la siguiente cadena:

```
X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H* 
```

### Filtro de spam

[**SpamAssassin**](https://spamassassin.apache.org) es un filtro de correo que analiza los mensajes entrantes y les asigna una puntuación basada en reglas predefinidas. Si la puntuación supera un umbral, el correo se marca como **spam**.  
Se integra con **Postfix** para filtrar correos antes de que lleguen a los buzones de los usuarios.  

SpamAssassin usa **varios métodos combinados** para analizar los correos y determinar si son spam.  

| Método | Descripción |
|--------|------------|
| **Reglas basadas en palabras clave** | Detecta frases típicas de spam en el asunto y el cuerpo del mensaje. |
| **Listas negras (DNSBL)** | Consulta bases de datos de direcciones IP de spammers conocidos. |
| **Análisis Bayesiano** | Aprende de los correos marcados como spam y legítimos para mejorar la precisión. |
| **Pruebas de heurística** | Aplica reglas predefinidas y suma puntuaciones según patrones sospechosos. |
| **RBL (Real-time Blackhole List)** | Consulta servicios externos para detectar servidores de spam. |
| **SPF (Sender Policy Framework)** | Verifica si el remitente tiene permiso para enviar correos desde el dominio. |
| **DKIM (DomainKeys Identified Mail)** | Comprueba la firma criptográfica del correo para detectar falsificaciones. |

Cada prueba añade o resta puntos a la puntuación final del correo. Si se supera el umbral (por defecto **5.0**), se considera spam.  

* [Los test de spamassassin](https://spamassassin.apache.org/old/tests_3_3_x.html)

Cada regla asigna una **puntuación positiva (spam) o negativa (ham, correo legítimo)**.  

Para que **Postfix** envíe los correos entrantes a SpamAssassin, se usa **`spamc`**.

* Aunque se puede usar amavis, si sólo vas a trabajar con spamassassin, se recomienda `spamc`.
* `spamc` es un cliente ligero que envía correos a SpamAssassin (spamd) para su análisis. En lugar de que SpamAssassin procese cada correo de forma independiente (lo que puede ser lento y consumir muchos recursos), spamc actúa como un intermediario que envía los correos al servicio spamd en segundo plano.
* ` spamc` consume menos recursos (ram y cpu) y nos proporcina mayor rendimeinto ya que puede procesar múltiples correos simultáneamente.

**¿Cómo funciona?**

1. Postfix recibe un correo.
2. Postfix lo reenvía a SpamAssassin a través de spamc.
3. spamc envía el correo al demonio spamd, que lo analiza.
4. spamd devuelve el correo con una cabecera X-Spam-Score.
5. Postfix entrega el correo, marcándolo como spam si es necesario.


**Verificación y pruebas**  

```bash
spamassassin -t < email.eml
```

El siguiente correo es considerado como spam:

```
Subject: Test spam mail (GTUBE)
Message-ID: <GTUBE1.1010101@example.net>
Date: Wed, 23 Jul 2003 23:30:00 +0200
From: Sender <sender@example.net>
To: Recipient <recipient@example.net>
Precedence: junk
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit

This is the GTUBE, the
	Generic
	Test for
	Unsolicited
	Bulk
	Email

If your spam filter supports it, the GTUBE provides a test by which you
can verify that the filter is installed correctly and is detecting incoming
spam. You can send yourself a test mail containing the following string of
characters (in upper case and with no white spaces and line breaks):

XJS*C4JDBQADN1.NSBN3*2IDNEN*GTUBE-STANDARD-ANTI-UBE-TEST-EMAIL*C.34X

You should send this test mail from an account outside of your network.
```
