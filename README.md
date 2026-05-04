# Vampire Buster — 3D OpenGL Action Game in C++

A first-person / third-person 3D vampire-hunting game built directly on the fixed-function OpenGL pipeline. The player roams a closed urban arena populated with vampires (called "creatures" in code), picks up melee weapons (a stick, then upgradeable blades), and clicks to kill enemies up close. Built with GLUT + GLEW + GLAUX on Windows, using a hand-written 3DS model loader for all geometry. Coursework project for **Computer Graphics (CSEN 471)** at the **German University in Cairo (GUC)**, ~2022.

> Honest note: this is the "Final commit" snapshot from the course. It boots, renders the world, lets you walk around in first/third person, swap weapons by walking onto pickups, and click-kill enemies across two levels — but it carries a lot of the usual coursework rough edges (debug `cout` spam every frame, hard-coded model paths, partially-wired keyboard, no Makefile). Nothing has been cleaned up post-submission.

---

## Gameplay

- **You play** a roaming hunter in a small open-world city block, viewed in either first-person or third-person camera.
- **Two levels**, gated by progress:
  - **Level 1** — three "creature2" vampires are scattered around the arena (`creature21`, `creature22`, `creature23`). Walk up to each one and **left-click** to one-shot it. Each kill plays `Sounds/VimpireDeath.wav`.
  - **Level 2** — kicks in automatically when all three Level 1 vampires are dead. A larger boss "creature1" with **3 HP** spawns. Walk up and click three times to kill it. Each hit plays `Sounds/MonsterDeath.wav`. A "leveling up" jingle (`Sounds/UptoLevel2.wav`) plays on the transition.
- **Weapons** — start with a stick. Two blade pickups (`blade.3ds`, `blade2.3ds`) sit on the map; walking onto either swaps your held weapon. Pickup state is one-way (no putting the stick back).
- **Win condition** — the boss dies. Triggers `Sounds/VictorySound.wav`.
- **Lose condition** — a 5-minute timer (`glutTimerFunc(5*60*1000, checkWin, 0)`); if the boss isn't dead by then, `Sounds/GameOver.wav` plays.
- **Collision** — the player can't walk through cars (`oldcar1`, `oldcar2`), the van, or out of three predefined `BoundingBox` regions covering the playable streets. Walking into the small `oldcar1` topples it (purely cosmetic — flips 90° and lies on its side).
- **Day/night cycle** — a sun light source (`GL_LIGHT0`) sweeps left-to-right across the sky and tints diffuse colour over time via a 500 ms timer (`timeSunMovement`).

### Controls

