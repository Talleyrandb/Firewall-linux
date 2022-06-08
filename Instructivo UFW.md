# Firewall UFW para Sistemas Operativos Ubuntu y CentOS

El kernel de Linux proporciona capacidades para supervisar y controlar el tráfico de red. Estas capacidades se exponen al usuario final a través del firewall iptables. Sin embargo, iptables es bastante complicado y confuso por lo que es más fácil emplear `UFW uncomplicated firewall`. UFW es como un front-end para el firewall iptables. Esto facilita el proceso de gestión de las reglas del firewall iptables que le dicen al kernel de Linux como gestionar con el tráfico de red de acuerdo con sus necesidades.

UFW funciona permitiéndole configurar reglas que
> Permiten o deniegan el tráfico 

> Contolan el tráfico de entrada o de salida

> Controlan origen y destino de los puertos

## Instalación y configuración del firewall (opcional)

```shell
yum install epel-release -y
```
```shell
yum install --enablerepo="epel" ufw -y
```
```shell
apt-get install ufw
```
## Activación del firewall (opcional)

Si al ejecutar el comando `ufw status` aparece el mensaje `Status: inactive`, significa que el firewall aún no está habilitado en el sistema. Tendrá que ejecutar el siguiente comando para habilitarlo.
```shell
ufw enable
```
>NOTA: Por defecto, cuando se activa UFW se bloquea el acceso externo a todos los puertos del servidor. En la práctica, eso significa que, si está conectado a un servidor a través de `SSH` y habilita ufw antes de permitir el acceso a través del puerto SSH, será desconectado. Asegúrese de seguir las instrucciones de cómo habilitar el acceso SSH que aparece en esta guía antes de habilitar el firewall si ese es su caso.
## Desactivar el firewall (opcional)
Si por alguna razón necesita desactivar UFW, puede hacerlo con el siguiente comando:
```shell
ufw disable
```
> Tenga en cuenta que este comando desactivará por completo el servicio de firewall en el sistema.
## Verificar el estado del firewall (opcional)

Para ver lo que está actualmente bloqueado o permitido en el firewall, puede utilizar el parámetro `verbose` cuando ejecute `ufw status`, como sigue:
```shell
ufw status verbose
```
## Eliminar una regla UFW (opcional)
Para eliminar una regla previamente configurada en UFW, utilice el comando `ufw delete` seguido de la regla `allow` o `deny` y la especificación de destino `from` . El siguiente ejemplo eliminaría una regla previamente configurada para permitir todas las conexiones desde una dirección IP de 181.173.61.205:
```shell
ufw delete allow from 181.173.61.205
```
Otra forma de especificar qué regla se quiere eliminar es proporcionando el ID de la regla. Esta información se puede obtener con el siguiente comando:
```shell
ufw status numbered
```
```shell
Output
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] Anywhere                   DENY IN     182.145.11.123             
[ 2] Anywhere on eth0           ALLOW IN    182.145.11.127 
```
En la salida, se puede ver que hay dos reglas activas. La primera regla, con los valores resaltados, deniega todas las conexiones procedentes de la dirección IP 182.145.11.123. La segunda regla permite las conexiones en la interfaz `on eth0` procedentes de la dirección IP 182.145.11.127.

Como por defecto UFW ya bloquea todos los accesos externos a no ser que estén explícitamente permitidos, la primera regla es redundante, así que puede eliminarla. Para eliminar una regla por su ID, ejecute:
```shell
ufw delete 1
```
Se le pedirá que confirme la operación y que se asegure de que el ID que está proporcionando se refiere a la regla correcta que desea eliminar.
```shell
Output 
Deleting:  
	deny from 182.145.11.123 
Proceed with operation (y|n)? y 
Rule deleted
```
Si vuelve a listar sus reglas con `ufw status`, verá que la regla fue eliminada.

## Bloquear y permitir el tráfico por defecto (mandatorio)

Indique a UFW que bloquee todo el tráfico entrante y que permita todo el tráfico saliente y después, permita únicamente los puertos necesarios.
```shell
ufw default allow outgoing
```
```shell
ufw default deny incoming
```
# Permitir (opcional)

