# Vulkan Ray Tracing Engine

Motor de ray tracing multiplataforma con dos backends de cómputo:
**CUDA + Vulkan** en Windows y **Metal + Vulkan** en macOS. La misma base
de código corre en ambas plataformas; solo cambia el backend de cómputo y
la capa de ventana, aislados en archivos específicos de plataforma.

## Vista previa

![Test render](test_render.png)

## Características

- **Path tracing progresivo** con acumulación temporal y Russian Roulette.
- **BVH con pruning exacto** para traversal eficiente de la escena.
- **Primitivas soportadas:** esferas, triángulos, cajas y cilindros.
- **Tonemapping HDR → LDR** con dithering estocástico.
- **Cero copias en macOS:** gracias a la memoria unificada de Apple Silicon,
  el kernel Metal escribe directamente en el buffer de Vulkan.
- **Ventana nativa** en cada plataforma (Win32 / Cocoa), sin dependencias
  externas como GLFW o SDL.
- **Resize, fullscreen y captura de mouse** funcionando en ambas plataformas.

## Plataformas soportadas

| Plataforma | Cómputo | API gráfica | Driver Vulkan | Estado |
|---|---|---|---|---|
| Windows | CUDA | Vulkan | Nativo (NVIDIA/AMD) | ✅ |
| macOS (Apple Silicon) | Metal | Vulkan | KosmicKrisp o MoltenVK | ✅ |
| macOS (Intel) | — | — | — | ❌ |

> **Nota sobre macOS Intel:** el motor asume memoria unificada y usa la API
> Metal para cómputo. No funcionará en Macs Intel con GPU discreta.

## Requisitos

### Windows

- **Visual Studio 2022** (con soporte C++, MSVC, y compilador CUDA).
- **CUDA Toolkit 12.x** o superior.
- **Vulkan SDK** de [LunarG](https://vulkan.lunarg.com/sdk/home).
- **CMake 3.21+**. Opcionalmente **Ninja** para builds más rápidos.
- **GPU NVIDIA** compatible con CUDA.

### macOS (Apple Silicon)

- **Xcode 16+** con **Metal Toolchain** instalado:

  ```bash
  sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
  sudo xcodebuild -runFirstLaunch
  xcodebuild -downloadComponent MetalToolchain
  ```

- **Vulkan SDK** de [LunarG](https://vulkan.lunarg.com/sdk/home/macos/download)
  (incluye MoltenVK y `glslc`). Exporta `VULKAN_SDK` en tu shell.
- **CMake 3.21+**. Puedes instalarlo con Homebrew:

  ```bash
  brew install cmake
  ```

## Compilación y ejecución

### Windows

Con el generador de Visual Studio (multi-config), `CMAKE_BUILD_TYPE` se
**ignora** — la configuración se elige con `--config` al compilar.

```powershell
# 1. Configurar el proyecto (genera build/engine.sln)
cmake -B build

# 2. Compilar en Release
cmake --build build --config Release --parallel

# 3. Ejecutar
.\build\Release\engine.exe
```

Si prefieres un binario único y builds más rápidos, usa **Ninja**
(single-config, aquí sí funciona `CMAKE_BUILD_TYPE`):

```powershell
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build
.\build\engine.exe
```

También puedes abrir la solución directamente en Visual Studio:

```powershell
start build\engine.sln
```

y pulsar `F5` (con debugger) o `Ctrl+F5` (sin debugger).

Los shaders compilados (`.spv`) se copian automáticamente junto al
ejecutable en `build\Release\shaders\`.

### macOS

Con el generador por defecto (Unix Makefiles, single-config), la
configuración se pasa en tiempo de configuración con `CMAKE_BUILD_TYPE`.

```bash
# 1. Apuntar al Vulkan SDK (ajusta la versión a la que tengas instalada)
export VULKAN_SDK="$HOME/VulkanSDK/1.3.275.0/macOS"
export PATH="$VULKAN_SDK/bin:$PATH"

# 2. Configurar
cmake -B build -DCMAKE_BUILD_TYPE=Release

# 3. Compilar
cmake --build build --parallel

# 4. Ejecutar
./build/engine
```

También puedes usar **Ninja** o **Xcode** como generador:

```bash
# Ninja (más rápido)
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build
./build/engine

# Xcode (multi-config, aquí --config es obligatorio)
cmake -B build -G Xcode
cmake --build build --config Release --parallel
./build/Release/engine
```

Los shaders compilados (`tonemap.spv` y `raygen.metallib`) se copian
automáticamente junto al ejecutable en `build/shaders/`.

## Problemas comunes

- **`CMake Warning: Manually-specified variables were not used by the
  project: CMAKE_BUILD_TYPE` en Windows** — es inofensivo. El generador
  de Visual Studio es *multi-config* e ignora `CMAKE_BUILD_TYPE`. No lo
  pases al configurar y elige la configuración con `--config Release`
  al compilar.

- **`glslc no encontrado` al configurar** — asegúrate de que `VULKAN_SDK`
  está exportado en la shell, o de que el `bin/` del SDK está en el `PATH`.

- **No aparece `engine.exe` / `engine` tras compilar** — revisa que la
  compilación terminó sin errores. El binario queda en
  `build\Release\engine.exe` (Windows + Visual Studio),
  `build\engine.exe` (Windows + Ninja) o `build/engine` (macOS + Makefiles/Ninja).

- **macOS: fallo al crear la instancia Vulkan** — confirma que el
  `VULKAN_SDK` cargado en tiempo de ejecución coincide con el usado al
  compilar, y que MoltenVK / `libvulkan` están en el `PATH`
  (y en `DYLD_LIBRARY_PATH` si aplica).