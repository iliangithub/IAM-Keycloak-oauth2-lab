# 1.0 Introducción.

En esta práctica monto desde cero un laboratorio de gestión de identidades y accesos (IAM) con Keycloak, sobre WSL y Docker.

No quiero quedarme en levantar la herramienta y que funcione. Lo que busco es entender por qué funciona así: qué es un token realmente, qué diferencia hay entre autenticar y autorizar, por qué hay tres tokens distintos y qué pasa cuando algo falla. Por eso una parte de la práctica consiste en romper cosas a propósito y apuntar lo que responde el servidor de verdad, no lo que yo esperaba que respondiera.

Voy a tocar dos flujos de OAuth 2.0, el de credenciales de cliente y el de código de autorización con PKCE, y a partir de ahí los tokens JWT: leerlos, validarlos, verlos caducar, rotarlos y revocarlos. También uso Postman.

Lo que no entra aquí: SAML, la federación contra LDAP o Active Directory y el aprovisionamiento con SCIM. Eso lo dejo para prácticas siguientes.<br>
Todo esto corre en mi máquina, sobre HTTP y sin TLS, y con contraseñas fáciles. Medidas poco realistas y entornos de simulación.

# 2.0 Definiciones.

Antes de empezar con la resolución de la práctica, es de interés primero comprender la teoría y el porqué de cada cosa.

## 2.1 IAM (Identity and Access Management) o Gestión de identidades y accesos: el marco.

Sin entrar en gran profundidad vamos a definir un poco esto.

La ciberseguridad es un concepto general que engloba múltiples disciplinas y abarca tanto un plano físico como uno lógico.

Al hablar de un plano lógico o físico, nos referimos a que no solo hablamos de software, prevenir malware, ciberataques, reducir superficies de ataque... Sino cosas que parecen ajenas y realmente forman parte de la disciplina, como el uso de cámaras de seguridad, sensores, escáneres, etc.

Cuando hablamos de múltiples disciplinas mencionamos:
- La ingeniería de redes (aunque no sea 100% específica de la ciberseguridad).
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
- El Blue Team y todo lo que engloba.
  - SOC: monitorización continua, SIEM, triaje y respuesta de primer y segundo nivel.
  - DFIR: análisis forense digital y respuesta a incidentes.
  - CTI: inteligencia de amenazas, indicadores, atribución, informes.
  - CTH: búsqueda proactiva de amenazas sin alerta previa.
  - Ingeniería de detección: creación y afinado de reglas (Sigma, YARA, KQL).
  - Gestión de vulnerabilidades y de exposición.
  - Purple Team: validación de detecciones contra técnicas reales.
- Red Team.
  - Test de intrusión sobre red, sistemas y aplicaciones.
  - Simulación de adversario y ejercicios de emulación.
  - Ingeniería social, phishing y pretexto.
  - Seguridad ofensiva de aplicaciones y desarrollo de exploits.
  - OSINT y reconocimiento.
  - Seguridad física ofensiva (intrusión, clonado de tarjetas).
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
> Algunas de estas siglas se usan también como nombre de puesto, pero no son lo mismo. SOC, DFIR, CTI y CTH son áreas y funciones. Los títulos cambian muchísimo de una empresa a otra, así que en esta lista pongo lo que se hace, no cómo se llama el que lo hace.
>

Por lo tanto, el IAM es una disciplina de la ciberseguridad.<br>
Que gobierna sobre el ciclo de vida de las identidades digitales (personas, cuentas de servicio y dispositivos) y de los accesos que se les conceden.

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
- Se prueba algo que sabes, algo que solo tú sabes.
  - Una contraseña, un PIN, una respuesta a una pregunta.
- Algo que solo tú tienes, una posesión.
  - Una llave, una tarjeta magnética (banco), una tarjeta de proximidad (RFID), un token físico, un certificado digital, el móvil que te servirá como MFA o recibirá SMS o que gracias a él tiene una tarjeta SIM que te identifica.
- Algo que eres.
  - La biometría, huella, retina, fisonomía, voz. Rasgos físicos inherentes.

Para establecer qué puedes hacer, es decir, para autorizarte: 
- Primero debes autenticarte.

Y ahora, se usan distintos modelos para saber qué puedes hacer:
- Basado en roles (RBAC).
  - Es decir, se crea un rol "administrador" y es el rol "administrador" quien tiene los permisos.
  - Y eres tú a quien se le asigna ese rol. Por lo tanto, heredas los permisos.
- Basado en atributos (ABAC).
  - La decisión depende del contexto, no solo de quién eres, sino del departamento, la hora, el dispositivo, la red desde la que conectas o el importe de la operación.
- Basado en listas de control de acceso (ACL).
  - Es decir, que los permisos van puestos directamente en un recurso, y ese recurso guarda una lista de quién puede hacer qué sobre él. Como los permisos de un fichero.

En la autorización es importante tener en cuenta:
- El principio del mínimo privilegio, darle a cada identidad solo lo imprescindible para su función y nada más.
- El principio de denegación por defecto: lo que no está permitido de forma explícita, se deniega.
- El principio de mediación completa, es decir, que cada intento de acceso se comprueba en el momento, sin dar por válida una decisión anterior.

Los tres salen del mismo sitio, y me pareció curioso: son parte de los ocho principios de diseño de sistemas seguros que publicaron Saltzer y Schroeder en 1975 (least privilege, fail-safe defaults y complete mediation). El famoso "nunca confíes, verifica siempre" de la confianza cero es el tercero de ellos aplicado a la red, o sea que la idea tiene ya cincuenta años.

Por último, hay dos cosas que se confunden mucho: el permiso y la decisión.

- El permiso concedido (en el sector se le llama entitlement) es estático.
  - Alguien lo solicitó, alguien lo aprobó y queda guardado en un sistema. Se administra: se solicita, se aprueba, se revisa y se revoca.
- La decisión es dinámica.
  - Se calcula en cada intento, con los datos de ese momento, y puede salir que no aunque el permiso siga concedido.

Es decir, tener el permiso no garantiza obtener el acceso. Son dos momentos distintos y dos sistemas distintos.

Un ejemplo físico lo deja claro. Tienes una tarjeta de acceso al edificio y estás dado de alta en la puerta principal: eso es el permiso, y sigue ahí aunque estés de vacaciones. Pasas la tarjeta a las tres de la madrugada y el torno te deniega el paso porque el horario de esa puerta es de 7 a 22: eso es la decisión.

