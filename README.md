Cómo configurar Github en Linux con Proxy y no morir en el intento
===

Hoy he logrado configurar mi cuenta de Github luego de cientos de infructuosos intentos por lo que voy a compartir mis nuevos conocimientos adquiridos con ustedes.

Creando el repo en Github
----

Primero que nada, hay que generar un repositorio en Github, es simple, hay que registrarse y luego crear un repositorio vacío en la página.

Github nos dará una URL al estilo: 

https://github.com/TuNombreDeUsuario/ElRepositorioQueCreaste.git

que nos permitirá clonarlo en la máquina en la que estemos. En mi caso es una máquina que se encuentra en una red corporativa con un proxy http (una hermosura, snif).

Configurando el proxy
----

Antes que nada hay que editar la configuración de git para establecer el proxy que debe usar, la configuración se encuentra en ~/.gitconfig, si no se encuentra hay que crear el archivo y establecer los siguientes parámetros en él:

    [http]
    proxy = http://proxy_url:proxy_port

Con lo que obtendremos el acceso http a través del proxy.

Clonando el repo
----

Ahora lo que tenemos que hacer es clonar el repo en la máquina, nos vamos a /ruta/de/nuestros/proyectos/ con

    $ cd /ruta/de/nuestros/proyectos/

y clonamos el repositorio

    $ git clone https://github.com/TuNombreDeUsuario/ElRepositorioQueCreaste.git

Todo debería haber funcionado como se esperaba, git se debería haber podido conectar vía http para clonar el repo.

Agregando archivos
----

Lista la clonación, entramos ahora al directorio del proyecto, en nuestro caso sería

    $ cd ElRepositorioQueCreaste

Ahora podemos crear el típico README.MD y escribir en él la descripción del proyecto (y/o agregar todos los archivos que queramos).

Obviamente, tenemos que agregarlos utilizando

    $ git add listado_de_archivos a_agregar

o simplemente

    $ git add *

que agregará todos los archivos del directorio.

Ahora la parte difícil, tenemos que subir nuestros cambios a Github, pero Github debe saber que somos nosotros, por lo que tendremos que crear un par clave-privada/clave-pública para poder conectarnos via ssh y subir nuestros cambios... Parece difícil, pero no lo es tanto (sí, lo digo después de haber hecho varios intentos infructuosos, pero con buenos resultados al final).

Generando el par de claves
----

Primero tenemos que verificar que no existan, si ya nos hemos conectado vía ssh es posible que ya estén ahí y si no lo hemos hecho nunca habrá que generarlas, veamos si existen:

    $ ls ~/.ssh/

Si en el listado aparecen id_rsa.pub o id_dsa.pub, las claves ya han sido generadas anteriormente, por lo que no habría que generarlas y pasaríamos al punto siguiente, sino generémoslas:

    $ ssh-keygen -t rsa -C "tu_email@dominio.com"

El comando anterior crea una clave ssh utilizando el email como etiqueta. Responderá diciendo que se está generando el par de claves rsa y nos pedirá el nombre del archivo para guardarla (el que es por defecto es el ~/.ssh/id_rsa), presionamos Enter para dejar el valor por defecto. Ahora tenemos que agregarla:

    $ ssh-add id_rsa

El sistema nos pedirá una passphrase y también deberemos repetirla para confirmarla. La passphrase es una clave que queda guardada en la máquina y permite que nos conectemos vía ssh. Aunque se encuentra en la máquina, no está en texto claro, es decir que no puede comprenderse y gracias a que está guardada, no deberemos escribirla nuevamente para conectarnos.

Un ejemplo de passphrase es algo al estilo: "unicamente palomas galopan sintiendo comezon" o "ciempieces ondulantes vuelan sobre honduras". Como escribí antes, no nos importa mucho ya que no volveremos a escribirla luego de confirmarla.

Luego de todo este proceso obtendremos el resultado, es decir la clave de identificación y la clave pública (que es la que nos interesa) con su fingerprint (su huella). La huella son 16 números hexadecimales separados por dos puntos (:) y es lo que nos permitirá identificar la clave.

Agregando la clave a Github
----

Para obtener la clave generada hacemos

    $ cat ~/.ssh/id_rsa.pub

