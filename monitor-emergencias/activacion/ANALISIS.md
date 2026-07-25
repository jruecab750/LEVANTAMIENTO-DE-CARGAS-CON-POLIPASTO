# Análisis: web ciudadana de activación inmediata ante incendio forestal

**Prototipo formativo** — Ciclo de Emergencias y Protección Civil, módulo de Intervención
Operativa. Caso de diseño: incendio forestal en Andalucía.

---

## 1. Problema

Cuando se declara un incendio con afectación a población, la información oficial a la
ciudadanía se dispersa: ruedas de prensa, X/redes, medios, boca a boca. El ciudadano
bajo estrés necesita **un solo lugar** que responda, en menos de 30 segundos, a tres
preguntas: **¿me afecta?, ¿qué hago ahora?, ¿dónde me informo si esto cambia?**

Es-Alert (alerta a móviles) despierta y ordena una acción breve, pero no puede sostener
información viva (cambia cada hora) ni detalle por municipio. Esta web es el **destino**
al que Es-Alert, el 112 y las cuentas oficiales pueden apuntar.

## 2. Concepto: activación inmediata

La web existe **antes** del incendio, desplegada y probada («preparada»). Activarla no es
construir nada: es **rellenar un bloque de datos** (el *payload*) y publicar. Objetivo de
tiempo: **menos de 15 minutos** desde la orden del director del plan hasta la publicación.

Tres estados, misma página:

| Estado | Qué muestra |
|---|---|
| `preparada` | Autoprotección preventiva general + aviso de que se activará en emergencia |
| `activada` | La emergencia en curso: semáforo por municipio, instrucciones, mapa, boletines |
| `desactivada` | Fin de la emergencia, información de retorno y recursos post-incendio |

## 3. Principios de comunicación de crisis aplicados

1. **Una pregunta primero**: «¿Dónde estás?» — todo lo demás se filtra por municipio.
   El ciudadano no debe leer lo que no le aplica.
2. **Órdenes claras con semáforo**: EVACUACIÓN / CONFINAMIENTO / ALERTA / NORMALIDAD,
   con color + icono + palabra (nunca solo color).
3. **Lectura fácil**: frases cortas, imperativo, sin jerga técnica (nada de «PEIF»,
   «nivel operativo 2»… sin traducir). Tipografía grande, contraste alto, pensada para
   leerse en un móvil al sol y con cobertura débil.
4. **Ligereza extrema**: una sola página estática, sin frameworks; debe cargar con 3G
   saturado. El mapa es secundario y carga en diferido: el texto manda.
5. **Fuente única y datada**: cada dato lleva la hora del boletín; se enumeran los
   boletines para que el ciudadano sepa si tiene la última versión.
6. **Contra el bulo**: bloque explícito «fuentes oficiales» + «no compartas información
   sin fuente»; la propia página es enlazable como desmentido.
7. **112 siempre visible**: botón fijo de llamada; y el recordatorio inverso — **no
   llamar al 112 para pedir información**, solo emergencias (se da el teléfono de
   información al ciudadano).
8. **Accesibilidad**: HTML semántico, roles ARIA en el semáforo, botones ≥48 px,
   texto real (no imágenes de texto).

## 4. Modelo de datos (el payload)

Bloque `PAYLOAD` al inicio del HTML, editable por personal no técnico:

- `estado`: preparada | activada | desactivada
- `emergencia`: nombre operativo (IF …), provincia, fecha/hora de declaración, nivel
- `boletin`: número, fecha/hora, autoridad emisora
- `municipios[]`: nombre + situación (evacuacion | confinamiento | alerta | normalidad)
  + instrucción específica + punto de encuentro / albergue (con coordenadas)
- `cortes[]`: vía, tramo, alternativa
- `albergues[]`: nombre, dirección, coordenadas, admite animales sí/no
- `telefonos`: información al ciudadano, albergues, personas desaparecidas
- `fuentes[]`: cuentas y webs oficiales de esa emergencia
- `mapa`: centro, zoom, polígono aproximado de zona afectada (opcional)

Todo campo vacío se oculta solo: el payload mínimo viable es estado + emergencia +
un municipio.

## 5. Flujo operativo de activación

1. **Preparación (hoy)**: la plantilla vive publicada en estado `preparada`; el gabinete
   dispone de un payload de ejemplo comentado y ha ensayado el proceso (simulacros).
2. **Orden de activación**: la da la dirección del plan (nivel 1+ con afectación a
   población). El gabinete rellena el payload con el primer boletín **validado por el
   CECOP** — la web nunca publica nada no confirmado.
3. **Publicación**: commit/subida del fichero (GitHub Pages u hosting estático: sin
   base de datos que caiga bajo pico de tráfico; una página estática aguanta virales).
4. **Boletines**: cada cambio relevante = nuevo boletín (número + hora). Es-Alert y las
   cuentas oficiales enlazan siempre a la misma URL.
5. **Desactivación**: estado `desactivada` con información de retorno; la página queda
   como registro y vuelve a `preparada` tras la campaña.

## 6. Qué NO es esta web

- No es el monitor del operador (ese es `monitor-emergencias`): aquí no hay capas, ni
  FIRMS, ni predicción — al ciudadano no se le dan datos crudos que deba interpretar.
- No sustituye a Es-Alert, al 112 ni a la cadena de mando del plan de emergencias.
- Este prototipo es **formativo**: no es un canal oficial y así lo declara un banner
  permanente no eliminable en el diseño.

## 7. Métricas de éxito (para el análisis en el aula)

- Tiempo orden→publicación (< 15 min) y orden→boletín siguiente (< 10 min).
- ¿Responde las 3 preguntas en 30 s? (test con usuarios: abuela-test).
- Peso de página < 200 KB sin mapa; funcional sin JavaScript para el contenido crítico.
- Cero datos no validados publicados.

## 8. Líneas de evolución

- Versión multiidioma (turismo: inglés mínimo; alemán/francés en costa).
- Generador de payload con formulario (para que el gabinete no toque código).
- Integración de la URL en la plantilla de mensaje Es-Alert de la comunidad autónoma.
- Réplica automática del semáforo municipal hacia el monitor del operador
  (`CONFIG.municipiosDeclarados`) — mismo dato, dos vistas.
