# 🧪 Reflected XSS in canonical link tag

![Plataforma](https://img.shields.io/badge/Plataforma-PortSwigger-purple)
![Dificultad](https://img.shields.io/badge/Dificultad-Practitioner-yellow)
![Categoría](https://img.shields.io/badge/Categor%C3%ADa-Reflected%20XSS%20%7C%20Accesskey-red)

## 📌 Resumen

Laboratorio de **PortSwigger Web Security Academy** que explota una
vulnerabilidad **Reflected XSS** en la etiqueta `<link rel="canonical">`
del `<head>`. Como los corchetes angulares están escapados, no se pueden
inyectar etiquetas nuevas. La solución pasa por **romper el atributo
`href`** e inyectar `accesskey` + `onclick` para ejecutar código cuando
el usuario pulsa una combinación de teclas.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Análisis | Reflexión en `<link rel="canonical">` | Contexto: atributo HTML |
| Bypass | Romper `href` con `'` | Inyectar atributos |
| Ejecución | `accesskey='x' onclick='alert(1)` | ✅ Lab resuelto |

---

## 🎯 Objetivo

Inyectar un atributo que ejecute `alert()` en la página de inicio.

---

## 📖 Contexto: el atributo `accesskey`

El atributo **`accesskey`** permite asignar una **combinación de teclado**
a un elemento HTML. Cuando el usuario pulsa esa combinación, el navegador
**enfoca y activa** el elemento, como si lo hubiera clicado.

Combinado con un **evento** (`onclick`, `onfocus`), permite ejecutar
JavaScript **sin que el elemento sea visible ni clicable**.

**Combinaciones de teclas por navegador:**

| Navegador | Combinación |
|-----------|-------------|
| Chrome (Windows/Linux) | `Alt + X` |
| Chrome (macOS) | `Ctrl + Alt + X` |
| Firefox | `Alt + Shift + X` |

> ⚠️ **El lab solo funciona en Chrome**. Otros navegadores pueden no
> disparar el evento `onclick` en elementos `<link>`.

---

## 🔍 Análisis

### 1. Identificar la reflexión

El input del usuario se refleja en la etiqueta `<link rel="canonical">`:

```html
<link rel="canonical" href="https://LAB-ID.web-security-academy.net/?TU-INPUT">
```

**Contexto:** dentro de un **atributo HTML** (`href`), no en el cuerpo.

### 2. Comprobar el escape

Los corchetes angulares (`<`, `>`) están **escapados**, así que no se
puede cerrar la etiqueta `<link>` ni inyectar `<script>`.

**Pero las comillas simples (`'`)** no están escapadas. Eso permite
**romper el atributo `href`** e inyectar atributos nuevos.

### 3. Confirmar con Burp Repeater

En Burp, se prueba a inyectar:

```
'accesskey='x'onclick='alert(1)
```

La respuesta refleja:

```html
<link rel="canonical" href="https://LAB-ID.../?&apos;accesskey=&apos;x&apos;onclick=&apos;alert(1)">
```

✅ Los atributos `accesskey` y `onclick` se inyectan correctamente.

---

## 💥 Explotación

### 4. Construir el payload

```
'accesskey='x'onclick='alert(1)
```

**Desglose:**

| Parte | Función |
|-------|---------|
| `'` | **Cierra el atributo `href`** (comilla simple). |
| `accesskey='x'` | Asigna la tecla `X` como atajo. |
| `onclick='alert(1)` | Ejecuta `alert(1)` al activar el elemento. |

### 5. Ejecutar el ataque

URL con el payload codificado:

```
https://YOUR-LAB-ID.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)
```

### 6. Activar el evento

**En Chrome**, pulsar:

- **Windows/Linux:** `Alt + X`
- **macOS:** `Ctrl + Alt + X`

Se ejecuta `alert(1)` y el lab se marca como **Solved**.

---

## 📸 Evidencia
<img width="1148" height="198" alt="image" src="https://github.com/user-attachments/assets/9d3b3749-8493-4b84-827b-9670bbc06f2e" />


---

## 🧠 Lecciones aprendidas

1. **El XSS no solo está en el `<body>`.** El `<head>` (con `<link>`,
   `<meta>`, `<title>`) también puede ser vulnerable.
2. **Los atributos HTML son un contexto de inyección.** Si puedes romper
   el atributo (con `'` o `"`), puedes inyectar nuevos atributos.
3. **`accesskey` es un vector poco común.** Permite activar elementos
   invisibles con combinaciones de teclado.
4. **El navegador importa.** Este lab solo funciona en Chrome porque el
   comportamiento de `accesskey` varía entre navegadores.
5. **El escape de `<>` no es suficiente.** Hay que escapar también `"`,
   `'`, `&`, y otros caracteres según el contexto.
6. **El ataque requiere interacción del usuario** (la combinación de
   teclas). En un ataque real, se podría engañar a la víctima para que
   pulse la combinación.

---

## 🛡️ ¿Cómo se previene?

1. **Escapar TODOS los caracteres peligrosos** en el contexto correcto:
   - En atributos HTML: escapar `"` y `'`.
   - En HTML: escapar `<`, `>`, `&`.
   - En JS: escapar `\`, `'`, `"`, etc.
2. **Usar funciones de escape dedicadas** según el contexto (HTML,
   atributo, JS, URL, CSS).
3. **No reflejar el input directamente en `<link>` ni en otros elementos
   del `<head>`**.
4. **Content Security Policy (CSP)** que bloquee scripts inline.
5. **Sanitizar el input** con librerías como DOMPurify.

---

## 🛠️ Herramientas utilizadas

- Burp Suite (Proxy, Repeater)
- Navegador Chrome
- DevTools (para inspeccionar el `<head>`)

---

## 📚 Referencias

- [PortSwigger - Reflected XSS in canonical link tag](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-canonical-link-tag)
- [MDN - accesskey](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/accesskey)
- [PortSwigger - XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-10-11
