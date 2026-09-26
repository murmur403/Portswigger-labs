# 🧪 Blind SQL Injection with Time Delays

![Plataforma](https://img.shields.io/badge/Plataforma-PortSwigger-purple)
![Dificultad](https://img.shields.io/badge/Dificultad-Practitioner-yellow)
![Categoría](https://img.shields.io/badge/Categor%C3%ADa-Blind%20SQLi-red)

## 📌 Resumen

Laboratorio de **PortSwigger Web Security Academy** que explota una
**inyección SQL ciega basada en tiempo** en la cookie `TrackingId` para causar
un retraso de **10 segundos** en la respuesta del servidor.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Identificación | Cookie `TrackingId` | Punto de inyección |
| DBMS | Comportamiento | **MySQL** |
| Payload | `'||SELECT SLEEP(10)` | Retraso de 10s |
| Confirmación | Tiempo de respuesta | ✅ Lab resuelto |

---

## 🎯 Objetivo

Explotar una vulnerabilidad de **inyección SQL ciega basada en tiempo** para
provocar un retraso de 10 segundos en la respuesta del servidor.

---

## 📖 Contexto

El laboratorio presenta una **SQLi ciega (blind)** en la cookie `TrackingId`:

- Los resultados de la consulta **no se devuelven** en la respuesta.
- La aplicación **no responde diferente** ante errores.
- Pero la consulta se ejecuta **de forma síncrona**.

Esto significa que podemos **inferir información** midiendo el **tiempo de
respuesta** del servidor. Es la base de la SQLi ciega basada en tiempo.

---

## 🔍 Análisis

### 1. Identificar el punto de inyección

Se intercepta la petición `GET /` con **Burp Suite** y se observa que la cookie
`TrackingId` se incluye en una consulta SQL en el servidor.

```
Cookie: TrackingId=2FIp3bG71f2NuclR; session=...
```

### 2. Determinar el gestor de base de datos

El laboratorio usa **MySQL** en esta instancia. La función de pausa en MySQL es:

```sql
SLEEP(segundos)
```

Otras bases de datos usan funciones distintas:

| DBMS | Función de pausa |
|------|------------------|
| MySQL | `SLEEP(10)` |
| PostgreSQL | `pg_sleep(10)` |
| Microsoft SQL Server | `WAITFOR DELAY '0:0:10'` |
| Oracle | `dbms_pipe.receive_message(('a'),10)` |

---

## 💥 Explotación

### 3. Construir el payload

Se añade el payload al valor de la cookie usando concatenación `||`:

```sql
'||SELECT SLEEP(10)
```

**Cookie final:**

```
Cookie: TrackingId=2FIp3bG71f2NuclR'||SELECT SLEEP(10)
```

> 💡 **¿Por qué `||`?** En MySQL, `||` es el operador de concatenación lógica
> (equivalente a `OR`). Permite unir la consulta original con la nuestra.

### 4. Ejecutar el ataque

Se envía la petición con **Forward** en Burp Proxy:

- El servidor tarda **10 segundos** en responder.
- La página se actualiza a **"Solved"**.

---

## 📸 Evidencia

![solved](https://github.com/user-attachments/assets/85c0440a-ebeb-4e30-bf9a-804223d22a8c)

✅ **Laboratorio completado.**

---

## 🧠 Lecciones aprendidas

1. **SQLi ciega (blind)**: cuando no ves el resultado ni errores, **el tiempo
   es tu canal de información**.
2. **Cada DBMS tiene su función de pausa**:
   - MySQL → `SLEEP()`
   - PostgreSQL → `pg_sleep()`
   - MSSQL → `WAITFOR DELAY`
   - Oracle → `dbms_pipe.receive_message()`
3. **La cookie `TrackingId`** es un punto de inyección clásico en labs de
   PortSwigger. Siempre revisa cookies, headers y parámetros.
4. **Burp Repeater es ideal** para medir tiempos de respuesta con precisión.
5. **Base para explotación avanzada**: con `SLEEP()` condicional se pueden
   extraer datos letra por letra (técnica que se usa en labs más avanzados).

---

## 🔬 Cómo se escala esto (para futuros labs)

Una vez confirmado que hay un retraso, se puede **inferir información**
usando `IF`:

```sql
'||SELECT IF(SUBSTRING(version(),1,1)='8', SLEEP(5), 0)--
```

Si el servidor tarda 5 segundos → la condición es verdadera.
Si responde rápido → la condición es falsa.

Repitiendo esto se puede extraer la versión, nombres de tablas, columnas y
credenciales **sin ver nunca el resultado** en pantalla. Es lento, pero
funciona.

---

## 🛠️ Herramientas utilizadas

- Burp Suite (Proxy, Repeater)
- Navegador web

---

## 📚 Referencias

- [PortSwigger - Blind SQL Injection](https://portswigger.net/web-security/sql-injection/blind)
- [PortSwigger - Time delays](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays)
- [MySQL - SLEEP()](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_sleep)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-09-23
