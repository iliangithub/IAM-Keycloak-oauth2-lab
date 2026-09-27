# 1.0 Introducción.

El objetivo de esta práctica no solo es profundizar en las disciplinas 

# 2.0 Definiciones.

Antes de empezar con la resolución de la práctica, es de interés primero comprender la teoría y el porqué de cada cosa.

## 2.1 IAM (Identity and Access Management) o Gestión de identidades y accesos: el marco.

Sin entrar en gran profundidad vamos a definir un poco esto.

La ciberseguridad, es un concepto general que engloba múltiples disciplinas y abarca tanto un plano físico como uno lógico.

Al hablar de un plano lógico o físico, nos referimos a que no solo hablamos de software, prevenir malware, ciberataques, reducir superficies de ataque... Si no cosas que parecen tan ajenas, pero realmente forman parte de la disciplina, como el uso de cámaras de seguridad, sensores, escáneres, etc...

Cuando hablamos de múltiples disciplinas mencionamos:
- La ingeniería de redes (aunque no sea 100% específico de la ciberseguridad).
  - Segmentación y microsegmentación (VLAN, zonas, DMZ).
  - Cortafuegos, NGFW y listas de control de acceso.
  - VPN e IPsec, acceso remoto.
  - Control de acceso a la red (NAC, 802.1X).
  - Proxies, filtrado DNS y de navegación.
  - Detección e inspección a nivel de red (IDS, IPS, NDR).
  - Arquitecturas de confianza cero aplicadas al transporte.
- La infraestructura.
  - Bastionado de sistemas operativos y servicios.
  - Gestión de parches y de configuración.
  - Servidores de directorio y servicios centrales.
  - Virtualización, contenedores y orquestación (Docker, Kubernetes).
  - Nube pública (AWS, Azure) y su modelo de responsabilidad compartida.
  - Copias de seguridad, recuperación y continuidad técnica.
  - Cifrado en reposo y gestión de claves y secretos.
  - Automatización e infraestructura como código, DevSecOps.
- El blueteam y todo lo que engloba.
  - SOC: monitorización continua, SIEM, triaje y respuesta de primer y segundo nivel.
  - DFIR: análisis forense digital y respuesta a incidentes.
  - CTI: inteligencia de amenazas, indicadores, atribución, informes.
  - CTH: búsqueda proactiva de amenazas sin alerta previa.
  - Ingeniería de detección: creación y afinado de reglas (Sigma, YARA, KQL).
  - Gestión de vulnerabilidades y de exposición.
  - Purple Team: validación de detecciones contra técnicas reales.
- RedTeam.
  - Test de intrusión sobre red, sistemas y aplicaciones.
  - Simulación de adversario y ejercicios de emulación.
  - Ingeniería social, phishing y pretexto.
  - Seguridad ofensiva de aplicaciones y desarrollo de exploits.
  - OSINT y reconocimiento.
Seguridad física ofensiva (intrusión, clonado de tarjetas).
- Gestión de proyectos, etc.
  - Gobierno, riesgo y cumplimiento (GRC).
  - Normativa y marcos: ISO 27001, ENS, NIS2, DORA, PCI DSS.
  - Análisis y tratamiento de riesgos.
  - Políticas, procedimientos y auditoría interna.
  - Continuidad de negocio y gestión de crisis.
  - Concienciación y formación del personal.
- Y por último, IAM.
  - Autenticación: MFA, SSO, gestión de credenciales.
  - Federación de identidad: SAML, OAuth 2.0, OpenID Connect.
  - Aprovisionamiento y ciclo de vida: SCIM, altas, cambios y bajas.
  - Gobierno de identidades (IGA): recertificación, segregación de funciones, catálogo de accesos.
  - PAM: cuentas privilegiadas, bóvedas de secretos, acceso de emergencia.
  - Gestión de directorios: Active Directory, LDAP, Entra ID.
  - CIAM: identidad de clientes y usuarios externos.

> [!NOTE]
> (Hemos mencionado puestos de trabajo / títulos, pero eso muy subjetivo y cada empresa pone el nombre que le da la gana realmente). SOC, DFIR, CTI y CTH no son sólo puestos, si no más bien áreas y funciones.
>

