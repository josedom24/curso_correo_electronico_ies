# Direcciones de correo electrónico

Para enviar un correo necesitamos la dirección de correo de un usuario, la forma de esta dirección es:

    usuario@nombre_servicio_correo

Normalmente el nombre del servicio de correo es el **nombre del dominio** de la organización a la que pertenece el usuario.

Cuando enviamos un correo a esa dirección, de alguna manera tenemos que determinar la dirección IP del servidor de correo de la organización:

1. Si el nombre detrás de la @ está asociado a un **registro A** en un servidor DNS, ya sabríamos la dirección del servidor de correo.
2. Como hemos comentado, el nombre del servicio de correo es un nombre de dominio que no suele estar asociado a una dirección IP (y menos del servidor de correo). Por lo tanto, es necesario en el servidor dns un **registro MX** que indique el nombre del servidor de correo asociado al nombre de dominio.
