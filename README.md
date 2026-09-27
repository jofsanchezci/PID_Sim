# Simulador interactivo de cibernética: controlador PID

Este documento explica el archivo `Simulador_PID_Cibernetica.html`, una actividad para el curso de **Sistemas Complejos**. La simulación muestra cómo un controlador recibe información de la salida de una planta, calcula el error respecto a una consigna y aplica una acción para regularla. Permite explorar la retroalimentación, el amortiguamiento y la respuesta frente a una perturbación.

## Inicio rápido

1. Guarda `README_Simulador_PID_Cibernetica.md` y `Simulador_PID_Cibernetica.html` en la misma carpeta. Si descargas ambos archivos, conserva sus nombres.
2. Abre el archivo HTML con un navegador moderno. No requiere servidor, conexión a Internet ni bibliotecas externas.
3. Elige una configuración de ejemplo o mueve los controles deslizantes.
4. Observa la respuesta temporal y los indicadores. Usa **Pausar**, **Reproducir** y **Reiniciar** para controlar la animación.
5. Activa **Aplicar perturbación en t = 8 s** para comparar la recuperación del sistema.

Cada cambio en las ganancias o en el amortiguamiento vuelve a calcular la trayectoria completa y reinicia la animación.

## Modelo de la planta

La planta es un sistema dinámico de segundo orden. Su salida es `y`, su velocidad de cambio es `y'`, la consigna es `r = 1` y la señal de control es `u`:

```text
e(t)  = r - y(t)
u(t)  = limitar[Kp·e(t) + Ki·I(t) - Kd·y'(t), -5, 5]
I'(t) = e(t)                       (con protección frente a saturación)
y''(t) = -2·ζ·ω·y'(t) - ω²·y(t) + ω²·u(t) + d(t)
```

La frecuencia natural de la planta es `ω = 2 rad/s`. La perturbación opcional vale `d(t) = 0` antes de los 8 segundos y `d(t) = -0,8` a partir de ese instante. La simulación dura 20 segundos y parte de `y(0) = 0`, `y'(0) = 0` e `I(0) = 0`.

El término derivativo se calcula a partir de la salida medida (`-Kd·y'`), por lo que no introduce un impulso al fijar la consigna. La señal `u` se limita a `[-5, 5]`. Cuando el control está saturado, la integral deja de acumular error en la dirección que aumentaría la saturación; esto reduce el *windup*. El programa integra el modelo con un paso fijo de `0,005 s` mediante una actualización numérica sencilla y dibuja una muestra cada `0,02 s`. Es un modelo educativo, no una identificación de una planta real.

## Controles

| Control | Función | Efecto habitual al aumentarlo |
| --- | --- | --- |
| `Kp` | Reacciona ante el error presente. | Respuesta inicial más intensa; puede aumentar el sobrepaso y, sin integral, reducir el error estacionario. |
| `Ki` | Acumula error a lo largo del tiempo. | Puede llevar el error estacionario a cero, aunque también producir mayor sobrepaso. |
| `Kd` | Reacciona a la rapidez de cambio de la salida. | Suele amortiguar la oscilación; valores altos pueden hacer más lenta la respuesta. |
| `ζ` | Define el amortiguamiento propio de la planta. | Atenúa oscilaciones, aunque puede ralentizar la evolución. |

Estos efectos dependen de la combinación de parámetros y de la saturación del actuador. No deben interpretarse como reglas universales.

## Configuraciones incluidas

| Botón | `Kp` | `Ki` | `Kd` | `ζ` | Propósito |
| --- | ---: | ---: | ---: | ---: | --- |
| Oscilatoria | 2,5 | 0 | 0 | 0,20 | Observar sobrepaso y oscilaciones. |
| Casi crítica | 2,5 | 0 | 1,67 | 0,20 | Aproximarse al amortiguamiento crítico. |
| Lenta sin oscilación | 2,5 | 0 | 3,00 | 0,20 | Comparar una respuesta sobreamortiguada. |
| PID con integral | 2,0 | 0,45 | 1,20 | 0,30 | Explorar la corrección del error estacionario. |

La perturbación está desactivada inicialmente. Las tres primeras configuraciones, sin acción integral ni perturbación, permiten comparar directamente las clases de amortiguamiento del modelo lineal.

## Cómo leer la gráfica

