# 🛰️ Monitor de Emergencias · España / Andalucía

Adaptación didáctica de los conceptos de [koala73/worldmonitor](https://github.com/koala73/worldmonitor)
y de [IncendiosEspaña.es](https://xn--incendiosespaa-2nb.es/) para el **Ciclo de Emergencias y
Protección Civil** (módulo de Intervención Operativa), centrada en la emergencia por
**incendios forestales con Nivel 3 declarado en la Comunidad de Madrid** y la vigilancia de **Andalucía**.

**Un único fichero** (`index.html`), sin backend. En vivo: `https://jruecab750.github.io/monitor-emergencias/`

> ⚠️ Este monitor es un ejercicio formativo. **No sustituye a los sistemas oficiales de
> emergencia.** En una emergencia real, llama al **112**.

---

## Novedades v3

- **Panel 🗂 Capas propio**: activar/desactivar, **reordenar (▲▼)** y **transparencia por capa**
  (deslizador). El estado se recuerda en el navegador.
- **Agrupación de focos FIRMS**: a zoom lejano los focos se agrupan en círculos con recuento
  (color = peor confianza del grupo, popup con FRP total); al acercar el zoom se separan.
- **🔎 Buscador** (arriba a la izquierda del mapa): 📍 vuela a una población (Nominatim) y
  🔥 busca un **incendio declarado** por su nombre operativo (p. ej. «IF Córdoba», como los
  nombra INFOCA) en las noticias de las últimas 48 h, con enlace a la búsqueda en vivo en X,
  y centra el mapa en la zona para ver los focos.
- **🌿 Vegetación / ocupación del suelo**: WMS INSPIRE oficial (SIOSE/CLC, IGN).
- **🌬 Viento y 🌡 Temperatura sobre el mapa**: rejilla de flechas (dirección + velocidad,
  color por racha) y valores de temperatura (color por umbral), de Open-Meteo, recalculada
  al mover el mapa.
- **Pestaña 🌬 Windy**: visor Windy.com embebido con animación (viento, rachas, temperatura,
  humedad).

## Fuentes cargadas

| Fuente | Qué aporta | Vía |
|---|---|---|
| **NASA FIRMS** (como IncendiosEspaña.es) | Focos térmicos VIIRS de las últimas 24 h como **puntos clicables**: tamaño = FRP (potencia radiativa), color = confianza, opacidad = antigüedad. Refresco cada 5 min | API de área (clave gratuita en `CONFIG.firmsKey`) |
| **NASA EONET** (como worldmonitor) | Incendios declarados abiertos | API directa |
| **EFFIS / Copernicus** (WMS) | Riesgo de incendio hoy (FWI), focos raster, **perímetros de área quemada de la temporada** | Teselas WMS |
| **NASA GIBS** | Anomalías térmicas VIIRS del día anterior | Teselas WMTS |
| **IGN** | Mapas base, ortofoto PNOA, **límites municipales** (WMS INSPIRE), sismicidad (RSS) | Teselas + WMS + RSS |
| **OpenStreetMap / Overpass** | **Parques de bomberos** (con detección de bases forestales por nombre: INFOCA, CEDEFO, BRIF, retén…), **puntos de agua** (balsas, depósitos, puntos de aspiración; hidrantes a zoom alto), **helipuertos**, **núcleos de población**. Se cargan según la vista del mapa | API Overpass (2 espejos) |
| **Nominatim (OSM)** | Geocodificación de municipios afectados/declarados | API directa (1 pet./s, caché local) |
| **MeteoAlarm (AEMET)** | Avisos meteorológicos oficiales | Atom vía proxy |
| **Open-Meteo** | **Regla 30-30-30** en 10 puntos (Madrid, Guadarrama, 8 capitales andaluzas) | API directa |
| **Google News RSS** | 3 paneles: Madrid N3, Andalucía/INFOCA, Protección Civil (consultas editables) | RSS vía proxy |
| **Cuentas oficiales de X** | 112, INFOCA, UME, DGPCE, AEMET, delegaciones (búsqueda en vivo)… | Enlaces directos |

### Municipios afectados en el mapa (🚩)

X no ofrece API pública, así que los municipios que declara la Delegación del Gobierno
se incorporan por dos vías, ambas geocodificadas y pintadas como 🚩:

1. **Detección automática**: se buscan ~100 municipios de la C. de Madrid en los titulares
   del panel de noticias (chip rojo 📰).
2. **Transcripción del operador**: lo leído en X se anota en `CONFIG.municipiosDeclarados`
   (`'Municipio'` o `'Municipio, Provincia'`), como haría el operador de un CECOP (chip naranja 📢).

### Aviso de proximidad (como IncendiosEspaña.es)

El botón **«📍 Vigilar mi posición»** geolocaliza al usuario, calcula la distancia al foco
FIRMS más cercano en cada refresco y, si baja de `CONFIG.alertKm` (25 km), muestra alerta
roja y notificación del navegador.

## Sobre la clave FIRMS

`CONFIG.firmsKey` contiene una clave **gratuita y de solo lectura** de datos públicos de la NASA
([obtener aquí](https://firms.modaps.eosdis.nasa.gov/api/map_key/)). Al ser una web sin servidor,
la clave es visible públicamente: es un riesgo asumido y habitual en este tipo de mapas
(tiene límite propio de peticiones y puede regenerarse en cualquier momento).

## Descartado y por qué

- **De worldmonitor**: mercados, cripto, militar, ciber, puertos, 500+ feeds globales y su
  backend (Vercel/Convex/Tauri) — sin relación con emergencias o inviable sin servidor.
- **De IncendiosEspaña.es**: EUMETSAT Meteosat/SEVIRI cada 15 min y el raspado de los
  servicios autonómicos (INFOCA, JCyL, Bombers, 112 CV) — ambos exigen backend propio.

## Notas técnicas

- Proxies CORS públicos con cadena de respaldo (`allorigins` → `corsproxy.io` → `codetabs`)
  para los RSS; las APIs con CORS abierto van directas y con respaldo por proxy (`auto`).
- Pie de página: estado ● de cada fuente con hora de última actualización.
- Refresco: FIRMS y noticias cada 5 min; avisos/meteo/sismos/EONET y capas WMS cada 10 min;
  capas OSM al mover el mapa (con zoom mínimo por capa).
- El banner «NIVEL 3» es **declarativo**: lo fija quien opera el monitor, como en un CECOP.
- Si una capa WMS de EFFIS no pinta, verificar nombres de capa en el
  [visor EFFIS](https://forest-fire.emergency.copernicus.eu/apps/effis_current_situation/).

## Uso didáctico sugerido

1. **Sala de crisis**: proyectar el monitor y asignar roles (focos, avisos, meteo, comunicación).
2. **Análisis de situación**: cruzar FWI + avisos AEMET + regla 30-30-30 para justificar
   niveles de preemergencia.
3. **Planificación operativa**: con las capas OSM, localizar el parque de bomberos, punto de
   agua y helipuerto más cercanos a un foco FIRMS y proponer un plan de ataque.
4. **Verificación de fuentes**: contrastar un titular con la cuenta oficial de X y transcribir
   los municipios declarados a `CONFIG.municipiosDeclarados`.
