# Urban Scooter: pruebas web, móviles y de API del flujo de reservas

## Resumen

Proyecto final del bootcamp de QA Engineer de TripleTen. Diseñé y ejecuté 166 casos de prueba sobre Urban Scooter, una aplicación de alquiler de scooters con entrega a domicilio, que cubren la versión web, la API del back-end y las funciones clave de la app móvil Android. Validé 15 endpoints y documenté 24 defectos en Jira, 10 de severidad alta.

## Objetivo

Validar que el flujo de pedido funcione de punta a punta: que una persona pueda hacer un pedido con datos correctos, seguir su estado (en el almacén, en camino, entregado y en alquiler), cancelarlo, y que el back-end y la app del repartidor reflejen esos cambios.

Es la función central del negocio. Si el pedido no se crea, se crea con datos incompletos (por ejemplo, sin dirección) o queda en un estado incorrecto, la entrega no se puede cumplir y se pierden ingresos y confianza del cliente.

## Alcance

**Qué se probó**

- **Web:** formulario "Hacer pedido" (nombre, apellido, dirección, estación de metro, teléfono, fecha de entrega, periodo de alquiler, color y comentario) y pantalla "Estado del pedido" (cadena de estados, cancelación del pedido y pedido atrasado).
- **API:** crear repartidor (`POST /api/v1/courier`), eliminar repartidor (`DELETE /api/v1/courier/{id}`) y consultar un pedido por su número de seguimiento (GET /api/v1/orders/track)
- **Base de datos:** verificación de los registros en PostgreSQL (tablas `Orders` y `Couriers`).
- **App móvil (Android):** notificación 2 horas antes del final del pedido y ventana "Sin acceso a Internet". La app también se usó como repartidor para mover los pedidos por los estados.

**Plataformas y entornos**

- **Web:** Chrome 154.0.8037.58 y Opera 137.0.6036.39 (64 bits), Windows 11, resoluciones 1920x1080 y 1280x720 (los requisitos piden Chrome 85+ y Opera 71+).
- **Móvil:** Android 9.0 en el emulador de Android Studio.
- **API:** Postman.

**Fuera del alcance**

- Página de inicio (landing).
- Resto de funciones de la app móvil (solo se usó para avanzar los pedidos por los estados).
- Dispositivos físicos (solo se usó el emulador).

## Técnicas utilizadas

- Análisis de requisitos y mapa mental de la función "Formulario de pedido".
- Clases de equivalencia y valores límite para validar los campos (web y API).
- Listas de comprobación para "Estado del pedido" y la API.
- Registro de zonas grises (requisitos ambiguos o no definidos) en una columna de observaciones, con el estado "Por confirmar".
- Pruebas de API con Postman y verificación de datos con consultas SQL.
- Pruebas en dos navegadores y dos resoluciones.

## Resultados

| Métrica | Resultado |
|---|---|
| Casos de prueba diseñados | 166 |
| Casos ejecutados | 166 |
| Endpoints validados | 15 |
| Defectos encontrados y documentados | 24 |
| Defectos críticos o altos | 10 |

**Hallazgos principales**

- En Chrome no se puede crear ningún pedido; en Opera sí.
- Un pedido se puede crear con la dirección vacía (UR-5).
- Las validaciones de los campos fallan en varios casos: Apellido no valida el máximo (UR-1), Teléfono acepta de 11 a 13 caracteres en lugar de 10 a 12 (UR-6), Dirección rechaza 50 caracteres (UR-3) y Fecha de entrega permite elegir fechas pasadas (UR-8).
- La API crea repartidores con logins y contraseñas inválidos y responde 201.
- Cancelar un pedido no lo elimina del sistema (UR-18), y eliminar un repartidor no borra sus pedidos vinculados.
- Al aceptar un pedido en la app móvil se duplican los registros en `Orders`.
- La notificación 2 horas antes del final del pedido no llega (UR-24).

## Extracto de casos de prueba

