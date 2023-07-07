# SonarQube
SonarQube Information


Es una herramienta de análisis de código estático autoadministrada que ayuda de forma sistemática a entregar un código limpio y de calidad. SonarQube se integra al flujo de trabajo existente, detectando  problemas en el código para que podamos entrar a corregirlos y lograr una mejora continua en nuestros proyectos. Con SonarQube se puede analizar más de 30 lenguajes de programación diferente y se puede integrar al flujo de CI para garantizar que el nuevo código que se va integrando al proyecto cumpla con los estándares establecidos. 


# Implementación con docker y docker compose (https://docs.sonarqube.org/latest/setup-and-upgrade/install-the-server/)


## 1. Servidor SonarQube

Se crea un archivo de nombre “docker-compose.yml” con el siguiente contenido:

![image](uploads/5a30a1184280d8bcf14130fc26c5e94b/image.png)

* La creación de los siguientes volúmenes ayuda a evitar la pérdida de información al actualizar a una nueva versión o actualizar a una edición superior:

   * sonarqube_data: contiene archivos de datos, como la base de datos H2 integrada y los índices de Elasticsearch.
   * sonarqube_logs: contiene registros de SonarQube sobre acceso, proceso web, proceso CE y Elasticsearch.
   * sonarqube_extensions: contendrá los complementos que instale y el controlador Oracle JDBC si es necesario.
   
_En el servicio de sonarqube hay que asegúrese de usar  los volúmenes  como se muestra en la definición anterior y no vincular una ruta. con esto ayuda a docker para que los complementos se completen correctamente._

![image](uploads/324aff15aa231d93fdb4614a866c7a92/image.png)

* SonarQube requiere una base de datos para almacenar información sobre proyectos, métricas de calidad del código, problemas de código, configuraciones de reglas, usuarios, etc. El propósito principal de almacenar estos datos es proporcionar un repositorio centralizado para el análisis de código, la generación de informes y la visualización de métricas de calidad del código. Además, la base de datos permite mantener un seguimiento histórico de los proyectos y la evolución de la calidad del código a lo largo del tiempo.

Para ejecutar el servidor se debe realizar con el siguiente comando:

`docker compose up -d`

![image](uploads/f148dace11635137110c44107f4b802e/image.png)

Una vez esté arriba, se puede acceder desde el navegador: http://localhost:9000/ 

![image](uploads/7ee407b2f461442569434828e59e3fe4/image.png)

Como manera inicial se puede acceder con las siguientes credenciales:

* Usuario: admin
* Contraseña: admin

Después de ingresar se solicita la actualización de las credenciales:

![image](uploads/53cb4c423982567a6b3d9ca2ce5d161c/image.png)

![image](uploads/f0449ecdf00b5b746632840ead0ce696/image.png)


Posterior a esto, se procede a generar un token de autenticación de SonarQube siguiendo los siguientes pasos:

* Se hace click en el ícono de usuario en la esquina superior derecha y se selecciona "My Account" (Mi cuenta).
* Se ingresa a la pestaña "Security" (Seguridad).
* Se ingresa un nombre para el token en el campo "Generate Tokens" (Generar tokens), de tipo User Token y luego se hace clic en "Generate" (Generar).
* Por último, se copia el token generado; ya que se necesitará en pasos siguientes.

![image](uploads/07e525a88705b97c522dedf2016c7dde/image.png)


**Excepciones**

Si al ejecutar “docker compose up” se genera un error como el siguiente:

`bootstrap check failure [1] of [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]`

![image](uploads/3f06dcae9766fc6c6f2a17c5be363bf0/image.png)

Es porque  el valor de **vm.max_map_count** en tu sistema es demasiado bajo para que Elasticsearch se ejecute correctamente. Elasticsearch requiere que este valor sea configurado adecuadamente en el sistema operativo para funcionar correctamente.

Para corregir este problema, se puede seguir los siguientes pasos según el sistema operativo:

**En Linux:**

1. Abrir una terminal y ejecutar el siguiente comando para verificar el valor actual de vm.max_map_count:

   `sysctl vm.max_map_count`

2. Si el valor es menor que 262144, se puede aumentar temporalmente ejecutando el siguiente comando:

   `sudo sysctl -w vm.max_map_count=262144`

3. Para aumentarlo de forma permanente, se debe editar el archivo **/etc/sysctl.conf**, agregando la siguiente línea al final del archivo:

   `vm.max_map_count=262144`

   Se debe guardar los cambios y ejecutar el siguiente comando para aplicar la configuración:

   `sudo sysctl -p`

**En Windows:**

1. Abrir una ventana de PowerShell con privilegios de administrador.

2. Ejecutar el siguiente comando para aumentar temporalmente el valor de vm.max_map_count:

   `wsl --sysctl vm.max_map_count=262144`

3. Para aumentarlo de forma permanente, se debe ejecutar el siguiente comando para abrir el archivo de configuración:

   `sudo nano /etc/wsl.conf`

   Luego, se debe agregar la siguiente línea al archivo:

   >    [wsl]
   >    kernelCommandLine = "sysctl.vm.max_map_count=262144"

   Ahora, se guardan los cambios y se cierra el archivo.

4. Por último, se debe reiniciar el sistema operativo Windows.

Después de aumentar el valor de **vm.max_map_count**, se puede volver a ejecutar **docker compose up** y el error **"bootstrap check failure"** debería estar resuelto.


## 2. Configuración de SonarQube en el proyecto Frontend

