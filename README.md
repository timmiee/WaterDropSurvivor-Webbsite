# WaterDrop Survivor - Official Website

A website related to the 0.2-NewVersion alpha test game.

---

## 🌊 THE LORE: A Psychological Odyssey

### The Nirvana Conscious Lake
In a realm beyond mortal comprehension, there exists the **Nirvana Lake** - a body of sentient water where individual consciousness fragments (water droplets) exist in perpetual bliss. Each droplet is a soul, experiencing eternal peace and unity with the collective consciousness.

### The Annunaki: Ancient Architects
The **Annunaki** - primordial beings who shaped reality itself - created this realm as a prison and a paradise simultaneously. They are the architects of consciousness, manipulating the very fabric of existence. Their influence permeates the world, manifesting as ancient structures and corrupted energies that disrupt the natural order.

### Aida's Betrayal
**Aida**, once a guardian of the lake, committed the ultimate betrayal. Driven by forbidden knowledge and corrupted by the Annunaki's whispers, she shattered the unity of the lake. Her actions ejected countless consciousness fragments (the player being one) from Nirvana, forcing them to fight for survival in a hostile realm filled with twisted entities born from corrupted droplets.

The player - a lone water drop - must now survive wave after wave of enemies, collect essence from fallen foes, unlock ancient powers, and uncover the truth behind Aida's betrayal and the Annunaki's grand design.

---

## 🎮 ENGINE 2.0: STRICT TECHNICAL RULES

**WaterDrop Survivor** is built on **Engine 2.0** - a revolutionary, physics-first approach to game development that rejects traditional conventions. This is not your typical survivor game.

### Core Principles

#### 1. **OBJECT POOLING IS MANDATORY**
- **EVERYTHING** uses Object Pooling: enemies, projectiles, blood particles, corpses, VFX, UI elements
- Maximum pool sizes enforced to prevent memory leaks
- Example: Corpse pool limited to 200 objects, recycling oldest when limit reached
- No dynamic memory allocation during gameplay - all objects pre-allocated at initialization

#### 2. **NO PRE-RENDERED ANIMATIONS**
- **ZERO sprite sheets allowed**
- **ZERO pre-baked animations**
- Every frame is computed in real-time using hardcoded pixel-physics
- Blood splatters are calculated per-pixel using canvas 2D context and real physics
- Enemy deformations (flattening, crushing, piercing) are mathematically computed in real-time
- Weapon-specific impact animations are procedurally generated based on force vectors

#### 3. **HARDCODED CANVAS TEXTURES**
- Ground textures are drawn pixel-by-pixel on Canvas 2D context
- Grass strokes, dirt patterns, and terrain details are algorithmically generated
- `texture.needsUpdate = true` MUST be called AFTER drawing operations complete
- Use MeshBasicMaterial for canvas textures to ensure visibility without lighting dependencies

#### 4. **BLOOD V2 PHYSICS SYSTEM**
- Blood is a living, physics-based system with:
  - Real-time fluid dynamics simulation
  - Particle pooling (max 5000 particles)
  - Persistent blood pools that remain 10-15 seconds
  - Z-fighting prevention via `depthWrite: false`
- Blood V2 MUST bypass "Low Graphics" mode limiters
- Update function MUST be called every frame in `requestAnimationFrame` loop

#### 5. **STRICT PERFORMANCE OPTIMIZATION**
- Object Pooling prevents garbage collection stutters
- Batch rendering for all particles and effects
- Culling systems for off-screen entities
- Maximum entity limits enforced at engine level
- No memory allocations in hot paths (game loop, physics updates)

#### 6. **DEEPLY CUSTOMIZED UI**
- 3D button elements rendered in Three.js scene
- Custom scroll boxes with physics-based momentum
- No standard HTML UI overlays in game view
- Camp Theme settings menu with advanced graphics controls:
  - **Auto Mode**: Adaptive graphics based on performance
  - **Manual Mode**: Forces all particle systems ON, overrides limiters

#### 7. **LORE-INTEGRATED MECHANICS**
- Every game system ties back to the narrative
- Annunaki Obelisks affect AI pathfinding and player health
- Lore Lake creates water physics simulation (Y-position sink, speed reduction)
- Codex entries unlock as player discovers lore fragments
- Skill Tree gates certain features (e.g., Auto-Aim requires unlocking)

---

## 🔧 CRITICAL ENGINE 2.0 IMPLEMENTATION NOTES

### For Developers Working on Game Code:

**This repository contains the promotional WEBSITE only.** The actual game engine code must be implemented separately. When implementing Engine 2.0 features:

1. **Settings Menu Collision Fix**: Remove legacy event listeners on settings/escape that trigger old pause menu
2. **Ground Texture**: Ensure `groundTexture.needsUpdate = true` after canvas drawing, use MeshBasicMaterial if lighting issues occur
3. **Blood V2**: Call update function in main loop, set `depthWrite: false` on blood materials
4. **Spawn Hole & Elevator**: Dark cylinder at Y<0, elevator raises player from shadow to light, doors close when player reaches Y=0
5. **Combat**: Remove HP bars, add weapon-specific physics animations, corpse pool persists 10-15s
6. **UI**: Build 3D Camp UI with Auto/Manual graphics modes, lock Auto-Aim behind skill tree
7. **Codex**: Golden Pyramid with Black Eye of Horus icon, glows when new entries available
8. **World**: Medium-large map with 4 regions, Lore Lake (water physics), Annunaki Obelisk (corrupted particles, AI disruption)

---

## 🌐 Website Development

This repository contains the official **WaterDrop Survivor** promotional website built with:
- Single-file HTML architecture (index.html)
- Embedded CSS with custom theming system
- Tab-based SPA navigation
- Responsive design with mobile support
- Dark atmospheric theme matching game aesthetics

The website showcases game features, lore, mechanics, upgrade systems, and provides player account functionality.

---

## 📞 Contact & Support

**WaterDrop Survivor** is a solo-developed passion project. For bug reports, feedback, or collaboration inquiries, please reach out through the website's contact form or community channels.

**Remember**: This is not a simple survival game. This is a psychological odyssey where consciousness, physics, and rage collide.