| ID | Descripción | Pasos | Resultado esperado | Resultado obtenido | Estado |
|---|---|---|---|---|---|
| TC-01 | Nombre con un espacio | 1. Abrir el formulario "Para quién es el scooter".<br>2. Escribir "Ed Castro" en Nombre. | Se acepta. | Se acepta. | Aprobado |
| TC-02 | Apellido con 16 caracteres | 1. Abrir el formulario.<br>2. Escribir "Quinterosohenaos" en Apellido. | El campo se resalta en rojo con el mensaje "Introduce un apellido válido". | No se resalta, no hay mensaje y permite avanzar (UR-1). | Fallido |
| TC-03 | Dirección vacía | 1. Completar los demás campos con valores válidos.<br>2. Dejar Dirección vacío y pulsar "Siguiente". | No permite avanzar. | Avanza y el pedido se crea sin dirección (UR-5). | Fallido |
| TC-04 | Teléfono con 10 caracteres | 1. Abrir el formulario.<br>2. Escribir "1234567890" en Teléfono. | Se acepta (el rango válido es de 10 a 12 caracteres). | Se rechaza (UR-6). | Fallido |
| TC-05 | Pedido atrasado | 1. Consultar en "Estado del pedido" un pedido cuya fecha de entrega ya venció. | El estado cambia a "El repartidor o repartidora se demoró", con el mensaje de retraso, en rojo. | El estado y el mensaje cambian y se resaltan en rojo. | Aprobado |
| TC-06 | API: login de 1 carácter | 1. Enviar `POST /api/v1/courier` con un login de 1 carácter y una contraseña válida. | Error con el mensaje "Usuario o contraseña no válido". | Responde 201 y crea el repartidor ([UR-X]). | Fallido |

## Defecto destacado

**Título:** En Chrome no se puede crear un pedido (el botón "Sí" de la ventana "¿Deseas hacer un pedido?" no hace nada) — [UR-11]

**Pasos para reproducirlo:**

1. Abrir Urban Scooter en Chrome y hacer clic en "Pedir".
2. Completar "Para quién es el scooter" con valores válidos y hacer clic en "Siguiente".
3. En "Alquiler", completar los campos obligatorios y hacer clic en "Pedir".
4. En la ventana "¿Deseas hacer un pedido?", hacer clic en "Sí".

**Resultado esperado:** Se crea el pedido y aparece una ventana con el número de pedido y el botón "Comprueba el estado".

**Resultado obtenido:** No pasa nada: no se crea el pedido ni aparece el número. La consola de DevTools no muestra errores. Con los mismos datos, en Opera el pedido sí se crea.

**Severidad y prioridad:** Alta / Alta. Bloquea el flujo principal en uno de los navegadores que exigen los requisitos.

**Evidencia:** captura del informe en Jira y de la consola de Chrome (`evidencias/`).

## Evidencias

- Capturas de Jira: lista de los 24 defectos y el detalle de cada informe.
- Capturas de Postman: solicitudes y respuestas de la API (por ejemplo, el 201 con un login inválido).
- Capturas de Android Studio: la notificación que no llega y la ventana "Sin acceso a Internet".
- Capturas de la base de datos (`psql`): registros duplicados en `Orders`.
- Capturas de la aplicación web con los defectos.

## Archivos

```
.
├── README.md
├── mapa-mental/              # Mapa mental del formulario de pedido
├── casos-de-prueba/
│   ├── validacion-de-campos  # Tabla de clases de equivalencia y valores límite (Excel o CSV)
│   ├── checklist-estado-del-pedido
│   ├── checklist-api
│   └── casos-app-movil
├── informes-de-defectos/     # UR-1 a UR-24
└── evidencias/
```

## Herramientas

Postman, Jira, Chrome DevTools, Android Studio (emulador con Android 9.0), Google Sheets, PostgreSQL (`psql` desde Cygwin con `ssh`) y Figma (consulta de diseños).
