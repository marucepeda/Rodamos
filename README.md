# Rodamos

**Todas las salidas platenses, en un solo lugar.**

Rodamos es una webapp para descubrir salidas grupales en bicicleta. Reúne en una agenda cronológica lo que hoy está disperso entre Instagram, WhatsApp y Facebook: cuándo sale cada grupo, desde dónde, hacia dónde, cuántos kilómetros y por qué tipo de camino. Cada salida lleva a su fuente original, donde se participa.

> **Estado: prototipo de MVP.** Es una interfaz funcional con datos de demostración ficticios. Todavía no tiene base de datos, panel de administración ni despliegue propio. Ver [Limitaciones](#limitaciones).

**Demo en vivo:** `https://<tu-usuario>.github.io/<nombre-del-repo>/` (reemplazá por tu URL de GitHub Pages cuando la actives).

## Qué hace hoy

- **Agenda cronológica** con la hora como eje visual y las salidas ordenadas por fecha y hora.
- **Selector de fechas**: «Todas», «Fin de semana», los próximos 14 días (con la cantidad de salidas de cada día) y un calendario para otra fecha.
- **Filtros** que se combinan con lógica AND: franja horaria, hora exacta, destino, distancia y tipo de camino. Muestran los filtros activos y se limpian con un clic.
- **Búsqueda compartible**: el estado de los filtros vive en la URL y se conserva al volver desde el detalle.
- **Detalle de cada salida** con enlace a la fuente original o al organizador. Si la salida está cancelada o ya pasó, la acción se desactiva.
- **Formulario para proponer una salida** con validación; la propuesta queda como «pendiente de revisión» y no se muestra públicamente.
- **Zona horaria por ciudad**: «hoy», «mañana» y el vencimiento se calculan con la hora local de la ciudad, no la del dispositivo.
- **Vencimiento automático**: una salida cuya hora ya pasó no aparece como próxima.
- **Modo claro y oscuro** según la preferencia del sistema.
- **Estados** de carga, error y sin resultados.

## Reglas de los filtros

| Filtro | Regla |
|---|---|
| Fecha | Día elegido, fin de semana (sábado y domingo vigentes, sin días ya pasados) o todas las próximas |
| Franja horaria | Mañana 06:00–11:59, Tarde 12:00–17:59, Noche 18:00–23:59 |
| Hora exacta | Salidas que parten desde esa hora (opcional) |
| Destino | Coincidencia de texto en destino y descripción; sin distinguir mayúsculas y tolerando espacios de más |
| Distancia | Hasta 20 km · 21–40 km · 41–70 km · más de 70 km. Se compara el valor numérico: 20 km entra en «hasta 20» y 40 km en «21–40» |
| Tipo de camino | Asfalto, rural o mixto. Las salidas sin tipo especificado siguen apareciendo en las búsquedas generales |

Todos los filtros se combinan con AND. Las franjas y los rangos se definen en un único lugar del código (`BANDS` y `RANGES`).

## Ejecutar el proyecto

No hay paso de compilación ni dependencias que instalar. Todo el proyecto es un único archivo.

```bash
# Opción 1: abrirlo directamente
open index.html          # macOS
xdg-open index.html      # Linux

# Opción 2: servirlo localmente
python3 -m http.server 8000
# luego abrir http://localhost:8000/index.html
```

La tipografía (Bricolage Grotesque) se carga desde Google Fonts. Sin conexión, la app funciona igual con las fuentes del sistema.

## Estructura

```
index.html   # HTML, CSS y JavaScript en un solo archivo
README.md
```

Dentro de `index.html`:

| Bloque | Contenido |
|---|---|
| Configuración | Ciudades, franjas horarias, rangos de distancia, tipos de camino |
| Lógica pura | `filterRides`, `isPublic`, `weekendDates`, `inRange`, `nowIn`, `addDays` (expuestas en `window.RodamosLogic`) |
| Datos de demostración | Salidas generadas de forma relativa a la fecha de hoy, incluidos casos límite |
| Vistas | Inicio (agenda), detalle y formulario, con ruteo por `location.hash` |

La lógica de filtrado está separada de la presentación para poder probarla y reutilizarla.

## Sistema visual

- Identidad **monocroma**: blanco, negro y grises. Sin color de acento.
- El **rojo** se reserva solo para errores (validación y fallos de carga). Las cancelaciones y los vencimientos se comunican con etiquetas monocromas, texto y tachado.
- Colores definidos como variables CSS con tema claro y oscuro; en oscuro se usan grises, no negro absoluto.
- Accesibilidad: foco visible, navegación por teclado, objetivos táctiles de al menos 40 px, estados que no dependen solo del color y respeto de `prefers-reduced-motion`.

## Datos de demostración

Todas las salidas incluidas son **ficticias** y están marcadas como «Demo». No tienen enlace de origen, por lo que su botón de fuente aparece desactivado. No representan actividades ni grupos reales.

## Limitaciones

- **Sin persistencia real.** Las propuestas del formulario se guardan solo en el navegador de quien las envía.
- **Sin administración.** No hay aprobación, edición, cancelación ni verificación de salidas, ni autenticación.
- **Sin pruebas automatizadas.** La lógica está preparada para probarse, pero todavía no hay suite de tests.
- **Un solo archivo.** No hay modularización ni framework; es adecuado para validar la experiencia, no para crecer.
- **Verificación visual parcial.** Falta una revisión sistemática en dispositivos reales y con lectores de pantalla.
- **Contrastes** calculados a mano, sin auditoría con herramienta.

## Próximos pasos

1. Base de datos relacional (PostgreSQL) con ciudades, organizadores y salidas, y políticas de acceso por fila.
2. Panel de administración protegido: flujo borrador → revisión → publicación, cancelación y verificación.
3. Pruebas automatizadas de filtrado, vencimiento y permisos.
4. Páginas individuales por salida y por ciudad, con metadatos para buscadores.
5. Incorporación de datos reales con permiso de los organizadores.
6. Despliegue, documentación de instalación y guía para sumar una ciudad nueva.

## Modelo de datos previsto

```
cities      (id, name, slug, country_code, timezone, is_active)
organizers  (id, city_id → cities, name, public_url, contact_url, description, is_active)
rides       (id, city_id → cities, organizer_id → organizers,
             start_date, start_time, meeting_point, meeting_point_address,
             meeting_point_maps_url, destination, distance_km,
             terrain_type [asfalto|rural|mixto|null], description,
             source_url, source_type,
             status [draft|pending_review|published|cancelled|completed|expired],
             created_at, updated_at, last_verified_at)
```

Solo las salidas publicadas y vigentes deben aparecer en el catálogo público. Las consultas públicas excluyen las salidas pasadas sin depender de una tarea programada.

## Privacidad

No se exige registro ni datos personales para consultar salidas, y no se recopila ubicación. Los enlaces apuntan a fuentes públicas o autorizadas por los organizadores.

## Licencia

Distribuido bajo licencia MIT. Ver el archivo [LICENSE](LICENSE).