Esto tiene una consecuencia práctica en IAM: una recertificación revisa permisos concedidos, mientras que un registro de accesos recoge decisiones. Un permiso que nadie ha usado nunca no aparece en ningún registro, y sigue siendo un riesgo porque está ahí esperando.

## 2.3 Certificación y recertificación de accesos.

Antes de nada, una aclaración para evitar confusiones: aquí "certificación" no tiene nada que ver con las certificaciones profesionales tipo CCNA o Security+. Comparten la palabra y nada más.

La certificación de accesos, también llamada recertificación o revisión de accesos, es el proceso periódico en el que un responsable revisa los permisos que tienen asignadas las personas a su cargo y confirma, uno por uno, si siguen siendo necesarios.

Existe porque los permisos se acumulan. Cada cambio de puesto, cada proyecto y cada sustitución temporal añade accesos, pero casi nadie retira los anteriores. A ese efecto se le llama acumulación de privilegios (privilege creep), y sin revisiones periódicas acaba produciendo personas que, sobre el papel, pueden hacer casi cualquier cosa.

Cómo funciona una campaña de recertificación:

- Se define el alcance. Qué aplicaciones, qué permisos y qué colectivo se revisan.
- Se asigna un revisor. Normalmente el responsable jerárquico de cada persona, o el propietario de la aplicación.
- El revisor decide sobre cada permiso: mantener o revocar.
- Lo revocado se ejecuta en los sistemas destino.
- Y queda la evidencia: quién revisó qué, cuándo y con qué resultado.

La frecuencia depende de la criticidad. Lo habitual es anual o semestral para accesos normales, y bastante más seguido para los accesos privilegiados.

Es importante entender que la recertificación es una red de seguridad, no un sustituto del proceso de bajas. Si el circuito de baja funciona, los accesos se retiran cuando la persona se va o cambia de puesto. La recertificación está para cazar lo que ese circuito dejó pasar.

Y en banca no es opcional: es un requisito de auditoría y de cumplimiento normativo, y de ahí que interese tanto la evidencia como el resultado.

## 2.4  Proveedor de identidad y parte confiante

Un proveedor de identidad (IdP, Identity Provider) es el sistema que se encarga de autenticar y de emitir afirmaciones verificables sobre quién es la identidad autenticada.

La parte confiante es la aplicación que confía en esas afirmaciones en lugar de comprobar credenciales por su cuenta. Según el protocolo recibe un nombre u otro:

- En OpenID Connect se le llama parte confiante (relying party).
- En SAML se le llama proveedor de servicio (service provider).

Es decir, la aplicación deja de preguntar contraseñas y pasa a preguntar al proveedor de identidad quién eres.

La gracia del modelo es que concentra el punto de verificación. Si veinte aplicaciones validan contraseñas por su cuenta, hay veinte almacenes de credenciales que proteger, veinte políticas que mantener y veinte sitios donde olvidarte de cerrar un acceso. Delegando, hay uno. Y como efecto secundario sale el inicio de sesión único (SSO): una vez autenticado ante el proveedor, el resto de aplicaciones aceptan esa autenticación sin volver a pedir nada.

Esa confianza no es automática, se establece antes. La aplicación y el proveedor se configuran mutuamente (registro del cliente, metadatos, secretos o claves públicas de firma), y a partir de ahí la aplicación puede comprobar que la afirmación que recibe viene realmente del proveedor y no ha sido manipulada.

En esta práctica los papeles son:

- Keycloak es el proveedor de identidad.
- La aplicación web y la API que protegeremos son las partes confiantes.

## 2.5  OAuth 2.0: delegación de autorización

## 2.6  OpenID Connect: la capa de identidad

## 2.7  Keycloak: la implementación concreta

## 2.8  Vocabulario propio de Keycloak

## 2.9  Qué es una API y qué significa REST

Una API (Application Programming Interface) es la cara pública de un programa: el conjunto de operaciones y funciones que declara al exterior, qué datos hay que enviarle en cada una y qué devuelve.

Quien use una API no tiene que saber cómo está hecha por dentro, ni qué lenguaje ni qué base de datos usa. Y se puede cambiar el interior, es decir, la implementación, sin romper a quien te usa la API, siempre que el contrato se respete. Si cambias qué hace una operación o qué devuelve, ahí sí rompes a quien te consume.

Una API no es solo vía web, ni todo es HTTP ni JSON. Una biblioteca de C tiene API. Pero a día de hoy, al decir API nos referimos normalmente a una API web.

## 2.10  Tokens: concepto, tipos y estructura JWT

## 2.11 Los tres tipos de tokens y sus funciones

## 2.12 Flujos de concesión utilizados

## 2.13 Endpoints del servidor de autorización

## 2.14 Herramientas: curl y Postman

## 2.15 Referencias normativas

# 3.0 Procedimientos.

## 3.1. PREPARACIÓN DEL ENTORNO

En mi Windows no tengo Docker instalado y ni pienso instalarlo. Voy directamente con WSL, vamos a ver que máquinas tenemos:

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

Si abro WSL desde una carpeta de PowerShell, me deja en su equivalente dentro de Linux, que suele ser algo como /mnt/c/Users/User. Eso que veo montado en /mnt/c es mi disco C: de Windows, accesible desde Linux como si fuera una carpeta más. Por eso el home de Linux y la carpeta de usuario de Windows son dos sitios distintos, y para irme al mío uso `cd ~`.

Sobre si es una máquina virtual: sí, pero no del tipo que tienes en la cabeza. WSL2 ejecuta un núcleo Linux real dentro de una máquina virtual ligera sobre Hyper-V. La diferencia con VirtualBox o VMware es que está muy integrada: arranca en un par de segundos, comparte el localhost con Windows y te monta los discos de Windows automáticamente. No es una emulación ni una capa de traducción, es Linux de verdad, solo que con las costuras muy disimuladas.

Un detalle que importa: yo trabajo siempre dentro de /home/user, no en /mnt/c. El acceso a los ficheros de Windows desde Linux pasa por una capa de traducción y va bastante más lento. Con Docker, clonando repositorios o compilando, se nota mucho. /mnt/c lo dejo solo para mover ficheros de un lado a otro.

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