## Permitir una dirección IP (opcional)

Para permitir todas las conexiones de red que se originen desde una dirección IP específica, ejecute el siguiente comando, sustituyendo la dirección IP resaltada por la dirección IP a la que desea permitir el acceso:
```shell
ufw allow from 181.173.61.205
```

## Permitir conexiones entrantes a una interfaz de red (opcional)

Para permitir las conexiones entrantes desde una dirección IP específica a una interfaz de red concreta, ejecute el siguiente comando, sustituyendo la dirección IP resaltada por la dirección IP que desea permitir.
```shell
ufw allow in on eth0 from 182.145.11.127
```
El parámetro `in` indica a ufw que aplique la regla sólo a las conexiones entrantes, y el parámetro `on eth0` especifica que la regla se aplica sólo a la interfaz `eth0`.
## Permitir SSH (mandatorio)

Al trabajar con servidores remotos, debe asegurarse de que el puerto SSH está abierto a las conexiones para que pueda entrar en su servidor de forma remota.

El siguiente comando habilitará el perfil de la aplicación OpenSSH UFW y permitirá todas las conexiones al puerto SSH por defecto en el servidor
```shell
ufw allow OpenSSH
```
Aunque es menos fácil de usar, una sintaxis alternativa es especificar el número de puerto exacto del servicio SSH, que suele ser el puerto por defecto TCP 22.
```shell
ufw allow 22
```

## Permitir SSH entrante desde una dirección IP o subred específica (opcional)

Para permitir conexiones entrantes desde una dirección IP o subred específica, incluirá una directiva `from` para definir el origen de la conexión. Esto requerirá que también especifique la dirección de destino con un parámetro `to`. Para bloquear esta regla sólo a SSH, limitarás el `proto` (protocolo) a TCP y luego usará el parámetro `port` y lo establecerá en 22, el puerto por defecto de SSH.

El siguiente comando permitirá sólo conexiones SSH provenientes de la dirección IP 203.0.113.103:
```shell
ufw allow from 203.0.113.103 proto tcp to any port 22
```
También puede utilizar una dirección de subred como parámetro de origen para permitir las conexiones SSH entrantes de toda una red
```shell
ufw allow from 203.0.113.0/24 proto tcp to any port 22
```
## Permitir en Nginx HTTP/HTTPS (mandatorio)

El servidor web Nginx establece unos cuantos perfiles UFW diferentes dentro del servidor. Una vez que tenga Nginx instalado y habilitado como servicio, ejecute el siguiente comando para identificar qué perfiles están disponibles:
```shell
ufw app list | grep Nginx
```
Para permitir tanto el tráfico HTTP como el HTTPS, elija Nginx Full. De lo contrario, elija Nginx HTTP para permitir sólo HTTP o Nginx HTTPS para permitir sólo HTTPS.

El siguiente comando permitirá el tráfico HTTP y HTTPS en el servidor (puertos 80 y 443):
```shell
ufw allow "Nginx Full"
```
## Permitir en Apache HTTP/HTTPS (mandatorio)

El servidor web Apache establece unos cuantos perfiles UFW diferentes dentro del servidor. Una vez que tenga Apache instalado y habilitado como servicio, ejecute el siguiente comando para identificar qué perfiles están disponibles:
```shell
ufw app list | grep Apache
```
Para habilitar tanto el tráfico HTTP como el HTTPS, elija Apache Full. En caso contrario, elija Apache para HTTP o Apache Secure para HTTPS.
El siguiente comando permitirá el tráfico HTTP y HTTPS en el servidor (puertos 80 y 443):
```shell
ufw allow "Apache Full"
```
## Permitir todo el HTTP entrante (puerto 80) (opcional)

Los servidores web, como Apache y Nginx, suelen escuchar las peticiones HTTP en el puerto 80. Si su política por defecto para el tráfico entrante está configurada para descartar o denegar, tendrá que crear una regla UFW para permitir el acceso externo en el puerto 80. Puede utilizar el número de puerto o el nombre del servicio (http) como parámetro de este comando.
Para permitir todas las conexiones HTTP entrantes (puerto 80), ejecute:
```shell
ufw allow http
```
## Permitir todo el HTTP entrante (puerto 80) (opcional)