Por lo tanto, el IAM es una disciplina de la ciberseguridad.<br>
Que gobierna sobre el ciclo de vida de las identidades digitales (personas, cuentas de servicio y dispositivos) y de los accesos o las cosas que pueden hacer.

<b>IAM responde a cuatro preguntas muy importantes:</b>
- Quién eres (autenticación).
- Qué puedes hacer (autorización).
- Quién autorizó que pudieras hacerlo (gobierno y trazabilidad).
- Si eso sigue siendo necesario hoy (recertificación).

## 2.2  Autenticación y autorización.

Son dos procesos secuenciales y separables.

Es decir:
1. Primero se establece quién eres. Autenticación.
2. Y una vez autenticado, se establece qué se te permite. Autorización.

Para establecer quién eres hay muchas formas y ninguna reemplaza a la otra porque cada uno tiene su caso de uso especial, ninguna es mejor que otra y a menudo se combinan. Para autenticarte:
- Se prueba algo que sabes, algo que sólo tú sabes.
  - Una contraseña, un PIN, una respuesta a una pregunta.
- Algo que sólo tú tienes, una posesión.
  - Una llave, una tarjeta magnetica (banco), una tarjeta de proximidad (RFID), un token físico, un certificado digital, el móvil que te servirá como MFA o recibirá SMS o que gracias a él tiene una tarjeta SIM que te identifica.
- Algo que eres.
  - La biometría, huella, retina, fisonomía, voz. Rasgos físicos inherentes.

Para establecer qué puedes saber, es decir, para autorizarte: 
- Primero debes autenticarte.

Y ahora, se usan distintos modelos para saber qué puedes hacer:
- Basado en roles (RBAC).
  - Es decir, se crea un rol "administrador" y es el rol "administrador" quien tiene los permisos.
  - Y eres tú a quién se le asigna ese rol. Por lo tanto, heredas los permisos. 
- Basado en atributos (ABAC).
  - La decisión depende del contexto, no solo quién eres, si no del departamento, la hora, el dispositivo, la red desde la que conectas o el importe de la operación.
- Basado en listas de control de acceso (ACL).
  - Es decir, que los permisos van puestos directamente en un recurso, este recurso guarda una lista de quién puede hacer sobre él. Como los permisos de un fichero.

En la autorización es importante tener en cuenta:
- El principio del mínimo privilegio, darle a cada identidad solo lo imprescindible para su función y nada más.
- El principio de denegación por defecto. Además de eso, lo que no tiene permitido de forma explícita, se deniega.
- Principio de mediación completa, es decir que cada intento de acceso se comprueba en el momento, sin dar por válida una decisión anterior.

Por último, conviene distinguir dos cosas que se confunden: el permiso y la decisión.

- El permiso concedido (en el sector se le llama entitlement) es estático.
  - Alguien lo solicitó, alguien lo aprobó y queda guardado en un sistema. Se administra: se solicita, se aprueba, se revisa y se revoca.
- La decisión es dinámica.
  - Se calcula en cada intento, con los datos de ese momento, y puede salir que no aunque el permiso siga concedido.

Es decir, tener el permiso no garantiza obtener el acceso. Son dos momentos distintos y dos sistemas distintos.

Un ejemplo físico lo deja claro. Tienes una tarjeta de acceso al edificio y estás dado de alta en la puerta principal: eso es el permiso, y sigue ahí aunque estés de vacaciones. Pasas la tarjeta a las tres de la madrugada y el torno te deniega el paso porque el horario de esa puerta es de 7 a 22: eso es la decisión.

Y de ahí sale una consecuencia práctica que importa en IAM: una recertificación revisa permisos concedidos, mientras que un registro de accesos recoge decisiones. Un permiso que nadie ha usado nunca no aparece en ningún registro, y sigue siendo riesgo porque está ahí esperando.

## 2.3 Certificación y recertificación de accesos.

La certificación de accesos, también llamada recertificación o revisión de accesos, es el proceso periódico en el que un responsable revisa los permisos que tienen asignadas las personas a su cargo y confirma, uno por uno, si siguen siendo necesarios.