- **Línea amarilla discontinua:** consigna `r = 1`.
- **Línea verde agua:** salida `y` de la planta; se lee en el eje vertical izquierdo.
- **Línea violeta:** señal de control `u`; se lee en el eje vertical derecho, cuya escala va de `-5` a `5`.
- **Línea vertical gris:** inicio de la perturbación, si está activada.
- **Marcador móvil:** instante y valor actual de la salida durante la animación.

La escala visible de la salida va de `0` a `2`. Ante parámetros extremos, una parte de la curva puede quedar fuera del área visible; los indicadores se calculan sobre los datos simulados.

### Indicadores

- **Sobrepaso:** porcentaje en que el máximo de la salida durante el tramo inicial supera el equilibrio estimado. Se observa antes de `t = 7,9 s`, para que la perturbación no contamine esta medida. Si `Ki = 0`, el equilibrio nominal es `Kp/(1 + Kp)`; si `Ki > 0`, se toma `1` como equilibrio esperado.
- **Tiempo de establecimiento ±2 %:** primer instante desde el cual todas las muestras restantes **del tramo anterior a 7,9 s** permanecen a no más de `0,02` unidades del equilibrio estimado. `> 8 s` significa que no se alcanzó ese criterio en dicho tramo. No representa necesariamente el tiempo de recuperación tras la perturbación.
- **Error final observado:** `|1 - y(20 s)|`. Cuando la perturbación está activa, incluye su efecto y puede diferir del error estacionario teórico.
- **Pico de control `|u|`:** valor absoluto máximo de la acción calculada a lo largo de los 20 segundos; su máximo posible es `5` por la saturación.

## Amortiguamiento y alcance de la clasificación

Sin integral (`Ki = 0`) ni perturbación, y mientras la señal de control no sature, la ecuación de lazo cerrado tiene la forma:

```text
y'' + (2·ζ·ω + ω²·Kd)·y' + ω²·(1 + Kp)·y = ω²·Kp·r

ζ_efectivo = (2·ζ·ω + ω²·Kd) / [2·ω·√(1 + Kp)]
y_infinito = [Kp/(1 + Kp)]·r
```

El programa etiqueta `ζ_efectivo < 0,96` como **subamortiguada**, valores entre `0,96` y `1,04` como **cercana a crítica**, y `ζ_efectivo > 1,04` como **sobreamortiguada**. La banda alrededor de `1` es una convención visual del simulador: matemáticamente, la respuesta crítica corresponde a `ζ_efectivo = 1`. Todas son respuestas amortiguadas; las categorías distinguen su forma.

Cuando `Ki > 0`, el sistema de lazo cerrado incorpora un estado integral y deja de ser estrictamente de segundo orden. Con saturación, tampoco vale la clasificación lineal exacta. En esos casos, o cuando se aplica la perturbación, el simulador presenta una **descripción observacional** del tramo inicial: con o sin sobrepaso mayor al 2 %. Es recomendable analizar la curva completa junto con el error y la señal de control.

## Actividades sugeridas

1. Selecciona **Oscilatoria** y aumenta `Kd` lentamente hasta que la curva deje de oscilar. Registra `ζ_efectivo` cerca de la transición.
2. Selecciona **Lenta sin oscilación** y reduce `Kd` para comparar la rapidez de respuesta con la aparición de sobrepaso.
3. Con `Ki = 0`, observa que la salida puede estabilizarse por debajo de la consigna. Después añade `Ki` y compara el error final.
4. Activa la perturbación y compara la recuperación con `Ki = 0` y `Ki > 0`. ¿Qué información obtiene el controlador de la retroalimentación?
5. Discute qué recursos adicionales necesitaría el regulador si, además del cambio externo, fallara el sensor. Relaciónalo con la ley de variedad requerida.

### Preguntas de análisis

- ¿Por qué elevar `Kp` puede acelerar la respuesta y, al mismo tiempo, aumentar la oscilación?
- ¿Qué se gana y qué se pierde al añadir acción derivativa?
- ¿En qué circunstancias el control saturado impide obtener la respuesta predicha por el modelo lineal?
- ¿Por qué no basta observar únicamente el error final para evaluar el desempeño?

## Archivos y compatibilidad

El proyecto consta de un único archivo HTML autónomo con CSS y JavaScript incorporados, más este README. Está diseñado para navegadores actuales de escritorio y dispositivos móviles. La gráfica usa la API Canvas del navegador. No guarda resultados ni envía datos.
