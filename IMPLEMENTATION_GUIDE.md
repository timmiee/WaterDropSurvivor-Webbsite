# ENGINE 2.0 IMPLEMENTATION GUIDE
## WaterDrop Survivor - Critical Mission Specifications

---

## ⚠️ REPOSITORY ARCHITECTURE NOTICE

**THIS REPOSITORY CONTAINS THE PROMOTIONAL WEBSITE ONLY.**

- **Location:** `timmiee/WaterDropSurvivor-Webbsite`
- **Contents:** Single-file HTML website (`index.html`), README, image assets
- **Does NOT contain:** Game engine code, Three.js implementation, physics systems, canvas rendering

**The actual game code must be implemented in a separate repository.**

---

## 🎯 MISSION OVERVIEW

This document provides comprehensive specifications for the Engine 2.0 overhaul of WaterDrop Survivor. These features must be implemented in the **actual game engine codebase** (not this website repository).

---

## STEP 1: ROOT CAUSE ANALYSIS & CONFLICT REMOVAL (MANDATORY)

### 1.1 Settings Menu Collision Fix

**Problem:** Legacy event listeners interfere with new Camp Theme UI.

**Solution:**
```javascript
// REMOVE or override legacy code:
// - Click handlers on settings button that trigger old pause menu
// - Keydown handlers on Escape key that show "Play" button
// - Any event.stopPropagation() calls that prevent new UI from receiving events

// IMPLEMENT:
function openCampThemeUI() {
    // Remove all legacy listeners first
    document.removeEventListener('keydown', legacyEscapeHandler);
    settingsButton.removeEventListener('click', legacySettingsHandler);

    // Open new Camp Theme UI
    campThemeUI.show();
}
```

### 1.2 Ground Texture Bug Fix

**Problem:** Hardcoded Canvas texture renders brown/incorrect colors.

**Root Cause:** `texture.needsUpdate` not called after 2D context drawing, OR using MeshStandardMaterial without adequate lighting.

**Solution:**
```javascript
// Create canvas texture
const canvas = document.createElement('canvas');
const ctx = canvas.getContext('2d');

// Draw grass strokes pixel-by-pixel
for (let i = 0; i < grassStrokeCount; i++) {
    ctx.strokeStyle = `rgb(${baseGreen}, ${variance}, ${baseBlue})`;
    ctx.beginPath();
    ctx.moveTo(x1, y1);
    ctx.lineTo(x2, y2);
    ctx.stroke();
}

// CRITICAL: Call needsUpdate AFTER all drawing is complete
const groundTexture = new THREE.CanvasTexture(canvas);
groundTexture.needsUpdate = true;  // ← MUST be after drawing

// Use MeshBasicMaterial to ensure visibility without lighting
const groundMaterial = new THREE.MeshBasicMaterial({
    map: groundTexture,
    side: THREE.DoubleSide
});

// OR if using MeshStandardMaterial, ensure adequate lighting:
const ambientLight = new THREE.AmbientLight(0xffffff, 0.8);
const directionalLight = new THREE.DirectionalLight(0xffffff, 0.6);
```

### 1.3 Blood V2 Blocker Fixes

**Problem:** Blood V2 not rendering, aggressive "Low Graphics" mode limiting particles.

**Solution:**
```javascript
// Main game loop
function gameLoop() {
    requestAnimationFrame(gameLoop);

    // CRITICAL: Blood V2 update MUST be called every frame
    bloodV2System.update(deltaTime);

    // Bypass Low Graphics limiters - force Blood V2 to render
    if (settingsMode === 'manual' || forceAllParticles) {
        bloodV2System.setParticleLimit(5000);  // Max particles
        bloodV2System.setRenderMode('full');   // Full fidelity
    }

    renderer.render(scene, camera);
}

// Blood material - fix Z-fighting
const bloodMaterial = new THREE.MeshBasicMaterial({
    color: 0x8B0000,
    transparent: true,
    opacity: 0.8,
    depthWrite: false,  // ← CRITICAL: Prevents Z-fighting
    depthTest: true
});
```

