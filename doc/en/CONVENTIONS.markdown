% CONVENTIONS(2) LibreVPN Manual | lvpn
% Nicolás Reynolds <fauno@endefensadelsl.org>
% 2013

# NAME

Notes for lvpn developers :)


# SYNOPSIS

lib/
:    commands

lib/skel/
:    base files for tinc

lib/skel/scripts/
:    directory of scripts/hooks

doc/
:    documentation

bin/
:    helper programs that are not specifically for lvpn

etc/
:    extra source code

hosts/
:    node files

nodos/
:    your own nodes

locale/
:    translations


# DESCRIPTION

## Where scripts go

The script _lvpn_ autodetects the commands following the
_lib/lvpn-yourscript_ convention.  It also sets some environment
variables that the scripts can use to know which network they are
working on.

The scripts can be in any programming language as long as they can
read those environment variables.


## Environment variables

These are the environment variables that lvpn exports for the rest of
the commands.

TINC
:    Location of the node directory, by default _/etc/tinc/lvpn_.

NETWORK
:    Name of the network, by default the last directory of _TINC_.

LVPN
:    Absolute path of _lvpn_.

LVPN\_DIR
:    Absolute path of the working directory, by default the base directory of _LVPN_

LVPN\_LIBDIR
:    Absolute path of the directory of commands.

LVPN\_HOSTS
:    Absolute path of the directory of hosts.

LVPN\_BEADLE
:    Absolute path of new advertised keys from strangers.

KEYSIZE
:    Default key size.

LVPN\_SUBNET
:    The IPv4 subnet range

LVPN\_SUBNET6
:    The IPv6 subnet range

TINCD\_FLAGS
:    Flags for the _tincd_ daemon.

PORT
:    Default port

sudo
:    Use this variable in front of any command that must run with root
     privileges, eg: _${sudo} rsync_.  Requires _"root=true"_ at the
     beginning of the script.


## Where is the documentation

The documentation documentación carries the full name of the script:
_doc/language/lvpn-yourscript.markdown_.  The function _help()_ in
_lib/msg_ takes the script name as an argument to show this
documentation.

Also, it takes the _PAGER_ environment variable to paginate the
output, by it uses _less_.


## Flags and arguments

_lvpn command_ -flags localnode extra arguments

Following _lvpn_ comes the command, to which flags are passed in order
(with their options).  The first extra argument always has to be the
local node where the action is performed.  Then come the extra
arguments (names of other nodes, for example).

## Common functions

En el archivo _lib/common_ se almacenan las funciones de uso común entre todos
los comandos.  Se la puede incluir en un script añadiendo la línea

    . "${LVPN_LIBDIR}"/common

al principio del script.

Nota: Agregar _root=true_ antes de common para poder correr funciones de
root.

### Variables

* self: nombre del script. Usado para obtener el nombre del script. Ejemplo:
  _help $self_ llama a la ayuda del script actual.

### Funciones

* _add\_to\_file()_: Agrega una línea al final de un archivo. Uso:
  _add\_to\_file archivo "Texto a agregar"_

* _requires()_: Indica que el script necesita que un programa se encuentre en el
  PATH.  Se recomienda cuando el script llama a un programa que puede no
  encontrarse en una instalación estándar.  Uso: _requires avahi-publish rsync_

* _get\_node\_dir()_: Encuentra el directorio de un nodo pasándole el nombre del
  nodo como argumento.  _node\_dir="$(get\_node\_dir ${node})"_

* _get\_node\_file()_: Encuentra el archivo de host de un nodo dentro del
  directorio del nodo.  _node\_file="$(get\_node\_file ${node})"_

* _get\_node\_name()_: Limpia el nombre del nodo de caracteres inválidos

* _get\_host\_file()_: Obtiene el archivo del nodo en $LVPN\_HOSTS

* _find\_init\_system()_: Encuentra el tipo de inicio de tinc.  Ver
  _lib/lvpn-install_.

* _get\_id()_: Obtiene nombre y mail del responsable del nodo usando git o
  usuario@hostname.

* _get\_ipv4()_: Genera una dirección IPv4 a partir de LVPN_SUBNET.

* _get\_ipv6()_: Genera una dirección IPv6 a partir de LVPN_SUBNET6.


## Messages

En _lib/msg_ se encuentran las funciones básicas para imprimir mensajes en la
salida de errores estándar.  Esto es para que no sean procesados como la salida
estándar de los scripts, que se reservan para poder enviar la información a una
tubería.

No es necesario incluirla ya que se llama desde _lib/common_.

Todas las funciones tienen soporte para traducciones utilizando gettext, por lo
que los mensajes que se completan con variables deben seguir el formato de
_printf_: _%s_ para reemplazar por cadenas, _%d_ para números enteros, etc.

Por ejemplo: _msg "Procesando el nodo %s..." "$node"_

* _msg()_: Información
* _error()_: Mensaje de error
* _warning()_: Alerta
* _fatal\_error()_: Imprime un mensaje de error y termina el programa
  inmediatamente
* _tip()_: Recomendaciones, por ejemplo, cual comando correr a continuación.


## Los comandos

La mayoria de los comandos solo configuran, luego hay que enviar los cambios a
directorio de instalación con el comando _lvpn init install_.
