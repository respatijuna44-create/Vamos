<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>3D Tembak-Tembakan: Tentara vs Musuh</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: 'Courier New', monospace;
        }
        #info {
            position: absolute;
            top: 20px;
            left: 20px;
            background: rgba(0,0,0,0.7);
            color: white;
            padding: 12px 20px;
            border-radius: 8px;
            backdrop-filter: blur(5px);
            pointer-events: none;
            z-index: 10;
            font-weight: bold;
            letter-spacing: 1px;
            border-left: 4px solid #ffaa00;
            box-shadow: 0 0 15px rgba(0,0,0,0.3);
        }
        #score {
            color: #ffaa00;
            font-size: 24px;
            font-weight: bold;
        }
        #health {
            color: #ff5555;
            font-size: 24px;
            font-weight: bold;
        }
        #controls {
            position: absolute;
            bottom: 20px;
            left: 20px;
            background: rgba(0,0,0,0.6);
            color: #ccc;
            padding: 8px 15px;
            border-radius: 8px;
            font-size: 12px;
            font-family: monospace;
            pointer-events: none;
            z-index: 10;
        }
        #ammo {
            position: absolute;
            bottom: 20px;
            right: 20px;
            background: rgba(0,0,0,0.7);
            color: #fff;
            padding: 8px 15px;
            border-radius: 8px;
            font-size: 18px;
            font-family: monospace;
            font-weight: bold;
            pointer-events: none;
            z-index: 10;
            border-right: 3px solid #ffaa00;
        }
        #game-over {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(0,0,0,0.85);
            color: white;
            padding: 30px 50px;
            text-align: center;
            border-radius: 20px;
            backdrop-filter: blur(8px);
            z-index: 20;
            display: none;
            border: 2px solid #ff5555;
            font-family: monospace;
            pointer-events: auto;
        }
        #game-over h1 {
            font-size: 48px;
            margin: 0 0 10px;
            color: #ff5555;
        }
        #restart-btn {
            background: #ffaa00;
            border: none;
            padding: 12px 24px;
            font-size: 18px;
            font-weight: bold;
            margin-top: 20px;
            cursor: pointer;
            border-radius: 30px;
            font-family: monospace;
            transition: 0.2s;
        }
        #restart-btn:hover {
            background: #ffcc44;
            transform: scale(1.05);
        }
        .instruction {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0,0,0,0.5);
            color: #ddd;
            padding: 5px 12px;
            border-radius: 20px;
            font-size: 12px;
            pointer-events: none;
            white-space: nowrap;
        }
        @media (max-width: 600px) {
            #info { font-size: 12px; top: 10px; left: 10px; padding: 6px 12px; }
            #score, #health { font-size: 18px; }
            #ammo { font-size: 14px; bottom: 10px; right: 10px; }
            .instruction { font-size: 10px; }
        }
    </style>
