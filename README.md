# 🧛 Vampire Buster — A 3D OpenGL Action Game

A 3D action game built from the ground up in C++ and OpenGL. Step into a fully textured urban arena, arm yourself with melee blades, and hunt down a swarm of vampires before facing off against a multi-HP boss. Built as a deliberate showcase of the classical OpenGL graphics pipeline — every model, light, texture, and camera movement is wired by hand.

---

## Gameplay

You play a vampire hunter exploring a 3D urban environment. Move through the streets, scoop up melee weapons left around the map, and close the distance on enemies before striking. The camera follows the action in third person, with a smooth swap to first person for tighter combat moments.

**Controls**

| Input | Action |
|---|---|
| **W** | Walk forward |
| **Mouse** | Look / aim camera |
| **Left click** | Attack |

The minimal control scheme keeps the focus where it belongs — on the world, the enemies, and the moment of the strike.

---

## Levels

**Level 1 — The Hunt.** A wave of vampires is spread across the arena. Track them down one by one, line each one up, and finish them off. Every kill is punctuated with audio feedback, and clearing the wave unlocks the next stage.

**Level 2 — The Boss Fight.** A larger, more dangerous vampire enters the arena with multiple hit points. This one won't go down on the first strike — read its position, time your attacks, and chip away at its health. A countdown timer adds the pressure: defeat it before time runs out to win, or hear the Game Over jingle if it survives.

---

## Rendering

The entire 3D world is rendered through the OpenGL pipeline — a deliberate exercise in mastering the fundamentals of real-time graphics:

- **3D models** loaded directly from Autodesk `.3DS` files via a custom C++ model loader, with full vertex, face, UV, and material parsing.
- **Multiple light sources** including an animated directional sun that sweeps across the sky and shifts the scene's diffuse colour over time, simulating a day/night cycle.
- **Textured skybox** rendered as a quadric sphere with a sky bitmap mapped across its interior, giving the arena a sense of openness.
- **Textured ground plane** with tiled BMP textures and proper UV repetition.
- **BMP texture loading** with mipmap generation for crisp visuals at any distance.
- **Hand-rolled camera system** with eye, center, and up vectors, switching cleanly between first-person and third-person perspectives via `gluLookAt`.
- **Bounding-box collision detection** keeping the player anchored to the playable streets and reacting on contact with enemies and props.
- **Win32 audio** layered on top of the renderer for atmospheric sound effects on every kill, level transition, and game-state change.

---

## Tech Stack

- **C++** — game logic, rendering callbacks, model and texture pipelines
- **OpenGL** — 3D rendering pipeline
- **GLUT** — windowing, input, and the main frame loop
- **GLEW** — OpenGL extension management on Windows
- **GLAUX** — BMP image loading
- **Visual Studio 2019** — IDE and build environment

---

## Project Structure

| File | Role |
|---|---|
| `OpenGLMeshLoader19.cpp` | Main game file — entry point, GLUT callbacks, camera, collision, level logic, and per-frame draw calls |
| `Model_3DS.h` / `.cpp` | 3DS-format model loader and renderer |
| `GLTexture.h` / `.cpp` | Texture wrapper supporting BMP loading and mipmap generation |
| `TextureBuilder.h` | Helpers for loading skybox and ground textures |
| `glut.h`, `glew.h`, `glaux.h` + `.lib` files | Vendored graphics and image-loading libraries |

---

## Building

The project is built and run in **Visual Studio 2019** as a C++ Win32 console application.

1. Open Visual Studio 2019 and create a new **Empty C++ Project**.
2. Add the source files (`OpenGLMeshLoader19.cpp`, `Model_3DS.cpp`, `GLTexture.cpp`) and the header files to the project.
3. Under **Project Properties → C/C++ → General**, add the project root to **Additional Include Directories** so `glut.h`, `glew.h`, and `glaux.h` resolve.
4. Under **Linker → General**, add the project root to **Additional Library Directories**.
5. Under **Linker → Input → Additional Dependencies**, add:
   ```
   glut32.lib;glew32.lib;glaux.lib;opengl32.lib;glu32.lib;winmm.lib;
   ```
6. Build with `Ctrl+Shift+B` and run with `F5`.

---

## Course Context

Built for the **Computer Graphics** course in the **B.Sc. Computer Science & Engineering** programme at the **German University in Cairo (GUC)**, 2022. The project applies and extends the OpenGL fundamentals covered in the syllabus — camera math, lighting, texturing, model loading, collision detection, and game-loop timing — into a full interactive 3D experience.

---

## Authors

Anas ElNemr  ·  Ahmed Eltawel