---

## STEP 2: THE SPAWN HOLE & ELEVATOR (FLAWLESS EXECUTION)

### 2.1 Door Translation Fix

**Problem:** Spawn doors open too far outside the hole.

**Solution:**
```javascript
// Calculate exact door positions relative to hole edges
const HOLE_RADIUS = 5.0;  // meters
const DOOR_WIDTH = 2.0;
const DOOR_THICKNESS = 0.2;

// Door positions when CLOSED (covering hole)
const door1ClosedPos = { x: -HOLE_RADIUS/2, z: 0 };
const door2ClosedPos = { x: HOLE_RADIUS/2, z: 0 };

// Door positions when OPEN (exactly at hole edges)
const door1OpenPos = { x: -HOLE_RADIUS - DOOR_WIDTH/2, z: 0 };
const door2OpenPos = { x: HOLE_RADIUS + DOOR_WIDTH/2, z: 0 };

// Animate doors opening
function openSpawnDoors() {
    gsap.to(door1.position, {
        x: door1OpenPos.x,
        duration: 2.0,
        ease: 'power2.out'
    });
    gsap.to(door2.position, {
        x: door2OpenPos.x,
        duration: 2.0,
        ease: 'power2.out'
    });
}
```

### 2.2 The Depth Illusion

**Create underground hole representation:**
```javascript
// Dark cylinder going down into Y < 0
const holeGeometry = new THREE.CylinderGeometry(
    HOLE_RADIUS,      // top radius
    HOLE_RADIUS,      // bottom radius
    20,               // height (extends deep underground)
    32                // radial segments
);

const holeMaterial = new THREE.MeshBasicMaterial({
    color: 0x000000,
    side: THREE.BackSide  // Render inside of cylinder
});

const holeCylinder = new THREE.Mesh(holeGeometry, holeMaterial);
holeCylinder.position.y = -10;  // Center at Y=-10 (going from Y=0 to Y=-20)
holeCylinder.castShadow = true;
scene.add(holeCylinder);
```

### 2.3 The Cinematic Sequence

**Implementation:**
```javascript
function startGameSequence() {
    // Player starts deep in darkness
    player.position.y = -15;
    camera.position.y = -14;

    // Elevator platform
    const elevatorGeometry = new THREE.CylinderGeometry(3, 3, 0.5, 32);
    const elevatorMaterial = new THREE.MeshStandardMaterial({
        color: 0x888888,
        metalness: 0.7,
        roughness: 0.3
    });
    const elevator = new THREE.Mesh(elevatorGeometry, elevatorMaterial);
    elevator.position.y = -15;
    scene.add(elevator);

    // Lighting setup - dark at bottom, light at top
    const bottomLight = new THREE.PointLight(0x330000, 0.1, 10);
    bottomLight.position.y = -15;
    const topLight = new THREE.DirectionalLight(0xffffee, 1.0);
    topLight.position.set(0, 10, 5);

    // Animate elevator rising
    gsap.timeline()
        .to([player.position, elevator.position, camera.position], {
            y: 0,
            duration: 5.0,
            ease: 'power1.inOut',
            onUpdate: () => {
                // Gradual light transition as player rises
                const progress = (player.position.y + 15) / 15;
                bottomLight.intensity = 0.1 * (1 - progress);
                topLight.intensity = 1.0 * progress;
            }
        })
        .add(() => {
            // Player reaches surface - doors snap shut
            closeDoors();
        }, '+=0.5');
}

function closeDoors() {
    gsap.to([door1.position, door2.position], {
        x: 0,  // Both doors move to center
        duration: 0.8,
        ease: 'power2.in',
        onComplete: () => {
            // Doors locked, game begins
            enablePlayerControl();
        }
    });
}
```

---

## STEP 3: COMBAT, GORE & ENEMY POOLING

### 3.1 Remove HP Bars

