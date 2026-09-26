# 🧪 PortSwigger Labs Writeups

Mis soluciones de los laboratorios de [PortSwigger Web Security Academy](https://portswigger.net/web-security).

Práctica de vulnerabilidades web en entornos controlados: SQL Injection, XSS,
CSRF, SSRF, y más.

---

## 📚 Índice de laboratorios

| Laboratorio | Categoría | Dificultad | Fecha | Writeup |
|-------------|-----------|-----------|-------|---------|
| SQL injection Oracle | SQL Injection | Practitioner | 2026-09-19 | [ver](./sql-injection-oracle.md) |
| SQL injection listing contents Oracle | SQL Injection | Practitioner | 2026-09-23 | [ver](./sql-injection-listing-contents-oracle.md) |
| Blind SQL injection time delays | SQL Injection | Practitioner | 2026-09-23 | [ver](./blind-sql-injection-time-delays.md) |


---

## 🛠️ Herramientas utilizadas

![Burp Suite](https://img.shields.io/badge/-Burp%20Suite-purple)
![SQLMap](https://img.shields.io/badge/-SQLMap-red)
![curl](https://img.shields.io/badge/-curl-blue)

- **Proxy/Intercept:** Burp Suite (Repeater, Intruder)
- **SQL Injection:** SQLMap, payloads manuales
- **Automatización:** curl, scripts Python

---

## 📖 Estructura de cada writeup

Cada archivo `.md` sigue esta plantilla:

1. **Resumen** — categoría, dificultad, objetivo.
2. **Enunciado** — descripción del lab.
3. **Análisis** — cómo identificar la vulnerabilidad.
4. **Explotación** — payloads y pasos.
5. **Solución** — cómo se resuelve el lab.
6. **Lecciones aprendidas** — qué se practicó.

---

## ⚠️ Disclaimer

Todos los writeups son de laboratorios **intencionadamente vulnerables**
de PortSwigger Web Security Academy. No se ha atacado ningún sistema real.

---

**Autor:** [Murmur](https://github.com/murmur403)
