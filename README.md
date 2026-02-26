# RoscoIntegra

Autor: Javier Feijóo  
Licencia: CC BY-NC-SA

## Proposito
RoscoIntegra es una webapp offline para aula, inspirada en un rosco tipo Pasapalabra. Esta pensada para proyectar en clase y gestionar una partida con control docente claro: carga de bancos JSON, temporizador global, avance por letras pendientes, audio opcional y resumen final exportable.

## Estado del proyecto
La aplicacion esta organizada en tres vistas sin frameworks:

1. Inicio
2. Configuracion
3. Juego

El flujo completo ya esta implementado y funcional en HTML, CSS y JavaScript modular.

## Estructura del repositorio
```text
/roscointegra
  index.html
  README.md
  /css
    styles.css
  /js
    app.js
    state.js
    bank.js
    rosco.js
    tts.js
    sfx.js
    timer.js
    storage.js
  /assets
    manifest.json
    preguntas_base_roscointegra.json
    /img
      logo.png
      centro_rosco.png
    /audio
      (opcional: audios fallback)
```

## Responsabilidad de cada modulo JavaScript
- `js/app.js`: orquestacion general de vistas, eventos UI, flujo de partida, resumen final y exportacion.
- `js/state.js`: estado global de la app y valores por defecto.
- `js/bank.js`: validacion, normalizacion y construccion de preguntas por letra para cada partida.
- `js/rosco.js`: renderizado circular del rosco con trigonometria y estados visuales.
- `js/tts.js`: capa de TTS del navegador y fallback a audio local.
- `js/sfx.js`: efectos sonoros de juego (acierto, fallo, tic-tac ultimos segundos, fin de tiempo).
- `js/timer.js`: temporizador global preciso basado en timestamps (mm:ss, pausa/reanudar/reinicio).
- `js/storage.js`: wrapper de localStorage para guardado/lectura de configuracion y cache.

## Como ejecutar
### Opcion recomendada
Usar servidor local para permitir `fetch` de bancos en `/assets`:

- VS Code Live Server, o
- `python -m http.server`, o
- cualquier servidor estatico.

### Opcion directa
Abrir `index.html` con `file://`.

Nota: en algunos navegadores, `fetch` a `/assets` falla en `file://`. En ese caso, usa Importar archivo o pegar JSON.

## Carga de bancos de preguntas
La app soporta tres vias:

1. Desde `/assets` con `manifest.json`.
2. Importar archivo `.json` (FileReader).
3. Pegar JSON en textarea.

### Descubrimiento estable de bancos
`/assets/manifest.json` define bancos disponibles y banco por defecto.

### Validaciones aplicadas
- Debe existir array de preguntas.
- Campos obligatorios por pregunta: `letra`, `tipo`, `pregunta`, `respuesta`.
- Campo opcional por pregunta: `distractores`.
  - Array simple: `distractores: ["..."]` (comun a cualquier tipo).
  - Objeto por logica:
    - `distractores.starts` o `distractores.empieza`
    - `distractores.contains` o `distractores.contiene`
    - `distractores.all|common|comun` (comunes)
- Letra normalizada a mayusculas.
- Deteccion de duplicados (bloqueante segun el criterio implementado del banco actual).
- Deteccion de letras ausentes (warning no bloqueante).
- Normalizacion de metadatos (`ciclo`, `modulo`, `dificultad`) cuando faltan.

Siempre se muestra en UI:
- origen de carga,
- archivo,
- numero total de preguntas,
- diagnostico de errores.

### Prioridad de distractores en juego
1. Se usan primero los distractores definidos en el JSON para esa pregunta (segun logica `empieza/contiene`).
2. Si faltan opciones, se completan con distractores compatibles generados desde el banco.
3. Si aun faltan, se repiten compatibles para mantener 4 botones sin romper la logica.

## Formato JSON recomendado
El banco debe seguir esta estructura base:

```json
{
  "titulo": "Nombre del banco",
  "version": "1.0",
  "fecha_generacion": "2026-02-26",
  "descripcion": "Descripcion breve",
  "letras_incluidas": ["A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V"],
  "preguntas_por_letra": 8,
  "preguntas": [
    {
      "letra": "A",
      "tipo": "empieza",
      "pregunta": "Con la A: paso finito y ordenado para resolver un problema.",
      "respuesta": "Algoritmo",
      "distractores": {
        "starts": ["Aplicacion", "Analisis", "Abstraccion"],
        "contains": ["Paralelo", "Catalogo", "Administrador"],
        "all": ["Arquitectura"]
      },
      "ciclo": "DAW",
      "modulo": "Programacion",
      "dificultad": "basica"
    }
  ]
}
```

Reglas clave:
- `tipo` admite variantes como `empieza` o `contiene` (la app las normaliza).
- `distractores` es opcional, pero recomendable.
- En preguntas `empieza`, los valores de `distractores.starts` deben empezar por la letra.
- En preguntas `contiene`, los valores de `distractores.contains` deben contener la letra.
- Evita repetir la `respuesta` correcta dentro de `distractores`.

## Prompt Generico Para IA
Usa este prompt para generar bancos compatibles:

```text
Genera un JSON valido para RoscoIntegra, sin markdown ni comentarios.

Objetivo:
- Crear un banco de preguntas tipo rosco con letras: A,B,C,D,E,F,G,H,I,J,L,M,N,O,P,Q,R,S,T,U,V
- 8 preguntas por letra (4 tipo "empieza" y 4 tipo "contiene")
- Nivel: [INDICA NIVEL]
- Dominio tematico: [INDICA TEMATICA]
- Idioma: espanol de Espana

Formato obligatorio:
- Raiz con campos: titulo, version, fecha_generacion (YYYY-MM-DD), descripcion, letras_incluidas, preguntas_por_letra, preguntas
- Cada elemento en preguntas debe incluir:
  letra, tipo, pregunta, respuesta, distractores, ciclo, modulo, dificultad
- distractores debe ser objeto con:
  starts (3 strings), contains (3 strings), all (array opcional)

Reglas de calidad:
- Respuestas de una sola palabra (sin espacios), naturales y usadas realmente.
- Sin numeros, codigos raros ni sufijos artificiales.
- Si tipo="empieza": respuesta empieza por la letra indicada.
- Si tipo="contiene": respuesta contiene la letra indicada.
- distractores.starts: 3 respuestas plausibles que empiecen por esa letra.
- distractores.contains: 3 respuestas plausibles que contengan esa letra.
- Ningun distractor puede ser igual a la respuesta correcta.
- Mantener coherencia semantica con pregunta, ciclo/modulo/dificultad.
- No duplicar pregunta ni respuesta dentro del banco.

Devuelve solo el JSON final.
```

### Prompt En Galego
```text
Xera un JSON valido para RoscoIntegra, sen markdown nin comentarios.

Obxectivo:
- Crear un banco de preguntas tipo rosco coas letras: A,B,C,D,E,F,G,H,I,J,L,M,N,O,P,Q,R,S,T,U,V
- 8 preguntas por letra (4 tipo "empieza" e 4 tipo "contiene")
- Nivel: [INDICA NIVEL]
- Dominio tematico: [INDICA TEMATICA]
- Idioma: galego (Galicia)

Formato obrigatorio:
- Raiz cos campos: titulo, version, fecha_generacion (YYYY-MM-DD), descripcion, letras_incluidas, preguntas_por_letra, preguntas
- Cada elemento en preguntas debe incluír:
  letra, tipo, pregunta, respuesta, distractores, ciclo, modulo, dificultad
- distractores debe ser obxecto con:
  starts (3 strings), contains (3 strings), all (array opcional)

Regras de calidade:
- Respostas dunha soa palabra (sen espazos), naturais e de uso real.
- Sen numeros, codigos raros nin sufixos artificiais.
- Se tipo="empieza": a resposta empeza pola letra indicada.
- Se tipo="contiene": a resposta contén a letra indicada.
- distractores.starts: 3 respostas plausibles que empecen por esa letra.
- distractores.contains: 3 respostas plausibles que conteñan esa letra.
- Ningún distractor pode ser igual á resposta correcta.
- Manter coherencia semantica con pregunta, ciclo/modulo/dificultad.
- Non duplicar pregunta nin resposta dentro do banco.

Devolve so o JSON final.
```