Existe porque los permisos se acumulan. Cada cambio de puesto, cada proyecto y cada sustitución temporal añade accesos, pero casi nadie retira los anteriores. A ese efecto se le llama acumulación de privilegios (privilege creep), y sin revisiones periódicas acaba produciendo personas que, sobre el papel, pueden hacer casi cualquier cosa.

Cómo funciona una campaña de recertificación:

Se define el alcance. Qué aplicaciones, qué permisos y qué colectivo se revisan.
Se asigna un revisor. Normalmente el responsable jerárquico de cada persona, o el propietario de la aplicación.
El revisor decide sobre cada permiso: mantener o revocar.
Lo revocado se ejecuta en los sistemas destino.
Y queda la evidencia: quién revisó qué, cuándo y con qué resultado.

La frecuencia depende de la criticidad. Lo habitual es anual o semestral para accesos normales, y bastante más seguido para los accesos privilegiados.

Es importante entender que la recertificación es una red de seguridad, no un sustituto del proceso de bajas. Si el circuito de baja funciona, los accesos se retiran cuando la persona se va o cambia de puesto. La recertificación está para cazar lo que ese circuito dejó pasar.

Y en banca no es opcional: es un requisito de auditoría y de cumplimiento normativo, y de ahí que interese tanto la evidencia como el resultado.

## 2.4  Proveedor de identidad y parte confiante

Un proveedor de identidad (IdP, Identity Provider) es el sistema que se encarga de autenticar y de emitir afirmaciones verificables sobre quién es la identidad autenticada.

La parte confiante es la aplicación que confía en esas afirmaciones en lugar de comprobar credenciales por su cuenta. Según el protocolo recibe un nombre u otro:

En OpenID Connect se le llama parte confiante (relying party).
En SAML se le llama proveedor de servicio (service provider).

Es decir, la aplicación deja de preguntar contraseñas y pasa a preguntar al proveedor de identidad quién eres.

El valor del modelo está en concentrar el punto de verificación. Si veinte aplicaciones validan contraseñas por separado, hay veinte almacenes de credenciales que proteger, veinte políticas que mantener y veinte sitios donde olvidarse de cerrar un acceso. Delegando, hay uno solo. Y de ahí sale, como efecto, el inicio de sesión único (SSO): una vez autenticado ante el proveedor, el resto de aplicaciones aceptan esa autenticación sin volver a pedir credenciales.

Esa confianza no es automática, se establece antes. La aplicación y el proveedor se configuran mutuamente (registro del cliente, metadatos, secretos o claves públicas de firma), y a partir de ahí la aplicación puede comprobar que la afirmación que recibe viene realmente del proveedor y no ha sido manipulada.

En esta práctica los papeles son:

Keycloak es el proveedor de identidad.
La aplicación web y la API que protegeremos son las partes confiantes.

## 2.5  OAuth 2.0: delegacion de autorizacion

## 2.6  OpenID Connect: la capa de identidad

## 2.7  Keycloak: la implementacion concreta

## 2.8  Vocabulario propio de Keycloak

## 2.9  Que es una API y que significa REST

Una API (Application Programming Interface) es la cara pública de un programa: el conjunto de operaciones y funciones que declara al exterior, qué datos hay que enviarle en cada una y qué devuelve.

Quien use una API no tiene que saber cómo está hecha por dentro, ni qué lenguaje ni qué base de datos usa. Y se puede cambiar el interior, es decir, la implementación, sin romper a quien te usa la API, siempre que el contrato se respete. Si cambias qué hace una operación o qué devuelve, ahí sí rompes a quien te consume.

Una API no es solo vía web, ni todo es HTTP ni JSON. Una biblioteca de C tiene API. Pero a día de hoy, al decir API nos referimos normalmente a una API web.

## 2.10  Tokens: concepto, tipos y estructura JWT

## 2.11 Los tres tipos tokens y sus funciones

## 2.12 Flujos de concesion utilizados

## 2.13 Endpoints del servidor de autorizacion

## 2.14 Herramientas: curl y Postman

## 2.15 Referencias normativas

# 3.0 Procedimientos.