Lo interesante es que la obligación viaja con la identidad, no con la aplicación. Da igual desde dónde intente entrar el usuario, la acción le salta igual, porque quien la impone es el proveedor de identidad.

Yo lo dejo vacío. Si marco cualquiera, cuando llegue al flujo con Postman el navegador me va a interrumpir pidiendo esa tarea y me rompe el ejercicio. Lo mismo con la contraseña temporal, que la desactivo en la pestaña Credentials.

<img width="942" height="483" alt="imagen" src="https://github.com/user-attachments/assets/17d2d022-497c-4b18-8457-d37354212dac" />

Y aceptamos y lo creamos. Como nos habremos dado cuenta, el estilo de la interfaz es muy AWS.

<img width="1575" height="891" alt="imagen" src="https://github.com/user-attachments/assets/95dbb0a7-1669-497d-85c9-d343eb3532d6" />


> [!IMPORTANT]
> El usuario se ha creado a mano, campo por campo. Eliminar exactamente ese trabajo<br>
> manual es la razón de existir de SCIM, que es el objeto de una práctica posterior.
>

Vamos además a añadirle una contraseña, más que nada porque sin contraseña no podrá autenticarse y el usuario por lo tanto es inservible literalmente:

<img width="966" height="562" alt="imagen" src="https://github.com/user-attachments/assets/03f20048-6021-4ec7-981c-374fabfef7f2" />

En mi caso contraseña "a".

## 3.3. CLIENTE CONFIDENCIAL Y FLUJO DE CREDENCIALES DE CLIENTE

<p><b>Qué es un cliente en OAuth</b></p>

Un cliente no es una persona ni un usuario. Es una aplicación registrada que tiene permiso para pedir tokens al proveedor de identidad. Cuando das de alta un cliente en Keycloak, lo que estás haciendo es decir: esta aplicación existe, se llama así, y puede solicitar tokens bajo estas condiciones.

<p><b>Público frente a confidencial</b></p>

La diferencia está en una sola pregunta: <b>¿esta aplicación puede guardar un secreto sin que nadie lo vea?</b>

- Un cliente público no puede. Vive en un navegador o en un móvil, su código es inspeccionable por el usuario, y cualquier contraseña que metiéramos dentro estaría a la vista. Por eso se autentica sin secreto y necesita protecciones adicionales como PKCE.
- Un cliente confidencial sí puede. Corre en un servidor controlado, nadie puede leer su configuración, y por tanto se le puede entregar un secreto. Ese secreto es su credencial: es lo que prueba que la petición viene de él y no de un impostor.

<p><b>El flujo de credenciales de cliente</b></p>

Es el flujo más simple de OAuth porque elimina a la persona de la ecuación. El cliente presenta su client_id y su client_secret al proveedor, y recibe un access token a su propio nombre.

<p><b>Comparado con el flujo con usuario, aquí desaparecen tres cosas:</b></p>

- No hay navegador, porque no hay nadie a quien mostrarle una pantalla de acceso.
- No hay consentimiento, porque nadie está delegando el acceso a sus datos. El programa actúa por sí mismo, no en nombre de otro.
- No hay refresh token, porque no hace falta. El cliente conserva su secreto y puede pedir un token nuevo cuando quiera, sin molestar a nadie.

<p><b>Dónde se usa esto de verdad</b></p>

- Un proceso nocturno que consolida datos y llama a varias APIs internas.
- Un microservicio que llama a otro microservicio.
- Un conector de aprovisionamiento que crea y borra cuentas en una aplicación destino. Es el caso de la práctica siguiente: el motor que empuja identidades por SCIM se autentica exactamente así.

<p><b>La cuenta de servicio</b></p>

Aquí hay un detalle propio de Keycloak. Un token tiene que hablar de alguien, necesita un sujeto. Como en este flujo no hay persona, Keycloak se crea por dentro un usuario que representa al propio cliente: la cuenta de servicio (service account).

No es un capricho, resuelve un problema real: gracias a esa cuenta se le pueden asignar roles al programa igual que a una persona. Y de aquí sale una idea que en IAM corporativo pesa mucho, y es que las aplicaciones también son identidades y también hay que gobernarlas. Tienen permisos, se acumulan, caducan y hay que recertificarlas. En un banco suele haber más cuentas de servicio que empleados, y son justo las que peor se controlan.

<p><b>Y una consecuencia de seguridad importante</b></p>

El secreto del cliente es una credencial de larga duración que no caduca sola. Quien lo tenga puede obtener tokens indefinidamente hasta que alguien lo rote. Por eso no se escribe en el código ni se sube a un repositorio, y por eso existen las bóvedas de secretos. Cuando en el .gitignore se excluye el fichero .env, es literalmente esto lo que se está protegiendo.

Entonces, vamos a crear el cliente. Muy importante, que estemos en el realm adecuado.

<img width="397" height="131" alt="imagen" src="https://github.com/user-attachments/assets/7f3c2293-9674-4795-ac6a-26715deaa8a1" />

y nos vamos a clients.

<img width="1405" height="677" alt="imagen" src="https://github.com/user-attachments/assets/3deb0fe4-b1e5-4b00-a14a-c9a75e1bb501" />

<img width="932" height="837" alt="imagen" src="https://github.com/user-attachments/assets/c8f602d2-82e4-4ba0-82bf-6fcef18b7353" />

<p><b>El client type es:</b></p>

Es el protocolo que va a hablar esa aplicación con Keycloak. Solo hay dos opciones y son excluyentes: un cliente habla OIDC o habla SAML, no los dos.

OpenID Connect es el moderno, construido sobre OAuth 2.0. Intercambia tokens JWT, funciona por peticiones HTTP con JSON, y es lo que usan las aplicaciones web actuales, las móviles y las APIs. Es el que usamos aquí.

SAML 2.0 es anterior, de principios de los 2000. Intercambia aserciones en XML firmado y el navegador las transporta mediante formularios que se autoenvían. Se diseñó pensando en el inicio de sesión único entre organizaciones, no en APIs.

SAML no está muerto ni de lejos: en banca, administración pública y universidades hay muchísimas aplicaciones que solo hablan SAML, y una plataforma de IAM tiene que sostener ambos. Por eso Keycloak los ofrece.

