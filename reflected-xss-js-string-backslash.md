# 🧪 Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped

![Plataforma](https://img.shields.io/badge/Plataforma-PortSwigger-purple)
![Dificultad](https://img.shields.io/badge/Dificultad-Practitioner-yellow)
![Categoría](https://img.shields.io/badge/Categor%C3%ADa-Reflected%20XSS%20%7C%20JS%20String-red)

## 📌 Resumen

Laboratorio de **PortSwigger Web Security Academy** que explota una
vulnerabilidad **Reflected XSS** en el buscador. El input se refleja
**dentro de una cadena de JavaScript** entre comillas simples. El servidor
**HTML-encodea** los corchetes angulares (`<`, `>`) y las comillas dobles
(`"`), y **escapa** las comillas simples (`'`). Sin embargo, **la barra
invertida (`\`) NO está escapada**, lo que permite usar `\'` para "escapar
el escape" y romper la cadena.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Análisis | Reflexión en `<script>` | Contexto: JS string |
| Escape | `'` → `\'`, pero `\` NO se escapa | Vulnerabilidad |
| Bypass | `\'` rompe la cadena | Inyectar código |
| Payload | `\'-alert(1)//` | ✅ Lab resuelto |

---

## 🎯 Objetivo

Romper la cadena de JavaScript y ejecutar `alert()`.

---

## 📖 Contexto: escape de comillas y barras invertidas

Cuando el input se refleja dentro de una cadena JavaScript, el servidor
suele escapar caracteres peligrosos:

| Carácter | Escape |
|----------|--------|
| `'` | `\'` |
| `"` | `\"` |
| `\` | `\\` |

**El problema:** si el servidor **escapa la comilla pero NO la barra
invertida**, el atacante puede usar `\'` para "escapar el escape". El
`\` del atacante **escapa la comilla que el servidor añadiría**, y la
comilla del atacante **rompe la cadena**.

---

## 🔍 Análisis

### 1. Confirmar la reflexión

Al buscar `test`, el servidor devuelve:

```html
<script>
var searchTerms = 'test';
...
</script>
```

**Contexto:** dentro de una **cadena JavaScript** entre comillas simples.

### 2. Probar el escape de comillas

Buscar `test'payload`:

```javascript
var searchTerms = 'test\'payload';
```

❌ La comilla simple (`'`) está escapada. No rompe la cadena.

### 3. Probar el escape de la barra invertida

Buscar `test\payload`:

```javascript
var searchTerms = 'test\payload';
```

✅ **La barra invertida (`\`) NO está escapada.** Esto es la clave.

### 4. Probar los corchetes angulares

Buscar `<script>`:

```html
0 search results for '&lt;script&gt;'
```

❌ Los corchetes angulares están **HTML-encoded** (`&lt;`, `&gt;`). No se
puede romper el bloque `<script>`.

**Conclusión:** hay que romper la **cadena**, no el bloque.

---

## 💥 Explotación

### 5. Construir el payload

Aprovechando que `\` no se escapa:

```
\'-alert(1)//
```

**Desglose:**

| Parte | Función |
|-------|---------|
| `\'` | El `\` del atacante **escapa la comilla que el servidor añadiría**. |
| `'` | **Rompe la cadena** de JavaScript. |
| `-alert(1)` | Ejecuta `alert(1)`. |
| `//` | **Comenta el resto** de la línea (la comilla final y el `;`). |

### 6. Cómo queda el código

Con el payload, el HTML resultante es:

```javascript
var searchTerms = '\'-alert(1)//';
```

El navegador lo interpreta así:

1. `'` → inicia la cadena.
2. `\'` → comilla escapada (dentro de la cadena).
3. `'` → **cierra la cadena**.
4. `-alert(1)` → **código JavaScript ejecutado**.
5. `//` → comenta el resto.

**Resultado:** se ejecuta `alert(1)`.

### 7. Ejecutar el ataque

Buscar en el buscador:

```
\'-alert(1)//
```

O en la URL (URL-encoded):

```
https://YOUR-LAB-ID.web-security-academy.net/?search=%5C%27-alert(1)%2F%2F
```

Se ejecuta `alert(1)` y el lab se marca como **Solved**.

---

## 📸 Evidencia

<img width="1159" height="463" alt="image" src="https://github.com/user-attachments/assets/750c300d-377a-462a-bfe4-2be3b4ba6757" />


---

## 🧠 Lecciones aprendidas

1. **El escape de comillas no es suficiente** si no se escapa **también la
   barra invertida**. El atacante puede usar `\'` para "escapar el escape".
2. **El orden de los escapes importa.** Un servidor bien configurado
   escapa primero `\` y luego `'`. Un servidor mal configurado escapa
   solo `'`, dejando la puerta abierta.
3. **HTML-encode de `<>` no protege** si el input se refleja dentro de un
   `<script>`. Hay que escapar **también** `</script>` y otros caracteres.
4. **XSS en contexto JavaScript** requiere entender **cómo funciona el
   parseo de strings** en JS.
5. **El escape contextual** (según dónde se refleje el input) es la única
   defensa real.

---

## 🛡️ ¿Cómo se previene?

1. **Escapar `\` ANTES que `'`**:
   ```javascript
   input = input.replace(/\\/g, '\\\\').replace(/'/g, "\\'");
   ```
2. **Escapar `</script>`** también, para prevenir el cierre del bloque.
3. **Usar `JSON.stringify()`** para pasar datos a JavaScript de forma segura.
4. **Content Security Policy (CSP)** que bloquee scripts inline.
5. **No reflejar input del usuario dentro de bloques `<script>`**.
6. **Usar `data-*` attributes** y leerlos desde JS con `dataset`, en lugar
   de generar código JS dinámicamente.

---

## 🛠️ Herramientas utilizadas

- Burp Suite (Proxy, Repeater)
- Navegador web

---

## 📚 Referencias

- [PortSwigger - Reflected XSS into JavaScript string (angle brackets HTML-encoded, quotes escaped)](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-angle-brackets-double-quotes-html-encoded-single-quotes-escaped)
- [PortSwigger - XSS contexts](https://portswigger.net/web-security/cross-site-scripting/contexts)
- [OWASP - XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-10-13
