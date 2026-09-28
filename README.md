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

El <b>required user actions</b> son tareas que Keycloak obliga a completar al usuario la próxima vez que inicie sesión, antes de dejarle pasar a ningún sitio.

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

<p><b>El client type es:</b></p>

Es el protocolo que va a hablar esa aplicación con Keycloak. Solo hay dos opciones y son excluyentes: un cliente habla OIDC o habla SAML, no los dos.

OpenID Connect es el moderno, construido sobre OAuth 2.0. Intercambia tokens JWT, funciona por peticiones HTTP con JSON, y es lo que usan las aplicaciones web actuales, las móviles y las APIs. Es lo que vas a practicar.

SAML 2.0 es anterior, de principios de los 2000. Intercambia aserciones en XML firmado y el navegador las transporta mediante formularios que se autoenvían. Se diseñó pensando en el inicio de sesión único entre organizaciones, no en APIs.

SAML no está muerto ni de lejos: en banca, administración pública y universidades hay muchísimas aplicaciones que solo hablan SAML, y una plataforma de IAM tiene que sostener ambos. Por eso Keycloak los ofrece.

La diferencia práctica que te importa ahora: con SAML no hay access token que enviar a una API. El resultado del flujo es una aserción que establece una sesión en la aplicación. Para proteger una API, que es lo que vas a hacer, SAML no sirve.

<p><b>El client ID es:</b></p>

Es el identificador público de la aplicación. El nombre con el que esa aplicación se presenta ante Keycloak.

La analogía directa: si comparas un cliente con una cuenta de usuario, el client_id es el nombre de usuario y el client_secret es la contraseña. Uno identifica, el otro demuestra.

Tres cosas que conviene tener claras:

No es secreto. Viaja en cada petición, aparece en las URL del navegador cuando el flujo es interactivo y cualquiera puede verlo. No pasa nada, porque identificar no es autenticar. Lo que hay que proteger es el secreto.

Es único dentro del realm. No puede haber dos clientes con el mismo client_id en lab-iam, aunque sí podría existir uno igual en otro realm, porque son mundos separados.

Es lo que vas a escribir en cada petición de token. Cuando dentro de un rato lances el curl, verás el parámetro client_id=api-backend. Con eso Keycloak sabe qué aplicación está pidiendo, qué flujos tiene permitidos y qué debe meter en el token.

Y una consecuencia práctica: cambiarlo después rompe todas las integraciones que ya lo usan, porque es la referencia que tienen configurada. Por eso se elige con cabeza y no se toca.

<img width="1142" height="772" alt="imagen" src="https://github.com/user-attachments/assets/83ac1052-2507-4caf-8034-2257018fdf37" />


Client authentication en On, solo Service account roles marcado, y todo lo demás apagado.<br>
Ahora, qué es cada cosa:

- Client authentication: Off es cliente público, On es confidencial. Al ponerlo en On, Keycloak le genera un secreto y le exige presentarlo en cada petición.
- Authorization: activa los servicios de autorización de grano fino de Keycloak, un motor de políticas propio suyo basado en recursos y permisos. Es una funcionalidad avanzada y no estándar.
- Authentication flow, que es donde eliges qué flujos puede usar este cliente:
  - Standard flow: el flujo de código de autorización, el que abre navegador y tiene una persona delante.
  - Direct access grants: el flujo en el que la aplicación recoge usuario y contraseña y se los manda a    - Keycloak. Está desaconsejado por el RFC 9700 porque obliga al usuario a entregar su contraseña a cada aplicación.
  - Implicit flow: devolvía el token directamente en la URL del navegador. Obsoleto y desaconsejado.
  - Service account roles: es el que habilita el flujo de credenciales de cliente y crea la cuenta de servicio.
  - Standard Token Exchange: permite canjear un token por otro, por ejemplo cuando un servicio necesita llamar a otro conservando la identidad del usuario original. Es útil en arquitecturas de microservicios encadenados.
  - JWT Authorization Grant: permite que el cliente se autentique presentando un JWT firmado en lugar de un secreto compartido. Es más seguro, porque la clave privada nunca viaja, pero más complejo de montar.
  - OAuth 2.0 Device Authorization Grant: el flujo de los dispositivos sin teclado cómodo. Es lo que hace una smart TV cuando te muestra un código y te dice que entres en una web desde el móvil.
  - OIDC CIBA Grant: autenticación desacoplada. El usuario aprueba desde otro canal, por ejemplo la app del banco, mientras la operación ocurre en otro sitio. Muy usado en banca abierta.

- Require PKCE: obliga a usar la protección PKCE en el flujo de código. Solo aplica cuando hay navegador, así que en este cliente es irrelevante.
- Require DPoP bound tokens: esto es interesante y merece que lo conozcas aunque no lo actives. DPoP (RFC 9449) ata el token a una clave criptográfica del cliente, de modo que un token robado no sirve para nada sin esa clave. Es la respuesta al problema de fondo de los tokens portadores, que es que quien los tiene los usa.

<img width="1020" height="852" alt="imagen" src="https://github.com/user-attachments/assets/f419e0d6-472c-4f8a-bb14-daee714e300a" />

Fíjate en un detalle que confirma que lo has configurado bien: aquí solo salen Root URL y Home URL. No aparecen las Valid redirect URIs ni los Web origins, porque al desmarcar Standard flow le has dicho a Keycloak que este cliente nunca va a pasar por un navegador, y sin navegador no hay retorno que autorizar.

Qué son esos dos campos, por si te los encuentras en el siguiente cliente:

- Root URL es la raíz de la aplicación. Sirve de prefijo para las demás URL, de modo que puedas escribirlas en relativo y cambiar el dominio en un solo sitio al pasar de desarrollo a producción.
- Home URL es a dónde se envía al usuario cuando entra a la aplicación desde Keycloak, por ejemplo desde la consola de cuenta.

