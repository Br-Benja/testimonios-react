# Práctica inicial de React — Aplicación de testimonios

TecNM Celaya · Ingeniería de Software · Unidad 3 · AGO–DIC 2026

Interfaz que muestra testimonios de uno en uno, con navegación manual (Anterior, Siguiente, Aleatorio) y reproducción automática cada cinco segundos (Pausar/Reanudar).

**Etapa terminada:** básica + ampliación con reproducción automática.

**Sitio publicado:** https://br-benja.github.io/testimonios-react/ (GitHub Pages, se actualiza con cada push a `main`).

> Los testimonios y fotografías son datos ficticios de demostración (avatares de pravatar.cc).

## Integrantes

| Nombre | Número de control | Rol |
|---|---|---|
| Víctor Benjamín Bautista Rojas (@Br-Benja) | | Conductor / observador (alternados) |
| Leonardo Manuel Olvera Álvarez | | Conductor / observador (alternados) |

## Versiones utilizadas

| Herramienta | Versión |
|---|---|
| Node.js (`node -v`) | v24.14.0 |
| npm (`npm -v`) | 11.9.0 |
| Vite | 8.3.5 |
| React | ver `package.json` |

## Instalación y ejecución

```bash
npm install      # instala dependencias
npm run dev      # servidor de desarrollo (http://localhost:5173)
npm run build    # genera la versión de producción en dist/
npm run preview  # revisa localmente la compilación (no publica)
```

## Estructura

```
testimonios/
├─ index.html              # contiene <div id="root"> y carga main.jsx
├─ package.json / package-lock.json
├─ public/                 # recursos estáticos
└─ src/
   ├─ main.jsx             # crea la raíz de React, monta <App/> e importa styles.css
   ├─ App.jsx              # estado (index, reproduciendo), navegación y temporizador
   ├─ data.js              # arreglo de 8 testimonios (módulo JS con export default)
   ├─ styles.css           # estilos globales y de accesibilidad
   └─ components/
      ├─ Testimonial.jsx   # presenta un testimonio (recibe item por props)
      └─ Controls.jsx      # botones; comunica acciones mediante callbacks
```

**Recorrido:** `index.html` → `main.jsx` (createRoot en `#root`) → `App` → `Testimonial` + `Controls`.

## Decisiones principales

- **Separación de responsabilidades:** `data.js` solo tiene datos; `Testimonial` solo presenta; `Controls` solo avisa intenciones; `App` es el único que conserva y cambia el estado.
- **Flujo de datos:** App baja a `Testimonial` el objeto `item={testimonios[index]}` y a `Controls` las funciones `onPrev`, `onNext`, `onRandom`. Los hijos no importan `data.js` ni modifican el estado directamente.
- **Índice circular con `%`:** `next` usa `(actual + 1) % total`; `prev` usa `(actual - 1 + total) % total` para evitar negativos.
- **Aleatorio sin repetir:** se elige un salto entre 1 y `total − 1`, por lo que nunca cae en el mismo índice cuando hay 2+ registros. Con 1 registro no hace nada.
- **Lista vacía:** si `total === 0` se muestra “No hay testimonios disponibles.” y no se renderizan tarjeta ni controles. Los Hooks se declaran antes de ese retorno condicional.
- **Autoplay con `useEffect`:** el intervalo se crea solo si `reproduciendo` es verdadero y hay 2+ registros; la limpieza `clearInterval` evita intervalos acumulados. Inicia pausado para permitir leer. Cualquier navegación manual pasa por `navegar()`, que pausa la reproducción.
- **Accesibilidad:** botones con texto visible (nombre accesible), `type="button"`, foco visible con `:focus-visible`, `min-height: 44px`, texto alternativo en las fotos, `h1` → `h2` jerárquico y `lang="es"`. Se retiró `aria-live` del contador en la etapa automática para evitar anuncios cada 5 s.
- **Diseño adaptable:** ancho máximo de 560 px, título con `clamp()`, controles con `flex-wrap` y `overflow-wrap: anywhere` para textos largos.

## Registro de pruebas — Etapa básica

Pruebas P01–P07 realizadas con los 3 registros iniciales; P08 con los 8 registros.

| Prueba | Acción y resultado esperado | Resultado obtenido | Estado | Corrección |
|---|---|---|---|---|
| P01 Inicio | Se muestra Ana y 1 de 3 | Ana Pérez, “Testimonio 1 de 3” | Aprobado | — |
| P02 Siguiente | Desde Ana, muestra Luis y 2 de 3 | Luis García, 2 de 3 | Aprobado | — |
| P03 Límite superior | Desde María, Siguiente muestra Ana | Ana Pérez, 1 de 3 | Aprobado | — |
| P04 Límite inferior | Desde Ana, Anterior muestra María | María López, 3 de 3 | Aprobado | — |
| P05 Aleatorio | Cambia a otro válido (10+ veces) | 20 clics, 0 repeticiones; la lógica del salto `1..total−1` lo garantiza | Aprobado | — |
| P06 Un registro | Todos los controles mantienen el registro | Se mantiene, sin errores | Aprobado | — |
| P07 Sin registros | Con `[]` aparece el mensaje, sin controles | “No hay testimonios disponibles.” | Aprobado | — |
| P08 Ampliación de datos | 8 registros, navegación completa | Contador “de 8”; 8 → 1 y 1 → 8 correctos | Aprobado | — |
| P09 Presentación | 320/768/1280 px, textos largos, teclado | Sin desplazamiento horizontal en 320 y 1280 px. El botón Reanudar quedaba pegado a los controles en móvil | Aprobado | Se agregó `.app > button { margin-top: 10px; }` |
| P10 Compilación | `npm run build` sin errores; consola limpia | Build exitoso; consola sin errores | Aprobado | — |

