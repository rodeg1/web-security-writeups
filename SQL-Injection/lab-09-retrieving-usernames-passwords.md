# SQL Injection UNION Attack – Recuperar nombres de usuario y contraseñas

## 🧪 Laboratorio

PortSwigger Web Security Academy – SQL Injection

**Laboratorio:** SQL injection UNION attack, retrieving usernames and passwords

**Categoría:** SQL Injection

**Dificultad:** Practitioner

---

## 🎯 Objetivo

La aplicación contiene una vulnerabilidad de SQL Injection en el filtro de categoría de productos.

El objetivo del laboratorio es realizar un ataque `UNION SELECT` para recuperar los nombres de usuario y las contraseñas almacenadas en una tabla llamada `users`.

Finalmente, debemos utilizar las credenciales obtenidas para iniciar sesión como el usuario `administrator`.

---

## 🔎 1. Determinar el número de columnas

El primer paso fue determinar cuántas columnas devuelve la consulta original.

Mediante pruebas con `UNION SELECT` se determinó que la consulta devuelve:

```text
2 columnas
```

Por lo tanto, cualquier consulta `UNION SELECT` utilizada posteriormente debe devolver exactamente dos columnas.

---

## 🗂️ 2. Enumerar las tablas

Aunque el laboratorio proporciona el nombre de la tabla objetivo, se realizó una enumeración para identificar las tablas disponibles.

Se utilizó:

```text
'+UNION+SELECT+NULL,table_name+FROM+information_schema.tables--
```

Entre los resultados apareció:

```text
users
```

La tabla `users` era la tabla relevante para el objetivo del laboratorio.

---

## 🔐 3. Identificar los datos objetivo

El laboratorio indica que la tabla `users` contiene las siguientes columnas:

```text
username
password
```

Por lo tanto, los datos que necesitamos recuperar son:

```text
username
password
```

---

## 🧩 4. Combinar Username y Password

Como la consulta original devuelve dos columnas y la segunda columna permite mostrar texto, se utilizó la concatenación de cadenas mediante el operador `||`.

El payload utilizado fue:

```text
'+UNION+SELECT+NULL,username||':'||password+FROM+users--
```

La consulta concatena los dos campos utilizando `:` como separador:

```text
username:password
```

De esta forma fue posible recuperar los nombres de usuario y sus respectivas contraseñas dentro de una única columna reflejada en la respuesta.

---

## 🔑 5. Recuperar las credenciales

La respuesta permitió identificar las credenciales correspondientes al usuario:

```text
administrator
```

Las credenciales obtenidas durante el laboratorio fueron utilizadas para acceder a la función de login.

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

Este laboratorio permitió combinar varias técnicas aprendidas anteriormente:

- Determinar el número de columnas de una consulta.
- Identificar una columna compatible con texto.
- Enumerar tablas mediante `information_schema`.
- Identificar la tabla `users`.
- Consultar información de otras tablas mediante `UNION SELECT`.
- Concatenar diferentes columnas utilizando `||`.
- Recuperar información sensible mediante una vulnerabilidad de SQL Injection.
- Utilizar las credenciales obtenidas para autenticarse como otro usuario.

La técnica general utilizada fue:

```text
Identificar columnas
        ↓
Identificar columna de texto
        ↓
Identificar tabla objetivo
        ↓
Consultar username/password
        ↓
Concatenar los valores
        ↓
Obtener credenciales
        ↓
Autenticarse
```

---

## 🛡️ Mitigación

La vulnerabilidad puede prevenirse evitando que los datos controlados por el usuario sean incorporados directamente en las consultas SQL.

Las principales medidas de protección son:

- Utilizar consultas parametrizadas (*prepared statements*).
- No concatenar datos proporcionados por el usuario directamente en consultas SQL.
- Implementar una validación adecuada de las entradas.
- Aplicar el principio de mínimo privilegio a las cuentas de base de datos.
- Evitar mostrar información detallada de errores SQL.
- Realizar pruebas de seguridad periódicas sobre la aplicación.

---

## 📚 Referencias

- PortSwigger Web Security Academy – SQL Injection