Ambos son comodidades de configuración, no tienen efecto de seguridad. Los que sí lo tienen son las Valid redirect URIs, que verás cuando crees el cliente público.

Y lo creamos:

<img width="1356" height="862" alt="imagen" src="https://github.com/user-attachments/assets/77e6cea3-2d0d-4519-8783-26875e4428b6" />

Nos dirigimos al apartado "Client scopes":

<img width="1476" height="432" alt="imagen" src="https://github.com/user-attachments/assets/6756f59c-e947-4f4f-83d3-6d78762f29f0" />

Pulsa en el ámbito dedicado, el que se llama api-backend-dedicated.

<img width="906" height="477" alt="imagen" src="https://github.com/user-attachments/assets/341c4800-0952-4acc-9e15-0beb541d1025" />

<img width="966" height="375" alt="imagen" src="https://github.com/user-attachments/assets/3dc61020-6af7-4cbb-8f90-986d3cc66650" />

<img width="612" height="777" alt="imagen" src="https://github.com/user-attachments/assets/29103470-b08d-4db4-93d7-b30d790fffe8" />

Ahora nos vamos al apartado de credenciales:

<img width="1232" height="757" alt="imagen" src="https://github.com/user-attachments/assets/31bdfc4c-cdfc-443d-9965-116a98d06e98" />

<p><b>Y copiamos el client secret.</b></p>

¿Qué es? Es la credencial de la aplicación. Sin él, Keycloak no tiene forma de saber que quien pide el token es realmente api-backend.
- Piénsalo así: el client_id es público y aparece en cualquier sitio. Si bastara con enviarlo, cualquiera que lo supiera podría pedir tokens haciéndose pasar por tu proceso, y con esos tokens entrar a tu API. El secreto es lo que convierte "digo que soy api-backend" en "demuestro que soy api-backend".
- Y en este flujo concreto tiene un peso especial, porque es la única credencial que hay. En el flujo con persona, la seguridad se apoya en la contraseña del usuario y en todo el proceso del navegador. Aquí no hay nada de eso: el secreto es lo único que separa a tu proceso de cualquier otro. Por eso este flujo solo se permite a clientes confidenciales, que son los que pueden custodiarlo.

De ahí sale lo que ya comentamos y que conviene que quede escrito en tu README: ese valor no se escribe en el código, no se sube al repositorio y se guarda en una variable de entorno o en una bóveda de secretos. Tiene el mismo valor que una contraseña, con el agravante de que no caduca sola y suele estar en manos de varios equipos.

Ahora nos volvemos al WSL e instalaremos el "jq" que es un procesador de JSON para la CLI

El problema que resuelve: una API te devuelve el JSON todo en una línea, sin saltos ni indentación, porque así ocupa menos. Para una máquina da igual, para ti es ilegible. Cuando pasas la salida por jq, la reformatea y la colorea.

```
sudo apt install -y jq
```

Como se vería sin el jq:

