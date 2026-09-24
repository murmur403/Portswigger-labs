# Blind SQL Injection with Time Delays

**Plataforma:** PortSwigger Web Security Academy
**Dificultad:** Practitioner
**Estado:** ✅ Resuelto
**Objetivo:** Explotar una vulnerabilidad de inyección SQL ciega basada en tiempo para causar un retraso de 10 segundos en la respuesta del servidor.

## Contexto
El laboratorio tiene una vulnerabilidad de inyección SQL ciega (blind) en la cookie `TrackingId`. Los resultados de la consulta no se devuelven en la respuesta, y la aplicación no responde de forma diferente ante errores. Sin embargo, la consulta se ejecuta de forma síncrona, por lo que es posible provocar retrasos condicionales para inferir información.

## Pasos

### 1. Identificar el punto de inyección
- Intercepté la petición `GET /` con Burp Suite.
- Observé que la cookie `TrackingId` se incluye en una consulta SQL en el servidor.

### 2. Determinar el gestor de base de datos
- El laboratorio usa **MySQL** en esta instancia.
- La función de pausa en MySQL es `SLEEP(segundos)`.

### 3. Construir el payload
- Añadí el siguiente payload al valor de la cookie: '||SELECT SLEEP(10)
- La cookie final quedó así: TrackingId=2FIp3bG71f2NuclR'||SELECT SLEEP(10)

### 4. Ejecutar el ataque
- Envié la petición con **Forward** en Burp Proxy.
- El servidor tardó **10 segundos** en responder.
- La página se actualizó a **"Solved"**.

## Resultado
✅ Laboratorio resuelto. Se confirmó la vulnerabilidad de inyección SQL ciega basada en tiempo.

## Evidencia
<img width="1189" height="303" alt="image" src="https://github.com/user-attachments/assets/85c0440a-ebeb-4e30-bf9a-804223d22a8c" />
