# 🛰️ Monitor de Emergencias · España / Andalucía

Adaptación didáctica del concepto de [koala73/worldmonitor](https://github.com/koala73/worldmonitor)
para el **Ciclo de Emergencias y Protección Civil** (módulo de Intervención Operativa),
centrada en la emergencia por **incendios forestales con Nivel 3 declarado en la
Comunidad de Madrid** y en la vigilancia permanente de **Andalucía**.

**Un único fichero** (`index.html`), sin backend ni claves API. Se abre en el navegador
o se sirve por GitHub Pages: `https://<usuario>.github.io/<repo>/monitor-emergencias/`

---

## Por qué no es un fork directo

WorldMonitor real necesita infraestructura propia que no aporta nada al aula:
proxy RSS en Vercel con lista blanca de dominios, backend Convex, claves API,
facturación/planes y app de escritorio Tauri. Este monitor conserva su **arquitectura
conceptual** (mapa + capas conmutables + paneles de feeds + panel de estado por fuente
+ auto-refresco) implementada de forma autocontenida.

## Fuentes DESCARTADAS de worldmonitor

| Fuente / módulo | Motivo |
|---|---|
| Mercados, cripto, 29 bolsas | Sin relación con emergencias |
| Bases militares, APT/ciber, puertos AIS, datacenters IA | Fuera de ámbito |
| 500+ feeds internacionales (Reuters, TASS, tech…) | Ruido para una emergencia nacional |
| Índice de inestabilidad de países | Sustituido por la regla 30-30-30 (ver abajo) |
| Backend propio (Vercel/Convex/Tauri/facturación) | Inviable y innecesario en el aula |

## Fuentes CARGADAS

| Fuente | Qué aporta | Vía |
|---|---|---|
| **NASA EONET** (la misma que usa worldmonitor) | Incendios declarados abiertos (bbox península) | API directa (CORS abierto) |
| **EFFIS / Copernicus** (WMS) | Riesgo de incendio hoy (FWI), focos MODIS/VIIRS, área quemada de la temporada | Teselas WMS |
| **NASA GIBS** | Anomalías térmicas VIIRS (equivalente FIRMS) | Teselas WMTS sin clave |
| **IGN** | Mapas base oficiales, ortofoto PNOA, sismicidad (RSS) | Teselas + RSS |
| **MeteoAlarm (AEMET)** | Avisos meteorológicos oficiales, con las zonas Madrid/Andalucía destacadas | Atom vía proxy |
| **Open-Meteo** | T, HR y rachas en 10 puntos (Madrid, Guadarrama y las 8 capitales andaluzas) → **regla 30-30-30** | API directa (CORS abierto) |
| **Google News RSS** | 3 paneles: *Madrid N3*, *Andalucía/INFOCA*, *Protección Civil* — consultas editables en `CONFIG.queries` | RSS vía proxy |
| **Cuentas oficiales de X** | 112 Madrid, 112 Andalucía, INFOCA, UME, DGPCE, AEMET, delegaciones (búsqueda en vivo)… | Enlaces directos (X no ofrece RSS público; worldmonitor tampoco lo integra) |
| **Visores oficiales** | INFOCA, EFFIS, avisos AEMET, Copernicus EMS, IGN, DGT eTraffic | Enlaces |

### Regla 30-30-30

Sustituye al «índice de inestabilidad» de worldmonitor por un indicador operativo real
de comportamiento extremo del fuego: **T > 30 °C, HR < 30 %, viento > 30 km/h**.
Cada punto vigilado se clasifica por condiciones superadas:
🟩 NORMAL (0/3) · 🟨 AVISO (1/3) · 🟧 GRAVE (2/3) · 🟥 CRÍTICO (3/3).

## Notas técnicas

- **Proxy CORS**: los RSS (Google News, MeteoAlarm, IGN) no permiten peticiones
  cross-origin, así que pasan por una cadena de proxies públicos con respaldo
  (`allorigins` → `corsproxy.io` → `codetabs`). Si uno cae, se prueba el siguiente.
  El pie de página muestra el estado ● de cada fuente y su última actualización.
- **Refresco**: noticias cada 5 min, datos (avisos, meteo, sismos, EONET) cada 10 min.
- **Configuración editable** en el objeto `CONFIG` al inicio del script: bbox, puntos
  de vigilancia, consultas de noticias y texto del banner de situación.
- El banner «NIVEL 3» es **declarativo** (lo fija quien opera el monitor, como en un
  CECOP real): edítalo en el HTML cuando cambie la situación operativa.
- Si alguna capa WMS de EFFIS no pinta, comprobar los nombres de capa vigentes en el
  [visor EFFIS](https://forest-fire.emergency.copernicus.eu/apps/effis_current_situation/)
  (los servicios de Copernicus renombran capas ocasionalmente).

## Uso didáctico sugerido

1. **Sala de crisis**: proyectar el monitor y asignar roles (seguimiento de avisos,
   focos, condiciones meteo, comunicación oficial).
2. **Análisis de situación**: cruzar la capa FWI con los avisos AEMET y la regla
   30-30-30 para justificar niveles de preemergencia.
3. **Verificación de fuentes**: contrastar una noticia de los paneles con la cuenta
   oficial de X correspondiente (competencia de comunicación en emergencias).