| Input | Action |
|---|---|
| Mouse move | Rotate player / camera (yaw only — mouse pointer is warped back to centre each frame) |
| `W` | Walk forward (in the direction you're facing) |
| `Space` | Toggle first-person ↔ third-person camera |
| `T` | Toggle top-down view |
| **Left mouse click** | Attack (kills any enemy whose bounding box contains the player) |
| Arrow keys | Manual camera pitch / yaw nudges (`rotateX` / `rotateY`) |
| Numpad `4 5 6 7 8 9` | Free-fly camera translate (debug) |
| `J K L U I O ; P` | Move debug marker / rotate debug value (developer only) |
| `M` | Cycle which debug marker is active |
| `Esc` | Quit |

`A`, `S`, `D` are wired into the keyboard switch but the cases are empty — only `W` actually moves you. Strafing and walking backward weren't implemented.

---

## Rendering

Pure **fixed-function / immediate-mode OpenGL** — no shaders, no VBOs, no VAOs. Everything goes through the OpenGL 1.x pipeline that the course covered.

- **Geometry**: every model in the game (player, vampires, cars, weapons, environment) is a **`.3DS` (Autodesk 3D Studio) file** loaded by Matthew Fairfax's `Model_3DS` C++ class (`Model_3DS.cpp` / `.h`) — a self-contained 3DS chunk-parser that reads vertices, faces, UVs, and material chunks straight from the binary file and stores them in plain arrays. `Model_3DS::Draw()` walks each object and pumps triangles through `glBegin(GL_TRIANGLES) … glEnd()`.
- **Ground**: a single textured `GL_QUADS` quad spanning `(-200,0,-200)` → `(200,0,200)` with `GL_REPEAT` UVs (5×5 tile factor) — see `RenderGround()`.
- **Skybox**: a `gluSphere` quadric of radius 250 with a sky bitmap mapped via `gluQuadricTexture`.
- **Lighting**: three `GL_LIGHT0/1/2` enabled, but only `GL_LIGHT0` is animated (the sun). `glColorMaterial(GL_FRONT, GL_AMBIENT_AND_DIFFUSE)` + `GL_NORMALIZE` + `GL_SMOOTH` shading.
- **Camera**: a hand-rolled `Camera` class with `eye`, `center`, `up` vectors and `gluLookAt`-driven view; switches between first-person (radius `0.1` from player) and third-person (radius `20`).
- **Projection**: `gluPerspective(60°, w/h, 0.001, 5000)`, set in `setupCamera()` every frame.
- **Textures**: BMPs decoded with the **GLAUX** library (`auxDIBImageLoadA`) and uploaded via `gluBuild2DMipmaps` — see `TextureBuilder.h::loadBMP` and `GLTexture.cpp`. There's also a PPM loader (`loadPPM`) and a TGA loader on the GLTexture class, neither of which the game actually uses.
- **Buffering**: `GLUT_DOUBLE | GLUT_DEPTH | GLUT_RGBA`, swapped via `glutSwapBuffers()`.
- **HUD**: minimal — `glRasterPos3f` + `glutBitmapCharacter(GLUT_BITMAP_TIMES_ROMAN_24, ...)` for any on-screen text (most calls are commented out).
- **Collision**: pure CPU AABB tests via the `BoundingBox` class — no physics library.

---

## Tech Stack

- **Language**: C++ (compiled as a Win32 console + GLUT app; uses `iostream`, `vector`, `deque`, `cmath`).
- **Graphics**: **OpenGL 1.x** fixed-function pipeline. **GLEW** is included for symbol loading on Windows, but no >1.x extensions are actually called.
- **Window / input**: **GLUT** (`glut.h` + `glut32.lib`). `glutMainLoop` drives the frame loop; `glutTimerFunc` powers the day/night animation, the level-up jingle delay, and the 5-minute lose timer.
- **Image loading**: **GLAUX** (`glaux.h` + `glaux.lib`) for `auxDIBImageLoadA`. No `stb_image` / SOIL / DevIL.
- **3D model loading**: **Matthew Fairfax's `Model_3DS`** (in-tree, `Model_3DS.cpp`) — bespoke 3DS chunk parser, MIT-style header attribution.
- **Texture wrapper**: **Matthew Fairfax's `GLTexture`** (in-tree, `GLTexture.cpp`) — BMP/TGA loader with mipmaps.
- **Sound**: **Win32 `PlaySound`** from `mmsystem.h` (`#include <Windows.h>` + `<mmsystem.h>`). All clips are `.wav` files in a sibling `Sounds/` directory, played with `SND_FILENAME | SND_ASYNC` (or `SND_SYNC` for the lose-state Game Over jingle).
- **Platform**: **Windows-only**. The code calls `MessageBoxA`, includes `Windows.h`, and uses `fopen_s`. Building on Linux/macOS would require swapping out GLAUX (deprecated by Microsoft) and the `PlaySound` audio path.

---

## Project Structure

| File | Role |
|---|---|
| `OpenGLMeshLoader19.cpp` | **Main game file** (1 421 lines). Holds `main()`, the GLUT callbacks (`myDisplay`, `myKeyboard`, `keySpecial`, `myMouse`, `myReshape`, `ReadMouseMotion`), the `Vector3f` / `BoundingBox` / `DrawableObject` / `Camera` / `Marker` classes, all model declarations, all per-frame draw calls, the collision-detection function, and both timer callbacks (sun motion, lose-condition timeout). |
| `Model_3DS.h` / `.cpp` | Matthew Fairfax's third-party 3DS file loader. Parses Autodesk `.3DS` chunks (vertices, faces, UVs, materials, textures) and renders via immediate-mode `glBegin(GL_TRIANGLES)`. |
| `GLTexture.h` / `.cpp` | Matthew Fairfax's third-party texture wrapper. Loads BMP / TGA from disk or Visual Studio resources, builds mipmaps, exposes `Use()` to bind. |
| `TextureBuilder.h` | Standalone helpers: `loadBMP` (GLAUX-based) and `loadPPM`. The skybox texture goes through `loadBMP`. |
| `glut.h`, `glut32.lib` | Vendored GLUT headers + import library. |
| `glew.h`, `glew32.lib` | Vendored GLEW. |
| `glaux.h`, `glaux.lib` | Vendored Microsoft GLAUX (deprecated since the early 2000s; still works on Windows). |
| `ReadMe.txt` | Auto-generated Visual Studio "AppWizard" placeholder — not the game's actual readme. |

> **Asset folders are NOT in the repo.** The game expects sibling directories `Models/` (containing `stick/L1.3DS`, `player/player.3ds`, `creature1/`, `creature2/`, `oldcar1/`, `oldcar2/`, `van/`, `blade/`, `blade2/`, `finalb2/`, `finalb3/`, `garbagebags/`), `Textures/` (containing `ground.bmp`, `ground2.bmp`, `blu-sky-3.bmp`), and `Sounds/` (`VictorySound.wav`, `UptoLevel2.wav`, `VimpireDeath.wav`, `MonsterDeath.wav`, `GameOver.wav`). These were submitted separately via the GUC course portal and aren't redistributed here. Without them, `LoadAssets()` will exit on the first missing file (the BMP loader pops a Win32 `MessageBoxA` and calls `exit(EXIT_FAILURE)`).

---

## How to Build & Run

The project was developed in **Visual Studio 2019** on Windows as a standard C++ Win32 console application linked against GLUT/GLEW/GLAUX. There's no `Makefile` or `.vcxproj` checked in (only the `ReadMe.txt` boilerplate references `OpenGL3DTemplate.vcxproj`, which isn't shipped), so you'll need to recreate the project.

