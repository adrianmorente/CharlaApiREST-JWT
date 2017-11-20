# Temas de que hablar en la presentación

Vamos a hablar de microservicios, de lo que suponen, las ventajas y desventajas que poseen, y cómo desplegar de forma segura una arquitectura basada en ellos.

Como en tan poco tiempo no podemos abarcar todos los términos, iremos compartiendo enlaces a blogs y páginas de documentación que complementen todos los contenidos que tratemos.

#### API (Interfaz de Programación de Aplicaciones)

> Conjunto de funciones y/o métodos ofrecidos por una aplicación para que otro software, a modo de capa de abstracción, la utilice. **Poner Twitter como ejemplo.** En los microservicios suele ser concisa y simple; y su código ha de estar bien testeado siempre.

#### REST (¿Transferencia de Estado Representacional?)

> Acceso y manipulación de recursos en línea mediante un conjunto de instrucciones predefinidas y uniformes. Utiliza los métodos HTTP para `crear, leer, actualizar y borrar` recursos. En un servicio web RESTful, la respuesta puede ser en XML, HTML o JSON.

#### Arquitecturas monolíticas VS Microservicios

> **Fight!** *(Explicar diferencias, convenciones, beneficios, desventajas y lo que hayas desayunado, si encarta.)*

---

# Seguridad en APIs REST

Hablaremos de convenciones y estándares que deberían usarse, algunos ejemplos de control de accesos, etc.

**Recomendar Open Web Application Security Project y [su manual](https://www.owasp.org/index.php/REST_Security_Cheat_Sheet)**.

#### HTTPS

> **Los servicios REST solo deben funcionar sobre el protocolo HTTPS.**

Por nuestra API viajarán usuarios, contraseñas, claves de sesión, etc. Hay que garantizar integridad y confidencialidad de los datos. [Guía sobre securizar Transport Layer](https://www.owasp.org/index.php/Transport_Layer_Protection_Cheat_Sheet).

#### Control de accesos

> En los sistemas monolíticos vistos con anterioridad, se controla el acceso al inicio (con autenticación y gestión de sesiones); y una vez dentro el usuario navega por todo el sistema. Para un sistema distribuido, esto es más difícil de manejar.

+ El control ha de ser local a cada API, y que en ella se tomen las decisiones  de acceso pertinentes. Esto causa mucha latencia.... Solución: Identity Provider + API Gateway.

#### Autenticando con un IdP

[**Autenticando con un IdP**](https://auth0.com/docs/connections/calling-an-external-idp-api)

Usaremos Auth0, por ejemplo.

1. Obtener token que permita conectar con la API de Auth0 durante 24 horas. Ésto se hace a través de la web de Auth0. Lo suyo es automatizar la petición.
2. Consultar a la API usando el token obtenido previamente. Por ejemplo, esto hace una petición del perfil de un usuario a partir de su ID:

```js
var request = require("request");

var options = {
  method: 'GET',
  url: 'https://adrianmorente.eu.auth0.com/api/v2/ADRIANMORENTE',
  headers: { authorization: 'Bearer TOKEN_OBTENIDO' }
};

request(options, function (error, response, body) {
  if (error) throw new Error(error);

  console.log(body);
});
```

3. Extraer el token devuelto por el IdP:

```json
{
  "email": "adrian.morente@cor.reo",
  "name": "Adrian Morente",
  "picture": "https://avatar/selfie.png",
  "gender": "male",
  "user_id": "ADRIANMORENTE",
  ...
}
```

## JSON Web Tokens

> Se trata de otra convención de tokens de seguridad, y son estructuras en JSON que comparten la información entre diferentes APIs. Se deben proteger con algún tipo de firma criptográfica **(explícate algo)**.

**Nunca se deben aceptar ni emitir JWTs sin cifrar, del tipo** `{ "alg" : "none" }`.

**Nunca.**

***NUNCA.***

> Se pueden proteger los JWTs con un **MAC** *(Código de Autenticación de Mensaje)* a modo de protección de integridad. Al usar uno de estos, cualquier servicio puede validar el token y utilizar la misma clave para crear tokens nuevos.

+ **Particularidad: al hacer ésto, todos los servicios deben confiar entre sí mutuamente.**
+ **Problema: si en un solo punto del sistema distribuido se compromete esta clave, se estaría viendo comprometido el sistema completo.**

> Una parte de la comunicación debe validar un JWT por su propia lógica programada, nunca por la información contenida en él mismo (en la cabecera, por ejemplo).

[Confusión de algoritmos en JWT](https://www.chosenplaintext.ca/2015/03/31/jwt-algorithm-confusion.html)

[***Librerías en cualquier lenguaje para tratamiento de JWT***](https://jwt.io/#libraries-io)

***¿Pero cómo funcionan?***

Cuando el usuario se autentica correctamente en el servicio con sus credenciales, se le guarda localmente un JSON Web Token (en almacenamiento local o en cookies). Esto difiere de las sesiones en el servidor y las cookies retornadas de los sistemas monolíticos. **El servidor no guarda nada --> stateless**.

---

## ID tokens

***Completar con info de OpenID Connect explained.***

```js
{
  "sub"       : "alice",
  "iss"       : "https://openid.c2id.com",
  "aud"       : "client-12345",
  "nonce"     : "n-0S6_WzA2Mj",
  "auth_time" : 1311280969,
  "acr"       : "c2id.loa.hisec",
  "iat"       : 1311280970,
  "exp"       : 1311281970,
}
```