<pre>
user@Usuario:~$ curl -s -X POST http://localhost:8080/realms/lab-iam/protocol/openid-connect/token   -d grant_type=client_credentials   -d client_id=api-backend   -d client_secret=PEGA_AQUI_EL_SECRETO
{"access_token":"eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJGQUJEWGcwckVKSmRVcVB6eVdTNnY1N29OMnRramJnaGZkR0tfWEpPZnlRIn0.eyJleHAiOjE3OTA1MjcwODgsImlhdCI6MTc5MDUyNjc4OCwianRpIjoidHJydGNjOjE3NDAwNGM3LWY0ZTMtYmEyNi1lZDEwLTU4OTcwY2RmNTIxMyIsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6ODA4MC9yZWFsbXMvbGFiLWlhbSIsImF1ZCI6ImFjY291bnQiLCJzdWIiOiI0OGVjMTI4OC1lMzVmLTQ2MmYtYmZjZS1lYmVkOTY5MWNmZmYiLCJ0eXAiOiJCZWFyZXIiLCJhenAiOiJhcGktYmFja2VuZCIsImFjciI6IjEiLCJhbGxvd2VkLW9yaWdpbnMiOlsiLyoiXSwicmVhbG1fYWNjZXNzIjp7InJvbGVzIjpbIm9mZmxpbmVfYWNjZXNzIiwiZGVmYXVsdC1yb2xlcy1sYWItaWFtIiwidW1hX2F1dGhvcml6YXRpb24iXX0sInJlc291cmNlX2FjY2VzcyI6eyJhY2NvdW50Ijp7InJvbGVzIjpbIm1hbmFnZS1hY2NvdW50IiwibWFuYWdlLWFjY291bnQtbGlua3MiLCJ2aWV3LXByb2ZpbGUiXX19LCJzY29wZSI6ImVtYWlsIHByb2ZpbGUiLCJjbGllbnRIb3N0IjoiMTcyLjE3LjAuMSIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwicHJlZmVycmVkX3VzZXJuYW1lIjoic2VydmljZS1hY2NvdW50LWFwaS1iYWNrZW5kIiwiY2xpZW50QWRkcmVzcyI6IjE3Mi4xNy4wLjEiLCJjbGllbnRfaWQiOiJhcGktYmFja2VuZCJ9.UfxKZkUVVvlFhD_pi2CmLovS8Dur0eEG8itwEX3QDKXCgrofKiOgeQU1GGqqCkDU4nUD4-3m7tFmNMRalhNsvhxD_7tV3yO6g5bhgYsjssGsqphGENoEPdK9yIz24QMioAu7taj7-lxWOWAIO2bgVmlu9PhwpDGTkKztxHkmf3JRFii1sSHuxuiHQBavhIgeHYxChbeaOkWcgffWomjMYyxoUMRb5QbWKVjPJ1DBBw9RHYhW7pJuGRsWIt9V3-eDLH8muZnBl-IGPMhO14ggGlJTs2t11uQ8nrnsIt8785w4Z2CwTOun1gzgcQmP9iOIwypZ8Sz1Lkx_FIFh7MPCUA","expires_in":300,"refresh_expires_in":0,"token_type":"Bearer","not-before-policy":0,"scope":"e
</pre>

Sin embargo con el jq:

```
curl -s -X POST http://localhost:8080/realms/lab-iam/protocol/openid-connect/token \
  -d grant_type=client_credentials \
  -d client_id=api-backend \
  -d client_secret=PEGA_AQUI_EL_SECRETO | jq
```

Qué hace cada parte, que es lo que importa:

- "-X POST:" el endpoint de token solo acepta POST. Las credenciales nunca viajan en la URL.
- "-d:" cada uno de estos es un campo del cuerpo de la petición, en formato de formulario.
- "grant_type=client_credentials": aquí es donde le dices a Keycloak qué flujo quieres. Este mismo endpoint atiende todos los flujos, y este parámetro es el que los distingue.
- "client_id y client_secret": quién eres y la prueba de que lo eres.
- "| jq": pasa la respuesta a jq para verla formateada en lugar de como una línea ilegible.

Y así es como se vería:

<img width="708" height="465" alt="imagen" src="https://github.com/user-attachments/assets/51a89e18-1ab1-4136-bf67-0b6a7f61e1eb" />

Funciona. Y confirma tres cosas que te anticipé, que conviene que veas escritas:
- No hay refresh_token. Fíjate además en refresh_expires_in: 0. Keycloak te está diciendo explícitamente que no emite uno, porque en este flujo no hace falta: el cliente tiene el secreto y puede pedir otro cuando quiera.
- `expires_in: 300`, cinco minutos. Ese es el valor por defecto del realm y es el que vas a bajar a un minuto más adelante para ver la caducidad en directo.
- `scope: "email profile"`, sin openid. Por eso este token no es OIDC, es OAuth puro. Y por eso, cuando pruebes el endpoint /userinfo con él, es probable que te lo rechace.

Ahora decodifica el cuerpo tú mismo, es decir, vamos a copiar ese "churro" de texto:

```
echo 'PEGA_AQUI_EL_TOKEN' | cut -d. -f2 | base64 -d 2>/dev/null | jq
```

<img width="915" height="892" alt="imagen" src="https://github.com/user-attachments/assets/883915aa-b446-4005-8938-e89eb33b14d4" />

<p><b>Tiempos</b></p>
- `iat` (issued at): 1790526999, que es el 27 de septiembre de 2026 a las 16:36:39 UTC, o sea las 18:36 hora peninsular. El instante en que Keycloak lo emitió.
- `exp` (expiration): 1790527299, las 16:41:39 UTC. Resta uno del otro y salen exactamente 300 segundos, los cinco minutos que anunciaba expires_in.

Ambos van en segundos desde el 1 de enero de 1970, que es el formato epoch de Unix. Se usa así porque es un entero sin zonas horarias ni ambigüedades de formato: cualquier sistema del mundo lo interpreta igual.

Quien valida el token compara exp con su propio reloj. De ahí un problema clásico en producción: si los relojes de dos servidores van desincronizados, uno puede rechazar tokens que el otro acaba de emitir. Por eso en entornos serios se sincroniza la hora por NTP y los validadores admiten un margen de tolerancia de unos segundos.
- `jti` (JWT ID): identificador único de este token concreto. Sirve para detectar reutilizaciones, para mantener listas de revocación y, sobre todo, para correlacionar en auditoría: si en el registro de tu API aparece una operación sospechosa con ese jti, puedes cruzarlo con el registro de emisión de Keycloak y saber exactamente quién y cuándo lo pidió. El prefijo trrtcc: es algo interno de Keycloak y no sé con certeza qué significa, no te lo voy a inventar.

<p><b>Quién y para quién</b></p>

- `iss` (issuer): http://localhost:8080/realms/lab-iam. Quién emitió el token. Tu API tendrá que comprobar que este valor coincide exactamente con el emisor que espera. Y ojo con esto, porque te va a morder más adelante: si montas la API dentro de otro contenedor, para ella localhost no es Keycloak, es ella misma. Ese desajuste entre la URL pública del emisor y la interna es uno de los errores más frecuentes al desplegar.
- `aud` (audience): account. Para quién está pensado el token. Aquí no aparece tu API, aparece el cliente interno de gestión de cuenta de Keycloak, porque es el único destinatario que el realm sabe añadir por defecto.

La consecuencia práctica: una API que valide la audiencia con rigor rechazaría este token, y haría bien. Un token emitido para un destinatario no debería servir en otro, porque si no, cualquier servicio que reciba tu token puede darse la vuelta y usarlo contra un tercero haciéndose pasar por ti. Se corrige añadiendo un mapeador de audiencia al cliente o al ámbito.

- `sub` (subject): 48ec1288-.... El identificador del sujeto del que habla el token. Es un UUID y no un nombre a propósito: los nombres cambian (una persona se casa, un cliente se renombra) y el identificador no debe cambiar nunca, porque es lo que las aplicaciones guardan para asociar sus datos a esa identidad.
- `azp` (authorized party): api-backend. Qué cliente pidió el token. En este caso coincide con quien lo va a usar, pero en flujos donde un cliente pide tokens destinados a otro, azp y aud son distintos y esa diferencia importa.
- `typ`: Bearer. El tipo de token, portador.

<p><b>Autenticación y autorización</b></p>

- `acr` (authentication context class reference): 1. Indica con qué nivel de garantía se autenticó el sujeto. Se usa para políticas del tipo "para transferir más de mil euros exijo que te hayas autenticado con doble factor en los últimos cinco minutos". Aquí es un valor básico, porque no hubo persona ni segundo factor.
- `realm_access.roles`: los roles de realm que trae el sujeto. Los tres que ves son de serie:
  - default-roles-lab-iam es un rol compuesto que Keycloak asigna automáticamente a todo el mundo y que agrupa los permisos mínimos.
  - offline_access permite solicitar tokens de sesión desconectada, los que sobreviven a que el usuario cierre el navegador.
  - uma_authorization va ligado al motor de autorización de grano fino, ese que dejaste en Off.

- resource_access.account.roles: roles sobre un cliente concreto, en este caso sobre account. Le permiten ver y gestionar su propio perfil.

Ahí tienes la diferencia entre los dos planos: los roles de realm valen en todo el realm, los de cliente solo tienen sentido dentro de una aplicación. Un mismo usuario puede ser "lector" en una aplicación y "administrador" en otra sin colisión, porque cada rol vive en su cliente.

- scope: email profile. Los ámbitos concedidos. Falta openid, y por eso este token no activa el comportamiento OIDC.

<p><b>Contexto y ruido</b></p>
- `clientHost` y `clientAddress`: `172.17.0.1`, la dirección desde la que se hizo la petición vista desde dentro del contenedor. Esa IP es la pasarela de la red de Docker, o sea tu WSL visto desde Keycloak. Es información de auditoría.
- `email_verified: false` y `preferred_username: service-account-api-backend`: vienen del perfil de la cuenta de servicio que Keycloak creó sola. El correo verificado aquí no significa nada, porque esta identidad no tiene correo.

## 3.4. LEER EL TOKEN POR DENTRO

Siguiente paso: la introspección, que es preguntarle al servidor si ese token sigue vivo.

Primero vamos a guardar el token en una variable para no andar pegándolo:

```
AT=$(curl -s -X POST http://localhost:8080/realms/lab-iam/protocol/openid-connect/token \
  -d grant_type=client_credentials -d client_id=api-backend \
  -d client_secret=TU_SECRETO | jq -r .access_token)
```

Ya se ha almacenado ese token en la variable:

```
echo $AT
```

Y ahora vamos a preguntar por él usando la variable:

```
curl -s -X POST http://localhost:8080/realms/lab-iam/protocol/openid-connect/token/introspect \
  -u api-backend:TU_SECRETO \
  -d token=$AT | jq
```

El -u es autenticación HTTP básica: manda usuario y contraseña en una cabecera. Aquí el usuario es el client_id y la contraseña el secreto. Fíjate en el detalle: para preguntar por un token también hay que estar autenticado. Keycloak no le cuenta a cualquiera qué contiene un token ajeno.

<img width="922" height="205" alt="imagen" src="https://github.com/user-attachments/assets/e5471001-780f-42c9-a37e-6fba012566fb" />

En mi caso devuelve false, porque han pasado más de 5 minutos con el token desde que lo pedí y ha caducado.

Deberías ver "active": true y, a continuación, las mismas afirmaciones que ya decodificaste.

<img width="720" height="952" alt="imagen" src="https://github.com/user-attachments/assets/4cc7ebf0-c003-4cda-81d7-c95512ddb00b" />

Cuando lo tengas, dos pruebas más que cierran este bloque:

```
curl -s http://localhost:8080/realms/lab-iam/protocol/openid-connect/certs | jq
```

En mi caso devuelve esto:

<pre>
{
  "keys": [
    {
      "kid": "iUFHWK-KvqUnWtO0t3mCL76O4Bv0Zfxw7du21T_Zw-E",
      "kty": "RSA",
      "alg": "RSA-OAEP",
      "use": "enc",
      "x5c": [
        "MIICnTCCAYUCBgGg43cmjjANBgkqhkiG9w0BAQsFADASMRAwDgYDVQQDDAdsYWItaWFtMB4XDTI2MDkyNzE1MjIxMVoXDTM2MDkyNzE1MjM1MVowEjEQMA4GA1UEAwwHbGFiLWlhbTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKKSvHNrGQLLCctZMoj5FgyrJHieq/kvRgkFR+NCR9z+0JImA4l0ql5bLXu4IoLq18DkZxjEVEVhcsBXxcglCiW9DYjE7uQR8qSwp1VXk6KoJXNzSv06W9ae2bVQoW5O+09RpWii6c+rKlSUBtD5PGFqxX3d5S+YypmRagWoYBd9gtHEax7QSOagsqwIXKDo6OMHni3e5/6b/JlDhUGTSC3OdrK7LdSZVDvfighKZjbnpzXW0oKM+lWdk0uwlJo9AqTRkW36p8l1a6stAvfmyHtdoqd01NFpDS/z912vZwJRLMhwXTDaMye6TQkApbN+BqmxAeaBufBJHxjurvITh6sCAwEAATANBgkqhkiG9w0BAQsFAAOCAQEAGzU+zikk4EMNwCsafkyBkDmx2a8xKUsdOm46Egg4wSRSY5BkH9wmkIodGI5Gz0D+RxX4JYrTdtSFCeDPGhl3V25vtvZuHz/MaYmu3I2hSWjTGKzW7VvpEadfjOm/67P59kKD7Jqtv6sX5I5g8rt2hIxgd6t0buxN0jtKKmqhR+mwFImk2Vftwj3tAp3FtKBjvx/ZqdyGDHOdMqzG4hyx8WLG2r2g9wJ7apiKTdXT2A3g3MKVLEE85QAYyUgO64J2DZVdZ1g6z83osJdIR4u6WYifPdhecZOpvlUtI93ah/D/YlphQxJqZDut0XHjrehPnBMdNSybrBZ9FaI57zD6ng=="
      ],
      "x5t": "bArxRSq0NvfoxU8oeJ01iLWsVQA",
      "x5t#S256": "yVlc_9dBdyaJKL0sR4UdeoHZwKDqBseOkp2cEtMjL4I",
      "n": "opK8c2sZAssJy1kyiPkWDKskeJ6r-S9GCQVH40JH3P7QkiYDiXSqXlste7gigurXwORnGMRURWFywFfFyCUKJb0NiMTu5BHypLCnVVeToqglc3NK_Tpb1p7ZtVChbk77T1GlaKLpz6sqVJQG0Pk8YWrFfd3lL5jKmZFqBahgF32C0cRrHtBI5qCyrAhcoOjo4weeLd7n_pv8mUOFQZNILc52srst1JlUO9-KCEpmNuenNdbSgoz6VZ2TS7CUmj0CpNGRbfqnyXVrqy0C9-bIe12ip3TU0WkNL_P3Xa9nAlEsyHBdMNozJ7pNCQCls34GqbEB5oG58EkfGO6u8hOHqw",
      "e": "AQAB"
    },
    {
      "kid": "FABDXg0rEJJdUqPzyWS6v57oN2tkjbghfdGK_XJOfyQ",
      "kty": "RSA",
      "alg": "RS256",
      "use": "sig",
      "x5c": [
        "MIICnTCCAYUCBgGg43clyTANBgkqhkiG9w0BAQsFADASMRAwDgYDVQQDDAdsYWItaWFtMB4XDTI2MDkyNzE1MjIxMVoXDTM2MDkyNzE1MjM1MVowEjEQMA4GA1UEAwwHbGFiLWlhbTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBALs3rlGecZ0ikNYWo72Qa1KzbU7+Nn7CsESbDM3oarikAZscwPdaItkEM7PiYY8XXDmUI+oZO/mnyqsYnCrdBSjPYFs4bOjb1xrca8u6PDU3uquLEkAIgvraaILQqZH7Z/yijZiwEv0lSOmX0dwkuw0JlTZmpbruzFkieDMLHDSVw6LSmCYsEjB8gRKThpIrZPKmfCITpXE7XpRKhLKxYAD8eVnUltoT+XUZ92/e7kv/ietg2xl04fXetCHb0Z9K5bxriCC00Lj9wklEafhuPeU1ybMHYVSd1mFYTIUyX6y5OVRgzgtyH3U7tm3YBhX1kHiNIT5ApN9iopL9veHM3HMCAwEAATANBgkqhkiG9w0BAQsFAAOCAQEAP1+49kF+jq4i0pclWxPaxnUw/RyDT02zug+rPyUIqnDd/vMha34JCpu9U7aFeOOOGJyoWFlOujDzECMYIzp2SX9OcyuFkI5GVxv5qVwqUz6YQeXtu6KkZ+ih0TSn6Uz67Zzwr0ZmCCv2H5AOQp/dzYh2E7dCw7h4Wi2o+qfiZD1KgZFSLZ9GXaXF9ZhlfLadUNl9IQNEmhamoRSCjuRbrkw4WoPEwRQi2HxasRVcsRhn26cjhnxWWzEfDK4jIFgH2z0+OQd0vXalJTtljKLS/Mxfrs6JddRYtTxHJFFC1EIjBQGKWV/IrQ2xUhXQerVS8MpxTVE6Jvgijm+QWtwjuw=="
      ],
      "x5t": "sZog5fYP_EyGfBe60XDkBgDocXk",
      "x5t#S256": "qiaBEm8vQ-rWD98-0xmLOsu0tfE2lbheNcsasT1r6Bo",
      "n": "uzeuUZ5xnSKQ1hajvZBrUrNtTv42fsKwRJsMzehquKQBmxzA91oi2QQzs-JhjxdcOZQj6hk7-afKqxicKt0FKM9gWzhs6NvXGtxry7o8NTe6q4sSQAiC-tpogtCpkftn_KKNmLAS_SVI6ZfR3CS7DQmVNmaluu7MWSJ4MwscNJXDotKYJiwSMHyBEpOGkitk8qZ8IhOlcTtelEqEsrFgAPx5WdSW2hP5dRn3b97uS_-J62DbGXTh9d60IdvRn0rlvGuIILTQuP3CSURp-G495TXJswdhVJ3WYVhMhTJfrLk5VGDOC3IfdTu2bdgGFfWQeI0hPkCk32Kikv294czccw",
      "e": "AQAB"
    }
  ]
}
</pre>


```
curl -s http://localhost:8080/realms/lab-iam/.well-known/openid-configuration | jq
```

<pre>
{
  "issuer": "http://localhost:8080/realms/lab-iam",
  "authorization_endpoint": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/auth",
  "token_endpoint": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/token",
  "introspection_endpoint": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/token/introspect",
  "userinfo_endpoint": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/userinfo",
  "end_session_endpoint": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/logout",
  "frontchannel_logout_session_supported": true,
  "frontchannel_logout_supported": true,
  "jwks_uri": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/certs",
  "check_session_iframe": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/login-status-iframe.html",
  "grant_types_supported": [
    "authorization_code",
    "client_credentials",
    "implicit",
    "password",
    "refresh_token",
    "urn:ietf:params:oauth:grant-type:device_code",
    "urn:ietf:params:oauth:grant-type:jwt-bearer",
    "urn:ietf:params:oauth:grant-type:token-exchange",
    "urn:ietf:params:oauth:grant-type:uma-ticket",
    "urn:openid:params:grant-type:ciba"
  ],
  "acr_values_supported": [
    "0",
    "1"
  ],
  "response_types_supported": [
    "code",
    "none",
    "id_token",
    "token",
    "id_token token",
    "code id_token",
    "code token",
    "code id_token token"
  ],
  "subject_types_supported": [
    "public",
    "pairwise"
  ],
  "prompt_values_supported": [
    "none",
    "login",
    "consent"
  ],
  "id_token_signing_alg_values_supported": [
    "PS384",
    "RS384",
    "EdDSA",
    "ES384",
    "HS256",
    "HS512",
    "ES256",
    "RS256",
    "HS384",
    "ES512",
    "PS256",
    "PS512",
    "RS512"
  ],
  "id_token_encryption_alg_values_supported": [
    "ECDH-ES+A256KW",
    "ECDH-ES+A192KW",
    "ECDH-ES+A128KW",
    "RSA-OAEP",
    "RSA-OAEP-256",
    "RSA1_5",
    "ECDH-ES"
  ],
  "id_token_encryption_enc_values_supported": [
    "A256GCM",
    "A192GCM",
    "A128GCM",
    "A128CBC-HS256",
    "A192CBC-HS384",
    "A256CBC-HS512"
  ],
  "userinfo_signing_alg_values_supported": [
    "PS384",
    "RS384",
    "EdDSA",
    "ES384",
    "HS256",
    "HS512",
    "ES256",
    "RS256",
    "HS384",
    "ES512",
    "PS256",
    "PS512",
    "RS512",
    "none"
  ],
  "userinfo_encryption_alg_values_supported": [
    "ECDH-ES+A256KW",
    "ECDH-ES+A192KW",
    "ECDH-ES+A128KW",
    "RSA-OAEP",
    "RSA-OAEP-256",
    "RSA1_5",
    "ECDH-ES"
  ],
  "userinfo_encryption_enc_values_supported": [
    "A256GCM",
    "A192GCM",
    "A128GCM",
    "A128CBC-HS256",
    "A192CBC-HS384",
    "A256CBC-HS512"
  ],
  "request_object_signing_alg_values_supported": [
    "PS384",
    "RS384",
    "EdDSA",
    "ES384",
    "HS256",
    "HS512",
    "ES256",
    "RS256",
    "HS384",
    "ES512",
    "PS256",
    "PS512",
    "RS512",
    "none"
  ],
  "request_object_encryption_alg_values_supported": [
    "ECDH-ES+A256KW",
    "ECDH-ES+A192KW",
    "ECDH-ES+A128KW",
    "RSA-OAEP",
    "RSA-OAEP-256",
    "RSA1_5",
    "ECDH-ES"
  ],
  "request_object_encryption_enc_values_supported": [
    "A256GCM",
    "A192GCM",
    "A128GCM",
    "A128CBC-HS256",
    "A192CBC-HS384",
    "A256CBC-HS512"
  ],
  "response_modes_supported": [
    "query",
    "fragment",
    "form_post",
    "query.jwt",
    "fragment.jwt",
    "form_post.jwt",
    "jwt"
  ],
  "registration_endpoint": "http://localhost:8080/realms/lab-iam/clients-registrations/openid-connect",
  "token_endpoint_auth_methods_supported": [
    "private_key_jwt",
    "client_secret_basic",
    "client_secret_post",
    "tls_client_auth",
    "client_secret_jwt"
  ],
  "token_endpoint_auth_signing_alg_values_supported": [
    "PS384",
    "RS384",
    "EdDSA",
    "ES384",
    "HS256",
    "HS512",
    "ES256",
    "RS256",
    "HS384",
    "ES512",
    "PS256",
    "PS512",
    "RS512"
  ],
  "introspection_endpoint_auth_methods_supported": [
    "private_key_jwt",
    "client_secret_basic",
    "client_secret_post",
    "tls_client_auth",
    "client_secret_jwt"
  ],
  "introspection_endpoint_auth_signing_alg_values_supported": [
    "PS384",
    "RS384",
    "EdDSA",
    "ES384",
    "HS256",
    "HS512",
    "ES256",
    "RS256",
    "HS384",
    "ES512",
    "PS256",
    "PS512",
    "RS512"
  ],
  "authorization_signing_alg_values_supported": [
    "PS384",
    "RS384",
    "EdDSA",
    "ES384",
    "HS256",
    "HS512",
    "ES256",
    "RS256",
    "HS384",
    "ES512",
    "PS256",
    "PS512",
    "RS512"
  ],
  "authorization_encryption_alg_values_supported": [
    "ECDH-ES+A256KW",
    "ECDH-ES+A192KW",
    "ECDH-ES+A128KW",
    "RSA-OAEP",
    "RSA-OAEP-256",
    "RSA1_5",
    "ECDH-ES"
  ],
  "authorization_encryption_enc_values_supported": [
    "A256GCM",
    "A192GCM",
    "A128GCM",
    "A128CBC-HS256",
    "A192CBC-HS384",
    "A256CBC-HS512"
  ],
  "claims_supported": [
    "iss",
    "sub",
    "aud",
    "exp",
    "iat",
    "auth_time",
    "name",
    "given_name",
    "family_name",
    "preferred_username",
    "email",
    "acr",
    "azp",
    "nonce"
  ],
  "claim_types_supported": [
    "normal"
  ],
  "claims_parameter_supported": true,
  "scopes_supported": [
    "openid",
    "address",
    "phone",
    "web-origins",
    "microprofile-jwt",
    "basic",
    "offline_access",
    "acr",
    "email",
    "service_account",
    "roles",
    "profile",
    "organization"
  ],
  "request_parameter_supported": true,
  "request_uri_parameter_supported": true,
  "require_request_uri_registration": true,
  "code_challenge_methods_supported": [
    "plain",
    "S256"
  ],
  "tls_client_certificate_bound_access_tokens": true,
  "dpop_signing_alg_values_supported": [
    "PS384",
    "RS384",
    "EdDSA",
    "ES384",
    "ES256",
    "RS256",
    "ES512",
    "PS256",
    "PS512",
    "RS512"
  ],
  "revocation_endpoint": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/revoke",
  "revocation_endpoint_auth_methods_supported": [
    "private_key_jwt",
    "client_secret_basic",
    "client_secret_post",
    "tls_client_auth",
    "client_secret_jwt"
  ],
  "revocation_endpoint_auth_signing_alg_values_supported": [
    "PS384",
    "RS384",
    "EdDSA",
    "ES384",
    "HS256",
    "HS512",
    "ES256",
    "RS256",
    "HS384",
    "ES512",
    "PS256",
    "PS512",
    "RS512"
  ],
  "backchannel_logout_supported": true,
  "backchannel_logout_session_supported": true,
  "device_authorization_endpoint": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/auth/device",
  "backchannel_token_delivery_modes_supported": [
    "poll",
    "ping"
  ],
  "backchannel_authentication_endpoint": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/ext/ciba/auth",
  "backchannel_authentication_request_signing_alg_values_supported": [
    "PS384",
    "RS384",
    "EdDSA",
    "ES384",
    "ES256",
    "RS256",
    "ES512",
    "PS256",
    "PS512",
    "RS512"
  ],
  "require_pushed_authorization_requests": false,
  "pushed_authorization_request_endpoint": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/ext/par/request",
  "mtls_endpoint_aliases": {
    "token_endpoint": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/token",
    "revocation_endpoint": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/revoke",
    "introspection_endpoint": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/token/introspect",
    "device_authorization_endpoint": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/auth/device",
    "registration_endpoint": "http://localhost:8080/realms/lab-iam/clients-registrations/openid-connect",
    "userinfo_endpoint": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/userinfo",
    "pushed_authorization_request_endpoint": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/ext/par/request",
    "backchannel_authentication_endpoint": "http://localhost:8080/realms/lab-iam/protocol/openid-connect/ext/ciba/auth"
  },
  "authorization_response_iss_parameter_supported": true
}
</pre>

```
curl -s http://localhost:8080/realms/lab-iam/protocol/openid-connect/userinfo \
  -H "Authorization: Bearer $AT" | jq
```

Y este último no nos devolverá nada.

Keycloak, en este endpoint, no manda un JSON de error. Pone toda la información en el código de estado y en la cabecera WWW-Authenticate. Así que jq recibe cero bytes, no tiene nada que formatear y no imprime nada. No es un fallo, es que estás mirando el sitio equivocado.

Por eso hace falta -i, que muestra las cabeceras además del cuerpo.

Y de ahí sale una lección práctica que conviene que quede en el README: no todas las APIs informan de los errores igual. Unas devuelven un JSON con el detalle, otras lo ponen en cabeceras, otras solo dejan el código de estado. Cuando algo "no devuelve nada", lo primero es mirar con -i antes de concluir que está roto.

Este otro comando (añadiendo el -i y quitando el jq), nos mostrará la cabecera:

```
curl -i -s http://localhost:8080/realms/lab-iam/protocol/openid-connect/userinfo \
  -H "Authorization: Bearer $AT"
```

<pre>
HTTP/1.1 403 Forbidden
Cache-Control: no-store
Pragma: no-cache
content-length: 0
Content-Type: text/plain;charset=utf-8
Referrer-Policy: no-referrer
Strict-Transport-Security: max-age=31536000; includeSubDomains
WWW-Authenticate: Bearer realm="lab-iam", error="insufficient_scope", error_description="Missing openid scope"
X-Content-Type-Options: nosniff
X-Robots-Tag: none
</pre>

### 3.4.1 Breve experimento

Experimentos que en ambos casos nos deben rechazar:

#### 3.4.1.1 Experimento 1: la caducidad.

```
SECRET=<poner aqui el secreto>

AT=$(curl -s -X POST http://localhost:8080/realms/lab-iam/protocol/openid-connect/token \
  -d grant_type=client_credentials -d client_id=api-backend \
  -d client_secret=$SECRET | jq -r .access_token)

date -u; curl -s -X POST http://localhost:8080/realms/lab-iam/protocol/openid-connect/token/introspect \
  -u api-backend:$SECRET -d token=$AT | jq '{active, exp, iat}'
```

Ahora esperamos algo más de cinco minutos sin volver a pedir token y lanzamos solo la segunda parte:

```
date -u; curl -s -X POST http://localhost:8080/realms/lab-iam/protocol/openid-connect/token/introspect \
  -u api-backend:$SECRET -d token=$AT | jq
```

Debe salir {"active": false} y nada más. 

<img width="1045" height="175" alt="imagen" src="https://github.com/user-attachments/assets/3fa11b3e-a1c1-4628-a8dd-7c9c5c6a7264" />

Fíjate en el detalle: cuando el token no es válido, la introspección no te cuenta nada de él. No dice que caducó, ni de quién era. Eso es deliberado: si diera detalles, serviría para sonsacar información sobre tokens ajenos.

Y el contraste que quieres documentar: decodifica ese mismo token caducado y verás que sus datos siguen perfectamente legibles. El token no se destruye ni se borra, simplemente deja de ser aceptado.

#### 3.4.1.2 Experimento 2: la firma manipulada.

Aquí vamos a modificar el contenido del token conservando la firma original, que es exactamente lo que intentaría un atacante para, por ejemplo, darse roles que no tiene.

```
H=$(echo $AT | cut -d. -f1)
P=$(echo $AT | cut -d. -f2)
S=$(echo $AT | cut -d. -f3)

# Cambiamos un valor dentro del cuerpo y lo volvemos a codificar
NEWP=$(echo $P | tr '_-' '/+' | base64 -d 2>/dev/null \
  | sed 's/"acr":"1"/"acr":"9"/' \
  | base64 -w0 | tr '/+' '_-' | tr -d '=')

FAKE="$H.$NEWP.$S"
```

Comprobemos primero que la manipulación surtió efecto:

```
echo $FAKE | cut -d. -f2 | tr '_-' '/+' | base64 -d 2>/dev/null | jq .acr
```

Debe decir "9". 

<img width="1165" height="402" alt="imagen" src="https://github.com/user-attachments/assets/22330036-28a7-415b-9513-adc04b38a76e" />

Ya tienes un token con el contenido alterado y la firma vieja. Ahora preséntalo:

```
curl -s -X POST http://localhost:8080/realms/lab-iam/protocol/openid-connect/token/introspect \
  -u api-backend:$SECRET -d token=$FAKE | jq

curl -i -s http://localhost:8080/realms/lab-iam/protocol/openid-connect/userinfo \
  -H "Authorization: Bearer $FAKE"
```

<img width="1362" height="452" alt="imagen" src="https://github.com/user-attachments/assets/78dea3c2-c1ba-4bc1-9674-3afcec0e9624" />

El contenido de un JWT es manipulable por cualquiera, porque base64 no protege nada. Lo que no se puede falsificar es la firma, porque haría falta la clave privada del realm, que nunca sale de Keycloak. De ahí la regla: nunca confíes en un JWT que no hayas verificado, aunque lo que pone dentro te parezca razonable.

## 3.6. CLIENTE PUBLICO CON PKCE DESDE POSTMAN

Vamos a crear un segundo cliente, `spa-web`.

General settings: 
- Client ID spa-web.

<img width="636" height="780" alt="imagen" src="https://github.com/user-attachments/assets/8d8f6fb5-d831-4d2d-a4b4-ccfc94e0fdf4" />

Capability config:
- Client authentication: Off
- Standard flow: marcado
- Require PKCE: marcado.
- PKCE Method: S256.
- Todo lo demás desmarcado

<img width="842" height="562" alt="imagen" src="https://github.com/user-attachments/assets/be967796-3e17-4462-87d1-2707d2e2369d" />

Login settings:
- Valid redirect URIs: https://oauth.pstmn.io/v1/callback
- Web origins: *

<img width="931" height="781" alt="imagen" src="https://github.com/user-attachments/assets/a21d12a7-1219-4f70-9ddf-771ca1c23822" />


Es el nombre que le vamos a poner al segundo cliente, el público. spa viene de Single Page Application, o sea una aplicación web de una sola página, de las que corren enteras en el navegador con JavaScript (Angular, React, Vue).

Es el caso típico de cliente público: su código se descarga al navegador del usuario, cualquiera puede abrir las herramientas de desarrollo y leerlo, así que no puede guardar ningún secreto. De ahí que necesite PKCE.

El nombre es arbitrario, lo elegí yo para que se entienda de un vistazo qué representa cada cliente:
- api-backend: un proceso de servidor, confidencial, sin persona.
- spa-web: una aplicación de navegador, pública, con persona.


Vamos a instalar postman para escritorio en Windows:

<a> https://www.postman.com/downloads/ </a>

Un cliente gráfico de HTTP pensado para trabajar con APIs. En esencia hace lo mismo que curl, pero con interfaz y con memoria.

Lo que aporta frente a la terminal:

Colecciones: guardas las peticiones organizadas en carpetas y las reutilizas. No dependes del historial de la shell.
Variables de entorno: defines una vez la URL base o el client_id y las usas en todas las peticiones con {{nombre}}. Cambiar de desarrollo a producción es cambiar de entorno, no reescribir cada petición.
Scripts: puedes ejecutar código antes y después de cada petición, por ejemplo para extraer el token de la respuesta y guardarlo en una variable automáticamente.
Ayudante de OAuth 2.0: y esta es la razón por la que lo usamos ahora. Ejecuta el flujo completo por ti, incluido el paso por el navegador para que te autentiques, y la gestión de PKCE. Con curl ese flujo es un incordio, porque hay que interceptar a mano el código de autorización que vuelve en la URL.

En el sector se usa a diario para probar endpoints, depurar integraciones y documentar APIs. Que lo sepas manejar era una de tus lagunas declaradas, y con esta práctica queda cubierta.

Un apunte de criterio que ya comentamos y que conviene mantener: el primer flujo lo hiciste con curl a propósito, para ver la mecánica sin capas intermedias. Ahora que entiendes lo que ocurre por debajo, Postman es comodidad, no magia.

En nuestro caso no vamos a iniciar sesión:

<img width="1601" height="987" alt="imagen" src="https://github.com/user-attachments/assets/8175cd91-79d3-427c-8e9a-f21435096472" />

Cuando estemos aquí vamos a mandar un GET:

```
http://localhost:8080/realms/lab-iam/protocol/openid-connect/userinfo
```

<img width="1440" height="387" alt="imagen" src="https://github.com/user-attachments/assets/50c74543-3531-445e-8710-38d0192152c8" />

<img width="1252" height="620" alt="imagen" src="https://github.com/user-attachments/assets/803ae42d-cd25-4cdb-8f49-0daeb5f03428" />

Rellena:
- Token Name: lab-iam
- Grant type: Authorization Code (With PKCE)
- Callback URL: https://oauth.pstmn.io/v1/callback y deja marcada la casilla Authorize using browser
- Auth URL: http://localhost:8080/realms/lab-iam/protocol/openid-connect/auth
- Access Token URL: http://localhost:8080/realms/lab-iam/protocol/openid-connect/token
- Client ID: spa-web
- Client Secret: vacío
- Code Challenge Method: SHA-256
- Code Verifier: déjalo vacío, Postman lo genera solo
- Scope: openid profile email
- State: pon cualquier cosa, por ejemplo xyz123
- Client Authentication: "Send client credentials in body"

Y le tenemos que dar a "Get new access token":

<img width="1305" height="530" alt="imagen" src="https://github.com/user-attachments/assets/9249bccf-7a7e-48a0-9224-08bac0fc781a" />

Y nos abrirá una nueva pestaña vía web, si nos sale un error es porque no lo hicimos en el realm correcto.
