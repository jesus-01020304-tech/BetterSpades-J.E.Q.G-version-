Instrucciones para GitHub Copilot (Agent mode) — Port de BetterSpades a Android
Cómo usar este archivo: guárdalo como .github/copilot-instructions.md
en la raíz de tu fork. Copilot lo lee automáticamente como contexto en
todas las conversaciones de ese repo, en VS Code, con Agent mode
activado. También puedes simplemente pegar este texto directo en el
chat si prefieres no crear el archivo.
Repositorio
Fork de xtreme8000/BetterSpades (rama standalone), cliente open-source
de Ace of Spades en C, licencia GPLv3.
Objetivo
Portar el cliente a Android y añadir soporte de mando (DualShock 4 / PS4
vía SDL_GameController), manteniendo el mismo esquema de controles que ya
existe para escritorio.
Contexto técnico ya investigado (no lo vuelvas a investigar, ya está confirmado)
src/window.c ya tiene un backend SDL2 completo bajo #ifdef USE_SDL,
con manejo de eventos táctiles (SDL_FINGERDOWN/UP/MOTION) pensado
para Android.
El CMakeLists.txt raíz ya define las opciones:
ENABLE_SDL, ENABLE_TOUCH, ENABLE_OPENGLES, ENABLE_ANDROID_FILE.
Falta soporte de mando bajo USE_SDL (si existe bajo USE_GLFW, para
escritorio, con la API de joystick vieja — úsalo como referencia de
qué botón hace qué, pero reimplementa con SDL_GameController, que es
la API moderna con detección de tipo de mando).
No hay proyecto Android/Gradle en el repo. Hay que basarse en la
plantilla oficial android-project/ que trae el repo de SDL2
(libsdl-org/SDL), copiando los fuentes de src/ ahí dentro.
Tareas, en orden — pide confirmación antes de pasar a la siguiente
1. Soporte de mando en src/window.c (bloque #ifdef USE_SDL)
Detectar y abrir el primer mando con SDL_GameControllerOpen,
reconexión en caliente vía SDL_CONTROLLERDEVICEADDED/REMOVED.
Detectar DualShock 4 con SDL_GameControllerGetType() == SDL_CONTROLLER_TYPE_PS4.
Mismo mapeo de acciones que el bloque USE_GLFW de ese archivo:
D-pad → cambio de herramienta, Start → Escape, hombro derecho → salto,
L3 → agacharse, hombro izquierdo → sprint, X/Square → recargar,
A/Cross → disparo, B/Circle → apuntar, stick izq. → movimiento,
stick der. → cámara.
Deadzone radial en los sticks (no solo un umbral por eje).
Extra opcional: rumble (SDL_GameControllerRumble) y barra de luz PS4
(SDL_GameControllerSetLED, comprobando SDL_GameControllerHasLED
antes de usarla).
Compílalo y pruébalo en escritorio primero:
cmake .. -DENABLE_SDL=ON -DENABLE_TOUCH=ON -DENABLE_GLFW=OFF
antes de tocar nada de Android — es mucho más rápido de depurar así.
2. Proyecto Android
Descarga android-project/ del repo de SDL2 (misma versión de SDL2
que ya usa BetterSpades).
Copia los fuentes de src/ de BetterSpades dentro, ajusta el build
nativo (Android.mk o CMakeLists.txt de la plantilla) para compilar
con ENABLE_SDL, ENABLE_TOUCH, ENABLE_OPENGLES,
ENABLE_ANDROID_FILE activados y ENABLE_GLFW desactivado.
Verifica que el AndroidManifest.xml pida un targetSdkVersion
actual (no el API 26 del APK viejo de 2018, que ya no instala en
móviles modernos).
3. Menú de ajustes táctil
Añadir dos opciones nuevas al .ini de configuración: deadzone y
sensibilidad de cámara del mando (mismo patrón que la sensibilidad de
ratón ya existente).
En el menú in-game, exponerlas como sliders táctiles grandes en vez
de campos de texto, coherente con el resto de opciones de ENABLE_TOUCH.
4. GitHub Actions
Crear .github/workflows/build-android.yml que instale NDK, CMake y
Gradle, y suba el APK resultante como artifact del workflow run.
Preferencias del usuario a respetar
En menús, si is_ps4 == true: Cross = confirmar, Circle = atrás
(convención PlayStation, al revés que Xbox).
Priorizar tener algo compilando y jugable cuanto antes por encima de
cubrir todas las características de golpe. Ir commit por commit,
cada uno compilable.
Antes de cada tarea grande, resume en 2-3 líneas qué vas a hacer y
por qué, para poder frenarte si algo no encaja con lo pensado.