La diferencia práctica más relevante aquí: con SAML no hay access token que enviar a una API. El resultado del flujo es una aserción que establece una sesión en la aplicación. Para proteger una API, SAML no sirve.

<p><b>El client ID es:</b></p>

Es el identificador público de la aplicación. El nombre con el que esa aplicación se presenta ante Keycloak.

La analogía directa: si comparas un cliente con una cuenta de usuario, el client_id es el nombre de usuario y el client_secret es la contraseña. Uno identifica, el otro demuestra.

Tres cosas a tener claras:
- No es secreto. Viaja en cada petición, aparece en las URL del navegador cuando el flujo es interactivo y cualquiera puede verlo. No pasa nada, porque identificar no es autenticar. Lo que hay que proteger es el secreto.
- Es único dentro del realm. No puede haber dos clientes con el mismo client_id en lab-iam, aunque sí podría existir uno igual en otro realm, porque son mundos separados.
- Es lo que se escribe en cada petición de token. Más adelante, al lanzar el curl, aparecerá el parámetro client_id=api-backend. Con eso Keycloak sabe qué aplicación está pidiendo, qué flujos tiene permitidos y qué debe meter en el token.

Y cambiarlo después rompe todas las integraciones que ya lo usan, porque es la referencia que tienen configurada.

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

Un detalle que me confirma que lo he configurado bien: aquí solo salen Root URL y Home URL. No aparecen las Valid redirect URIs ni los Web origins, y es porque al desmarcar Standard flow le he dicho a Keycloak que este cliente nunca va a pasar por un navegador. Sin navegador no hay retorno que autorizar.

Qué son esos dos campos, por si te los encuentras en el siguiente cliente:

- Root URL es la raíz de la aplicación. Sirve de prefijo para las demás URL, de modo que puedas escribirlas en relativo y cambiar el dominio en un solo sitio al pasar de desarrollo a producción.
- Home URL es a dónde se envía al usuario cuando entra a la aplicación desde Keycloak, por ejemplo desde la consola de cuenta.

Ambos son comodidades de configuración, no tienen efecto de seguridad. Los que sí lo tienen son las Valid redirect URIs, que verás cuando crees el cliente público.

Y lo creamos:

<img width="1356" height="862" alt="imagen" src="https://github.com/user-attachments/assets/77e6cea3-2d0d-4519-8783-26875e4428b6" />

Antes de pedir el primer token le voy a añadir al cliente un mapeador de audiencia. Esto no lo tenía previsto, lo puse después de pegarme un rato con un fallo, y lo dejo aquí en su sitio para que no le pase a nadie más.

Por defecto, el token que emite Keycloak para este cliente lleva `"aud": "account"`, o sea que va dirigido al cliente interno de gestión de cuenta y no a mi API. Y resulta que desde la versión 26.6.2 el endpoint de introspección comprueba que el cliente que pregunta esté dentro de la audiencia del token, como arreglo de una vulnerabilidad (CVE-2026-37979). Así que sin este mapeador, al introspeccionar mi propio token la respuesta es `{"active": false}`, aunque el token sea perfectamente válido. Me volví loco un rato hasta dar con esto.

Visto con calma es razonable: si cualquier cliente pudiera introspeccionar cualquier token, bastaría con darse de alta un cliente cualquiera para andar espiando tokens ajenos.

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
- Piénsalo así: el client_id es público y aparece en cualquier sitio. Si bastara con enviarlo, cualquiera que lo supiera podría pedir tokens haciéndose pasar por mi proceso, y con esos tokens entrar a mi API. El secreto es lo que convierte "digo que soy api-backend" en "demuestro que soy api-backend".
- Y en este flujo concreto tiene un peso especial, porque es la única credencial que hay. En el flujo con persona, la seguridad se apoya en la contraseña del usuario y en todo el proceso del navegador. Aquí no hay nada de eso: el secreto es lo único que separa a mi proceso de cualquier otro. Por eso este flujo solo se permite a clientes confidenciales, que son los que pueden custodiarlo.

De aquí sale una regla práctica: ese valor no se escribe en el código ni se sube al repositorio, se guarda en una variable de entorno o en una bóveda de secretos. Vale lo mismo que una contraseña, con el agravante de que no caduca sola y suele estar en manos de varios equipos.

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

Funciona, y confirma tres cosas:
- No hay refresh_token. Y además `refresh_expires_in: 0`, o sea que Keycloak lo dice de forma explícita. En este flujo no hace falta: el cliente tiene el secreto y puede pedir otro token cuando quiera.
- `expires_in: 300`, cinco minutos. Ese es el valor por defecto del realm, y más adelante lo bajaremos a un minuto para ver la caducidad en directo.
- `scope: "email profile"`, sin openid. Por eso este token no es OIDC, es OAuth puro. Y por eso, al probar el endpoint /userinfo con él, lo rechaza. Lo comprobamos más abajo.

Ahora vamos a decodificar el cuerpo del token, copiando ese "churro" de texto:

```
echo 'PEGA_AQUI_EL_TOKEN' | cut -d. -f2 | base64 -d 2>/dev/null | jq
```

<img width="915" height="892" alt="imagen" src="https://github.com/user-attachments/assets/883915aa-b446-4005-8938-e89eb33b14d4" />

<p><b>Tiempos</b></p>
- `iat` (issued at): 1790526999, que es el 27 de septiembre de 2026 a las 16:36:39 UTC, o sea las 18:36 hora peninsular. El instante en que Keycloak lo emitió.
- `exp` (expiration): 1790527299, las 16:41:39 UTC. Resta uno del otro y salen exactamente 300 segundos, los cinco minutos que anunciaba expires_in.

Ambos van en segundos desde el 1 de enero de 1970, que es el formato epoch de Unix. Se usa así porque es un entero sin zonas horarias ni ambigüedades de formato: cualquier sistema del mundo lo interpreta igual.

Quien valida el token compara exp con su propio reloj. De ahí un problema clásico en producción: si los relojes de dos servidores van desincronizados, uno puede rechazar tokens que el otro acaba de emitir. Por eso en entornos serios se sincroniza la hora por NTP y los validadores admiten un margen de tolerancia de unos segundos.
- `jti` (JWT ID): identificador único de este token concreto. Sirve para detectar reutilizaciones, para mantener listas de revocación y, sobre todo, para correlacionar en auditoría: si en el registro de mi API apareciera una operación sospechosa con ese jti, podría cruzarlo con el registro de emisión de Keycloak y saber exactamente quién y cuándo lo pidió. El prefijo trrtcc: es algo interno de Keycloak, y no he encontrado documentación que confirme su significado, así que lo dejo señalado como pendiente en lugar de suponerlo.