### Prompt In English
```text
Generate a valid JSON for RoscoIntegra, with no markdown and no comments.

Goal:
- Create a rosco-style question bank using letters: A,B,C,D,E,F,G,H,I,J,L,M,N,O,P,Q,R,S,T,U,V
- 8 questions per letter (4 with type "empieza" and 4 with type "contiene")
- Level: [SET LEVEL]
- Domain: [SET TOPIC]
- Language: English

Required format:
- Root fields: titulo, version, fecha_generacion (YYYY-MM-DD), descripcion, letras_incluidas, preguntas_por_letra, preguntas
- Each item in preguntas must include:
  letra, tipo, pregunta, respuesta, distractores, ciclo, modulo, dificultad
- distractores must be an object with:
  starts (3 strings), contains (3 strings), all (optional array)

Quality rules:
- One-word answers only (no spaces), natural and commonly used.
- No numbers, weird codes, or artificial suffixes.
- If tipo="empieza": respuesta must start with the target letter.
- If tipo="contiene": respuesta must contain the target letter.
- distractores.starts: 3 plausible options starting with the letter.
- distractores.contains: 3 plausible options containing the letter.
- No distractor can be identical to the correct answer.
- Keep semantic coherence with pregunta, ciclo/modulo/dificultad.
- Do not duplicate questions or answers in the bank.

Return only the final JSON.
```

## Reglas de partida implementadas
- `Ver respuesta` es obligatorio para habilitar `Acertada` y `Fallada`.
- `PASA` no revela respuesta.
- Solo se avanza por letras `pending`.
- Letras `correct` y `wrong` se saltan.
- Una letra `wrong` queda cerrada (sin reintento).
- Al cambiar de letra, se oculta respuesta y botones de evaluacion.
- Fin de partida por:
  - tiempo en 0, o
  - ausencia total de pendientes.

## Temporizador
- Cuenta atras global (no por turno).
- Formato visible `mm:ss`.
- Arranca automaticamente al iniciar partida.
- Botones de `Pausar`, `Reanudar` y `Reiniciar`.
- Al llegar a 0 se bloquea la interaccion y aparece resumen final.

## Audio
### Modo audio
- `audioEnabled = false`: no hay TTS ni efectos ni audio local.
- `audioEnabled = true`: se habilitan TTS y efectos.

### TTS
- Auto-lectura de pregunta al entrar en cada letra (si audio habilitado).
- Lectura manual de pregunta y respuesta.
- Ajustes de voz, rate y pitch persistidos.

### Efectos sonoros (SFX)
- Sonido al acertar.
- Sonido al fallar.
- Tic-tac en los ultimos 10 segundos.
- Sonido de fin de tiempo.

## Persistencia (localStorage)
Se guardan y restauran automaticamente:
- tiempo total y unidad,
- puntos por acierto y penalizacion,
- filtros (`ciclo`, `modulo`, `dificultad`),
- shuffle,
- `audioEnabled`,
- preferencias TTS,
- modo docente,
- ultimo banco cargado (cache local).

## Resumen final
El cierre muestra un dashboard visual con:
- puntuacion,
- correctas,
- falladas,
- pasadas,
- porcentaje,
- tiempo restante,
- tiempo consumido.

Incluye boton `Exportar resultados` para descargar JSON con timestamp, configuracion y resultados.

## Atajos de teclado
- Espacio: start/pause
- R: ver respuesta
- P: PASA
- A: acertada (si respuesta visible)
- F: fallada (si respuesta visible)
- C y E: alias opcionales (si respuesta visible)
- Flechas: anterior/siguiente pendiente

## Guia de pruebas manuales
1. Carga de assets en servidor local: validar `manifest.json` y banco por defecto.
2. Carga en `file://`: verificar aviso de limitacion y uso de importar/pegado.
3. JSON invalido: comprobar bloqueo y mensaje de error visible.
4. Filtros que dejan 0 resultados: no debe iniciar partida.
5. Flujo docente completo: Ver respuesta, Acertada/Fallada, PASA y vueltas.
6. Temporizador: pausa/reanuda/reinicia y fin por tiempo.
7. Audio: auto-lectura por letra, lectura manual y SFX de acierto/fallo/tic-tac.
8. Exportacion: verificar que el JSON descargado contiene configuracion y resultados.

## Notas
- Proyecto sin dependencias externas de runtime.
- Pensado para funcionar offline, con mejor experiencia de carga de assets usando servidor local.
