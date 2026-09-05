# SQL Injection UNION Attack – Recuperar múltiples valores en una sola columna

## 🧪 Laboratorio

PortSwigger Web Security Academy – SQL Injection

**Laboratorio:** SQL injection UNION attack, retrieving multiple values in a single column

**Categoría:** SQL Injection

**Dificultad:** Practitioner

---

## 🎯 Objetivo

La aplicación contiene una vulnerabilidad de SQL Injection en el filtro de categoría de productos.

El objetivo del laboratorio es realizar un ataque `UNION SELECT` para recuperar los nombres de usuario y las contraseñas almacenadas en la tabla `users`.

La dificultad adicional de este laboratorio consiste en que debemos recuperar **múltiples valores dentro de una única columna**.

Finalmente, debemos utilizar las credenciales obtenidas para iniciar sesión como el usuario `administrator`.

---

## 🔎 1. Determinar el número de columnas

El primer paso fue determinar el número de columnas devueltas por la consulta original.

Mediante pruebas con `UNION SELECT` se determinó que la consulta devuelve:

```text
2 columnas
```

Por lo tanto, el `UNION SELECT` debe devolver exactamente dos columnas.

---

## 🗂️ 2. Enumerar las tablas

Para identificar las tablas disponibles en la base de datos se consultó `information_schema.tables`.

Payload utilizado:

```text
Pets' UNION SELECT NULL,table_name FROM information_schema.tables-- -
```

Entre los resultados obtenidos se encontró:

```text
users
```

Esta tabla contiene la información necesaria para completar el laboratorio.

---

## 🧱 3. Identificar los datos objetivo

La tabla `users` contiene dos columnas relevantes:

```text
username
password
```

El objetivo es recuperar ambos valores.

Sin embargo, necesitamos hacerlo utilizando una única columna de salida.

---

## 🧩 4. Recuperar múltiples valores en una sola columna

Para combinar los valores de `username` y `password` se utilizó el operador `||`, que permite concatenar cadenas.

El payload utilizado fue:

```text
'+UNION+SELECT+NULL,username||':'||password+FROM+users--
```

La estructura de la consulta es:

```sql
SELECT NULL, username || ':' || password
FROM users
```

El operador `||` concatena ambos valores y `:` se utiliza como separador.

El resultado tiene el siguiente formato:

```text
username:password
```

Por ejemplo:

```text
administrator:********
usuario:********
```

---

## 🔐 5. Obtener las credenciales

La consulta permitió recuperar los nombres de usuario y las contraseñas almacenadas en la tabla `users`.

Entre los resultados se identificó el usuario:

```text
administrator
```

Las credenciales obtenidas durante el laboratorio fueron utilizadas para acceder al formulario de autenticación.

> Las credenciales obtenidas durante el laboratorio no se incluyen en este write-up.

---

## 🏁 6. Autenticación

Se utilizaron las credenciales obtenidas para iniciar sesión como:

```text
administrator
```

La autenticación fue exitosa y el laboratorio quedó marcado como **Solved**.

---

## 🧠 ¿Qué aprendimos?

Este laboratorio demuestra cómo recuperar varios valores cuando solamente disponemos de una columna compatible con datos de tipo texto.

La técnica consiste en concatenar diferentes campos mediante el operador `||`.

El flujo utilizado fue:

```text
Identificar número de columnas
        ↓
Identificar tabla users
        ↓
Identificar username y password
        ↓
Concatenar ambos valores
        ↓
Recuperar las credenciales
        ↓
Autenticarse como administrator
```

Esta técnica es especialmente útil en ataques `UNION SELECT` cuando la cantidad de columnas compatibles con texto es limitada.

---

## 🛡️ Mitigación

La vulnerabilidad de SQL Injection puede prevenirse evitando que los datos proporcionados por el usuario sean incorporados directamente en las consultas SQL.

Las principales medidas de protección son:

- Utilizar consultas parametrizadas (*prepared statements*).
- No concatenar datos controlados por el usuario directamente en consultas SQL.
- Implementar una validación adecuada de las entradas.
- Aplicar el principio de mínimo privilegio a las cuentas de base de datos.
- Evitar mostrar errores detallados de la base de datos.
- Realizar pruebas de seguridad periódicas sobre las aplicaciones.

---

## 📚 Referencias

- PortSwigger Web Security Academy – SQL Injection