<p><b>Quién y para quién</b></p>

- `iss` (issuer): http://localhost:8080/realms/lab-iam. Quién emitió el token. Una API tiene que comprobar que este valor coincide exactamente con el emisor que espera. Y aquí hay una trampa que veo venir para más adelante: si la API corre dentro de otro contenedor, para ella localhost no es Keycloak, es ella misma. Ese desajuste entre la URL pública y la interna rompe la validación, y por lo que he leído es de los fallos más típicos al desplegar.
- `aud` (audience): account. Para quién está pensado el token. Aquí no aparece mi API, aparece el cliente interno de gestión de cuenta de Keycloak, porque es el único destinatario que el realm sabe añadir por defecto.

La consecuencia práctica: una API que valide la audiencia con rigor rechazaría este token, y haría bien. Un token emitido para un destinatario no debería servir en otro, porque si no, cualquier servicio que reciba tu token puede darse la vuelta y usarlo contra un tercero haciéndose pasar por ti. Se corrige añadiendo un mapeador de audiencia al cliente o al ámbito.

- `sub` (subject): 48ec1288-.... El identificador del sujeto del que habla el token. Es un UUID y no un nombre a propósito: los nombres cambian (una persona se casa, un cliente se renombra) y el identificador no debe cambiar nunca, porque es lo que las aplicaciones guardan para asociar sus datos a esa identidad.
- `azp` (authorized party): api-backend. Qué cliente pidió el token. En este caso coincide con quien lo va a usar, pero en flujos donde un cliente pide tokens destinados a otro, azp y aud son distintos y esa diferencia importa.
- `typ`: Bearer. El tipo de token, portador.

<p><b>Autenticación y autorización</b></p>

- `acr` (authentication context class reference): 1. Indica con qué nivel de garantía se autenticó el sujeto. Se usa para políticas del tipo "para transferir más de mil euros exijo que te hayas autenticado con doble factor en los últimos cinco minutos". Más adelante veremos que en el flujo con persona este valor cambia según se haya autenticado de nuevo o se reutilice una sesión existente.
- `realm_access.roles`: los roles de realm que trae el sujeto. Los tres que ves son de serie:
  - default-roles-lab-iam es un rol compuesto que Keycloak asigna automáticamente a todo el mundo y que agrupa los permisos mínimos.
  - offline_access permite solicitar tokens de sesión desconectada, los que sobreviven a que el usuario cierre el navegador.
  - uma_authorization va ligado al motor de autorización de grano fino, ese que dejaste en Off.

- resource_access.account.roles: roles sobre un cliente concreto, en este caso sobre account. Le permiten ver y gestionar su propio perfil.

Ahí se ve la diferencia entre los dos planos: los roles de realm valen en todo el realm, y los de cliente solo dentro de una aplicación. Un mismo usuario puede ser "lector" en una aplicación y "administrador" en otra sin que choquen, porque cada rol vive en su cliente.

- scope: email profile. Los ámbitos concedidos. Falta openid, y por eso este token no activa el comportamiento OIDC.

<p><b>Contexto y ruido</b></p>
- `clientHost` y `clientAddress`: `172.17.0.1`, la dirección desde la que se hizo la petición vista desde dentro del contenedor. Esa IP es la pasarela de la red de Docker, o sea tu WSL visto desde Keycloak. Es información de auditoría.
- `email_verified: false` y `preferred_username: service-account-api-backend`: vienen del perfil de la cuenta de servicio que Keycloak creó sola. El correo verificado aquí no significa nada, porque esta identidad no tiene correo.

## 3.4. INTROSPECCIÓN Y ENDPOINTS DEL REALM

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

El -u es autenticación HTTP básica: manda usuario y contraseña en una cabecera. Aquí el usuario es el client_id y la contraseña el secreto. Detalle que no me esperaba: para preguntar por un token también hay que estar autenticado. Keycloak no le cuenta a cualquiera qué lleva dentro un token ajeno.

Con un token recién pedido, la respuesta es `"active": true` seguida de las mismas afirmaciones que ya habíamos decodificado.

<img width="922" height="205" alt="imagen" src="https://github.com/user-attachments/assets/e5471001-780f-42c9-a37e-6fba012566fb" />

En mi primera prueba me devolvió false, porque habían pasado más de cinco minutos desde que pedí el token y ya se me había caducado. Lo curioso es que el token seguía siendo perfectamente legible al decodificarlo, pero el servidor ya no lo aceptaba.

Aquí hay dos formas de validar un token y merecen una comparación:

- **Validación local**: rápida y sin depender del proveedor en cada petición, pero no detecta una revocación hasta que el token caduca por su cuenta.
- **Introspección**: detecta la revocación al instante, pero añade una llamada de red y acopla el servicio al proveedor.

En banca conviven las dos. La elección depende de cuánto duelen los milisegundos frente a cuánto duele un token revocado que sigue siendo aceptado.

<img width="720" height="952" alt="imagen" src="https://github.com/user-attachments/assets/4cc7ebf0-c003-4cda-81d7-c95512ddb00b" />

Dos consultas más cierran este bloque. La primera devuelve las claves públicas del realm:

```
curl -s http://localhost:8080/realms/lab-iam/protocol/openid-connect/certs | jq
```

Lo primero que hice fue comparar el `kid` de la clave de firma con el `kid` de la cabecera de mi token, y coinciden. Ahí se cierra el círculo: la cabecera del token dice con qué clave se firmó, y este endpoint publica esa clave para que cualquiera lo compruebe sin preguntarle nada a Keycloak.

Aparecen dos claves con funciones distintas: una con `"use": "sig"` y algoritmo RS256, que es la que firma, y otra con `"use": "enc"`, que sirve para cifrar tokens cuando se configura esa opción. Los campos `n` y `e` son el módulo y el exponente de la clave pública RSA. Nada de esto es secreto, por eso se publica en abierto.

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


