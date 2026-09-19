# VisualSPHysics — guía para compilar en Ubuntu

Este repo es un add-on de Blender (Python + dos módulos de extensión C++: `vtkimporter` y
`diffuseparticles`, construidos con CMake). Ya fue compilado y validado en macOS en una sesión
anterior; esta guía resume lo aprendido para que compilar en Ubuntu sea directo y para que sepas
qué verificar en vez de asumir que "make" exitoso significa que el add-on funciona.

## Pasos de compilación

```bash
sudo apt update
sudo apt install -y build-essential cmake git python3-dev libvtk9-dev
# Ubuntu 20.04: usar libvtk7-dev en vez de libvtk9-dev

cd VisualSPHysics
mkdir -p build && cd build
cmake ..
make -j$(nproc)
```

Debería generar `vtkimporter/vtkimporter.so` y `foamsimulator/diffuseparticles.so` sin pasos
extra. Los fixes de C++17 / `<filesystem>` / `<iostream>` ya están en el código fuente (commit
`7b4e52e`), así que en Linux con un compilador moderno esto debería compilar limpio a la primera.

Si `cmake` del sistema es menor a 3.10, o VTK no se encuentra, revisar la sección "Building on
Ubuntu" del `README.md` (tiene más detalle y alternativas de paquetes).

## Lo más importante: no asumas que compiló = que funciona

En macOS, `make` terminó "exitosamente" dos veces con módulos que **no cargaban** dentro de
Blender (una vez por linkear contra el Python equivocado, otra por el sufijo `.dylib` en vez de
`.so`). La única forma confiable de confirmar que el add-on realmente funciona es importarlo
dentro del propio intérprete embebido de Blender:

```bash
/ruta/a/blender --background --python-expr "
import sys
sys.path.insert(0, '/ruta/al/repo/build/vtkimporter')
sys.path.insert(0, '/ruta/al/repo/build/foamsimulator')
import vtkimporter
import diffuseparticles
print('OK')
"
```

Si esto no imprime `OK`, no des la compilación por terminada — mirá el traceback.

## Causa probable de fallos: versión de Python

Blender trae su propio Python embebido, que casi seguro **no coincide** con el `python3-dev` de
apt. Compilar contra el Python equivocado no siempre falla en `make` — puede compilar bien y
fallar recién al hacer `import` dentro de Blender (símbolos de la API de Python con firma o
tamaño distinto).

1. Comprobar qué versión usa Blender:
   ```bash
   /ruta/a/blender --background --python-expr "import sys; print(sys.version)"
   ```
2. Comparar con `python3 --version`. Si no coinciden, instalar los headers de la versión que
   Blender realmente usa (ej. `sudo apt install python3.11-dev`) y pasarle a cmake:
   ```bash
   cmake .. -DPYTHON_INCLUDE_DIR=/usr/include/python3.11 \
            -DPYTHON_LIBRARY=/usr/lib/x86_64-linux-gnu/libpython3.11.so
   make clean && make
   ```
3. Volver a correr el test de `import` de arriba.

En Linux, a diferencia de macOS, normalmente **sí** se linkea contra una `libpython3.X.so` real
(el `CMakeLists.txt` ya lo hace así fuera de `APPLE`), así que no debería hacer falta tocar nada
de `-undefined dynamic_lookup` ni el sufijo `.so` — eso es una particularidad de macOS que ya
está encapsulada en bloques `if (APPLE)` dentro de `foamsimulator/CMakeLists.txt` y
`vtkimporter/CMakeLists.txt`. Si aun así el import falla, sospechar primero de un mismatch de
versión de Python antes de tocar flags de linkeo.

## Empaquetar el add-on

```bash
cd build
cpack
```

Genera `visualsphysics.zip` con `VisualSPHysics.py`, `vtkimporter.so` y `diffuseparticles.so`.
Instalar en Blender vía **Edit > Preferences > Add-ons > Install...**, apuntando a ese zip. Tras
instalarlo, confirmar en la UI de Blender (no solo con el test de `import` por consola) que el
panel del add-on aparece y no tira errores en la consola de Blender.