Los servidores web, como Apache y Nginx, suelen escuchar las peticiones HTTP en el puerto 80. Si su política por defecto para el tráfico entrante está configurada para descartar o denegar, tendrá que crear una regla UFW para permitir el acceso externo en el puerto 80. Puede utilizar el número de puerto o el nombre del servicio (http) como parámetro de este comando.

Para permitir todas las conexiones HTTP entrantes (puerto 80), ejecute:
```shell
ufw allow http
```
Una sintaxis alternativa es especificar el número de puerto del servicio HTTP:
```shell
ufw allow 80
```
## Permitir todos los HTTPS entrantes (puerto 443) (mandatorio)
HTTPS normalmente se ejecuta en el puerto 443. Si su política por defecto para el tráfico entrante está configurada para descartar o denegar, tendrá que crear una regla UFW para permitir el acceso externo en el puerto 443.

Puede utilizar el número de puerto o el nombre del servicio (https) como parámetro de este comando.
```shell
ufw allow https
```
Una sintaxis alternativa es especificar el número de puerto del servicio HTTP:
```shell
ufw allow 443
```
## Permitir todo el tráfico entrante HTTP y HTTPS (opcional)

Si quiere permitir tanto el tráfico HTTP como el HTTPS, puede crear una única regla que permita ambos puertos. Este uso requiere que también defina el protocolo con el parámetro `proto`, que en este caso debe establecerse como tcp.
Para permitir todas las conexiones entrantes HTTP y HTTPS (puertos 80 y 443), ejecute:
```shell
ufw allow proto tcp from any to any port 80,443
```
## Permitir la conexión del DNS por puerto UDP (opcional)
el servicio DNS funciona en el puerto 53 tanto para protocolo TCP como UDP, para especificar que este servicio escuche por el protocolo UDP ejecute el siguiente comando:
```shell
ufw allow 53/udp
```
## Permitir la conexión de MySQL desde una dirección IP o subred específica (opcional)

MySQL escucha las conexiones de los clientes en el puerto TCP 3306. Si su servidor de base de datos MySQL está siendo utilizado por un cliente en un servidor remoto, necesitará crear una regla UFW para permitir ese acceso.

Para permitir conexiones entrantes de MySQL desde una dirección IP o subred específica, utilice el parámetro `from` para especificar la dirección IP de origen y el parámetro `port` para establecer el puerto 3306 de destino.
El siguiente comando permitirá que la dirección IP 203.0.113.103 se conecte al puerto MySQL del servidor:
```shell
ufw allow from 203.0.113.103 to any port 3306
```
Para permitir que toda la subred 203.0.113.0/24 pueda conectarse a su servidor MySQL, ejecute el siguiente comando:
```shell
ufw allow from 203.0.113.0/24 to any port 3306
```
## Permitir la conexión de PostgreSQL desde una dirección IP o subred específica (opcional)

PostgreSQL escucha conexiones de clientes en el puerto 5432. Si su servidor de base de datos PostgreSQL está siendo utilizado por un cliente en un servidor remoto, debe asegurarse de permitir ese tráfico.

Para permitir las conexiones entrantes de PostgreSQL desde una dirección IP o subred específica, especifique el origen con el parámetro `from`, y establezca el puerto en 5432:
```shell
ufw allow from 203.0.113.103 to any port 5432
```
Para permitir que toda la subred 203.0.113.0/24 se pueda conectar a su servidor PostgreSQL, ejecute el siguiente comando:
```shell
ufw allow from 203.0.113.0/24 to any port 5432
```
# Denegar (opcional)

## Bloquear una dirección IP (opcional)

Para bloquear todas las conexiones de red que se originan en una dirección IP específica, ejecute el siguiente comando, sustituyendo la dirección IP resaltada por la dirección IP que desea bloquear:
```shell
ufw deny from 182.145.11.123
```
## Bloquear una subred (opcional)