La segunda es el documento de descubrimiento, que es el mapa completo del realm y el punto de partida correcto para integrarse con cualquier proveedor de identidad ajeno:

```
curl -s http://localhost:8080/realms/lab-iam/.well-known/openid-configuration | jq
```

De toda esa parrafada me quedo con cinco cosas:

- `grant_types_supported` lista todos los flujos que el realm sabe hacer, incluidos `implicit` y `password`. Cuidado con leerlo mal: eso es lo que admite el realm, no lo que cada cliente tiene permitido. En `api-backend` esos flujos los dejé cerrados.
- `code_challenge_methods_supported` con `S256` confirma que PKCE está disponible, cosa que necesitaremos en el siguiente bloque.
- `scopes_supported` incluye `openid`, que es justo el ámbito que le falta a nuestro token actual.
- `revocation_endpoint` y `end_session_endpoint` son URL distintas: revocar un token concreto y cerrar sesión no son lo mismo.
- `dpop_signing_alg_values_supported` indica que el realm admite DPoP, el mecanismo que ata un token a una clave y que aparecía como interruptor al crear el cliente.

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

Y esta última llamada no nos devolverá nada.

Keycloak, en este endpoint, no manda un JSON de error. Pone toda la información en el código de estado y en la cabecera WWW-Authenticate. Así que jq recibe cero bytes, no tiene nada que formatear y no imprime nada. No es un fallo, es que estás mirando el sitio equivocado.

Por eso hace falta -i, que muestra las cabeceras además del cuerpo.

La lección que me llevo: no todas las APIs informan de los errores igual. Unas devuelven un JSON con el detalle, otras lo meten en cabeceras y otras solo dejan el código de estado. Cuando algo "no devuelve nada", lo primero es mirar con -i antes de dar por hecho que está roto.

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

Aquí tengo la diferencia entre 401 y 403 en vivo, que es justo lo que quería ver. Cuando el token es inválido o ha caducado, sale un **401** con `invalid_token`: el servidor no puede establecer quién soy, así que ni llega a plantearse qué permitirme. Cuando el token es válido pero le falta el ámbito, sale un **403** con `insufficient_scope`: sabe perfectamente quién soy, y justo por eso puede denegarme.

Ese formato de respuesta lo define el RFC 6750, y no es un adorno: permite que un cliente reaccione distinto según el motivo. Ante `invalid_token` pide uno nuevo, y ante `insufficient_scope` ya sabe que pedir otro igual no le va a servir y que lo que tiene que cambiar son los ámbitos.

### 3.4.1 Provocar los fallos: caducidad y firma

Dos experimentos que en ambos casos nos deben rechazar:

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

Detalle que me llamó la atención: cuando el token no es válido, la introspección no cuenta absolutamente nada de él. No dice que caducó, ni de quién era. Y es a propósito, porque si diera detalles serviría para sonsacar información sobre tokens ajenos.

El contraste está en que si decodifico ese mismo token caducado, sus datos siguen ahí perfectamente legibles. El token no se destruye ni se borra, simplemente deja de ser aceptado.

#### 3.4.1.2 Experimento 2: la firma manipulada.

Aquí voy a modificar el contenido del token dejando la firma original, que es lo que intentaría un atacante para, por ejemplo, darse roles que no tiene.

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

Lo que saco de aquí: el contenido de un JWT lo puede cambiar cualquiera, porque base64 es codificación y no cifrado. Lo que no se puede falsificar es la firma, porque haría falta la clave privada del realm y esa no sale de Keycloak. La regla entonces es no fiarse nunca de un JWT sin verificar, por muy razonable que parezca lo que pone dentro.

## 3.5. CLIENTE PÚBLICO CON PKCE DESDE POSTMAN

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

PKCE (RFC 7636) funciona así: el cliente se inventa un valor aleatorio, manda su resumen SHA-256 al pedir el código de autorización, y luego presenta el valor original al canjearlo. Si alguien intercepta el código por el camino no puede hacer nada con él, porque no conoce ese valor.

El nombre es arbitrario, elegido para que se entienda de un vistazo qué representa cada cliente:
- api-backend: un proceso de servidor, confidencial, sin persona.
- spa-web: una aplicación de navegador, pública, con persona.

Dos cosas sobre los campos de Login settings, que estos sí tienen efecto de seguridad:

- La **URI de retorno** es el control principal de este flujo. Keycloak solo entrega el código de autorización a una dirección que esté en esa lista. Si no existiera esa comprobación, cualquiera podría arrancar el flujo con mi client_id y pedir que el código aterrizara en su propio servidor. Por eso en producción no se pone un comodín ahí ni de broma.
- Los **Web origins** son otra cosa distinta: controlan CORS, o sea desde qué dominios puede el navegador llamar a Keycloak por JavaScript. El asterisco me lo permito porque esto es un laboratorio.


Vamos a instalar postman para escritorio en Windows:

<a> https://www.postman.com/downloads/ </a>

Un cliente gráfico de HTTP pensado para trabajar con APIs. En esencia hace lo mismo que curl, pero con interfaz y con memoria.

Lo que aporta frente a la terminal:

Colecciones: guardas las peticiones organizadas en carpetas y las reutilizas. No dependes del historial de la shell.
Variables de entorno: defines una vez la URL base o el client_id y las usas en todas las peticiones con {{nombre}}. Cambiar de desarrollo a producción es cambiar de entorno, no reescribir cada petición.
Scripts: puedes ejecutar código antes y después de cada petición, por ejemplo para extraer el token de la respuesta y guardarlo en una variable automáticamente.
Ayudante de OAuth 2.0: y esta es la razón por la que lo usamos ahora. Ejecuta el flujo completo por ti, incluido el paso por el navegador para que te autentiques, y la gestión de PKCE. Con curl ese flujo es un incordio, porque hay que interceptar a mano el código de autorización que vuelve en la URL.

En el sector se usa a diario para probar endpoints, depurar integraciones y documentar APIs.

El primer flujo lo hice con curl a propósito, para ver la mecánica sin capas por encima. Ahora que entiendo lo que pasa por debajo, Postman es comodidad y no magia.

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

Dos de esos campos hay que explicarlos. El **State** no está de adorno: es un valor que genera el cliente, viaja en la petición y vuelve en la respuesta, y sirve para comprobar que lo que recibe corresponde a algo que pidió él. Es la defensa clásica contra CSRF en este flujo. Y el **Code Verifier** que Postman rellena solo es la pieza de PKCE que explicaba antes.