</head>
<body>
    <div id="info">
        🔫 SOLDIER ASSAULT<br>
        Score: <span id="score">0</span> &nbsp;|&nbsp;
        ❤️ Health: <span id="health">100</span>
    </div>
    <div id="ammo">
        🔫 AMMO: <span id="ammo-count">∞</span>
    </div>
    <div id="controls">
        🖱️ Mouse: Lihat & Tembak (Klik Kiri) | 🏃‍♂️ WASD untuk bergerak | 🔄 Geser mouse
    </div>
    <div class="instruction">
        💥 TEMBAK MUSUH! Hindari atau tembak sebelum mereka menyerang!
    </div>
    <div id="game-over">
        <h1>GAME OVER</h1>
        <p>Kamu terluka parah dalam pertempuran.</p>
        <p>Skor Akhir: <span id="final-score">0</span></p>
        <button id="restart-btn">🔫 MULAI LAGI 🔫</button>
    </div>

    <!-- Import Three.js core dan addons via CDN -->
    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.128.0/build/three.module.js",
                "three/addons/": "https://unpkg.com/three@0.128.0/examples/jsm/"
            }
        }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
        import { CSS2DRenderer, CSS2DObject } from 'three/addons/renderers/CSS2DRenderer.js';

        // --- Setup Scene, Cameras, Renderers ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x0a1030); // warna malam/senja
        scene.fog = new THREE.FogExp2(0x0a1030, 0.008); // kabut ringan

        // Camera (First Person)
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(0, 1.6, 5);
        
        // WebGL Renderer
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true; // bayangan realistis
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        document.body.appendChild(renderer.domElement);
        
        // CSS2 Renderer untuk teks health bar
        const labelRenderer = new CSS2DRenderer();
        labelRenderer.setSize(window.innerWidth, window.innerHeight);
        labelRenderer.domElement.style.position = 'absolute';
        labelRenderer.domElement.style.top = '0px';
        labelRenderer.domElement.style.left = '0px';
        labelRenderer.domElement.style.pointerEvents = 'none';
        document.body.appendChild(labelRenderer.domElement);
        
        // --- Controls untuk First Person (tanpa Orbit, kita buat custom)
        // Tidak menggunakan OrbitControls, kita buat manual dengan PointerLock + event
        
        // Variabel pergerakan
        const keyState = {
            ArrowUp: false, ArrowDown: false, ArrowLeft: false, ArrowRight: false,
            w: false, s: false, a: false, d: false
        };
        let moveForward = false, moveBack = false, moveLeft = false, moveRight = false;
        let mouseX = 0, mouseY = 0;
        let cameraRotationX = 0, cameraRotationY = 0;
        const cameraSpeed = 0.12;
        const mouseSensitivity = 0.002;
        
        // Pointer lock untuk FPS style
        const canvas = renderer.domElement;
        
        canvas.addEventListener('click', () => {
            canvas.requestPointerLock();
        });
        
        document.addEventListener('pointerlockchange', lockChange);
        document.addEventListener('mozpointerlockchange', lockChange);
        
        function lockChange() {
            if (document.pointerLockElement === canvas) {
                document.addEventListener('mousemove', onMouseMove);
                console.log("Pointer locked - mode menembak aktif");
            } else {
                document.removeEventListener('mousemove', onMouseMove);
                console.log("Pointer unlock");
            }
        }
        
        function onMouseMove(e) {
            if (!document.pointerLockElement) return;
            mouseX += e.movementX * mouseSensitivity;
            mouseY += e.movementY * mouseSensitivity;
            // batasi vertikal agar tidak over rotate
            mouseY = Math.max(-Math.PI / 2.2, Math.min(Math.PI / 2.2, mouseY));
            camera.rotation.order = 'YXZ';
            camera.rotation.y = mouseX;
            camera.rotation.x = mouseY;
        }
        
        // Keyboard event
        window.addEventListener('keydown', (e) => {
            const key = e.key.toLowerCase();
            if (key === 'w') moveForward = true;
            if (key === 's') moveBack = true;
            if (key === 'a') moveLeft = true;
            if (key === 'd') moveRight = true;
            // cegah scroll default
            if (['w','s','a','d','arrowup','arrowdown','arrowleft','arrowright'].includes(key)) e.preventDefault();
        });
        
        window.addEventListener('keyup', (e) => {
            const key = e.key.toLowerCase();
            if (key === 'w') moveForward = false;
            if (key === 's') moveBack = false;
            if (key === 'a') moveLeft = false;
            if (key === 'd') moveRight = false;
        });
        
        // --- Game state ---
        let score = 0;
        let health = 100;
        let enemies = [];
        let bullets = [];
        let lastShootTime = 0;
        const shootCooldown = 200; // ms
        let gameActive = true;
        
        // Update UI
        const scoreElement = document.getElementById('score');
        const healthElement = document.getElementById('health');
        const ammoElement = document.getElementById('ammo-count');
        const gameOverDiv = document.getElementById('game-over');
        const finalScoreSpan = document.getElementById('final-score');
        
        function updateUI() {
            scoreElement.textContent = score;
            healthElement.textContent = Math.max(0, health);
            // ammo unlimited tapi tetap tampilkan
            ammoElement.textContent = '∞';
        }
        
        // --- World Environment: Ground + Grid + Lighting + decorations
        // Ground
        const groundMat = new THREE.MeshStandardMaterial({ color: 0x2c3e2f, roughness: 0.8, metalness: 0.1 });
        const ground = new THREE.Mesh(new THREE.PlaneGeometry(80, 80), groundMat);
        ground.rotation.x = -Math.PI / 2;
        ground.position.y = -0.5;
        ground.receiveShadow = true;
        scene.add(ground);
        
        // Grid helper
        const gridHelper = new THREE.GridHelper(80, 20, 0x88aa88, 0x446644);
        gridHelper.position.y = -0.4;
        scene.add(gridHelper);
        
        // Lingkungan sederhana: pohon-pohon atau kubus sebagai rintangan
        const rockMat = new THREE.MeshStandardMaterial({ color: 0x887766, roughness: 0.9 });
        for (let i = 0; i < 60; i++) {
            const rock = new THREE.Mesh(new THREE.BoxGeometry(0.8, 0.5 + Math.random() * 1, 0.8), rockMat);
            const angle = Math.random() * Math.PI * 2;
            const radius = 12 + Math.random() * 25;
            const x = Math.cos(angle) * radius;
            const z = Math.sin(angle) * radius;
            rock.position.set(x, -0.2, z);
            rock.castShadow = true;
            rock.receiveShadow = true;
            scene.add(rock);
        }
        
        // Tambahan dinding / barikade sederhana
        const wallMat = new THREE.MeshStandardMaterial({ color: 0xaa8866 });
        for (let i = -5; i <= 5; i++) {
            if (Math.abs(i) < 2) continue;
            const barricade = new THREE.Mesh(new THREE.BoxGeometry(1.2, 1.2, 1.2), wallMat);
            barricade.position.set(i * 2.5, 0, -8);
            barricade.castShadow = true;
            scene.add(barricade);
        }
        
        // Light: Ambient + Directional + titik
        const ambientLight = new THREE.AmbientLight(0x404060);
        scene.add(ambientLight);
        const dirLight = new THREE.DirectionalLight(0xfff5e0, 1);
        dirLight.position.set(10, 20, 5);
        dirLight.castShadow = true;
        dirLight.receiveShadow = false;
        dirLight.shadow.mapSize.width = 1024;
        dirLight.shadow.mapSize.height = 1024;
        scene.add(dirLight);
        const fillLight = new THREE.PointLight(0x5577aa, 0.3);
        fillLight.position.set(0, 5, 0);
        scene.add(fillLight);
        const backLight = new THREE.PointLight(0xffaa66, 0.2);
        backLight.position.set(-3, 2, -5);
        scene.add(backLight);
        
        // partikel efek tembakan sederhana
        function createMuzzleFlash(position) {
            const flashMat = new THREE.MeshStandardMaterial({ color: 0xffaa44, emissive: 0xff4400, emissiveIntensity: 0.8 });
            const flash = new THREE.Mesh(new THREE.SphereGeometry(0.15, 4, 4), flashMat);
            flash.position.copy(position);
            scene.add(flash);
            setTimeout(() => scene.remove(flash), 80);
        }
        
        // --- Enemy Class (CSS2D + Model)
        class Enemy {
            constructor(x, z) {
                this.health = 40;
                this.speed = 1.2;
                this.damage = 18;
                this.attackRange = 1.8;
                this.attackCooldown = 0;
                this.lastAttack = 0;
                
                // model: kubus + senjata merah + kepala
                const group = new THREE.Group();
                const bodyGeo = new THREE.BoxGeometry(0.8, 1.2, 0.8);
                const bodyMat = new THREE.MeshStandardMaterial({ color: 0xaa3333, roughness: 0.4 });
                const body = new THREE.Mesh(bodyGeo, bodyMat);
                body.castShadow = true;
                body.position.y = 0.6;
                group.add(body);
                
                const headGeo = new THREE.SphereGeometry(0.45, 16, 16);
                const headMat = new THREE.MeshStandardMaterial({ color: 0xcc8866 });
                const head = new THREE.Mesh(headGeo, headMat);
                head.position.y = 1.2;
                head.castShadow = true;
                group.add(head);
                
                const eyeMat = new THREE.MeshStandardMaterial({ color: 0xff0000 });
                const leftEye = new THREE.Mesh(new THREE.SphereGeometry(0.1, 8, 8), eyeMat);
                leftEye.position.set(-0.2, 1.35, 0.45);
                const rightEye = new THREE.Mesh(new THREE.SphereGeometry(0.1, 8, 8), eyeMat);
                rightEye.position.set(0.2, 1.35, 0.45);
                group.add(leftEye);
                group.add(rightEye);
                
                // Senjata
                const weapon = new THREE.Mesh(new THREE.BoxGeometry(0.2, 0.2, 0.6), new THREE.MeshStandardMaterial({ color: 0x442200 }));
                weapon.position.set(0.5, 0.8, 0.4);
                group.add(weapon);
                
                group.position.set(x, -0.2, z);
                scene.add(group);
                this.model = group;
                
                // CSS2D Health Bar
                const div = document.createElement('div');
                div.textContent = `❤️ ${this.health}`;
                div.style.backgroundColor = 'rgba(0,0,0,0.7)';
                div.style.color = '#ff6666';
                div.style.padding = '2px 6px';
                div.style.borderRadius = '12px';
                div.style.fontSize = '12px';
                div.style.fontWeight = 'bold';
                div.style.fontFamily = 'monospace';
                div.style.border = '1px solid red';
                div.style.whiteSpace = 'nowrap';
                const label = new CSS2DObject(div);
                label.position.set(0, 1.6, 0);
                group.add(label);
                this.label = label;
                this.updateHealthLabel();
            }
            
            updateHealthLabel() {
                if (this.label && this.label.element) {
                    this.label.element.textContent = `❤️ ${Math.max(0, this.health)}`;
                    if (this.health < 20) this.label.element.style.color = '#ffaaaa';
                }
            }
            
            takeDamage(amount) {
                this.health -= amount;
                this.updateHealthLabel();
                if (this.health <= 0) {
                    this.die();
                    return true;
                }
                return false;
            }
            
            die() {
                scene.remove(this.model);
                const index = enemies.indexOf(this);
                if (index !== -1) enemies.splice(index, 1);
                // efek ledakan kecil
                const expGeo = new THREE.SphereGeometry(0.4, 6, 6);
                const expMat = new THREE.MeshStandardMaterial({ color: 0xff6600, emissive: 0xff2200 });
                const explosion = new THREE.Mesh(expGeo, expMat);
                explosion.position.copy(this.model.position);
                scene.add(explosion);
                setTimeout(() => scene.remove(explosion), 150);
            }
            
            update(deltaTime, playerPos) {
                if (!this.model) return;
                // chasing player
                const direction = new THREE.Vector3().subVectors(playerPos, this.model.position);
                direction.y = 0;
                const dist = direction.length();
                if (dist > 0.3) {
                    direction.normalize();
                    this.model.position.x += direction.x * this.speed * deltaTime;
                    this.model.position.z += direction.z * this.speed * deltaTime;
                    // rotasi menghadap player
                    const angle = Math.atan2(direction.x, direction.z);
                    this.model.rotation.y = angle;
                }
                // attack if in range
                if (dist < this.attackRange && gameActive) {
                    const now = Date.now();
                    if (now - this.lastAttack > 1200) {
                        this.lastAttack = now;
                        // serangan ke player
                        if (gameActive) {
                            health = Math.max(0, health - this.damage);
                            updateUI();
                            // efek darah di layar (visual sederhana dengan flash merah)
                            const bloodFlash = document.createElement('div');
                            bloodFlash.style.position = 'fixed';
                            bloodFlash.style.top = 0;
                            bloodFlash.style.left = 0;
                            bloodFlash.style.width = '100%';
                            bloodFlash.style.height = '100%';
                            bloodFlash.style.backgroundColor = 'rgba(150, 0, 0, 0.3)';
                            bloodFlash.style.pointerEvents = 'none';
                            bloodFlash.style.zIndex = 100;
                            document.body.appendChild(bloodFlash);
                            setTimeout(() => bloodFlash.remove(), 150);
                            
                            if (health <= 0) {
                                gameActive = false;
                                endGame();
                            }
                        }
                    }
                }
            }
        }
        
        // --- Projectile (Peluru)
        class Bullet {
            constructor(position, direction) {
                this.life = true;
                const geometry = new THREE.SphereGeometry(0.1, 6, 6);
                const material = new THREE.MeshStandardMaterial({ color: 0xffcc44, emissive: 0xff6600 });
                this.mesh = new THREE.Mesh(geometry, material);
                this.mesh.position.copy(position);
                this.mesh.castShadow = true;
                scene.add(this.mesh);
                this.direction = direction.clone().normalize();
                this.speed = 35;
                this.range = 50;
                this.travelled = 0;
            }
            
            update(deltaTime) {
                if (!this.life) return false;
                const step = this.direction.clone().multiplyScalar(this.speed * deltaTime);
                this.mesh.position.add(step);
                this.travelled += step.length();
                if (this.travelled > this.range) return false;
                
                // cek collision dengan musuh
                for (let i = 0; i < enemies.length; i++) {
                    const enemy = enemies[i];
                    const enemyPos = enemy.model.position;
                    const distToEnemy = this.mesh.position.distanceTo(enemyPos);
                    if (distToEnemy < 0.8) {
                        const wasAlive = enemy.takeDamage(35);
                        if (wasAlive && enemy.health <= 0) {
                            score += 10;
                            updateUI();
                        } else if (!wasAlive) {
                            // sudah mati di takeDamage
                            score += 10;
                            updateUI();
                        } else {
                            // hanya damage
                            if (enemy.health <= 0) score += 10;
                            updateUI();
                        }
                        return false;
                    }
                }
                return true;
            }
            
            dispose() {
                scene.remove(this.mesh);
            }
        }
        
        // --- Spawn Enemies ---
        let spawnInterval;
        function startSpawning() {
            if (spawnInterval) clearInterval(spawnInterval);
            spawnInterval = setInterval(() => {
                if (!gameActive) return;
                if (enemies.length < 12) {
                    let angle = Math.random() * Math.PI * 2;
                    let radius = 12 + Math.random() * 10;
                    let x = Math.cos(angle) * radius;
                    let z = Math.sin(angle) * radius;
                    // jangan spawn terlalu dekat dengan player
                    const playerPos = camera.position;
                    if (Math.hypot(x - playerPos.x, z - playerPos.z) < 5) {
                        x += (x > 0 ? 5 : -5);
                        z += (z > 0 ? 5 : -5);
                    }
                    const enemy = new Enemy(x, z);
                    enemies.push(enemy);
                }
            }, 2200);
        }
        
        // --- Shoot function dengan raycast untuk akurasi lebih (opsional, tapi kita menggunakan bullet object)
        function shoot() {
            if (!gameActive) return;
            const now = Date.now();
            if (now - lastShootTime < shootCooldown) return;
            lastShootTime = now;
            
            // posisi kamera sebagai sumber tembakan
            const cameraPos = camera.position.clone();
            // arah tembakan dari arah pandang kamera
            const direction = new THREE.Vector3(0, 0, -1).applyQuaternion(camera.quaternion);
            createMuzzleFlash(cameraPos.clone().add(direction.clone().multiplyScalar(0.5)));
            
            const bullet = new Bullet(cameraPos, direction);
            bullets.push(bullet);
            
            // efek suara tanpa audio tag (opsional, tapi kita skip AudioContext karena butuh user gesture)
            // visual efek kecil
        }
        
        // Event click untuk menembak
        window.addEventListener('click', (e) => {
            if (document.pointerLockElement === canvas && gameActive) {
                shoot();
            }
        });
        
        // --- Game loop & Movement & Collision---
        let lastTime = performance.now();
        
        function updateMovement(deltaTime) {
            if (!gameActive) return;
            const actualSpeed = cameraSpeed * deltaTime * 60; // normalize ke 60fps approx
            const forward = new THREE.Vector3(0, 0, -1).applyQuaternion(camera.quaternion);
            const right = new THREE.Vector3(1, 0, 0).applyQuaternion(camera.quaternion);
            forward.y = 0;
            forward.normalize();
            right.y = 0;
            right.normalize();
            
            let move = new THREE.Vector3(0, 0, 0);
            if (moveForward) move.add(forward);
            if (moveBack) move.sub(forward);
            if (moveRight) move.add(right);
            if (moveLeft) move.sub(right);
            move.multiplyScalar(actualSpeed);
            
            let newPos = camera.position.clone().add(move);
            // batasan arena radius 35
            if (Math.abs(newPos.x) < 35 && Math.abs(newPos.z) < 35) {
                camera.position.copy(newPos);
            }
            // sederhana: jaga y tetap 1.6
            camera.position.y = 1.6;
        }
        
        function updateEnemies(deltaTime) {
            const playerPos = camera.position.clone();
            for (let i = 0; i < enemies.length; i++) {
                enemies[i].update(deltaTime, playerPos);
            }
        }
        
        function updateBullets(deltaTime) {
            for (let i = bullets.length-1; i >= 0; i--) {
                const alive = bullets[i].update(deltaTime);
                if (!alive) {
                    bullets[i].dispose();
                    bullets.splice(i,1);
                }
            }
        }
        
        function endGame() {
            gameActive = false;
            finalScoreSpan.textContent = score;
            gameOverDiv.style.display = 'block';
            if (spawnInterval) clearInterval(spawnInterval);
            document.exitPointerLock();
        }
        
        function restartGame() {
            // reset state
            gameActive = true;
            score = 0;
            health = 100;
            updateUI();
            // hapus semua musuh
            for (let e of enemies) {
                scene.remove(e.model);
            }
            enemies = [];
            for (let b of bullets) {
                b.dispose();
            }
            bullets = [];
            camera.position.set(0, 1.6, 5);
            camera.rotation.set(0,0,0);
            mouseX = 0; mouseY = 0;
            camera.rotation.y = 0;
            camera.rotation.x = 0;
            gameOverDiv.style.display = 'none';
            startSpawning();
        }
        
        document.getElementById('restart-btn').addEventListener('click', () => {
            restartGame();
        });
        
        // Animasi efek latar: menambahkan partikel debu atau cahaya dinamis
        const starField = new THREE.Points(new THREE.BufferGeometry(), new THREE.PointsMaterial({ color: 0xffffff, size: 0.1 }));
        const starCount = 800;
        const starPositions = new Float32Array(starCount * 3);
        for (let i = 0; i < starCount; i++) {
            starPositions[i*3] = (Math.random() - 0.5) * 400;
            starPositions[i*3+1] = (Math.random() - 0.5) * 100 + 20;
            starPositions[i*3+2] = (Math.random() - 0.5) * 150 - 80;
        }
        starField.geometry.setAttribute('position', new THREE.BufferAttribute(starPositions, 3));
        scene.add(starField);
        
        // Mulai spawning musuh
        startSpawning();
        updateUI();
        
        // --- Animasi Loop ---
        function animate() {
            const now = performance.now();
            let delta = Math.min(0.033, (now - lastTime) / 1000);
            lastTime = now;
            
            if (gameActive) {
                updateMovement(delta);
                updateEnemies(delta);
                updateBullets(delta);
                // sedikit efek dynamic light mengikuti player
                fillLight.position.x = camera.position.x;
                fillLight.position.z = camera.position.z;
            }
            
            renderer.render(scene, camera);
            labelRenderer.render(scene, camera);
            requestAnimationFrame(animate);
        }
        
        animate();
        
        // handling resize window
        window.addEventListener('resize', onWindowResize, false);
        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
            labelRenderer.setSize(window.innerWidth, window.innerHeight);
        }
        
        console.log("Game siap! Arahkan mouse, klik untuk kunci kursor, WASD bergerak, klik kiri tembak!");
    </script>
</body>
</html>
