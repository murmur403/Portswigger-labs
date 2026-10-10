# 🧪 Reflected XSS into HTML context with all tags blocked except custom ones

![Plataforma](https://img.shields.io/badge/Plataforma-PortSwigger-purple)
![Dificultad](https://img.shields.io/badge/Dificultad-Practitioner-yellow)
![Categoría](https://img.shields.io/badge/Categor%C3%ADa-Reflected%20XSS%20%7C%20WAF%20Bypass-red)

## 📌 Resumen

Laboratorio de **PortSwigger Web Security Academy** que explota una
vulnerabilidad **Reflected XSS** en el buscador. El WAF bloquea **todas las
etiquetas HTML estándar** (`<script>`, `<img>`, `<body>`...), pero permite
**etiquetas personalizadas**. Aprovechando que los navegadores aceptan
cualquier etiqueta desconocida y que los atributos de evento funcionan en
ellas, se inyecta `<xss>` con `onfocus` y se dispara automáticamente
mediante un ancla (`#x`) y `tabindex`.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Análisis | WAF bloquea etiquetas estándar | Buscar alternativa |
| Bypass | Etiqueta personalizada `<xss>` | Permitida por el WAF |
| Ejecución | `onfocus` + `tabindex` + `#x` | Foco automático → `alert()` |

---

## 🎯 Objetivo

Inyectar una etiqueta personalizada y ejecutar `alert(document.cookie)`
automáticamente, sin interacción del usuario.

---

## 📖 Contexto: ¿Por qué funcionan las etiquetas personalizadas?

Los navegadores están diseñados para ser **tolerantes** con el HTML. Cuando
encuentran una etiqueta que no conocen (como `<xss>` o `<foo>`), **no la
ignoran**: la tratan como un **elemento HTML genérico**. Y como cualquier
elemento HTML, **acepta atributos**, incluidos los de evento (`onfocus`,
`onclick`, etc.) .

**Esto significa que bloquear `<script>`, `<img>`, `<svg>` y demás
etiquetas estándar no protege contra XSS**: siempre queda la puerta abierta
para etiquetas personalizadas.

---

## 🔍 Análisis

### 1. Confirmar la reflexión

El buscador refleja el input:

```
/?search=test
```

Al probar `<script>alert(1)</script>` o `<img src=1 onerror=alert(1)>`,
el WAF devuelve **400 Bad Request**.

### 2. Probar etiquetas personalizadas

Al probar `<xss>test</xss>`, el WAF devuelve **200 OK**. ✅

**Conclusión:** el WAF bloquea etiquetas conocidas, pero **permite cualquier
etiqueta personalizada**.

### 3. El reto: disparar el evento sin interacción

Con `<xss onfocus=alert(1) tabindex=1>`, el evento `onfocus` se dispara
cuando el elemento **recibe el foco**. Pero el usuario tendría que hacer
clic o tabular hasta él.

**Solución:** usar un **ancla en la URL** (`#x`) con un `id` en el elemento.
Al cargar la página, el navegador **enfoca automáticamente** el elemento
con ese `id`.

---

## 💥 Explotación

### 4. Construir el payload

```
<xss id=x onfocus=alert(document.cookie) tabindex=1>#x
```

**Desglose:**

| Parte | Función |
|-------|---------|
| `<xss>` | Etiqueta personalizada (no bloqueada por el WAF). |
| `id=x` | Identificador para que el ancla `#x` lo encuentre. |
| `onfocus=alert(document.cookie)` | Se ejecuta al recibir el foco. |
| `tabindex=1` | **Hace el elemento enfocable** (sin esto, `onfocus` no se dispara). |
| `#x` | Al final de la URL → el navegador enfoca el elemento `x` automáticamente. |

### 5. Entregar el exploit

En el **Exploit Server** de PortSwigger, en el campo "Body":

```html
<script>
location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
</script>
```

1. Clic en **"Store"**.
2. Clic en **"Deliver exploit to victim"**.

### 6. ¿Por qué funciona sin interacción?

1. La víctima visita el Exploit Server.
2. El `<script>` redirige a la URL vulnerable con el payload.
3. El navegador carga la página y encuentra `<xss id=x ... tabindex=1>`.
4. El ancla `#x` hace que el navegador **enfoque automáticamente** el elemento.
5. El `onfocus` se dispara → `alert(document.cookie)`.
6. **Cero interacción del usuario.** ✅

---

## 📸 Evidencia

<img width="1170" height="189" alt="image" src="https://github.com/user-attachments/assets/52efbe12-85a9-4135-8dfd-5fd0ddaf8749" />


---

## 🧠 Lecciones aprendidas

1. **Bloquear etiquetas estándar NO protege contra XSS.** Los navegadores
   aceptan cualquier etiqueta personalizada.
2. **Los atributos de evento funcionan en etiquetas personalizadas.**
   `<xss onfocus=...>` es HTML válido para el navegador.
3. **`tabindex` es clave**: sin él, muchos elementos no pueden recibir foco
   y `onfocus` no se dispara.
4. **El ancla (`#x`)** permite enfocar un elemento automáticamente al cargar
   la página, **sin interacción del usuario**.
5. **Las blacklists son frágiles.** La única defensa sólida es **escapar el
   input** correctamente, no listar lo prohibido.
6. **El Exploit Server** permite entregar exploits a la víctima de forma
   controlada.

---

## 🛡️ ¿Cómo se previene?

1. **Escapar caracteres peligrosos** (`<`, `>`, `"`, `'`, `&`) en el input
   del usuario. **Esta es la única defensa real.**
2. **Sanitizar con librerías** como DOMPurify, que eliminan etiquetas y
   atributos peligrosos.
3. **Content Security Policy (CSP)** que bloquee scripts inline.
4. **Escape contextual**: según dónde se refleje el input (HTML, atributo,
   JS, URL).
5. **No usar blacklists.** Siempre serán incompletas.

---

## 🛠️ Herramientas utilizadas

- Burp Suite (Proxy, Repeater)
- Exploit Server (PortSwigger)
- Navegador web

---

## 📚 Referencias

- [PortSwigger - Reflected XSS with custom tags](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-all-tags-blocked-except-custom-ones)
- [PortSwigger - XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [MDN - tabindex](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/tabindex)
- [MDN - onfocus](https://developer.mozilla.org/en-US/docs/Web/API/Element/focus_event)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-10-09
