<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Spatial Discovery Pod - Class 12 Carnot Engine with P-V Graph</title>
  <style>
    body { margin: 0; overflow: hidden; background: #070b14; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; color: #fff; }
    #canvas-container { width: 100vw; height: 100vh; position: absolute; }
    #webcam { position: absolute; bottom: 24px; right: 24px; width: 200px; height: 150px; border-radius: 12px; border: 2px solid #38bdf8; transform: scaleX(-1); object-fit: cover; box-shadow: 0 8px 24px rgba(0,0,0,0.6); }
    #hud { position: absolute; top: 24px; left: 24px; background: rgba(15, 23, 42, 0.88); backdrop-filter: blur(10px); padding: 18px 22px; border-radius: 12px; border: 1px solid #1e293b; max-width: 380px; box-shadow: 0 10px 30px rgba(0,0,0,0.5); }
    h2 { margin: 0 0 6px 0; font-size: 1.25rem; color: #38bdf8; letter-spacing: -0.02em; }
    .sub { font-size: 0.82rem; color: #94a3b8; margin-bottom: 12px; text-transform: uppercase; letter-spacing: 0.05em; }
    .status-row { display: flex; justify-content: space-between; margin: 6px 0; font-size: 0.88rem; border-bottom: 1px solid #334155; padding-bottom: 4px; }
    .label { color: #94a3b8; }
    .val { font-weight: 600; color: #f8fafc; font-family: monospace; }
    .instructions { margin-top: 14px; font-size: 0.82rem; color: #cbd5e1; line-height: 1.45; }
    .badge { display: inline-block; background: #0369a1; color: #e0f2fe; padding: 3px 8px; border-radius: 4px; font-size: 0.72rem; font-weight: 700; margin-top: 10px; }

    /* Real-Time P-V Graph Styling */
    #graph-container {
      position: absolute;
      top: 24px;
      right: 24px;
      background: rgba(15, 23, 42, 0.92);
      backdrop-filter: blur(10px);
      padding: 16px 18px;
      border-radius: 12px;
      border: 1px solid #334155;
      box-shadow: 0 10px 30px rgba(0,0,0,0.6);
      z-index: 10;
    }
    .graph-title {
      font-size: 0.92rem;
      font-weight: 700;
      color: #38bdf8;
      margin-bottom: 8px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    #pvCanvas {
      border-left: 2px solid #94a3b8;
      border-bottom: 2px solid #94a3b8;
      display: block;
    }
  </style>

  <!-- Three.js + MediaPipe Hands -->
  <script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js"></script>
</head>
<body>
  <div id="canvas-container"></div>
  <video id="webcam" autoplay playsinline></video>

  <!-- Left: Telemetry & State -->
  <div id="hud">
    <h2>Carnot Ideal Heat Engine</h2>
    <div class="sub">Class 12 Physics • Thermodynamics Module</div>
    
    <div class="status-row">
      <span class="label">Current Base Contact:</span>
      <span class="val" id="stage-text" style="color: #f59e0b;">1. Heat Source (T₁)</span>
    </div>
    <div class="status-row">
      <span class="label">Thermodynamic Process:</span>
      <span class="val" id="process-text" style="color: #38bdf8;">Isothermal Expansion</span>
    </div>
    <div class="status-row">
      <span class="label">Piston Height (Volume V):</span>
      <span class="val" id="volume-text">V₁ → V₂</span>
    </div>
    <div class="status-row">
      <span class="label">Theoretical Efficiency:</span>
      <span class="val">η = 1 - (T₂ / T₁)</span>
    </div>

    <div class="instructions">
      • <strong>Move Hand Left / Right:</strong> Shifts the cylinder across Source ($T_1$), Insulating Stand, and Sink ($T_2$).<br>
      • <strong>Pinch & Pull (Thumb + Index):</strong> Compresses or expands the working gas chamber.
    </div>
    <div class="badge">Gemini Spatial Discovery Engine</div>
  </div>

  <!-- Right: Dynamic P-V Graph -->
  <div id="graph-container">
    <div class="graph-title">
      <span>P-V Indicator Diagram</span>
      <span style="font-size: 0.72rem; color: #94a3b8; font-weight: normal;">Carnot Cycle</span>
    </div>
    <canvas id="pvCanvas" width="280" height="210"></canvas>
  </div>

  <script>
    // --- 1. Scene Setup ---
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(55, window.innerWidth / window.innerHeight, 0.1, 100);
    camera.position.set(-0.5, 3, 7.5);
    camera.lookAt(-0.5, 0.5, 0);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(window.devicePixelRatio);
    document.getElementById('canvas-container').appendChild(renderer.domElement);

    // Lights
    scene.add(new THREE.AmbientLight(0xffffff, 0.7));
    const mainLight = new THREE.DirectionalLight(0xffffff, 1.5);
    mainLight.position.set(5, 8, 5);
    scene.add(mainLight);

    // --- 2. Build the Three Thermal Platforms ---
    const platformGroup = new THREE.Group();
    platformGroup.position.y = -1.2;

    function createPlatform(x, color, labelText) {
      const p = new THREE.Mesh(
        new THREE.CylinderGeometry(0.9, 0.9, 0.4, 32),
        new THREE.MeshStandardMaterial({ color: color, roughness: 0.3, metalness: 0.6 })
      );
      p.position.x = x;
      return p;
    }

    // Source (Hot T1) = Red/Orange; Stand (Adiabatic) = Dark Charcoal; Sink (Cold T2) = Cyan
    const sourcePlatform = createPlatform(-2.2, 0xef4444); // Source T1
    const standPlatform  = createPlatform(0.0,  0x334155); // Insulating Stand
    const sinkPlatform   = createPlatform(2.2,  0x06b6d4); // Sink T2

    platformGroup.add(sourcePlatform);
    platformGroup.add(standPlatform);
    platformGroup.add(sinkPlatform);
    scene.add(platformGroup);

    // --- 3. Build the Carnot Cylinder & Piston Apparatus ---
    const cylinderRig = new THREE.Group();

    // Transparent Insulated Glass Wall (Non-conducting)
    const glassGeo = new THREE.CylinderGeometry(0.85, 0.85, 2.4, 32, 1, true);
    const glassMat = new THREE.MeshPhysicalMaterial({
      color: 0x93c5fd,
      transparent: true,
      opacity: 0.35,
      roughness: 0.1,
      metalness: 0.1,
      transmission: 0.8,
      side: THREE.DoubleSide
    });
    const glassCylinder = new THREE.Mesh(glassGeo, glassMat);
    glassCylinder.position.y = 0.2;
    cylinderRig.add(glassCylinder);

    // Perfectly Conducting Base (Copper / Gold tone)
    const baseGeo = new THREE.CylinderGeometry(0.85, 0.85, 0.15, 32);
    const baseMat = new THREE.MeshStandardMaterial({ color: 0xf59e0b, metalness: 0.9, roughness: 0.2 });
    const conductingBase = new THREE.Mesh(baseGeo, baseMat);
    conductingBase.position.y = -1.0;
    cylinderRig.add(conductingBase);

    // Piston Assembly (Frictionless, Non-conducting)
    const pistonHead = new THREE.Mesh(
      new THREE.CylinderGeometry(0.82, 0.82, 0.2, 32),
      new THREE.MeshStandardMaterial({ color: 0x64748b, metalness: 0.5, roughness: 0.3 })
    );
    const pistonRod = new THREE.Mesh(
      new THREE.CylinderGeometry(0.08, 0.08, 1.8, 16),
      new THREE.MeshStandardMaterial({ color: 0x94a3b8, metalness: 0.8, roughness: 0.2 })
    );
    pistonRod.position.y = 0.9;
    pistonHead.add(pistonRod);
    cylinderRig.add(pistonHead);

    // Working Substance: Dynamic Gas Particles inside Cylinder
    const particleCount = 120;
    const particleGeo = new THREE.BufferGeometry();
    const positions = new Float32Array(particleCount * 3);
    for(let i = 0; i < particleCount * 3; i += 3) {
      positions[i] = (Math.random() - 0.5) * 1.2;
      positions[i+1] = -0.8 + Math.random() * 0.8;
      positions[i+2] = (Math.random() - 0.5) * 1.2;
    }
    particleGeo.setAttribute('position', new THREE.BufferAttribute(positions, 3));
    const particleMat = new THREE.PointsMaterial({ color: 0x38bdf8, size: 0.08 });
    const gasParticles = new THREE.Points(particleGeo, particleMat);
    cylinderRig.add(gasParticles);

    scene.add(cylinderRig);

    // --- 4. 2D P-V Graph Drawing Logic ---
    const pvCanvas = document.getElementById('pvCanvas');
    const ctx = pvCanvas.getContext('2d');

    const ptA = { x: 45,  y: 35,  label: "A" };
    const ptB = { x: 125, y: 75,  label: "B" };
    const ptC = { x: 235, y: 155, label: "C" };
    const ptD = { x: 135, y: 145, label: "D" };

    function drawPVDiagram(activeNormVol, stageIdx) {
      ctx.clearRect(0, 0, pvCanvas.width, pvCanvas.height);

      // Axis hints
      ctx.fillStyle = "#64748b";
      ctx.font = "10px monospace";
      ctx.fillText("P ↑", 6, 14);
      ctx.fillText("V →", pvCanvas.width - 25, pvCanvas.height - 8);

      // 1. Curve A -> B (Isothermal Expansion T1)
      ctx.strokeStyle = "#ef4444";
      ctx.lineWidth = 2.5;
      ctx.beginPath();
      ctx.moveTo(ptA.x, ptA.y);
      ctx.quadraticCurveTo(80, 52, ptB.x, ptB.y);
      ctx.stroke();

      // 2. Curve B -> C (Adiabatic Expansion)
      ctx.strokeStyle = "#a855f7";
      ctx.beginPath();
      ctx.moveTo(ptB.x, ptB.y);
      ctx.quadraticCurveTo(185, 110, ptC.x, ptC.y);
      ctx.stroke();

      // 3. Curve C -> D (Isothermal Compression T2)
      ctx.strokeStyle = "#06b6d4";
      ctx.beginPath();
      ctx.moveTo(ptC.x, ptC.y);
      ctx.quadraticCurveTo(180, 155, ptD.x, ptD.y);
      ctx.stroke();

      // 4. Curve D -> A (Adiabatic Compression)
      ctx.strokeStyle = "#10b981";
      ctx.beginPath();
      ctx.moveTo(ptD.x, ptD.y);
      ctx.quadraticCurveTo(80, 95, ptA.x, ptA.y);
      ctx.stroke();

      // Draw State Markers (A, B, C, D)
      [ptA, ptB, ptC, ptD].forEach(pt => {
        ctx.fillStyle = "#f8fafc";
        ctx.beginPath();
        ctx.arc(pt.x, pt.y, 3.5, 0, Math.PI * 2);
        ctx.fill();
        ctx.fillStyle = "#94a3b8";
        ctx.font = "10px sans-serif";
        ctx.fillText(pt.label, pt.x - 4, pt.y - 6);
      });

      // Compute marker position on curves
      let curX = ptA.x, curY = ptA.y;
      if (stageIdx === 1) {
        curX = ptA.x + (ptB.x - ptA.x) * activeNormVol;
        curY = ptA.y + (ptB.y - ptA.y) * Math.pow(activeNormVol, 0.7);
      } else if (stageIdx === 2) {
        curX = ptB.x + (ptC.x - ptB.x) * activeNormVol;
        curY = ptB.y + (ptC.y - ptB.y) * Math.pow(activeNormVol, 0.85);
      } else if (stageIdx === 3) {
        curX = ptC.x - (ptC.x - ptD.x) * activeNormVol;
        curY = ptC.y - (ptC.y - ptD.y) * activeNormVol;
      } else {
        curX = ptD.x - (ptD.x - ptA.x) * activeNormVol;
        curY = ptD.y - (ptD.y - ptA.y) * activeNormVol;
      }

      // Glowing Active Indicator Dot
      ctx.fillStyle = "#facc15";
      ctx.shadowColor = "#facc15";
      ctx.shadowBlur = 10;
      ctx.beginPath();
      ctx.arc(curX, curY, 6, 0, Math.PI * 2);
      ctx.fill();
      ctx.shadowBlur = 0;
    }

    // --- 5. Gesture Tracking & Physics Targets ---
    let targetCylX = -2.2; // Start on Source
    let targetPistonY = 0.2; // Mid height

    function onResults(results) {
      if (results.multiHandLandmarks && results.multiHandLandmarks.length > 0) {
        const lm = results.multiHandLandmarks[0];
        
        // Horizontal hand movement maps cylinder position across the 3 stations
        const normalizedX = (0.5 - lm[0].x) * 7.0;
        targetCylX = THREE.MathUtils.clamp(normalizedX, -2.2, 2.2);

        // Pinch distance controls piston height (V)
        const pinchDist = Math.hypot(lm[8].x - lm[4].x, lm[8].y - lm[4].y);
        targetPistonY = THREE.MathUtils.clamp((pinchDist - 0.08) * 4.5 - 0.6, -0.7, 1.1);
      }
    }

    const hands = new Hands({ locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}` });
    hands.setOptions({ maxNumHands: 1, modelComplexity: 1, minDetectionConfidence: 0.6, minTrackingConfidence: 0.6 });
    hands.onResults(onResults);

    const cam = new Camera(document.getElementById('webcam'), {
      onFrame: async () => { await hands.send({ image: document.getElementById('webcam') }); },
      width: 640, height: 480
    });
    cam.start();

    // --- 6. UI Updating & Animation Loop ---
    const stageEl = document.getElementById('stage-text');
    const procEl = document.getElementById('process-text');
    const volEl = document.getElementById('volume-text');

    function updateThermodynamicState(xPos, pHeight) {
      const normVol = THREE.MathUtils.clamp((pHeight - (-0.7)) / 1.8, 0, 1);
      volEl.innerText = pHeight > 0.2 ? `V (Expanded: ${(1.0 + normVol * 2.5).toFixed(1)} L)` : `V (Compressed: ${(1.0 + normVol * 2.5).toFixed(1)} L)`;

      let currentStage = 1;

      if (xPos < -0.8) {
        currentStage = 1;
        stageEl.innerText = "1. Heat Source (T₁)";
        stageEl.style.color = "#ef4444";
        procEl.innerText = "A → B: Isothermal Expansion (Q₁ in)";
        particleMat.color.setHex(0xf97316); // Hot gas
      } else if (xPos >= -0.8 && xPos <= 0.8) {
        currentStage = (normVol > 0.5) ? 2 : 4;
        stageEl.innerText = "2/4. Insulating Stand";
        stageEl.style.color = "#94a3b8";
        procEl.innerText = (currentStage === 2) ? "B → C: Adiabatic Expansion" : "D → A: Adiabatic Compression";
        particleMat.color.setHex(0xa855f7); // Insulated gas
      } else {
        currentStage = 3;
        stageEl.innerText = "3. Heat Sink (T₂)";
        stageEl.style.color = "#06b6d4";
        procEl.innerText = "C → D: Isothermal Compression (Q₂ out)";
        particleMat.color.setHex(0x38bdf8); // Cold gas
      }

      drawPVDiagram(normVol, currentStage);
    }

    function animate() {
      requestAnimationFrame(animate);

      // Smooth kinematics
      cylinderRig.position.x += (targetCylX - cylinderRig.position.x) * 0.12;
      pistonHead.position.y += (targetPistonY - pistonHead.position.y) * 0.15;

      // Confine gas particles under the piston
      const pos = gasParticles.geometry.attributes.position.array;
      const maxY = pistonHead.position.y;
      for (let i = 1; i < pos.length; i += 3) {
        pos[i] += (Math.random() - 0.5) * 0.02;
        if (pos[i] > maxY - 0.1) pos[i] = maxY - 0.15;
        if (pos[i] < -0.9) pos[i] = -0.85;
      }
      gasParticles.geometry.attributes.position.needsUpdate = true;

      updateThermodynamicState(cylinderRig.position.x, pistonHead.position.y);
      renderer.render(scene, camera);
    }
    animate();

    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  </script>
</body>
</html>
