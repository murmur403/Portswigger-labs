# 🧪 Stored DOM XSS

![Plataforma](https://img.shields.io/badge/Plataforma-PortSwigger-purple)
![Dificultad](https://img.shields.io/badge/Dificultad-Practitioner-yellow)
![Categoría](https://img.shields.io/badge/Categor%C3%ADa-Stored%20DOM%20XSS-red)

## 📌 Resumen

Laboratorio de **PortSwigger Web Security Academy** que explota una
vulnerabilidad **Stored DOM XSS** en la funcionalidad de **comentarios** de un
blog. El sitio usa `replace()` para sanear los caracteres `<` y `>`, pero
**solo reemplaza la primera aparición** de cada uno. Aprovechando esto, se
puede inyectar un payload que ejecute `alert(1)`.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Identificación | Formulario de comentarios | Punto de entrada |
| Análisis | `replace()` solo reemplaza la 1ª vez | Bypass del filtro |
| Payload | `<><img src=1 onerror=alert(1)>` | ✅ Lab resuelto |

---

## 🎯 Objetivo

Explotar la vulnerabilidad **Stored DOM XSS** en los comentarios para ejecutar
`alert()`.

---

## 📖 Contexto: ¿Qué es Stored DOM XSS?

Es una variante de XSS donde:

- **Stored**: el payload se **almacena** en el servidor (en este caso, como
  comentario).
- **DOM**: el JavaScript del cliente procesa el comentario de forma insegura.

Cada vez que alguien visita la página con el comentario malicioso, el payload
se ejecuta. Es **más peligroso** que el XSS reflejado porque afecta a **todos
los visitantes**, no solo a quien hace clic en un enlace.

---

## 🔍 Análisis

### 1. Identificar el punto de entrada

El blog tiene una sección de **comentarios** donde los usuarios pueden publicar
mensajes. Los comentarios se **almacenan** y se muestran a todos los
visitantes.

### 2. Analizar el filtro

El código JavaScript del cliente usa `replace()` para eliminar los caracteres
peligrosos:

```javascript
comment = comment.replace('<', '&lt;').replace('>', '&gt;');
```

**Problema:** `replace()` con un string **solo reemplaza la PRIMERA
ocurrencia**. Si el comentario tiene **dos** pares de `<>`, el filtro solo
sanea el primero. El segundo pasa sin ser tocado.

### 3. Confirmar el fallo

Si envías:

```
<><img src=1 onerror=alert(1)>
```

- El primer `<>` se reemplaza por `&lt;&gt;`.
- El segundo `<img src=1 onerror=alert(1)>` **pasa sin filtrar**.
- El navegador lo interpreta como HTML y ejecuta el `onerror`.

---

## 💥 Explotación

### 4. Construir el payload

```
<><img src=1 onerror=alert(1)>
```

**Desglose:**

| Parte | Función |
|-------|---------|
| `<>` | "Consume" el filtro `replace()` (primera ocurrencia). |
| `<img src=1 onerror=alert(1)>` | Pasa sin filtrar → ejecuta `alert(1)` al fallar la imagen. |

### 5. Ejecutar el ataque

1. Ir a cualquier **post** del blog.
2. En el campo de **comentario**, introducir el payload:

```
<><img src=1 onerror=alert(1)>
```

3. Rellenar los demás campos (nombre, email, web) como quieras.
4. Clic en **"Post Comment"**.
5. Clic en **"Back to Blog"**.
6. El navegador intenta cargar la imagen rota, dispara `onerror`, y se ejecuta
   `alert(1)`.

✅ **Lab resuelto.**

---

## 📸 Evidencia

<img width="1161" height="187" alt="image" src="https://github.com/user-attachments/assets/2f4fa44d-18ea-41b5-8b36-ca251c0612e4" />


✅ **Laboratorio completado.**

---

## 🧠 Lecciones aprendidas

1. **`replace()` con string solo reemplaza la primera ocurrencia.** Para
   reemplazar todas, se debe usar una **expresión regular con flag `g`**:
   ```javascript
   comment.replace(/</g, '&lt;').replace(/>/g, '&gt;');
   ```
2. **Los filtros basados en `replace()` son frágiles.** Cualquier carácter
   repetido puede "gastar" el filtro.
3. **Stored XSS afecta a todos los visitantes**, no solo a quien hace clic en
   un enlace. Es más grave que el reflejado.
4. **`<img src=1 onerror=...>`** es un payload clásico que funciona cuando no
   se pueden usar etiquetas `<script>` (por ejemplo, si están bloqueadas).
5. **En bug bounty, Stored XSS suele ser High/Critical** porque afecta a todos
   los usuarios que visiten la página.

---

## 🛡️ ¿Cómo se previene?

1. **Usar `replaceAll()`** o regex con flag `g`:
   ```javascript
   comment = comment.replaceAll('<', '&lt;').replaceAll('>', '&gt;');
   ```
2. **Escapar el HTML correctamente** con funciones dedicadas (por ejemplo,
   `textContent` en lugar de `innerHTML`).
3. **Sanitizar en el servidor** además de en el cliente.
4. **Usar librerías de sanitización** como DOMPurify.
5. **Content Security Policy (CSP)** que bloquee scripts inline.

---

## 🛠️ Herramientas utilizadas

- Navegador web
- Burp Suite (opcional)

---

## 📚 Referencias

- [PortSwigger - Stored DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-stored)
- [PortSwigger - DOM-based XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based)
- [MDN - String.replace()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace)
- [MDN - String.replaceAll()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replaceAll)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-10-02