Para ello, en mi Windows no tengo el docker instalado y ni pienso instalarlo. Voy directamente con WSL, vamos a ver que máquinas tenemos:

```
wsl -l -v
```

En mi caso tenía una:

```
wsl -d Ubuntu-24.04
```

Si no tienes creas una, para saber qué máquina crear, vamos a ver el catálogo:

```
wsl --list --online
```

```
wsl --install -d Debian
```

<pre>
PS C:\Users\User> wsl --install -d Debian
Descargando: Debian GNU/Linux
[==========================70,2%=========                  ]
</pre>

Estás en /mnt/c/Users/User porque abriste WSL desde esa carpeta en PowerShell y te dejó en su equivalente dentro de Linux. Eso que ves montado en /mnt/c es tu disco C: de Windows, accesible desde Linux como si fuera una carpeta más. Por eso tu home de Linux y la carpeta de usuario de Windows son dos sitios distintos.

Sobre si es una máquina virtual: sí, pero no del tipo que tienes en la cabeza. WSL2 ejecuta un núcleo Linux real dentro de una máquina virtual ligera sobre Hyper-V. La diferencia con VirtualBox o VMware es que está muy integrada: arranca en un par de segundos, comparte el localhost con Windows y te monta los discos de Windows automáticamente. No es una emulación ni una capa de traducción, es Linux de verdad, solo que con las costuras muy disimuladas.

Y un consejo que importa para lo que vas a hacer: trabaja siempre dentro de /home/user, no en /mnt/c. El acceso a los ficheros de Windows desde Linux pasa por una capa de traducción y es bastante más lento. Con Docker, clonando repositorios o compilando, la diferencia se nota mucho. Deja /mnt/c solo para cuando necesites mover un fichero entre los dos mundos.

<pre>
user@Usuario:/mnt/c/Users/User$ pwd
/mnt/c/Users/User
user@Usuario:/mnt/c/Users/User$ cd ~
user@Usuario:~$ ls
user@Usuario:~$ pwd
/home/user
user@Usuario:~$
</pre>

Instalamos docker:

```
sudo apt update && sudo apt install -y curl && curl -fsSL https://get.docker.com | sh && sudo usermod -aG docker $USER && sudo service docker start && sudo docker --version
```

Montamos el contenedor de KeyCloak:

```
sudo docker run -d --name kc-lab -p 8080:8080 \
  -e KC_BOOTSTRAP_ADMIN_USERNAME=admin \
  -e KC_BOOTSTRAP_ADMIN_PASSWORD=admin \
  quay.io/keycloak/keycloak:26.7.0 start-dev
```

Vamos a ver si está montado de verdad:

```
sudo docker ps
```

Tenemos que ver algo así:

<pre>
  CONTAINER ID   IMAGE                              COMMAND                  CREATED         STATUS         PORTS                                                             NAMES
2b57a5267113   quay.io/keycloak/keycloak:26.7.0   "/opt/keycloak/bin/k…"   9 seconds ago   Up 8 seconds   8443/tcp, 0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp, 9000/tcp   kc-lab
</pre>

<a>http://localhost:8080</a>

<img width="1917" height="957" alt="imagen" src="https://github.com/user-attachments/assets/5d147032-cbb5-4505-a849-f714255b729e" />

Las credenciales son:
- admin
- admin

<img width="1917" height="911" alt="imagen" src="https://github.com/user-attachments/assets/af78cb05-accb-4816-b2f7-199eb2a575ef" />

Lo que vemos es el panel de control.

KeyCloak es el proveedor de identidad del laboratorio. Su trabajo es autenticar y emitir tokens. Es decir, concentra las credenciales en un sitio y las aplicaciones dejan de gestionarlas: cuando una aplicación necesita saber quién eres o qué puedes hacer, se lo pregunta a Keycloak y este responde con un token firmado.

En lo práctico, tres funciones:
- Guarda o federa las identidades (usuarios, grupos, roles).
- Autentica y ejecuta los flujos de OAuth 2.0, OpenID Connect y SAML.
- Emite, renueva y revoca los tokens.

## 3.2. CREAR EL REALM Y UN USUARIO

