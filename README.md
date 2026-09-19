# VisualSPHysics
VisualSPHysics is an add-on for [Blender](https://www.blender.org/) that allows the user to render realistic images and animations for [DualSPHysics](http://dual.sphysics.org/) simulations.

VisualSPHysics also includes a foam simulator for DualSPHysics fluid simulations. It is based on the method of [M. Ihmsen et al.](https://doi.org/10.1007/s00371-012-0697-9)

More details about this tool can be read on the reference paper:

[García-Feal, O., Crespo, A. J. C., & Gómez-Gesteira, M. **VisualSPHysics: advanced fluid visualization for SPH models**. Computational Particle Mechanics, 1-14.
](https://link.springer.com/article/10.1007/s40571-020-00386-7).

![](http://dual.sphysics.org/blender/img/screenshot.png)

## Instructions
Check out the [wiki](https://github.com/EPhysLab-UVigo/VisualSPHysics/wiki) to read the installation and usage documentation.

## Building on Ubuntu

These steps compile the native Python modules (`vtkimporter` and `diffuseparticles`) from source and package the add-on for Blender.

### 1. Install build dependencies

```bash
sudo apt update
sudo apt install -y build-essential cmake git python3-dev libvtk9-dev
```

- `build-essential` provides `gcc`/`g++`/`make` and OpenMP support (`libgomp`) out of the box — no extra package is needed for the foam simulator's multithreading.
- `libvtk9-dev` is available on Ubuntu 22.04 and newer. On older releases (e.g. 20.04) use `libvtk7-dev` instead.
- If your distro's `cmake` is older than 3.10, install a newer one from [Kitware's apt repo](https://apt.kitware.com/) or with `pip install cmake`.

### 2. Clone and build

```bash
git clone https://github.com/EPhysLab-UVigo/VisualSPHysics.git
cd VisualSPHysics
mkdir build && cd build
cmake ..
make -j$(nproc)
```

This produces `vtkimporter/vtkimporter.so` and `foamsimulator/diffuseparticles.so`.

### 3. Match Blender's bundled Python (important)

The compiled `.so` modules must be built against the same Python version your Blender install uses internally — Blender ships its own embedded Python, which often does **not** match the system's `python3`. Check it with:

```bash
/path/to/blender --background --python-expr "import sys; print(sys.version)"
```

If that version differs from the `python3-dev` package apt installed (`python3 --version`), install the matching headers instead (e.g. for Python 3.11) and point cmake at them:

```bash
sudo apt install -y python3.11-dev
cmake .. -DPYTHON_INCLUDE_DIR=/usr/include/python3.11 -DPYTHON_LIBRARY=/usr/lib/x86_64-linux-gnu/libpython3.11.so
make clean && make
```

### 4. Package and install into Blender

```bash
cpack
```

This generates `visualsphysics.zip` inside `build/` containing `VisualSPHysics.py`, `vtkimporter.so` and `diffuseparticles.so`. In Blender, go to **Edit > Preferences > Add-ons > Install...** and select that zip.

## Examples

https://www.youtube.com/watch?v=EvSDFRfJToQ
https://www.youtube.com/watch?v=U6lloRvgoXA