**Implementation:**
```javascript
// REMOVE all HP bar rendering code
// Search for and delete:
// - Canvas context operations drawing bars above enemies
// - Three.js Sprite/Plane meshes for HP display
// - Any UI elements showing enemy health

// Example code to DELETE:
// ❌ ctx.fillRect(enemy.x - 20, enemy.y - 40, 40 * (enemy.hp / enemy.maxHp), 4);
// ❌ hpBarSprite.scale.x = enemy.hp / enemy.maxHp;
```

### 3.2 Hit & Kill Physics

**Fix flattening bug and add weapon-specific animations:**
```javascript
class Enemy {
    onHit(weapon, damage, impactVector) {
        this.hp -= damage;

        if (this.hp <= 0) {
            this.die(weapon, impactVector);
        } else {
            this.playHitAnimation(weapon, impactVector);
        }
    }

    die(weapon, impactVector) {
        // Weapon-specific death animations
        switch(weapon.type) {
            case 'BULLET':
                this.pierceDeath(impactVector);
                break;
            case 'BLUNT':
                this.crushDeath(impactVector);
                break;
            case 'EXPLOSIVE':
                this.splatterDeath(impactVector);
                break;
        }

        // Create corpse that persists
        const corpse = corpsePool.acquire();
        corpse.initFromEnemy(this, weapon.type);
        corpse.lifespan = 10 + Math.random() * 5; // 10-15 seconds
    }

    pierceDeath(impactVector) {
        // Bullet pierces through - minimal deformation
        this.mesh.scale.z *= 0.3;  // Flatten in impact direction
        this.velocity.add(impactVector.multiplyScalar(5));
        this.createBloodSpray(impactVector, 'pierce');
    }

    crushDeath(impactVector) {
        // Blunt weapon crushes - significant deformation
        gsap.to(this.mesh.scale, {
            y: 0.1,    // Flatten vertically
            x: 1.5,    // Expand horizontally
            z: 1.5,
            duration: 0.3,
            ease: 'power2.out'
        });
        this.createBloodSpray(impactVector, 'crush');
    }

    splatterDeath(impactVector) {
        // Explosive completely destroys - spawn multiple chunks
        for (let i = 0; i < 8; i++) {
            const chunk = this.createGibChunk();
            const randomDir = new THREE.Vector3(
                Math.random() - 0.5,
                Math.random(),
                Math.random() - 0.5
            ).normalize();
            chunk.velocity = randomDir.multiplyScalar(10);
        }
        this.createBloodSpray(impactVector, 'splatter');
    }
}
```

### 3.3 Corpse Object Pool

**Implementation:**
```javascript
class CorpsePool {
    constructor(maxSize = 200) {
        this.maxSize = maxSize;
        this.pool = [];
        this.active = [];

        // Pre-allocate all corpses
        for (let i = 0; i < maxSize; i++) {
            this.pool.push(new Corpse());
        }
    }

    acquire() {
        let corpse;

        if (this.pool.length > 0) {
            // Reuse from pool
            corpse = this.pool.pop();
        } else {
            // Pool exhausted - recycle oldest active corpse
            corpse = this.active.shift();
            corpse.cleanup();
        }

        this.active.push(corpse);
        return corpse;
    }

    update(deltaTime) {
        for (let i = this.active.length - 1; i >= 0; i--) {
            const corpse = this.active[i];
            corpse.lifespan -= deltaTime;

            // Fade out in last 2 seconds
            if (corpse.lifespan < 2.0) {
                corpse.mesh.material.opacity = corpse.lifespan / 2.0;
            }

            if (corpse.lifespan <= 0) {
                // Return to pool
                this.release(corpse);
            }
        }
    }

    release(corpse) {
        const index = this.active.indexOf(corpse);
        if (index !== -1) {
            this.active.splice(index, 1);
        }
        corpse.reset();
        this.pool.push(corpse);
    }
}

// Initialize global corpse pool
const corpsePool = new CorpsePool(200);
```

---

## STEP 4: NEW UI & THE CODEX

### 4.1 Camp Theme UI