Para bloquear una subred completa, puede utilizar la dirección de la subred como parámetro from en el comando `ufw deny`. Esto bloquearía todas las direcciones IP de la subred de ejemplo 203.0.113.0/24, ejecute el siguiente comando:
```shell
ufw deny from 203.0.113.0/24
```

## Bloquear las conexiones entrantes a una interfaz de red (opcional)

Para bloquear las conexiones entrantes desde una dirección IP específica a una interfaz de red concreta, ejecute el siguiente comando, sustituyendo la dirección IP resaltada por la dirección IP que desea bloquear:
```shell
ufw deny in on eth0 from 182.145.11.123
```
## Bloquear el correo SMTP saliente (opcional)

Los servidores de correo, como Sendmail y Postfix, suelen utilizar el puerto TCP 25 para el tráfico SMTP. Si su servidor no debería enviar correo saliente, es posible que quiera bloquear ese tipo de tráfico. Para bloquear las conexiones SMTP salientes, Esto configura su firewall UFW para que rechace todo el tráfico saliente en el puerto 25. Si necesita rechazar las conexiones salientes en un número de puerto diferente, puede repetir este comando y sustituir 25 por el número de puerto que desea bloquear. ejecute el siguiente comando:
```shell
ufw deny out 25
```

# Trabajar con aplicaciones (opcional)

## Lista de perfiles de aplicación disponibles (opcional)
Algunas aplicaciones que dependen de las comunicaciones de red, para esto se suele configurar un perfil UFW que permita la conexión desde direcciones externas. Esto suele ser lo mismo que ejecutar `ufw allow from`, con la ventaja de proporcionar un acceso directo que abstrae los números de puerto específicos que utiliza un servicio y proporciona una nomenclatura fácil de usar a los servicios referenciados.

Para listar qué perfiles están actualmente disponibles, ejecute el siguiente comando:
```shell
ufw app list
```

Si ha instalado un servicio, como un servidor web u otro software dependiente de la red, y el perfil no está disponible en UFW, asegúrese primero de que el servicio está activado. En el caso de los servidores remotos, lo normal es que tenga disponible OpenSSH:
```shell
Output
Available applications:
OpenSSH
```

## Habilitar el perfil de aplicación (opcional)

Para habilitar un perfil de aplicación UFW, ejecute `ufw allow` seguido del nombre del perfil de aplicación que desea habilitar, que puede obtener con un comando `ufw app list`. En el siguiente ejemplo, se habilitar el perfil OpenSSH, que permitirá todas las conexiones SSH entrantes en el puerto SSH predeterminado.
```shell
ufw allow “OpenSSH”
```

Recuerde citar los nombres de los perfiles que constan de varias palabras, como Nginx HTTPS.

## Desactivar el perfil de aplicación (opcional)
Para desactivar un perfil de aplicación que haya configurado previamente en UFW, deberá eliminar su regla correspondiente. Por ejemplo, considere la siguiente salida:
```shell
ufw status
```
```shell
Output
Status: active
To 					Action      From
--              	------      ----
OpenSSH				ALLOW		Anywhere                               
Nginx Full			ALLOW		Anywhere                  
OpenSSH (v6)		ALLOW		Anywhere (v6)                   
Nginx Full (v6)		ALLOW		Anywhere (v6)
```

Esta salida indica que el perfil de aplicación completa de Nginx está actualmente activado, permitiendo todas y cada una de las conexiones al servidor web tanto a través de HTTP como de HTTPS. Si quiere permitir sólo las solicitudes HTTPS desde y hacia su servidor web, tendría que habilitar primero la regla más restrictiva, que en este caso sería Nginx HTTPS, y luego desactivar la regla Nginx Full actualmente activa:
```shell
ufw allow "Nginx HTTPS"
```
```shell
ufw delete allow "Nginx Full"
```

Recuerda que puede listar todos los perfiles de aplicaciones disponibles con ejecutar el comando:
```shell
ufw app list.
```

fin de la guía 