Lo primero que vamos a crear es un realm y un usuario.

Un realm es una frontera de aislamiento: sus usuarios, clientes, roles y claves
de firma le pertenecen y no se comparten con otros realms. Equivale
conceptualmente a un tenant.

<img width="1551" height="502" alt="imagen" src="https://github.com/user-attachments/assets/93f1d204-130f-47bc-9682-12435c01346d" />

Al crearlo nos pedirá:

<img width="1250" height="767" alt="imagen" src="https://github.com/user-attachments/assets/5f35c061-41a7-44a4-9549-78ba8140b5e8" />

- Resource file: sirve para crear el realm importando un fichero JSON en lugar de configurarlo a mano. Keycloak permite exportar un realm completo (clientes, roles, grupos, mapeadores, ajustes) a un JSON, y ese fichero se puede volver a importar aquí para reconstruirlo idéntico. <br> Es la puerta de entrada a la configuración como código: en lugar de documentar cuarenta capturas de pantalla, guardas el JSON en el repositorio y cualquiera regenera tu laboratorio en un minuto. Ahora lo dejas vacío, porque estás creando el realm desde cero, pero cuando termines la práctica te interesa exportarlo y subirlo al repositorio.

- Enabled: indica si el realm está activo. Si lo desactivas, sus usuarios no pueden autenticarse y sus clientes dejan de obtener tokens, aunque toda la configuración se conserva y tú sigues viéndola desde la consola de administración. <br> Sirve para suspender un entorno entero sin destruirlo, por ejemplo durante una migración o al cerrar el acceso a una filial. Es el mismo concepto que desactivar una cuenta en lugar de borrarla, pero aplicado al realm completo.

<img width="1057" height="440" alt="imagen" src="https://github.com/user-attachments/assets/becc7af0-deb1-4163-8d25-5538a6b31dc1" />

<img width="1676" height="572" alt="imagen" src="https://github.com/user-attachments/assets/a2c9ae32-4f3d-4813-a194-f4f73b5d8960" />

<img width="1915" height="875" alt="imagen" src="https://github.com/user-attachments/assets/204f92fb-4b52-495e-8b48-8d1120e8744c" />

Son tareas que Keycloak obliga a completar al usuario la próxima vez que inicie sesión, antes de dejarle pasar a ningún sitio.

Las típicas del desplegable:
- <b>Update Password:</b> le fuerza a cambiar la contraseña.
- <b>Verify Email:</b> le manda un correo de verificación y no le deja entrar hasta que pulse el enlace.
- <b>Update Profile:</b> le obliga a completar nombre, apellidos o correo si faltan.
- <b>Configure OTP:</b> le fuerza a dar de alta el segundo factor con una aplicación de códigos.
- <b>Terms and Conditions:</b> le muestra las condiciones y exige aceptarlas.

Para qué sirven en la realidad: es el mecanismo de alta de empleados. Recursos Humanos crea la cuenta con una contraseña provisional y marca Update Password, así el administrador nunca conoce la contraseña definitiva. O la organización decide implantar MFA y marca Configure OTP a toda la plantilla, de modo que cada uno lo configura la primera vez que entra, sin que nadie tenga que perseguirlos.

Fíjate en el detalle interesante: es una obligación que viaja con la identidad, no con la aplicación. Da igual desde qué aplicación intente entrar, la acción le salta igual, porque quien la impone es el proveedor de identidad.

En tu caso, déjalo vacío. Si marcas cualquiera, cuando llegues al flujo con Postman el navegador te va a interrumpir pidiendo esa tarea y te va a romper el ejercicio. Lo mismo que la contraseña temporal, que desactivarás en la pestaña Credentials.

<img width="942" height="483" alt="imagen" src="https://github.com/user-attachments/assets/17d2d022-497c-4b18-8457-d37354212dac" />

Y aceptamos y lo creamos. Como nos habremos dado cuenta, el estilo de la interfaz es muy AWS.

<img width="1575" height="891" alt="imagen" src="https://github.com/user-attachments/assets/95dbb0a7-1669-497d-85c9-d343eb3532d6" />


