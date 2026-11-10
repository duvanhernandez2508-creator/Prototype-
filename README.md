# AgroSahagún — Prototipo de bajo nivel

Aplicación demostrativa para pequeños productores de Sahagún y la Sabana de Córdoba. Permite iniciar sesión por rol, gestionar fincas y cultivos, registrar observaciones, visualizar alertas y seguir una estrategia offline-first.

## Arquitectura propuesta

`Navegador/PWA → almacenamiento local y cola pendiente → Firebase (opcional) → panel web`.

- **Interfaz:** HTML, CSS y JavaScript sin dependencias; puede alojarse gratuitamente en GitHub Pages.
- **Operación rural:** `localStorage` conserva los datos del prototipo aun sin internet. El service worker deja disponible la interfaz después de la primera visita. La cola `pending` representa los cambios que deben sincronizarse.
- **Usuarios:** la aplicación permite crear una cuenta local con usuario, contraseña y rol, e iniciar sesión en el mismo dispositivo. La contraseña se guarda como un resumen criptográfico local, únicamente para probar el flujo. En producción, Firebase Authentication controla la identidad y Firestore Security Rules controla los permisos.
- **Datos:** los registros precargados llevan el rótulo `DEMO / SIMULADO`; no son mediciones reales de Sahagún ni de sensores.

## Sensores y lecturas reales

El prototipo no está conectado a hardware. Una lectura real debe diferenciarse de una simulada con estos campos: `farmId`, `sensorId`, `metric`, `value`, `unit`, `capturedAt`, `source: sensor`, `signalQuality` y `calibrationDate`.

Flujo recomendado: sensor de humedad calibrado → ESP32 o gateway → red disponible (Wi‑Fi/LoRa/celular) → API con autenticación → Firestore. Si no hay red, el gateway almacena temporalmente y reintenta. Antes de recomendar riego, el productor o asesor confirma la lectura en el terreno.

## Ejecutar localmente

Abra la carpeta con un servidor estático. Por ejemplo, con VS Code use “Live Server”; con Node use `npx serve .`. No abra directamente el HTML si desea probar el modo instalable/offline, pues el navegador requiere `localhost` o HTTPS para el service worker.

## Publicar en GitHub Pages

1. Cree un repositorio y cargue el contenido de esta carpeta en la rama `main`.
2. En GitHub, abra **Settings → Pages**.
3. Seleccione **Deploy from a branch**, rama `main` y carpeta `/(root)`.
4. Guarde y abra la URL publicada. La aplicación usa rutas relativas, por lo cual funciona también dentro de un repositorio de proyecto.

## Integración opcional con Firebase

1. Cree un proyecto en [Firebase Console](https://console.firebase.google.com/) y registre una aplicación web.
2. Active **Authentication** (correo/contraseña o cuentas institucionales) y **Cloud Firestore**.
3. Cree `firebase-config.js` a partir de `firebase-config.example.js`; no publique claves privadas ni archivos de cuenta de servicio.
4. Añada el SDK web de Firebase y un adaptador que reemplace `persist()` por escrituras locales + Firestore al recuperar conexión.
5. Modele colecciones: `users/{uid}`, `farms/{farmId}`, `entries/{entryId}` y `recommendations/{id}`. Asocie cada finca con `ownerId`.
6. Aplique reglas: productor lee/escribe solo sus fincas; asesor consulta fincas asignadas y crea recomendaciones; administrador gestiona usuarios. Pruebe reglas en el emulador antes de producción.

El archivo `firestore.rules.example` es una base orientativa para esas reglas. Ajústelo al modelo final (en especial, la lectura de entradas para asesores asignados) y pruébelo con cuentas reales antes de habilitarlo.

La configuración pública de Firebase identifica el proyecto, pero las reglas de Firestore son el control que protege los datos. Las credenciales administrativas deben quedarse fuera del navegador.

## Datos de prueba

Al abrir la demo encontrará tres fincas ficticias y tres lecturas con fecha de noviembre de 2026. Todos están señalados como `DEMO / SIMULADO`. Cree una finca nueva y un registro manual para evaluar la cola sin conexión.

## Plan de experimento y KPI — Etapa 4

**Objetivo:** evaluar si un prototipo de bajo nivel permite registrar y consultar información relevante para decisiones agrícolas aun con conectividad limitada.

| Prueba | Participantes/escenario | Indicador | Meta inicial |
|---|---|---|---|
| Registro de finca | 3 productores, cada uno registra una finca | Tasa de tareas completadas | ≥ 80% sin ayuda |
| Captura de observación | Productor registra humedad manual sin internet | Registros conservados | 100% al recargar la página |
| Consulta técnica | 1 asesor identifica fincas con alerta | Tiempo para detectar prioridad | ≤ 2 minutos |
| Comprensión de alertas | Productor explica la recomendación | Comprensión validada por entrevista | ≥ 80% |
| Sincronización futura | 10 registros en cola con conexión recuperada | Registros enviados sin duplicado | 100% |

**Evidencias:** capturas de pantalla, formulario breve de usabilidad, número de tareas completas, tiempo por tarea, observaciones del asesor y listado de registros antes/después de recuperar conectividad.

**Criterio ético:** solicitar consentimiento informado, usar fincas ficticias en pruebas académicas, no usar recomendaciones automáticas como sustituto de una visita técnica y permitir corrección o eliminación de los registros por su responsable.
