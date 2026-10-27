# Portarretrato

Portarretrato digital para una tablet Android antigua (Galaxy Tab 2): calendario mensual a la izquierda, y a la derecha una presentación de fotos con reloj.

Toda la app es un único archivo estático, [index.html](index.html) (CSS y JS en línea). No requiere compilación, dependencias ni servidor.

## Características

- Calendario del mes con navegación y próximo evento.
- Reloj (12/24 h, con o sin segundos).
- Presentación de fotos aleatoria con fundido cruzado.
- Fotos guardadas en el dispositivo (IndexedDB), reducidas automáticamente y con orientación EXIF corregida.
- Modo noche: atenúa la pantalla en un horario configurable.
- Sincronización opcional con Google Calendar y Google Drive mediante Apps Script.
- Indicador de batería (si el navegador o Fully Kiosk lo permiten).
- Se recarga a las 4:00 a. m. para liberar memoria.

## Uso

1. Abre `index.html` en un navegador (el objetivo es Chrome o Fully Kiosk Browser en la tablet).
2. Toca la pantalla para entrar a pantalla completa y mostrar el panel inferior.
3. Desde el panel puedes agregar fotos, quitar la actual o borrarlas todas. El panel se cierra solo tras 20 s sin actividad.

## Configuración

Todos los ajustes están en el objeto `CONFIG` al inicio del script de [index.html](index.html):

| Opción | Descripción |
| --- | --- |
| `SEGUNDOS_POR_FOTO` | Segundos entre fotos. |
| `AJUSTE_FOTO` | `'contain'` (foto completa) o `'cover'` (llena el recuadro recortando). |
| `MOSTRAR_SEGUNDOS` | Mostrar segundos en el reloj. |
| `LUNES_PRIMERO` | La semana empieza el lunes. |
| `TAMANO_MAX` | Tamaño máximo (px) al guardar fotos. |
| `NOCHE_DESDE` / `NOCHE_HASTA` | Horas del modo noche; `-1` en ambas lo desactiva. |
| `FOTOS_LOCALES` | Rutas de fotos junto al archivo, ej. `['fotos/1.jpg']`. |
| `URL_SCRIPT` | URL `/exec` del Apps Script (Google Calendar/Drive). Vacío = desactivado. |
| `CLAVE` | Clave compartida con el Apps Script. |
| `MINUTOS_ACTUALIZAR` | Cada cuántos minutos se sincroniza con Google. |

## Google Calendar / Drive (opcional)

Si `URL_SCRIPT` está definido, la página consulta un Apps Script publicado como web app (`Codigo.gs`, que no está en este repositorio) mediante JSONP, enviando `CLAVE` como secreto compartido. La respuesta tiene la forma:

```json
{ "events": [{ "start": "...", "end": "...", "title": "...", "allDay": false }], "photos": ["idDrive1", "idDrive2"] }
```

Los datos se guardan en `localStorage` (`pr_datos`), así que la app sigue mostrando lo último sincronizado sin conexión.

## Notas de desarrollo

- Solo ES5 y sin librerías: `var`, funciones normales, sin `let`/`const`/flechas/plantillas/`Promise`.
- El CSS usa flexbox con y sin prefijo `-webkit-`, y unidades `vh`/`vw` para adaptarse a cualquier pantalla. El modo vertical se define en el bloque `@media (orientation: portrait)`.
- Textos de la interfaz e identificadores del código están en español.