**Recommended (Visual Studio):**

1. Open VS 2019/2022 → **Create a new project** → **Empty Project** (C++ / Windows / Console).
2. Drop the repo files into the project folder. Add `OpenGLMeshLoader19.cpp`, `Model_3DS.cpp`, `GLTexture.cpp` to **Source Files**, and the `.h` files to **Header Files**.
3. Project Properties → **C/C++ → General → Additional Include Directories** → add the project root (so `glut.h`, `glew.h`, `glaux.h` resolve).
4. **Linker → General → Additional Library Directories** → also the project root.
5. **Linker → Input → Additional Dependencies** → add `glut32.lib;glew32.lib;glaux.lib;opengl32.lib;glu32.lib;winmm.lib;` (the `winmm.lib` covers `PlaySound`; the other three `.lib` files are vendored next to the headers).
6. Drop `glut32.dll` (and `glew32.dll` if you have it) next to the eventual `.exe` — or into `C:\Windows\System32` — so the game can launch.
7. Place the asset folders (`Models/`, `Textures/`, `Sounds/`) next to the `.exe` (or in the working directory you run from).
8. Build (`Ctrl+Shift+B`) and run (`F5`).

**Command-line (MSVC `cl.exe`, from a Developer Command Prompt):**

```bat
cl /EHsc /I. OpenGLMeshLoader19.cpp Model_3DS.cpp GLTexture.cpp ^
   /link /LIBPATH:. glut32.lib glew32.lib glaux.lib opengl32.lib glu32.lib winmm.lib
```

**Linux / macOS:** would need a port. GLAUX has no modern equivalent (replace with `stb_image` and a hand-written BMP path), and `PlaySound` would need to be swapped for SDL_mixer / OpenAL / `aplay`. The fixed-function GL calls themselves still work on Linux through Mesa, and on macOS up through 10.14 via the legacy GL profile.

---

## Coursework Context

Built for **Computer Graphics** at the **German University in Cairo (GUC)**, ~2022. The project brief was open-ended ("apply the OpenGL fixed-function pipeline you learned in lectures to a small interactive 3D scene"), and we picked a vampire-hunting concept so we could exercise:

- The **camera math** (eye/center/up, switching projection setups, swapping first/third person).
- **Loading and animating** non-trivial 3D models via the 3DS chunk format (rather than hand-coded cubes).
- **Lighting** (animated directional sun, multiple `GL_LIGHT*` sources, per-material specular/shininess, day-night colour shift).
- **Texturing** (mipmapped BMPs, repeat-wrapped ground, a `gluSphere` skybox).
- **Collision detection** with axis-aligned bounding boxes.
- **Game-loop timing** via `glutTimerFunc` for the sun cycle, the level-up jingle, and the lose-condition countdown.
- **Audio** layered on top of GL via Win32 `PlaySound` (since the syllabus didn't mandate a particular audio library).

It was deliberately scoped so two students could finish it in the semester window — hence the "click to kill at zero range" combat instead of projectile physics, and the small fixed enemy count instead of waves.

---

## Authors

- **Anas ElNemr** — [@anaselnemr](https://github.com/anaselnemr)
- **Ahmed Eltawel** — [@ahmedeltawel](https://github.com/ahmedeltawel)

---

## Acknowledgements

- Original team mirror: [github.com/ahmedeltawel/Vampire-Buster-OpenGL](https://github.com/ahmedeltawel/Vampire-Buster-OpenGL) — same codebase, hosted on Ahmed's account.
- **`Model_3DS`** and **`GLTexture`** classes by **Matthew Fairfax**, included verbatim under their original headers. They predate this project by many years and remain the workhorse of the rendering layer.
- **GLUT**, **GLEW**, and **GLAUX** import libraries are vendored in-tree at the versions our course's lab machines shipped with (Windows / Visual Studio 2019).
- The Computer Graphics teaching staff at the **German University in Cairo (GUC)** for the project brief and the lecture material that the camera, lighting, and texturing code is built directly out of.