**3D Settings Menu Implementation:**
```javascript
class CampThemeUI {
    constructor(scene, camera) {
        this.scene = scene;
        this.camera = camera;
        this.buttons = [];
        this.scrollBox = null;

        this.createUI();
    }

    createUI() {
        // 3D button geometry
        const buttonGeometry = new THREE.BoxGeometry(4, 1, 0.3);

        // Auto Graphics Mode button
        const autoButton = this.createButton('Auto Graphics', buttonGeometry);
        autoButton.position.set(-5, 2, -10);
        autoButton.onClick = () => this.setGraphicsMode('auto');

        // Manual Graphics Mode button
        const manualButton = this.createButton('Manual Graphics', buttonGeometry);
        manualButton.position.set(-5, 0, -10);
        manualButton.onClick = () => this.setGraphicsMode('manual');

        // Create scroll box for detailed settings
        this.createScrollBox();
    }

    setGraphicsMode(mode) {
        if (mode === 'manual') {
            // FORCE all particles ON
            bloodV2System.setParticleLimit(5000);
            particleEmitters.forEach(e => e.enabled = true);
            lowGraphicsMode = false;
        } else {
            // Auto - adaptive based on performance
            this.enableAdaptiveGraphics();
        }
    }

    createButton(text, geometry) {
        const material = new THREE.MeshStandardMaterial({
            color: 0xF0C030,
            metalness: 0.3,
            roughness: 0.6
        });

        const button = new THREE.Mesh(geometry, material);

        // Add text label (using canvas texture)
        const textTexture = this.createTextTexture(text);
        const textMaterial = new THREE.MeshBasicMaterial({ map: textTexture });
        const textPlane = new THREE.Mesh(
            new THREE.PlaneGeometry(3.5, 0.8),
            textMaterial
        );
        textPlane.position.z = 0.16;
        button.add(textPlane);

        this.scene.add(button);
        this.buttons.push(button);

        return button;
    }

    createScrollBox() {
        // Physics-based momentum scrolling
        // Implementation details for 3D scroll box with inertia
    }
}
```

### 4.2 Auto-Aim Toggle

**Skill Tree Gated Implementation:**
```javascript
class SettingsMenu {
    updateAutoAimOption() {
        const autoAimUnlocked = playerSkillTree.hasUnlocked('AUTO_AIM_SKILL');

        if (autoAimUnlocked) {
            this.autoAimToggle.enabled = true;
            this.autoAimToggle.material.opacity = 1.0;
        } else {
            this.autoAimToggle.enabled = false;
            this.autoAimToggle.material.opacity = 0.3;

            // Show lock icon and tooltip
            this.showLockIcon(this.autoAimToggle);
            this.setTooltip(this.autoAimToggle,
                'Unlock Auto-Aim in Skill Tree first');
        }
    }
}
```

### 4.3 Codex System