> [!IMPORTANT]
> El usuario se ha creado a mano. Eliminar exactamente ese trabajo<br>
> manual es la razon de existir de SCIM, que es el objeto<br>
> de una practica posterior.
>


## 3.3. CLIENTE CONFIDENCIAL Y FLUJO DE CREDENCIALES DE CLIENTE

<p><b>Qué es un cliente en OAuth</b></p>

Un cliente no es una persona ni un usuario. Es una aplicación registrada que tiene permiso para pedir tokens al proveedor de identidad. Cuando das de alta un cliente en Keycloak, lo que estás haciendo es decir: esta aplicación existe, se llama así, y puede solicitar tokens bajo estas condiciones.

<p><b>Público frente a confidencial</b></p>

La diferencia está en una sola pregunta: <b>¿esta aplicación puede guardar un secreto sin que nadie lo vea?</b>

Un cliente público no puede. Vive en un navegador o en un móvil, su código es inspeccionable por el usuario, y cualquier contraseña que metieras dentro estaría a la vista. Por eso se autentica sin secreto y necesita protecciones adicionales como PKCE.
Un cliente confidencial sí puede. Corre en un servidor que tú controlas, nadie puede leer su configuración, y por tanto se le puede entregar un secreto. Ese secreto es su credencial: es lo que prueba que la petición viene de él y no de un impostor.

<p><b>El flujo de credenciales de cliente</b></p>

Es el flujo más simple de OAuth porque elimina a la persona de la ecuación. El cliente presenta su client_id y su client_secret al proveedor, y recibe un access token a su propio nombre.

<p><b>Comparado con el flujo con usuario, aquí desaparecen tres cosas:</b></p>

No hay navegador, porque no hay nadie a quien mostrarle una pantalla de acceso.
No hay consentimiento, porque nadie está delegando el acceso a sus datos. El programa actúa por sí mismo, no en nombre de otro.
No hay refresh token, porque no hace falta. El cliente conserva su secreto y puede pedir un token nuevo cuando quiera, sin molestar a nadie.

<p><b>Dónde se usa esto de verdad</b></p>

Un proceso nocturno que consolida datos y llama a varias APIs internas.
Un microservicio que llama a otro microservicio.
Un conector de aprovisionamiento que crea y borra cuentas en una aplicación destino. Este es tu caso en la práctica siguiente: el motor que empuja identidades por SCIM se autentica exactamente así.

<p><b>La cuenta de servicio</b></p>

Aquí viene un detalle propio de Keycloak que conviene entender. Un token tiene que hablar de alguien, necesita un sujeto. Como aquí no hay persona, Keycloak crea internamente un usuario que representa al propio cliente: la cuenta de servicio (service account).

Eso no es un capricho de implementación, resuelve un problema real: gracias a esa cuenta puedes asignarle roles al programa, igual que se los asignarías a una persona. Y de ahí sale una idea que en IAM corporativo pesa mucho: las aplicaciones también son identidades, y también hay que gobernarlas. Tienen permisos, se acumulan, caducan y hay que recertificarlas. En un banco suele haber más cuentas de servicio que empleados, y son las que peor se controlan.

<p><b>Y una consecuencia de seguridad que conviene documentar</b></p>

El secreto del cliente es una credencial de larga duración que no caduca sola. Quien lo tenga puede obtener tokens indefinidamente hasta que alguien lo rote. Por eso no se escribe en el código ni se sube a un repositorio, y por eso existen las bóvedas de secretos. Cuando en tu .gitignore excluyas el .env, es literalmente esto lo que estás protegiendo.

Entonces, vamos a crear el cliente. Muy importante, que estemos en el realm adecuado.

<img width="397" height="131" alt="imagen" src="https://github.com/user-attachments/assets/7f3c2293-9674-4795-ac6a-26715deaa8a1" />

y nos vamos a clients.

<img width="1405" height="677" alt="imagen" src="https://github.com/user-attachments/assets/3deb0fe4-b1e5-4b00-a14a-c9a75e1bb501" />

<img width="932" height="837" alt="imagen" src="https://github.com/user-attachments/assets/c8f602d2-82e4-4ba0-82bf-6fcef18b7353" />