**Corrección registrada:** en P09, a 320 px el botón Reanudar quedaba pegado a la fila de controles. Se agregó un margen superior en `styles.css` y se verificó de nuevo en el ancho móvil.

Navegación con teclado comprobada: Tab llega a cada botón con foco visible, Enter y Espacio los activan. Ancho de 768 px sin desplazamiento horizontal.

La documentación completa está en `../Testimonios_Documentacion.pdf` y las capturas en `evidencias/`.

## Registro de pruebas — Reproducción automática

| Prueba | Resultado esperado | Resultado obtenido | Estado |
|---|---|---|---|
| A01 Activación | Reanudar avanza cada ~5 s | 5 → 6 a los 5 s, 6 → 7 a los 10 s | Aprobado |
| A02 Pausa | Pausar mantiene el testimonio | Tras 5.5 s sigue en el mismo | Aprobado |
| A03 Acción manual | Navegar cambia tarjeta y deja en pausa | Cambió y el botón volvió a “Reanudar” | Aprobado |
| A04 Reanudación | Varias veces sin aceleración | Intervalos constantes de ~5 s | Aprobado |
| A05 Límites | Del último vuelve al primero | 8 → 1 | Aprobado |
| A06 Pocos datos | Con 0 o 1 registro no hay intervalo | No aparece el botón; el efecto retorna antes de `setInterval` | Aprobado |
| A07 Limpieza | Sin intervalos acumulados | `return () => clearInterval(intervalo)` en el efecto | Aprobado |

### Traza obligatoria (3 registros)

| Acción | Índice antes | Índice después | Posición visible |
|---|---|---|---|
| Inicio | — | 0 | 1 de 3 |
| Siguiente | 0 | 1 | 2 de 3 |
| Siguiente | 1 | 2 | 3 de 3 |
| Siguiente | 2 | 0 | 1 de 3 |
| Anterior | 0 | 2 | 3 de 3 |

## Capturas

En `evidencias/`: `ancho-1280.png`, `ancho-768.png`, `ancho-320.png`, `teclado-foco.png`, `autoplay-activo.png`, `tres-registros.png`, `un-registro.png`, `sin-registros.png`, `limite-inferior.png` y `reutilizacion.png`.

## Explicación de conceptos

**Flujo de un clic:** el usuario presiona *Siguiente* → React invoca el callback `onNext` que `Controls` recibió → se ejecuta `next()` en App → `setIndex` programa un nuevo estado → React vuelve a ejecutar `App` (renderizado), que calcula `testimonios[index]` y lo pasa a `Testimonial` → React confirma en el DOM solo los cambios necesarios (nombre, foto, texto, contador).

**Qué baja de App a cada hijo:** a `Testimonial` el objeto `item`; a `Controls` tres funciones. Los datos bajan por props; las acciones suben por callbacks.

**`useState`:** `useState(0)` devuelve el valor actual y la función para cambiarlo. Cambiar una variable local (`index++`) no provoca un nuevo renderizado ni se conserva entre renderizados.

**Por qué no `testimonios[index + 1]`:** en el último elemento el índice se sale del arreglo, se obtiene `undefined` y la desestructuración en `Testimonial` falla. Por eso se usa `%`.

**JSX:** no es una cadena HTML; es una descripción que se compila a llamadas de JavaScript. Las llaves `{nombre}` insertan una expresión JavaScript.

**`useEffect` vs. evento:** el clic es una acción del usuario y se maneja en su evento. El temporizador es un sistema externo que debe sincronizarse con el estado `reproduciendo`, por eso va en un efecto con limpieza.

**`clearInterval`:** evita que al pausar/reanudar (o con StrictMode en desarrollo) se acumulen intervalos y el carrusel se acelere.

**No existe un Hook `render`:** renderizar es lo que React hace al ejecutar el componente; `createRoot(...).render()` es un método, no un Hook.

**Si los datos vinieran de una API:** el arreglo tendría que guardarse en estado, cargarse con un efecto, manejar estados de carga y error, y ajustar `index` si `total` cambia y el índice queda fuera de rango.

## Experimento de reutilización

Se agregaron temporalmente dos `<Testimonial>` en una `<section>` y se cambió una regla de `.testimonial-card`: ambas tarjetas recibieron el cambio desde un solo lugar.

| Cambio solicitado | Lugar principal del cambio |
|---|---|
| Agregar un testimonio | `data.js` |
| Modificar la tarjeta de todas las personas | `Testimonial.jsx` y estilos |
| Cambiar los textos de los controles | `Controls.jsx` |
| Cambiar el comportamiento de navegación | `App.jsx` |
| Modificar apariencia sin cambiar navegación | `styles.css` |

## Proceso (Scrum / XP)

Tablero: Pendiente · En proceso · En revisión · Terminado.

Programación en pareja: los roles de conductor y observador se alternaron entre Víctor Benjamín Bautista Rojas y Leonardo Manuel Olvera Álvarez en cada avance.

| Tarea | Puntos | Estado |
|---|---|---|
| Entorno (Vite, Node) | 1 | Terminado |
| Datos (`data.js`) | 1 | Terminado |
| Tarjeta (`Testimonial`) | 2 | Terminado |
| Navegación (`Controls`, estado) | 3 | Terminado |
| Estilos y accesibilidad | 2 | Terminado |
| Verificación (P01–P10) | 2 | Terminado |
| Autoplay (A01–A07) | 3 | Terminado |

## Referencias

- https://es.react.dev/learn
- https://es.react.dev/learn/thinking-in-react
- https://es.react.dev/learn/state-a-components-memory
- https://es.react.dev/learn/render-and-commit
- https://es.react.dev/learn/synchronizing-with-effects
- https://vite.dev/guide/