**Golden Pyramid with Eye of Horus:**
```javascript
class CodexButton {
    constructor(scene) {
        this.scene = scene;
        this.hasNewEntries = false;

        this.createPyramidIcon();
        this.createEyeOfHorus();
        this.createGlowEffect();
    }

    createPyramidIcon() {
        // Golden pyramid geometry
        const pyramidGeometry = new THREE.ConeGeometry(0.5, 1, 4);
        const pyramidMaterial = new THREE.MeshStandardMaterial({
            color: 0xFFD700,  // Gold
            metalness: 0.8,
            roughness: 0.2,
            emissive: 0xFFD700,
            emissiveIntensity: 0.0  // Will animate when new entries
        });

        this.pyramid = new THREE.Mesh(pyramidGeometry, pyramidMaterial);
        this.pyramid.rotation.y = Math.PI / 4;
        this.pyramid.position.set(8, 4, -10);
        this.scene.add(this.pyramid);
    }

    createEyeOfHorus() {
        // Black Eye of Horus texture on pyramid face
        const eyeCanvas = document.createElement('canvas');
        const ctx = eyeCanvas.getContext('2d');
        eyeCanvas.width = 256;
        eyeCanvas.height = 256;

        // Draw Eye of Horus (simplified)
        ctx.fillStyle = '#000000';
        ctx.beginPath();
        // ... draw eye shape
        ctx.fill();

        const eyeTexture = new THREE.CanvasTexture(eyeCanvas);
        eyeTexture.needsUpdate = true;

        const eyePlane = new THREE.Mesh(
            new THREE.PlaneGeometry(0.3, 0.3),
            new THREE.MeshBasicMaterial({
                map: eyeTexture,
                transparent: true
            })
        );
        eyePlane.position.set(0, 0.2, 0.4);
        this.pyramid.add(eyePlane);
    }

    createGlowEffect() {
        // Glow when new entries available
        const glowGeometry = new THREE.SphereGeometry(0.7, 16, 16);
        const glowMaterial = new THREE.MeshBasicMaterial({
            color: 0xFFD700,
            transparent: true,
            opacity: 0.0
        });

        this.glow = new THREE.Mesh(glowGeometry, glowMaterial);
        this.pyramid.add(this.glow);
    }

    setNewEntryNotification(hasNew) {
        this.hasNewEntries = hasNew;

        if (hasNew) {
            // Animate golden glow
            gsap.to(this.glow.material, {
                opacity: 0.4,
                duration: 1.0,
                repeat: -1,
                yoyo: true
            });
            gsap.to(this.pyramid.material, {
                emissiveIntensity: 0.5,
                duration: 1.0,
                repeat: -1,
                yoyo: true
            });
        } else {
            gsap.killTweensOf(this.glow.material);
            gsap.killTweensOf(this.pyramid.material);
            this.glow.material.opacity = 0.0;
            this.pyramid.material.emissiveIntensity = 0.0;
        }
    }
}
```

---

## STEP 5: THE WORLD TWEAKS & NEW LORE ENTITIES

### 5.1 Map Size & Boundaries

**Implementation:**
```javascript
const MAP_CONFIG = {
    size: 'medium-large',
    width: 200,   // meters
    height: 200,
    regions: [
        { name: 'spawn', center: { x: 0, z: 0 } },
        { name: 'lore_lake', center: { x: -80, z: -80 } },
        { name: 'annunaki_obelisk', center: { x: 80, z: 80 } },
        { name: 'combat_arena', center: { x: 80, z: -80 } },
        { name: 'sanctuary', center: { x: -80, z: 80 } }
    ]
};

// Invisible boundaries
const boundaryMaterial = new THREE.MeshBasicMaterial({
    visible: false,
    transparent: true,
    opacity: 0
});

const boundaries = [
    new THREE.Mesh(new THREE.PlaneGeometry(200, 1), boundaryMaterial),  // North
    new THREE.Mesh(new THREE.PlaneGeometry(200, 1), boundaryMaterial),  // South
    new THREE.Mesh(new THREE.PlaneGeometry(1, 200), boundaryMaterial),  // East
    new THREE.Mesh(new THREE.PlaneGeometry(1, 200), boundaryMaterial)   // West
];
```

### 5.2 The Lore Lake

