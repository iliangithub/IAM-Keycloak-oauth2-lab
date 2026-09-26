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

## 2.10  Tokens: concepto, tipos y estructura JWT

## 2.11 Los tres tipos tokens y sus funciones

## 2.12 Flujos de concesion utilizados

## 2.13 Endpoints del servidor de autorizacion

## 2.14 Herramientas: curl y Postman

## 2.15 Referencias normativas

# 3.0 Procedimientos.