Copiamos el resultado vamos al apartado "Account Settings > SSH Keys" de Github y hacemos click en "Add SSH Key". Pegamos la clave en el campo Key y le damos un nombre (este nombre nos permite recordar de dónde viene esta clave). Hacemos click en "Add Key", nos pedirá contraseña (la de Github) y listo, nos mandará al listado de claves donde podremos verificar el fingerprint de la misma con el generado en nuestra máquina, si corresponden, todo estuvo bien.

Como por un túnel
----

Hasta este punto, todo parece de una belleza y simpleza increible, aunque no todo lo que reluce es oro y nuestro proxy está hecho de papel maché pintado de dorado, no porque Squid sea una bazofia, sino porque a mi entender debería ser un proxy transparente y únicamente para http, pero el filtrado es para todo y con WPAD lo que lo hace todavía más complicado de configurar, en fin... Tendremos que instalar un túnel para conectarnos vía SSH sobre HTTP.

Como mi entorno es Ubuntu, solo tengo que utilizar apt para instalar corkscrew (algo simple de instalar y fácil de configurar)

    $ sudo apt-get install corkscrew

Cuando se haya instalado tendremos que editar el archivo ~/.ssh/config (si no existiera tendremos que crearlo) y agregar la siguiente línea

    ProxyCommand /ruta/a/corkscrew proxy_url proxy_port %h %p

Obviamente colocando la ruta correcta (generalmente /usr/bin/corkscrew), cambiando proxy_url por la url de nuestro proxy y proxy_port por el puerto que tenemos que utilizar.

Voilá, ahora cada vez que se quiera conectar vía ssh ejecutará el comando corkscrew con los parámetros que se le suministren.

Último detalle de configuración
----

Si solamente vamos a utilizar Github para gestionar nuestros repos, ciertas configuraciones globales funcionarán de maravilla, como en mi caso quiero poder gestionar repositorios no únicamente vía Github, configuraré localmente (en cada repositorio) la información de conexión al mismo.

Dentro de la carpeta de nuestro proyecto se encuenta una carpeta llamada .git/ dicha carpeta contiene información sobre los objetos y todo el historial del repo, eso es interesante pero no nos incumbe ahora, lo que realmente nos interesa es el archivo de configuración, o sea .git/config

Veremos secciones denotadas por corchetes y un nombre entre ellos, ejemplos son [core], [remote "origin"], [branch "master"], etc. Este archivo de configuración se mezcla ~/.gitconfig (el que editamos para configurar el proxy en el segundo punto) y permite sobreescribir la configuración anterior, es decir que si queremos que los cambios sean globales, editaríamos ~/.gitconfig y si pretendemos que los cambios sean locales (de cada proyecto) editaremos .git/config

Tendremos que agregar la sección [user] y setear las claves "name" y "email" (sin comillas) a los valores de nuestra cuenta en Github y verificar que en la sección [remote "origin"] la clave url esté seteada de la siguiente manera:

    url = ssh://git@github.com/TuNombreDeUsuario/ElRepositorioQueCreaste.git

para que nos permita conectarnos vía ssh.

Testeando todo
----

La mejor forma de testear todo, como sabemos es hacer

    $ git pull

que nos conectaría con Github y mandaría todos los cambios hacia allá. Es el paso obligado y con la configuración anterior, todo debería funcionar correctamente.

¿Y si no anda?
----

Bueno, hay muchas cosas por probar: que el proxy funcione, que permita las conexiones https, que nos podamos conectar via ssh...

Una buena para probar es intentando conectarnos con Github vía ssh pero con salida verborrágica:

    $ ssh -vT git@github.com

Lo que nos va a tirar un chorro de información sobre el intento de autenticación y los resultados del mismo, eso nos puede ayudar a detectar el problema (o no... pero es un primer acercamiento).

De todas maneras, hay mucha información en internet pero poca para este escenario exclusivo (sí exclusivo no solo porque es casi único, sino porque nos vive excluyendo de una buena conexión a internet, snif snif). Uno de los grandes lugares para encontrar información es https://help.github.com/categories/56/articles que recopila los grandes interrogantes aunque stackoverflow.com tiene más variedad de escenarios y algunas soluciones bastante imaginativas.

El listado de los TO-DOs
----

Como escribí anteriormente, la configuración de algunas cosas es global y afecta otras configuraciones precedentes, por ejemplo, la configuración del proxy en ~/.ssh/config altera toda las conexiones vía ssh, una posibilidad para solucionar este problema sería crear un script que, dependiendo de hacia dónde nos queramos conectar establezca o no la configuración del proxy, pero esto lo veremos más adelante.