**Water Physics Implementation:**
```javascript
class LoreLake {
    constructor(scene, center) {
        this.center = center;
        this.radius = 15;
        this.waterLevel = 0;

        this.createLake();
        this.createWaterLilies();
    }

    createLake() {
        // Lake surface
        const lakeGeometry = new THREE.CircleGeometry(this.radius, 64);
        const lakeMaterial = new THREE.MeshStandardMaterial({
            color: 0x1E88E5,
            transparent: true,
            opacity: 0.7,
            metalness: 0.9,
            roughness: 0.1
        });

        const lakeSurface = new THREE.Mesh(lakeGeometry, lakeMaterial);
        lakeSurface.rotation.x = -Math.PI / 2;
        lakeSurface.position.set(this.center.x, 0, this.center.z);
        scene.add(lakeSurface);
    }

    createWaterLilies() {
        const lilyCount = 12;
        for (let i = 0; i < lilyCount; i++) {
            const angle = (Math.PI * 2 * i) / lilyCount;
            const dist = this.radius * (0.6 + Math.random() * 0.3);

            const lily = this.createLily();
            lily.position.set(
                this.center.x + Math.cos(angle) * dist,
                0.05,
                this.center.z + Math.sin(angle) * dist
            );
            scene.add(lily);
        }
    }

    createLily() {
        const lilyGeometry = new THREE.CircleGeometry(0.5, 16);
        const lilyMaterial = new THREE.MeshStandardMaterial({
            color: 0x4CAF50,
            side: THREE.DoubleSide
        });
        return new THREE.Mesh(lilyGeometry, lilyMaterial);
    }

    updatePlayer(player, deltaTime) {
        const distFromCenter = Math.sqrt(
            Math.pow(player.position.x - this.center.x, 2) +
            Math.pow(player.position.z - this.center.z, 2)
        );

        if (distFromCenter < this.radius) {
            // Player is in water
            // Sink Y-position so only head visible
            const sinkDepth = -1.5;  // Player height is ~2m, sink 1.5m
            player.position.y = THREE.MathUtils.lerp(
                player.position.y,
                sinkDepth,
                deltaTime * 3.0
            );

            // Drastically reduce movement speed
            player.movementSpeed = player.baseSpeed * 0.2;  // 80% slower

            // Visual feedback - ripples, slower animation
            player.animationSpeed *= 0.3;
            this.createRipples(player.position);
        } else {
            // Restore normal state
            player.position.y = THREE.MathUtils.lerp(
                player.position.y,
                0,
                deltaTime * 2.0
            );
            player.movementSpeed = player.baseSpeed;
            player.animationSpeed = 1.0;
        }
    }
}
```

### 5.3 The Annunaki Weeping Obelisk

**Implementation:**
```javascript
class AnnunakiObelisk {
    constructor(scene, center) {
        this.center = center;
        this.height = 20;
        this.width = 3;
        this.corruptionRadius = 10;
        this.damagePerSecond = 5;

        this.createObelisk();
        this.createCorruptedParticles();
    }

    createObelisk() {
        // Massive black obsidian pillar
        const obeliskGeometry = new THREE.BoxGeometry(
            this.width,
            this.height,
            this.width
        );
        const obeliskMaterial = new THREE.MeshStandardMaterial({
            color: 0x0A0A0A,
            metalness: 0.9,
            roughness: 0.1,
            emissive: 0x330000,
            emissiveIntensity: 0.2
        });

        this.obelisk = new THREE.Mesh(obeliskGeometry, obeliskMaterial);
        this.obelisk.position.set(
            this.center.x,
            this.height / 2,
            this.center.z
        );
        this.obelisk.castShadow = true;
        scene.add(this.obelisk);

        // Ancient glyphs (using canvas texture)
        this.addGlyphs();
    }

    createCorruptedParticles() {
        // Object pooled corrupted particles
        this.particlePool = new ObjectPool(500);

        // Dripping effect - spawn particles at top
        this.dripTimer = 0;
        this.dripInterval = 0.1; // 10 particles per second
    }

    update(deltaTime, enemies, player) {
        // Spawn dripping corrupted particles
        this.dripTimer += deltaTime;
        if (this.dripTimer >= this.dripInterval) {
            this.dripTimer = 0;
            this.spawnCorruptedDrip();
        }

        // Update all active particles
        this.particlePool.updateActive(deltaTime);

        // Affect nearby enemies - slow them down
        enemies.forEach(enemy => {
            const distToObelisk = enemy.position.distanceTo(
                this.obelisk.position
            );

            if (distToObelisk < this.corruptionRadius) {
                // AI disruption - reduce speed
                const slowFactor = 1 - (distToObelisk / this.corruptionRadius);
                enemy.speed *= (1 - slowFactor * 0.7);  // Up to 70% slower

                // Visual corruption effect on enemy
                enemy.mesh.material.emissive.setHex(0x330000);
                enemy.mesh.material.emissiveIntensity = slowFactor * 0.5;
            }
        });

        // Damage player if too close
        const playerDist = player.position.distanceTo(this.obelisk.position);
        if (playerDist < this.corruptionRadius) {
            const damageFactor = 1 - (playerDist / this.corruptionRadius);
            const damage = this.damagePerSecond * damageFactor * deltaTime;
            player.takeDamage(damage);

            // Visual feedback - red vignette
            this.showCorruptionEffect(damageFactor);
        }
    }

    spawnCorruptedDrip() {
        const particle = this.particlePool.acquire();

        // Start at top of obelisk
        particle.position.set(
            this.center.x + (Math.random() - 0.5) * this.width,
            this.height,
            this.center.z + (Math.random() - 0.5) * this.width
        );

        // Red glowing particle
        particle.color = 0xFF0000;
        particle.velocity.y = -2.0;  // Fall down
        particle.lifetime = 10.0;    // 10 second lifetime

        // Glow effect
        particle.emissiveIntensity = 0.8;
    }
}
```