Se debe crear un archivo de nombre **“sonar-project.properties”** en la raíz del proyecto.

Este archivo se utiliza para definir y personalizar la configuración del proyecto que posteriormente se analizará en SonarQube. Para ello se agrega el siguiente contenido al archivo. Observe que se agrega el token generado anteriormente:

![image](uploads/b635436980f453d2c788717a59a7ab33/image.png)

**Configurar Scanner**

A Continuación se configura el archivo “package.json” para permitir ejecutar SonarQube a nivel del proyecto, para ello se agrega una nueva dependencia y un nuevo script:

![image](uploads/82d614139e8d9728476c23667b515dfb/image.png)
 
Luego, se instalan las dependencias del proyecto con el comando npm install, específicamente se instalará “sonarqube-scanner” como una dependencia de desarrollo del proyecto.

![image](uploads/ef1aa56f6b8f7139696df861f821cd32/image.png)

Ahora, se procede a ejecutar el **escaner de SonarQube** a nivel de proyecto con el siguiente comando: `npm run sonar`

![image](uploads/fc693e7c52fb60a1a2c280b3a5689f27/image.png)

![image](uploads/1e4e8d4ab8e92fd02c71c5c2ea3eaac0/image.png)

Después de realizado el análisis del proyecto del frontend, se puede ver el resultado en la interfaz web de sonarqube:

![image](uploads/3cfa2eb0d40e552bc1fd19559b52d80a/image.png)


## 3. Configuración de SonarQube en el proyecto Backend

Tal como se hizo en el paso anterior, se debe crear un archivo de nombre **“sonar-project.properties”** en la raíz del proyecto.

Este archivo se utiliza para definir y personalizar la configuración del proyecto que posteriormente se analizará en SonarQube.  Para ello se agrega el siguiente contenido al archivo. Observe que se agrega el token generado anteriormente:

![image](uploads/cc6de2f00c3e85e45a004aefca440417/image.png)


**Configurar Scanner**

En el caso del backend, una opción es crear un archivo “.yml” para ejecutar el sonar scanner a partir de una imagen docker usando docker compose. Para ello se crea un archivo en la raíz del proyecto con el siguiente contenido:

![image](uploads/521078cdec9fa87ab4031adcc9d09618/image.png)


* Se indica **“network_mode: "host"”** para lograr desde el contenedor del escáner, la conexión con el servidor de sonarqube en el ambiente local “localhost:9000”.

_**Nota**: de esta misma forma se podría agregar un nuevo servicio en el archivo para escanear otro proyecto, por ejemplo el del frontend. Cabe mencionar, que este archivo .yml no necesariamente debe estar dentro del proyecto del backend. Se puede crear un nuevo proyecto que contenga dicho archivo, etc._

Ahora, se procede a ejecutar el **escaner de SonarQube** a nivel de proyecto con el siguiente comando: 

`docker compose -f sonar-scanner.yml up sonar-scanner-uni2-backend`

![image](uploads/4167d8d78e821bde67e1566d6b8acade/image.png)

![image](uploads/eb64d1d2b91d0e9cad742b4c4f2fed45/image.png)


Después de realizado el análisis del proyecto del backend, se puede ver el resultado en la interfaz web de sonarqube:

![image](uploads/535fa4a025a1ab546050166ce20fe222/image.png)


## Integración con SonarLint

https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarlint-vscode#connected-mode 

El modo conectado de SonarLint conecta SonarLint con el proyecto de SonarQube y brinda beneficios adicionales que no obtendría al usar SonarLint o SonarQube de forma independiente.

Al usar SonarLint, el perfil de calidad (Quality Profile) de **Sonar Way** es el que se usa de forma predeterminada y **los usuarios pueden personalizar su conjunto de reglas**. Si se utiliza un perfil de calidad diferente en SonarQube, es posible que surjan nuevos problemas en SonarQube aunque SonarLint todo se vea limpio.

Con el modo conectado, se aplica el mismo conjunto de reglas personalizadas tanto en el IDE como en SonarQube, y se notifica en el IDE cuando la instancia local no cumple con los estándares de puerta de calidad [Quality Gates](https://docs.sonarqube.org/latest/user-guide/quality-gates/) del proyecto.


**Configuración del modo conectado en VS** 

1. Se accede a la opción de Sonar Lint, luego se debe abrir el Modo Conectado, y seleccionar Add sonarQube Connection:

![image](uploads/ce0705670b6cca920b9e08e2437ad761/image.png)


2. A Continuación se debe ingresar los datos solicitados y por ultimo dar click en Save Connection:

![image](uploads/b4f5f47630ba31cece45a2c2e8902553/image.png)

![image](uploads/08605ec129019ccb2027c121f51610ce/image.png)


3. Como paso siguiente, se debe enlazar el proyecto que se va a analizar. Para ello, dirigirse a la vista MODO CONECTADO DE SONARLINT en VSCode Explorer y seleccionar “Agregar enlace de proyecto” para agregar la conexión deseada.

![image](uploads/3a304d5323dc46c1e555a9d7c3fa3a31/image.png)

Se selecciona el proyecto.

![image](uploads/0328b0497e2521c6f7ed2d3008edf699/image.png)

Al final, se debe ver reflejado como en la imagen siguiente:

![image](uploads/1e87123ac63bc69bb2084ee36f730eb8/image.png)

Con lo anterior, ya se puede hacer uso del modo conectado de Sonar Lint.

