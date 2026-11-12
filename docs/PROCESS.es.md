# Cómo se hizo: prompts, iteración y supervisión humana

Este documento explica el proceso con el que se construyó el comparador de **Laponia 2027**: una IA agéntica trabajando bajo dirección humana, con un agente orquestador que reparte el trabajo en subagentes y un humano que decide, corrige y valida. Complementa al [README principal](../README.md) · [English version](PROCESS.md).

> **Idea central:** la IA no eligió el viaje. Aceleró la búsqueda, el cálculo y la comparación; las decisiones que cambiaron el resultado final las tomó siempre una persona.

---

## 1. Los tres niveles

| Nivel | Quién | Qué hace | Qué NO hace |
|---|---|---|---|
| **Dirección** | Humano (Alejandro) | Define el objetivo, las restricciones y los criterios; descarta o añade opciones; valida cada resultado. | No busca precios a mano ni hace las cuentas. |
| **Orquestación** | Agente principal | Descompone cada petición en tareas, las reparte, integra los resultados, comprueba coherencia y lo explica. | No da por bueno un dato sin fuente; no reserva ni paga. |
| **Ejecución** | Subagentes | Tareas acotadas y en paralelo: vuelos, alojamiento, coches, actividades, revisión de cifras. | No toman decisiones de alcance ni cambian el planteamiento. |

```
            ┌────────────────────────────┐
            │   HUMANO: objetivo,        │◄──────────── revisa, corrige,
            │   restricciones, decisión  │              descarta, valida
            └─────────────┬──────────────┘
                          │ prompt
                          ▼
            ┌────────────────────────────┐
            │  AGENTE ORQUESTADOR        │
            │  descompone · integra ·    │
            │  verifica · explica        │
            └──┬─────────┬─────────┬─────┘
               ▼         ▼         ▼
          Vuelos    Alojamiento  Coches / actividades
          (Kiwi,    (Booking)    (Kayak, operadores)
           Kayak)
               └─────────┴─────────┘
                         │ resultados con fuente
                         ▼
                  comparador (index.html)
```

---

## 2. Cronología de iteraciones

Cada ronda empezó con una corrección humana y terminó con un resultado mejor que el anterior.

### Ronda 0 · Punto de partida
- **Humano:** parte del planner de viaje de un amigo y pide ampliarlo para poder comparar destinos.
- **Resultado:** una única ruta, sin alternativas.

### Ronda 1 · Barato, pero con calidad
- **Humano:** prioriza alojamientos baratos *con calidad garantizada* y pide alternativas, incluida Rovaniemi (Laponia finlandesa).
- **Agente:** busca alojamiento filtrando por valoración y compara varias opciones.
- **Mejora:** el criterio de calidad evita elegir solo por precio.

### Ronda 2 · Foco y margen de seguridad
- **Humano:** se centra en **Kiruna vs Rovaniemi**, fija las fechas (25–31 de enero, 4–5 noches) y exige **sobreestimar antes que infraestimar**.
- **Agente:** ajusta el modelo de costes con colchón y marca cada partida como *fuente* o *estimación*.
- **Mejora:** los totales dejan de ser optimistas y cada cifra es auditable.

### Ronda 3 · El grupo cambia
- **Humano:** corrige el tamaño del grupo (de 20 a **10–14 personas**) y reorienta la ruta de Kiruna a **Kiruna + Noruega** (Narvik, Abisko).
- **Agente:** recalcula alojamiento por tamaño de grupo, coches necesarios y costes por persona.
- **Mejora:** el modelo pasa a ser un **selector de 10–14 personas**, no un número fijo.

### Ronda 4 · Tres opciones y restricciones de vuelo
- **Humano:** compara tres opciones, pide **volar solo con maleta de mano** y **coches familiares/SUV** en lugar de furgonetas.
- **Agente:** repite la búsqueda de vuelos con esa condición y revisa los coches (seguro, conductor joven, conductor extra, combustible, peajes y ferris).
- **Mejora:** precios realistas para el viaje que de verdad se va a hacer.

### Ronda 5 · Cambio de tercera opción
- **Humano:** sustituye *Full Noruega* por **Solo Tromsø** (plan de un amigo, con coche todos los días).
- **Agente:** reconstruye esa opción completa: vuelo directo, casa en Kvaløya, Lyngen, Sommarøy.
- **Mejora:** la comparación final pasa a ser entre tres planes que el grupo considera viables.