---

## 🔧 OBJECT POOLING IMPLEMENTATION

**Generic Object Pool Class:**
```javascript
class ObjectPool {
    constructor(maxSize, ObjectClass) {
        this.maxSize = maxSize;
        this.ObjectClass = ObjectClass;
        this.pool = [];
        this.active = [];

        // Pre-allocate all objects
        for (let i = 0; i < maxSize; i++) {
            this.pool.push(new ObjectClass());
        }
    }

    acquire() {
        let obj;

        if (this.pool.length > 0) {
            obj = this.pool.pop();
        } else {
            // Pool exhausted - recycle oldest
            obj = this.active.shift();
            obj.reset();
        }

        this.active.push(obj);
        return obj;
    }

    release(obj) {
        const index = this.active.indexOf(obj);
        if (index !== -1) {
            this.active.splice(index, 1);
        }
        obj.reset();
        this.pool.push(obj);
    }

    updateActive(deltaTime) {
        for (let i = this.active.length - 1; i >= 0; i--) {
            const obj = this.active[i];
            obj.update(deltaTime);

            if (obj.shouldRemove()) {
                this.release(obj);
            }
        }
    }
}

// Usage examples:
const enemyPool = new ObjectPool(100, Enemy);
const projectilePool = new ObjectPool(500, Projectile);
const bloodParticlePool = new ObjectPool(5000, BloodParticle);
const corpsePool = new ObjectPool(200, Corpse);
const vfxPool = new ObjectPool(300, VisualEffect);
```

---

## 📝 TESTING CHECKLIST

When implementing these features in the actual game code, verify:

- [ ] Settings menu opens Camp Theme UI without legacy conflicts
- [ ] Ground texture renders correct colors (green grass visible)
- [ ] Blood V2 particles render every frame, persist 10-15s
- [ ] Spawn sequence: player rises from darkness, doors close
- [ ] Enemy HP bars completely removed
- [ ] Weapon-specific death animations working (pierce/crush/splatter)
- [ ] Corpse pool maintains max 200, recycles oldest
- [ ] Camp UI renders in 3D with working buttons
- [ ] Auto-Aim locked unless skill tree unlocked
- [ ] Codex pyramid glows gold when new entries exist
- [ ] Lore Lake slows player, sinks Y-position
- [ ] Annunaki Obelisk damages player, slows enemies
- [ ] All systems use Object Pooling (zero dynamic allocation)

---

## 🚀 DEPLOYMENT NOTES

1. **This guide documents features for the GAME ENGINE, not the website**
2. **Game code must be in a separate repository**
3. **Website has been updated with lore and roadmap information**
4. **README.md contains comprehensive Engine 2.0 rules**
5. **All specifications are exact and ready for implementation**

---

**Status:** Documentation complete. Ready for game engine implementation.
**Last Updated:** March 2025
**Version:** Engine 2.0 Specifications v1.0
