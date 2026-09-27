# 🧪 DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

![Plataforma](https://img.shields.io/badge/Plataforma-PortSwigger-purple)
![Dificultad](https://img.shields.io/badge/Dificultad-Practitioner-yellow)
![Categoría](https://img.shields.io/badge/Categor%C3%ADa-DOM%20XSS%20%7C%20CSTI-red)

## 📌 Resumen

Laboratorio de **PortSwigger Web Security Academy** que explota una vulnerabilidad
**DOM-based XSS** a través de una **expresión de AngularJS** en la funcionalidad
de búsqueda. A diferencia de los XSS clásicos, aquí **no se inyectan etiquetas
HTML** (los corchetes angulares y comillas dobles están codificados), sino que
se aprovecha el **motor de plantillas de AngularJS** para ejecutar código
JavaScript mediante expresiones `{{ }}`.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Framework | `ng-app` en `<body>` | AngularJS detectado |
| Vector | Expresiones `{{ }}` | CSTI |
| Payload | `{{$on.constructor('alert(1)')()}}` | ✅ Lab resuelto |

---

## 🎯 Objetivo

Ejecutar una **expresión de AngularJS** que llame a la función `alert()`.

---

## 📖 Contexto: ¿Qué es Client-Side Template Injection (CSTI)?

Cuando un framework del lado del cliente (AngularJS, Vue, React) **evalúa input
del usuario como si fuera parte de su plantilla**, se produce una vulnerabilidad
de **CSTI**. En el caso de AngularJS, cualquier contenido dentro de `{{ }}` se
evalúa como una **expresión de JavaScript** .

Esto es especialmente peligroso porque **no necesitas inyectar etiquetas HTML**:
basta con que el input caiga dentro de un contexto donde AngularJS esté
"escaneando" el DOM (gracias a la directiva `ng-app`).

---

## 🔍 Análisis

### 1. Confirmar AngularJS

En el HTML del `<body>` se encuentra la directiva:

```html
<body ng-app>
```

La directiva **`ng-app`** le dice a AngularJS que trate todo el documento como
una plantilla. A partir de ahí, cualquier `{{ expresión }}` será evaluada.

### 2. Confirmar codificación

El enunciado indica que `<>` y `"` están **HTML-encoded**. Esto descarta los
payloads clásicos como `<script>` o `<img onerror>`.

**Solución:** usar la sintaxis de AngularJS (`{{ }}`) para inyectar una
expresión.

---

## 💥 Explotación

### 3. El payload

```
{{$on.constructor('alert(1)')()}}
```

### 4. Desglose del payload

| Parte | Función |
|-------|---------|
| `{{ ... }}` | Le dice a AngularJS: "evalúa lo que hay aquí dentro". |
| `$on` | Objeto interno del *scope* de AngularJS, siempre disponible. |
| `.constructor` | Accede al constructor de la función (`Function`). |
| `('alert(1)')` | Crea una nueva función que ejecuta `alert(1)`. |
| `()` | **Ejecuta** la función recién creada. |

**En palabras simples:** AngularJS evalúa `$on.constructor`, obtiene el
constructor de `Function`, crea una nueva función con el código `alert(1)` y
la ejecuta.

### 5. Ejecutar el ataque

1. Escribir el payload en el buscador:

```
{{$on.constructor('alert(1)')()}}
```

2. Pulsar Enter.
3. Se ejecuta `alert(1)` y el lab se marca como **Solved**.

---

## 📸 Evidencia

<img width="1161" height="187" alt="image" src="https://github.com/user-attachments/assets/f100a374-024f-45a8-bd98-de9e688cb4bf" />



✅ **Laboratorio completado.**

---

## 🧠 Lecciones aprendidas

1. **CSTI es un tipo de XSS diferente**: no se inyectan etiquetas HTML, sino
   expresiones que el framework evalúa.
2. **AngularJS (`ng-app`) es un vector clásico**: cuando está presente,
   cualquier `{{ }}` puede ser peligroso.
3. **La codificación de `<>` y `"` no protege** contra CSTI. El framework
   interpreta el contenido **antes** de que el navegador lo renderice.
4. **`$on.constructor`** es el punto de partida más fiable para explotar
   AngularJS, porque `$on` siempre está disponible en el scope.
5. **Frameworks modernos** (React, Vue, Svelte) **no evalúan expresiones del
   usuario** de la misma forma, por lo que son más seguros por diseño.

---

## 🛡️ ¿Cómo se previene?

1. **No usar `ng-app` en todo el documento**: restringir AngularJS a zonas
   específicas y controladas.
2. **Escapar `{{ }}`** en el input del usuario antes de insertarlo en el DOM.
3. **Usar `ng-bind`** en lugar de interpolación directa.
4. **Migrar a frameworks modernos** (React, Vue) que no evalúan expresiones
   del usuario.
5. **Implementar CSP** (Content Security Policy) que bloquee la evaluación de
   scripts inline.

---

## 🛠️ Herramientas utilizadas

- Navegador web (DevTools → Elements)
- Burp Suite (opcional)

---

## 📚 Referencias

- [PortSwigger - DOM XSS in AngularJS expression](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-angularjs-expression)
- [PortSwigger - DOM-based XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based)
- [PortSwigger - AngularJS CSP bypass](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#angularjs)
- [OWASP - Client-Side Template Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/11-Client-side_Testing/13-Testing_for_Client-side_Template_Injection)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-09-26