Y le tenemos que dar a "Get new access token":

<img width="1305" height="530" alt="imagen" src="https://github.com/user-attachments/assets/9249bccf-7a7e-48a0-9224-08bac0fc781a" />

Y nos abrirá una nueva pestaña en el navegador. A mí me salió un error "Client not found", y era porque había creado el cliente en el realm equivocado. Los realms son fronteras estrictas, y un cliente que vive en `master` no existe para `lab-iam`.

> [!NOTE]
> Otra cosa que me pasó: al terminar la autenticación no volvía nada a Postman. Era el bloqueador de ventanas emergentes del navegador. El retorno del flujo se hace abriendo una ventana nueva, así que si el navegador la bloquea el token no llega nunca, aunque te hayas autenticado bien.

<img width="1427" height="717" alt="imagen" src="https://github.com/user-attachments/assets/d15e9161-0b4e-4e05-9845-426e86055284" />

Finalmente nos autenticamos con el único usuario que tenemos en el realm, que es cesar23:

<img width="945" height="372" alt="imagen" src="https://github.com/user-attachments/assets/f88354e4-a46e-4e4e-b96a-da13c26c51e3" />

Y aquí lo tenemos:

<img width="1381" height="807" alt="imagen" src="https://github.com/user-attachments/assets/f76dcb85-c2d0-46ee-b951-42ebe12cc2a1" />

### 3.5.1 Los tres tokens, comparados

Para mí este panel es el momento clave de toda la práctica, porque es la primera vez que veo los tres tokens juntos. Antes de seguir los voy a comparar, porque cada uno sirve para una cosa distinta y eso se nota en lo que llevan dentro.

Decodificando los tres cuerpos, lo que mejor los distingue es la audiencia:

- access_token: `"aud": "account"`, `"azp": "spa-web"`, `"typ": "Bearer"`.
- id_token: `"aud": "spa-web"`, `"typ": "ID"`.
- refresh_token: `"aud": "http://localhost:8080/realms/lab-iam"`, `"typ": "Refresh"`.

Y ahí está la teoría que había leído, pero ahora con datos delante. El ID token va dirigido al cliente, porque su trabajo es contarle a la aplicación quién ha entrado. El refresh token va dirigido al propio realm, porque solo el servidor de autorización debería recibirlo. Y el access token va dirigido a una API.

Hay otra cosa que a mí se me habría pasado y que explica mucho: los algoritmos de firma no son los mismos.

- El access token y el id token van firmados con **RS256**, y su `kid` es el mismo que aparece publicado en el endpoint de claves.
- El refresh token va firmado con **HS512**, y su `kid` no aparece en ese endpoint.

RS256 es asimétrico: Keycloak firma con su clave privada y cualquiera valida con la pública, que está publicada. Lógico, porque esos dos tokens los van a validar terceros. HS512 es simétrico: la misma clave firma y valida, y por eso no la publica en ningún sitio. También lógico, porque nadie que no sea Keycloak tiene por qué validar un refresh token. Ese token no está hecho para que lo lea nadie, está hecho para volver a casa.

Las vidas también difieren: 300 segundos el access token y 1800 el refresh token. En producción la distancia es mucho mayor, pero la proporción ya ilustra la idea de que uno se usa mucho y dura poco, y el otro se usa poco y dura mucho.

En cuanto al contenido, el id token trae nombre, usuario y correo, pero ningún rol. El access token trae los roles además del perfil. Y el refresh token no trae ni nombre ni roles, solo identificadores, porque nadie lo va a leer para decidir nada.

Aparecen además dos campos que no existían en el flujo anterior: `sid` y `session_state`, con el mismo valor. Es el identificador de la sesión de usuario en Keycloak. En el flujo de credenciales de cliente no había ninguno, porque no había persona ni sesión que mantener. Es lo que permite el inicio de sesión único: si otra aplicación inicia un flujo con este mismo navegador, Keycloak reconoce la sesión y no vuelve a pedir credenciales.

También aparece `auth_time`, que dice cuándo me autentiqué de verdad, y que no tiene por qué coincidir con el momento en que se emitió el token. En mi caso había varios minutos de diferencia entre `auth_time` e `iat`, y es porque se estaba reutilizando una sesión que ya tenía abierta. Fijándome más, el `acr` valía 0 cuando reutilizaba la sesión y 1 cuando me acababa de autenticar. Lo dejo como observación de lo que he medido yo, no como algo que haya podido confirmar en la documentación.

### 3.5.2 Qué devuelve /userinfo

/userinfo devuelve las afirmaciones de identidad de la persona dueña del token, limitadas a los ámbitos concedidos. Con los ámbitos openid email profile, la respuesta es esta:

<pre>
{
  "sub": "920ba24c-b418-4c90-99af-48c49078b7a1",
  "email_verified": true,
  "name": "cesar venegas",
  "preferred_username": "cesar23",
  "given_name": "cesar",
  "family_name": "venegas",
  "email": "cesar.venegas@companiaficticia.com"
}
</pre>

Datos de perfil y nada más. Ni roles, ni tokens, ni información de sesión.

Y la pregunta natural es: si eso ya viene dentro del id_token, ¿para qué existe este endpoint? Por dos motivos.

El id_token es una foto del momento del inicio de sesión. Si el usuario cambia su correo media hora después, el id_token que tiene la aplicación sigue diciendo el antiguo. /userinfo se consulta en vivo y devuelve el estado actual.

Permite mantener los tokens pequeños. Un proveedor puede emitir un id_token mínimo y dejar que quien necesite el perfil completo lo pida aparte. Los tokens viajan en cada petición, así que cada byte cuenta.

Otra cosa a mirar es el sujeto: ese sub es el mismo que sale en los tres tokens. Es el identificador de cesar23 dentro de este realm y no cambia nunca, así que es lo que una aplicación guardaría en su base de datos para asociarle sus datos. Nunca el nombre de usuario ni el correo, que sí pueden cambiar.

Y lo que cierra el bloque: es exactamente el mismo endpoint que antes me devolvió un 403 por no llevar el ámbito openid. Misma URL y mismo servidor. Lo único que ha cambiado es qué token presento.

