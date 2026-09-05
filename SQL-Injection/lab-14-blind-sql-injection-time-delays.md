# Lab 14 — Blind SQL Injection con retrasos de tiempo

## Resumen

**Plataforma:** PortSwigger Web Security Academy  
**Categoría:** SQL Injection  
**Tipo:** Blind SQL Injection — Time Delays  
**Dificultad:** Practitioner

### Objetivo

Explotar una vulnerabilidad de Blind SQL Injection presente en la cookie `TrackingId` para provocar un retraso de aproximadamente 10 segundos en la respuesta de la aplicación.

---

## Vulnerabilidad

La aplicación utiliza la cookie `TrackingId` para realizar seguimiento de usuarios y posteriormente incorpora su valor dentro de una consulta SQL.

En este laboratorio:

- Los resultados de la consulta SQL no se muestran.
- La aplicación no responde de forma diferente dependiendo de si la consulta devuelve resultados.
- Los errores SQL tampoco proporcionan una diferencia útil.
- La consulta se ejecuta de forma síncrona.

Sin embargo, es posible utilizar **retrasos de tiempo controlados** como canal lateral para determinar si una expresión SQL está siendo ejecutada.

El concepto es:

```text
Solicitud normal
      ↓
Respuesta inmediata

Solicitud con función de espera
      ↓
La base de datos espera
      ↓
Respuesta aproximadamente 10 segundos después
```

---

## 1. Identificación del punto de inyección

Se interceptó una petición `GET /` utilizando Burp Suite y se identificó la cookie:

```http
Cookie: TrackingId=...
```

El objetivo inicial era comprobar si era posible introducir una función de retraso dentro de la consulta SQL.

---

## 2. Prueba de retraso temporal

Se utilizó el siguiente payload:

```text
' || pg_sleep(10)--
```

La cookie quedó conceptualmente:

```http
Cookie: TrackingId=' || pg_sleep(10)--
```

La expresión utiliza:

```sql
pg_sleep(10)
```

para solicitar al servidor de base de datos una espera de aproximadamente 10 segundos.

El operador:

```sql
||
```

permite concatenar expresiones en PostgreSQL.

Finalmente:

```sql
--
```

comenta el resto de la consulta original.

---

## 3. Resultado

Después de enviar la petición mediante Burp Repeater, la respuesta presentó un retraso aproximado de:

```text
10 segundos
```

En comparación con una petición normal, que respondía prácticamente de inmediato.

Esto confirmó que:

1. El valor de `TrackingId` podía influir en la consulta SQL.
2. La expresión SQL inyectada estaba siendo ejecutada.
3. La función `pg_sleep()` podía utilizarse para introducir un retraso controlado.
4. El tiempo de respuesta podía utilizarse como canal lateral.

El laboratorio quedó resuelto al conseguir el retraso de 10 segundos requerido.

---

## 4. Concepto de Time-Based Blind SQL Injection

En una Time-Based Blind SQL Injection, la aplicación no proporciona información directamente mediante el contenido de la respuesta.

En su lugar, el atacante utiliza el tiempo de respuesta como indicador.

Por ejemplo:

```text
Condición SQL
      ↓
Función de espera
      ↓
Retraso observable
```

Esto permite construir posteriormente condiciones como:

```text
Si la condición es verdadera
        ↓
esperar 10 segundos

Si la condición es falsa
        ↓
responder inmediatamente
```

De esta forma, incluso cuando la aplicación no devuelve datos ni errores útiles, es posible inferir información mediante las diferencias en los tiempos de respuesta.

---

## Metodología

```text
Identificar TrackingId
        ↓
Modificar el valor de la cookie
        ↓
Inyectar una función de espera
        ↓
Enviar la petición mediante Burp Repeater
        ↓
Comparar el tiempo de respuesta
        ↓
Confirmar retraso de aproximadamente 10 segundos
        ↓
Laboratorio resuelto
```

---

## Herramientas utilizadas

- Burp Suite
- Burp Repeater
- PortSwigger Web Security Academy

---

## Conceptos aprendidos

### Blind SQL Injection

Una Blind SQL Injection puede existir incluso cuando la aplicación no muestra directamente los resultados de la consulta.

En estos escenarios es necesario utilizar canales laterales para obtener información.

### Time-Based SQL Injection

Los retrasos controlados constituyen uno de esos canales laterales.

En este laboratorio se utilizó:

```sql
pg_sleep(10)
```

para generar una diferencia temporal observable.

### PostgreSQL

Durante la explotación se utilizó:

```sql
pg_sleep()
```

como función de espera para introducir un retraso controlado en la ejecución de la consulta.

---

## Comparación con laboratorios anteriores

Este laboratorio permite observar la evolución de diferentes técnicas de Blind SQL Injection:

```text
Lab 11
Conditional Responses
        ↓
Contenido de la respuesta


Lab 12
Conditional Errors
        ↓
Errores HTTP


Lab 13
Visible Error-Based SQL Injection
        ↓
Información directamente en el mensaje de error


Lab 14
Time Delays
        ↓
Tiempo de respuesta
```

Cada técnica utiliza un comportamiento diferente de la aplicación como canal lateral.

---

## Conclusión

Este laboratorio demostró cómo una vulnerabilidad de Blind SQL Injection puede explotarse incluso cuando la aplicación no devuelve resultados diferentes ni proporciona errores útiles.

La clave fue aprovechar la ejecución síncrona de la consulta para introducir un retraso controlado mediante:

```sql
pg_sleep(10)
```

El retraso observable de aproximadamente 10 segundos confirmó que la expresión SQL inyectada estaba siendo ejecutada.

Este tipo de vulnerabilidad demuestra que una aplicación puede filtrar información no solamente mediante su contenido o mensajes de error, sino también mediante características aparentemente indirectas como el **tiempo de respuesta**.

**Estado del laboratorio: Resuelto ✅**
