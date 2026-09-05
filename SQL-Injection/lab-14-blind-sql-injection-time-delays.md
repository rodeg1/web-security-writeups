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