Le damos a `Use this token`.

Y ahora ya finalmente le podemos dar a `Send`.

<img width="1410" height="657" alt="imagen" src="https://github.com/user-attachments/assets/c9e63bae-bd6f-4244-9647-df6792acddd9" />

Y abajo nos aparecerá como la "query":

<img width="1406" height="787" alt="imagen" src="https://github.com/user-attachments/assets/ea15641b-3271-4e3f-9a70-d750fd47780e" />

## 3.6 VER LA RENOVACIÓN Y LA ROTACIÓN EN DIRECTO

Desde Keycloak, vamos al realm `lab-iam`.

<img width="1746" height="772" alt="imagen" src="https://github.com/user-attachments/assets/efd8e780-73cd-47a0-8756-b2849ce2802f" />

Ponemos el "Access Token Lifespan" a 1 minuto. Bajamos para abajo:

<img width="782" height="617" alt="imagen" src="https://github.com/user-attachments/assets/c00560e6-0555-40c5-84ce-3436c6bb2fef" />

Encontraremos el "Revoke Refresh Token: On" y el "Refresh Token Max Reuse: 0". Y guardamos.
Ahora en el Postman, volvemos a pedir un access token:

<img width="996" height="607" alt="imagen" src="https://github.com/user-attachments/assets/9aff61ea-fd05-40d7-8e54-03593df2289a" />

Copiaremos el refresh token en una variable:

```
RT='PEGA_AQUI_EL_REFRESH_TOKEN'
```

Y luego lo "canjeamos":

```
curl -s -X POST http://localhost:8080/realms/lab-iam/protocol/openid-connect/token \
  -d grant_type=refresh_token \
  -d client_id=spa-web \
  -d refresh_token=$RT | jq
```

Con esto confirmaremos la rotación: el refresh token que ha llegado es distinto al que enviamos.

Dentro de ese refresh token nuevo aparece además un campo que antes no estaba, `reuse_id`. Sale justo al activar Revoke Refresh Token, y es el marcador con el que Keycloak sigue la cadena de rotaciones para pillar a quien intente usar un eslabón ya gastado.

<img width="1476" height="237" alt="imagen" src="https://github.com/user-attachments/assets/f6375909-8aa4-47f2-88bd-73c6fdbe97a4" />

Finalmente, vamos a ver que pasa si volvemos a ejecutar este mismo comando, el de antes:

```
curl -s -X POST http://localhost:8080/realms/lab-iam/protocol/openid-connect/token \
  -d grant_type=refresh_token \
  -d client_id=spa-web \
  -d refresh_token=$RT | jq
```

<img width="1247" height="200" alt="imagen" src="https://github.com/user-attachments/assets/f13e7073-4506-4674-80f3-750d1d2dfdc7" />

La respuesta es `invalid_grant` con la descripción "Maximum allowed refresh token reuse exceeded", que enlaza directamente con el Refresh Token Max Reuse que puse a 0.

Nos da error y eso está bien, porque es así como lo configuramos. Lo que hay detrás es esto: si alguien presenta un refresh token ya canjeado, significa que hay dos manos sobre la misma credencial, y lo seguro ante esa señal es cortar en lugar de seguir emitiendo tokens.

## 3.7 REVOCAR UN TOKEN TODAVÍA VÁLIDO

Vamos a Postman de nuevo y pedir un juego nuevo de tokens, tanto el refresh como el access.



Ponemos el refresh en esta variable:
```
RT2='PEGA_EL_REFRESH_NUEVO'
```

Ponemos el access en esta variable:

```
AT2='PEGA_EL_ACCESS_DEL_MISMO_JUEGO'
```

Revocamos:

```
curl -i -s -X POST http://localhost:8080/realms/lab-iam/protocol/openid-connect/revoke \
  -d client_id=spa-web \
  -d token=$RT2 \
  -d token_type_hint=refresh_token
```

<img width="1261" height="292" alt="imagen" src="https://github.com/user-attachments/assets/136a9ee3-e6a8-4c92-a2af-00ed3fe0bd4f" />

Intentaremos canjearlo después:

```
curl -s -X POST http://localhost:8080/realms/lab-iam/protocol/openid-connect/token \
  -d grant_type=refresh_token -d client_id=spa-web \
  -d refresh_token=$RT2 | jq
```

La respuesta es un error, que es justo lo que buscábamos:

```json
{
  "error": "invalid_grant",
  "error_description": "Session not active"
}
```

El mensaje dice más de lo que parece. No responde que el token sea inválido, responde que la sesión no está activa. O sea que revocar un refresh token en Keycloak no mata solo esa credencial, cierra la sesión del usuario entera.

Para ver hasta dónde llega, lanzo el access token del mismo juego contra /userinfo justo después de revocar, sin darle tiempo a caducar:

```
curl -i -s http://localhost:8080/realms/lab-iam/protocol/openid-connect/userinfo \
  -H "Authorization: Bearer $AT2"
```

<img width="1332" height="457" alt="imagen" src="https://github.com/user-attachments/assets/03b242af-76f2-4946-8771-1f174cc2b1f6" />

También lo rechaza, con un 401. Y el motivo es que /userinfo es un endpoint del propio Keycloak, así que comprueba que la sesión asociada al token siga viva.

Pero esto no se puede generalizar, y es lo más importante que saco de esta parte: **el alcance de una revocación depende de cómo valide cada servicio**. Una API mía que verifique el JWT en local contra las claves públicas del JWKS no le pregunta nada a Keycloak, así que seguiría aceptando ese access token hasta que se le caducara solo.

Ahí está el compromiso de diseño. Si quiero que una revocación tenga efecto inmediato en todas partes, mis servicios tienen que introspeccionar, y eso cuesta una llamada de red en cada petición. Si prefiero validar en local por rendimiento, asumo una ventana en la que un token revocado sigue funcionando, y esa ventana es exactamente la vida del access token. Por eso se le da una vida tan corta.

Para cerrar el contraste, generamos otro juego de tokens y lo canjeamos sin revocar nada:

<img width="1462" height="765" alt="imagen" src="https://github.com/user-attachments/assets/a1b020a5-26c3-4c7c-bce5-884415759af4" />

Canjeado y funcionando. Lo único que cambia entre las dos pruebas es la llamada al endpoint de revocación.