### Ronda 6 · Publicación y documentación
- **Humano:** pide actualizar y publicar el README del repositorio y documentar el proceso.
- **Agente:** reescribe el README y comprueba el resultado renderizado.
- **Mejora:** el repositorio explica qué hay, cómo se calculó y qué es estimación.

---

## 3. Qué corrigió el humano (y la IA no habría detectado sola)

| Intervención humana | Por qué importaba | Efecto en el output |
|---|---|---|
| "Barato pero con calidad" | La IA tiende a minimizar precio. | Filtro por valoración en alojamiento. |
| "Sobreestima antes que infraestimar" | Un presupuesto optimista rompe un viaje de grupo. | Colchón del +5 %, partidas conservadoras. |
| Grupo de 20 → 10–14 | Cambia coches, casas y precio por persona. | Recalculo por tamaño de grupo. |
| Solo maleta de mano | Un detalle personal que cambia el precio del vuelo. | Vuelos buscados sin equipaje facturado. |
| SUV en lugar de furgonetas | Conocimiento práctico de nieve y carretera. | Flota de coches revisada. |
| Sustituir la tercera opción | Información que solo tenía el grupo. | Comparación más útil. |
| Conseguir permiso del trabajo para las fechas | Restricción real fuera del alcance de la IA. | Ventana de fechas realista. |

---

## 4. Qué aportó la orquestación con subagentes

- **Paralelismo:** vuelos, alojamiento, coches y actividades se investigan a la vez, no uno detrás de otro.
- **Contexto acotado:** cada subagente recibe solo lo que necesita (fechas, grupo, restricciones), con menos ruido y menos errores.
- **Integración con criterio:** el orquestador unifica monedas (10,95 SEK/€, 11,5 NOK/€, 7,46 DKK/€), formatos y supuestos antes de mostrar nada.
- **Verificación cruzada:** una pasada final busca incoherencias (noches, personas, divisas, precios sin fuente).
- **Trazabilidad:** cada partida indica si es **fuente** o **estimación**, para que el humano sepa en qué puede confiar.

---

## 5. Plantillas de prompts reutilizables

Plantillas generalizadas del método; no son transcripciones literales.

**Orquestador: arranque**
```
Objetivo: comparar [N] opciones de viaje para [grupo, fechas, origen].
Prioridad: [la que importa]. Restricciones: [maleta de mano, tipo de coche, presupuesto...].
Descompón en tareas independientes, lanza subagentes en paralelo,
y devuélveme una tabla comparativa con fuente o estimación en cada partida.
Sobreestima antes que infraestimar. No reserves ni pagues nada.
```

**Subagente: tarea acotada**
```
Tarea: buscar [vuelos | alojamiento | coche] para [fechas, personas].
Restricciones: [...]. Devuelve: opciones ordenadas, precio total y por persona,
enlace o fuente, y fecha de consulta. Marca como "estimación" lo que no esté confirmado.
```

**Verificador: revisión final**
```
Revisa el resultado como un auditor: ¿noches y personas coinciden en todas las partidas?
¿Hay precios sin fuente? ¿Se mezclan divisas? ¿Algún supuesto contradice las restricciones?
Lista los problemas; no los corrijas sin avisar.
```

**Humano: corrección de rumbo**
```
Cambia [X] por [Y] porque [motivo]. Recalcula solo lo afectado y dime qué totales se mueven.
```

---

## 6. Límites y buenas prácticas

- Los precios se consultaron el **9–10 de noviembre de 2026** y pueden cambiar al reservar.
- La herramienta es de planificación: **no reserva ni paga**. Eso lo decide y lo hace una persona.
- Una cifra marcada como estimación es una hipótesis, no un dato; se confirma antes de comprometer dinero.
- Cuando la IA y el humano discrepan, manda el criterio humano; la IA debe explicar el impacto del cambio.

---

## 7. Qué demuestra este proyecto

1. **Más iteraciones, mejor resultado:** cada corrección humana mejoró el modelo (grupo, calidad, equipaje, coches, opciones).
2. **Supervisión efectiva:** las decisiones de alcance y de riesgo no se delegaron.
3. **División del trabajo clara:** una persona dirige, un orquestador coordina, los subagentes ejecutan en paralelo.
4. **Resultado auditable:** cada cifra dice de dónde sale y cuánto margen lleva.

---

*Documento de proceso, no es una reserva ni una oferta.*
