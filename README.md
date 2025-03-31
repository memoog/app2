<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Simulador Quirúrgico Retiniano – Versión Mejorada</title>
  <style>
    /* ================== BASE ================== */
    body {
      margin: 0;
      padding: 0;
      background: #000;
      overflow: hidden;
      font-family: Arial, sans-serif;
      color: white;
    }
    #container {
      position: relative;
      width: 100vw;
      height: 100vh;
      perspective: 1200px;
    }
    /* CONTENEDOR PRINCIPAL DEL OJO */
    #eye-chamber {
      position: absolute;
      width: 100%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    /* ================== RETINA CENTRAL ================== */
    .retina-container {
      position: relative;
      width: 80vmin;
      height: 80vmin;
      transform-style: preserve-3d;
      perspective: 1200px;
      border-radius: 50%;
      overflow: hidden;
      transition: transform 0.3s ease;
    }
    .retina-sphere {
      position: absolute;
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background: radial-gradient(circle at center, #400000 0%, #300000 40%, #200000 70%, #100000 90%);
      transform: rotateX(20deg) translateZ(50px);
      box-shadow: inset 0 0 150px rgba(200,0,0,0.3),
                  inset 0 0 50px rgba(255,0,0,0.2),
                  0 0 100px rgba(0,0,0,0.9);
      position: relative;
    }
    .retina-texture {
      position: absolute;
      width: 100%;
      height: 100%;
      background: radial-gradient(circle at 50% 50%, rgba(255,200,200,0.3) 0%, rgba(255,150,150,0.2) 80%),
                  url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="100" height="100" viewBox="0 0 100 100"><circle cx="50" cy="50" r="40" fill="none" stroke="rgba(255,100,100,0.2)" stroke-width="1"/></svg>');
      background-size: 100%, 20px 20px;
      opacity: 0.9;
      border-radius: 50%;
      mix-blend-mode: multiply;
    }
    .macula {
      position: absolute;
      width: 80px;
      height: 80px;
      background: radial-gradient(circle at center, rgba(255,220,180,0.5) 0%, rgba(255,200,150,0.3) 70%, transparent 100%);
      border-radius: 50%;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      z-index: 2;
    }
    .blood-vessels {
      position: absolute;
      width: 100%;
      height: 100%;
      z-index: 3;
      pointer-events: none;
    }
    .retina-nerve {
      position: absolute;
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200"><path d="M0,0 Q50,50 100,0 T200,0" stroke="rgba(255,255,255,0.05)" stroke-width="2" fill="none"/></svg>') center/cover;
      pointer-events: none;
      opacity: 0.5;
      z-index: 4;
    }
    .optic-disc {
      position: absolute;
      width: 70px;
      height: 70px;
      background-color: rgba(200,100,100,0.3);
      border-radius: 50%;
      top: 50%;
      left: 25%;
      transform: translate(-50%, -50%);
      box-shadow: inset 0 0 15px rgba(200,50,50,0.4), 0 0 20px rgba(200,50,50,0.3);
      z-index: 5;
    }
    .optic-cup {
      position: absolute;
      width: 30px;
      height: 30px;
      background-color: rgba(200,50,50,0.4);
      border-radius: 50%;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      box-shadow: inset 0 0 10px rgba(150,30,30,0.5);
      z-index: 6;
    }
    /* ================== CAMPO DE VISIÓN – Endoiluminador ================== */
    /* (El joystick para el efecto de luz se mantiene) */
    #light-mask {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0,0,0,0.85);
      mask-image: radial-gradient(circle at var(--light-x, 50%) var(--light-y, 50%), transparent var(--light-size, 80px), black calc(var(--light-size, 80px) + 40px));
      -webkit-mask-image: radial-gradient(circle at var(--light-x, 50%) var(--light-y, 50%), transparent var(--light-size, 80px), black calc(var(--light-size, 80px) + 40px));
      transition: all 0.3s ease;
      border-radius: 50%;
      pointer-events: none;
      z-index: 7;
    }
    #light-reflection {
      position: absolute;
      width: 100%;
      height: 100%;
      background: radial-gradient(circle at var(--light-x, 50%) var(--light-y, 50%), rgba(255,255,220,0.9) calc(var(--light-size, 80px)*0.2), rgba(255,200,150,0.5) calc(var(--light-size, 80px)*0.4), transparent var(--light-size, 80px));
      opacity: 0;
      transition: opacity 0.3s ease;
      border-radius: 50%;
      pointer-events: none;
      z-index: 8;
    }
    /* ================== INSTRUMENTOS QUIRÚRGICOS ================== */
    /* Se posicionan dentro de la retina central; se mantienen los estilos originales */
    #vitrectome {
      width: 20px;
      height: 100px;
      background: linear-gradient(to bottom, #ccc, #fff);
      border: 1px solid #aaa;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.5);
      position: absolute;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
      display: none;
    }
    #vitrectome::after {
      content: '';
      position: absolute;
      bottom: -5px;
      left: 50%;
      transform: translateX(-50%);
      width: 12px;
      height: 12px;
      background: radial-gradient(circle, #fff, #ccc);
      border-radius: 50%;
    }
    #end-grasper {
      width: 18px;
      height: 90px;
      background: linear-gradient(to bottom, #ddd, #eee);
      border: 1px solid #bbb;
      border-radius: 10px;
      box-shadow: 0 0 8px rgba(0,0,0,0.5);
      position: absolute;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
      display: none;
    }
    #laser-probe {
      width: 4px;
      height: 80px;
      background: linear-gradient(to bottom, #f00, #ff0);
      border-radius: 2px;
      box-shadow: 0 0 5px rgba(255,0,0,0.8);
      position: absolute;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
      display: none;
    }
    /* ================== MINI MAPA (diseño según app5 original) ================== */
    #miniMapContainer {
      position: absolute;
      top: 20px;
      right: 20px;
      width: 300px;
      height: 225px;
      overflow: hidden;
      border-radius: 10px;
      z-index: 2000;
    }
    #eyeCrossSection {
      width: 100%;
      height: 100%;
      display: block;
    }
    /* ================== PANEL DE INSTRUMENTOS – BOTONES TOGGLE ================== */
    .instrument-panel {
      position: absolute;
      top: 20px;
      left: 50%;
      transform: translateX(-50%);
      background: rgba(0,0,0,0.7);
      padding: 15px;
      border-radius: 10px;
      z-index: 5000;
    }
    .toggle-btn {
      background: #1e3a8a;
      color: white;
      padding: 8px 12px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      font-size: 0.9em;
      margin: 0 5px;
    }
    .toggle-btn.active {
      background: #3b82f6;
      box-shadow: 0 0 10px #3b82f6;
    }
    /* ================== NUEVO BOTÓN DE ACCIÓN “PRECIONAR” ================== */
    #btn-precionar {
      background: #1e3a8a;
      color: white;
      padding: 8px 12px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      font-size: 0.9em;
    }
    /* ================== PANEL DE MONITOREO (parámetros) ================== */
    .control-panel {
      position: absolute;
      left: 50%;
      bottom: 20px;
      transform: translateX(-50%);
      background: rgba(10,10,10,0.85);
      padding: 15px;
      border-radius: 8px;
      z-index: 5000;
      font-size: 14px;
      width: 280px;
    }
    .vital-sign {
      display: flex;
      justify-content: space-between;
      margin: 8px 0;
    }
    .vital-label {
      color: #ccc;
    }
    .vital-value {
      font-family: 'Courier New', monospace;
      font-weight: bold;
    }
    .normal { color: #2ecc71; }
    .warning { color: #f39c12; }
    .danger { color: #e74c3c; }
    #iop-gauge {
      width: 100%;
      height: 8px;
      background: linear-gradient(to right, #2ecc71 0%, #2ecc71 30%, #f39c12 30%, #f39c12 70%, #e74c3c 70%, #e74c3c 100%);
      border-radius: 4px;
      margin-top: 5px;
      overflow: hidden;
    }
    #iop-level {
      height: 100%;
      width: 50%;
      background: rgba(255,255,255,0.3);
      border-radius: 4px;
      transition: width 0.5s ease;
    }
    /* ================== JOYSTICKS ================== */
    .joystick-container {
      position: absolute;
      bottom: 20px;
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 10px;
      z-index: 3000;
    }
    #joystick-vitrectomo-container {
      left: 20px;
    }
    #joystick-light-container {
      right: 20px;
    }
    .joystick {
      width: 100px;
      height: 100px;
      background: rgba(255,255,255,0.1);
      border: 2px solid rgba(255,255,255,0.3);
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      touch-action: none;
      position: relative;
    }
    .joystick .joystick-handle {
      width: 40px;
      height: 40px;
      background: rgba(255,255,255,0.5);
      border-radius: 50%;
      position: absolute;
      transition: transform 0.1s ease;
    }
    /* ================== NUEVAS REGLAS PARA FUNCIONALIDADES ================== */
    .laser-spot {
      width: 25px;
      height: 25px;
      background: radial-gradient(circle, rgba(255,50,50,0.8), transparent 70%);
      border-radius: 50%;
      position: absolute;
      pointer-events: none;
      animation: laser-fade 2.5s ease-out forwards;
    }
    @keyframes laser-fade {
      0% { opacity: 1; transform: scale(1); }
      100% { opacity: 0; transform: scale(0.3); }
    }
    .laser-burn {
      width: 6px;
      height: 6px;
      background: radial-gradient(circle, white 0%, rgba(255,255,255,0.6) 60%, transparent 80%);
      border-radius: 50%;
      position: absolute;
      pointer-events: none;
    }
    .vitreous-particle {
      width: 5px;
      height: 5px;
      background: rgba(255,255,255,0.9);
      border-radius: 50%;
      position: absolute;
      pointer-events: none;
      animation: float-particle 1.5s ease-out forwards;
    }
    @keyframes float-particle {
      100% { transform: translate(var(--tx, 0px), var(--ty, 0px)); opacity: 0; }
    }
    /* Estilos para la inyección (copiados de práctica6) */
    .injection-bubble {
      position: absolute;
      background: rgba(200,230,255,0.7);
      border-radius: 50%;
      pointer-events: none;
      z-index: 3;
      filter: blur(1px);
      animation: bubble-float 4s ease-in-out forwards;
    }
    @keyframes bubble-float {
      100% { 
        transform: translate(calc(var(--tx)*1px), calc(var(--ty)*1px));
        opacity: 0;
      }
    }
  </style>
</head>
<body>
  <div id="container">
    <!-- MINI MAPA -->
    <div id="miniMapContainer">
      <svg id="eyeCrossSection" viewBox="0 0 800 600" preserveAspectRatio="xMidYMid meet">
        <defs>
          <radialGradient id="bgGradient" cx="50%" cy="50%" r="70%">
            <stop offset="0%" stop-color="#1E293B" />
            <stop offset="100%" stop-color="#0F172A" />
          </radialGradient>
          <radialGradient id="lensGradient" cx="50%" cy="50%" r="50%">
            <stop offset="0%" stop-color="#E0F7FA" stop-opacity="0.9" />
            <stop offset="70%" stop-color="#B2EBF2" stop-opacity="0.8" />
            <stop offset="100%" stop-color="#80DEEA" stop-opacity="0.7" />
          </radialGradient>
          <filter id="dropShadow" x="-20%" y="-20%" width="140%" height="140%">
            <feGaussianBlur in="SourceAlpha" stdDeviation="8" />
            <feOffset dx="0" dy="4" result="offsetblur" />
            <feComponentTransfer>
              <feFuncA type="linear" slope="0.3" />
            </feComponentTransfer>
            <feMerge>
              <feMergeNode />
              <feMergeNode in="SourceGraphic" />
            </feMerge>
          </filter>
        </defs>
        <rect width="800" height="600" fill="url(#bgGradient)" />
        <ellipse cx="400" cy="150" rx="100" ry="65" fill="url(#lensGradient)" filter="url(#dropShadow)" />
        <ellipse cx="400" cy="150" rx="95" ry="60" fill="none" stroke="#B3E5FC" stroke-width="2" />
        <circle cx="400" cy="150" r="55" fill="none" stroke="url(#irisGradient)" stroke-width="15" />
        <circle cx="400" cy="150" r="30" fill="#000000" />
        <circle cx="400" cy="300" r="250" fill="none" stroke="#E0E0E0" stroke-width="4" />
        <circle cx="400" cy="300" r="240" fill="#400000" opacity="0.8" />
        <circle id="miniCenter" cx="400" cy="300" r="8" fill="#00f7ff" />
        <!-- Palitos blancos en el mini mapa -->
        <!-- Palito izquierdo (vitrectomo) -->
        <line id="probeLight" x1="225" y1="100" x2="350" y2="300" stroke="#FFFFFF" stroke-width="9" stroke-linecap="round" />
        <line id="probeLightInner" x1="225" y1="100" x2="350" y2="300" stroke="#FFFFFF" stroke-width="6" stroke-linecap="round" />
        <!-- Palito derecho (anteriormente endoiluminador, ahora se usará para la inyección) -->
        <line id="probeForceps" x1="575" y1="100" x2="450" y2="300" stroke="#FFFFFF" stroke-width="9" stroke-linecap="round" />
        <line id="probeForcepsInner" x1="575" y1="100" x2="450" y2="300" stroke="#FFFFFF" stroke-width="6" stroke-linecap="round" />
      </svg>
    </div>

    <!-- PANEL DE INSTRUMENTOS – BOTONES TOGGLE -->
    <div class="instrument-panel">
      <button class="toggle-btn" id="btn-vitrectomo">Vitrectomo</button>
      <button class="toggle-btn" id="btn-peeling">Peeling</button>
      <button class="toggle-btn" id="btn-laser">Láser</button>
      <!-- Se sustituye el botón de Endoiluminador por Inyección -->
      <button class="toggle-btn" id="btn-injection">Inyección</button>
    </div>

    <!-- NUEVO BOTÓN "PRECIONAR" (ubicado justo arriba de los joysticks) -->
    <div id="action-button" style="position:absolute; bottom:120px; left:50%; transform:translateX(-50%); z-index:3000;">
      <button id="btn-precionar">Precionar</button>
    </div>

    <!-- CONTENEDOR DEL OJO CON RETINA CENTRAL -->
    <div id="eye-chamber">
      <div class="retina-container" id="retina">
        <div class="retina-sphere">
          <div class="retina-texture"></div>
          <div class="retina-nerve"></div>
          <div class="optic-disc">
            <div class="optic-cup"></div>
          </div>
          <div class="macula"></div>
          <svg class="blood-vessels" viewBox="0 0 600 600">
            <path d="M100,300 Q250,200 300,300 T500,300" fill="none" stroke="#8b0000" stroke-width="2" stroke-opacity="0.7"/>
            <path d="M100,280 Q250,400 300,280 T500,280" fill="none" stroke="#8b0000" stroke-width="1.5" stroke-opacity="0.6"/>
            <path d="M300,100 Q200,250 300,300 T300,500" fill="none" stroke="#8b0000" stroke-width="1.8" stroke-opacity="0.7"/>
          </svg>
        </div>
        <!-- Efectos de luz -->
        <div id="light-mask"></div>
        <div id="light-reflection"></div>
        <!-- Instrumentos (se agregan dentro de la retina central) -->
        <div id="vitrectome" class="instrument"></div>
        <div id="end-grasper" class="instrument"></div>
        <div id="laser-probe" class="instrument"></div>
      </div>
    </div>

    <!-- PANEL DE MONITOREO (parámetros) -->
    <div class="control-panel">
      <div class="vital-sign">
        <span class="vital-label">Presión Intraocular:</span>
        <span class="vital-value normal" id="iop-value">16 mmHg</span>
      </div>
      <div id="iop-gauge">
        <div id="iop-level"></div>
      </div>
      <div class="vital-sign">
        <span class="vital-label">Vitreo Removido:</span>
        <span class="vital-value" id="vitreous-progress">0%</span>
      </div>
      <div class="vital-sign">
        <span class="vital-label">Estado Retina:</span>
        <span class="vital-value normal" id="retina-status">Estable</span>
      </div>
    </div>

    <!-- JOYSTICKS -->
    <div id="joystick-vitrectomo-container" class="joystick-container">
      <div id="joystick-vitrectomo" class="joystick">
        <div class="joystick-handle"></div>
      </div>
      <div class="slider-container z-control">
        <label for="vitrectomo-z-slider">Vitrectomo Z:</label>
        <input type="range" id="vitrectomo-z-slider" min="-250" max="-50" value="-150">
      </div>
    </div>
    <div id="joystick-light-container" class="joystick-container">
      <div id="joystick-light" class="joystick">
        <div class="joystick-handle"></div>
      </div>
      <div class="slider-container z-control">
        <label for="endo-z-slider">Endoiluminador Z (tamaño luz):</label>
        <input type="range" id="endo-z-slider" min="0" max="200" value="50">
      </div>
    </div>
  </div>

  <!-- ALERTAS -->
  <div class="alert-modal" id="alert-modal" style="display:none; position:fixed; top:50%; left:50%; transform:translate(-50%,-50%); background:rgba(20,0,0,0.9); padding:20px; border-radius:8px; z-index:10000;">
    <div class="alert-title" id="alert-title" style="color:#f44; font-size:18px; margin-bottom:10px;">Alerta</div>
    <div class="alert-message" id="alert-message" style="margin-bottom:15px;"></div>
    <button class="alert-btn" onclick="closeAlert()" style="padding:8px 15px; background:#800; color:white; border:none; border-radius:4px; cursor:pointer;">Aceptar</button>
  </div>

  <!-- ================== SCRIPTS ================== -->
  <script>
    /* VARIABLES GLOBALES PARA JOYSTICKS Y Z */
    let lightJoystickX = 50, lightJoystickY = 50;
    let vitrectomoJoystickX = 50, vitrectomoJoystickY = 50;
    let currentDepth = parseInt(document.getElementById('vitrectomo-z-slider').value);

    /* Variables de estado y parámetros */
    let activeInstrument = null;
    let iop = 16; // mmHg
    let vitreousRemoved = 0;
    let retinaStatus = 'Estable';

    /* Función de alerta */
    function showAlert(title, message) {
      document.getElementById('alert-title').innerText = title;
      document.getElementById('alert-message').innerText = message;
      document.getElementById('alert-modal').style.display = 'block';
    }
    function closeAlert() {
      document.getElementById('alert-modal').style.display = 'none';
    }

    /* Función toggle para los botones de instrumentos */
    function toggleInstrument(btnId, instrumentId) {
      const btn = document.getElementById(btnId);
      const instr = document.getElementById(instrumentId);
      if(btn.classList.contains('active')) {
        btn.classList.remove('active');
        if(instr) instr.style.display = 'none';
        if(btnId === 'btn-peeling') {
          const mem = document.getElementById('membrane');
          if(mem) mem.remove();
        }
      } else {
        btn.classList.add('active');
        if(instr) instr.style.display = 'block';
        activeInstrument = instrumentId;
        if(btnId === 'btn-peeling') {
          initPeeling();
        }
      }
    }
    document.getElementById('btn-vitrectomo').addEventListener('click', function(){
      toggleInstrument('btn-vitrectomo', 'vitrectome');
    });
    document.getElementById('btn-peeling').addEventListener('click', function(){
      toggleInstrument('btn-peeling', 'end-grasper');
    });
    document.getElementById('btn-laser').addEventListener('click', function(){
      toggleInstrument('btn-laser', 'laser-probe');
    });
    // El botón de inyección ejecuta directamente la función de inyección
    document.getElementById('btn-injection').addEventListener('click', function(){
      performInjection();
    });

    /* NUEVO BOTÓN: "PRECIONAR" */
    document.getElementById('btn-precionar').addEventListener('click', () => {
      // Según el instrumento activo, se simula la acción correspondiente usando la posición actual de la figura
      if(activeInstrument === 'laser-probe'){
         const instrument = document.getElementById('laser-probe');
         const retinaRect = document.getElementById('retina').getBoundingClientRect();
         const instRect = instrument.getBoundingClientRect();
         const x = instRect.left + instRect.width/2 - retinaRect.left;
         const y = instRect.top + instRect.height/2 - retinaRect.top;
         // Se crea un evento sintético para disparar la función de láser
         const syntheticEvent = { clientX: retinaRect.left + x, clientY: retinaRect.top + y };
         laserFunction(syntheticEvent);
      } else if(activeInstrument === 'vitrectome'){
         const instrument = document.getElementById('vitrectome');
         const retinaRect = document.getElementById('retina').getBoundingClientRect();
         const instRect = instrument.getBoundingClientRect();
         const x = instRect.left + instRect.width/2 - retinaRect.left;
         const y = instRect.top + instRect.height/2 - retinaRect.top;
         const syntheticEvent = { clientX: retinaRect.left + x, clientY: retinaRect.top + y };
         vitrectomyFunction(syntheticEvent);
      } else if(activeInstrument === 'end-grasper'){
         // Para el peeling, se simula el corte forzando el trayecto hasta el borde del overlay
         const membrane = document.getElementById('membrane');
         const peelCanvas = document.getElementById('peelCanvas');
         const pinza = document.getElementById('pinza');
         if(membrane && peelCanvas && pinza) {
            const memRect = membrane.getBoundingClientRect();
            const centerX = memRect.width/2;
            const centerY = memRect.height/2;
            const radius = memRect.width/2;
            // Se simula que el corte llega al borde derecho
            pinza.setAttribute("cx", centerX + radius);
            pinza.setAttribute("cy", centerY);
            const tearPath = document.getElementById('tearPath');
            const d = `M ${centerX} ${centerY} L ${centerX + radius} ${centerY}`;
            tearPath.setAttribute("d", d);
            activarDesgarroPeel(membrane, pinza, peelCanvas);
         }
      }
    });

    /* Actualización de Vitals */
    function updateVitals() {
      iop += (Math.random() - 0.5) * 0.2;
      if(activeInstrument === 'vitrectome') iop -= 0.1;
      document.getElementById('iop-value').innerText = iop.toFixed(1) + " mmHg";
      document.getElementById('iop-level').style.width = Math.min(100, Math.max(0, iop)) + "%";
      document.getElementById('vitreous-progress').innerText = Math.min(100, vitreousRemoved).toFixed(0) + "%";
      document.getElementById('retina-status').innerText = retinaStatus;
    }
    setInterval(updateVitals, 1000);

    /* Actualización de posición del instrumento vitrectome (usado también para peeling y láser) */
    function updateVitrectomoPosition(normX, normY) {
      const retinaRect = document.getElementById('retina').getBoundingClientRect();
      const maxOffset = retinaRect.width / 2;
      const offsetX = (normX - 50) / 50 * maxOffset;
      const offsetY = (normY - 50) / 50 * maxOffset;
      const zVal = document.getElementById('vitrectomo-z-slider').value;
      if(activeInstrument) {
        const instr = document.getElementById(activeInstrument);
        if(instr) {
          instr.style.transform = `translate(calc(-50% + ${offsetX}px), calc(-50% + ${offsetY}px)) translateZ(${zVal}px)`;
        }
      }
      vitrectomoJoystickX = normX;
      updateMiniLeftLine(normX, normY);
    }
    /* Actualización del efecto de luz del endoiluminador (joystick se mantiene) */
    function updateEndoLightEffect(normX, normY) {
      document.documentElement.style.setProperty('--light-x', normX + '%');
      document.documentElement.style.setProperty('--light-y', normY + '%');
      document.getElementById('light-reflection').style.opacity = 0.7;
      const zVal = parseInt(document.getElementById('endo-z-slider').value);
      document.getElementById('light-mask').style.transform = `translateZ(${zVal}px)`;
      const retinaRect = document.getElementById('retina').getBoundingClientRect();
      const minLightSize = 10;
      const maxLightSize = retinaRect.width;
      const newLightSize = minLightSize + (maxLightSize - minLightSize) * (zVal / 200);
      document.documentElement.style.setProperty('--light-size', newLightSize + 'px');
      lightJoystickX = normX;
      updateMiniRightLine(normX, normY);
    }
    /* Funciones para actualizar los palitos blancos en el mini mapa */
    function updateMiniLeftLine(normX, normY) {
      const defaultTipX = 350;
      const defaultTipY = 300;
      const scaleX = 2.5;
      const scaleY = 1.8;
      const offsetX = (normX - 50) * scaleX;
      const offsetY = (normY - 50) * scaleY;
      let tipX = defaultTipX + offsetX;
      let tipY = defaultTipY + offsetY;
      const miniLeft = document.getElementById('probeLight');
      const miniLeftInner = document.getElementById('probeLightInner');
      miniLeft.setAttribute('x2', tipX);
      miniLeftInner.setAttribute('x2', tipX);
      miniLeft.setAttribute('y2', tipY);
      miniLeftInner.setAttribute('y2', tipY);
    }
    function updateMiniRightLine(normX, normY) {
      const defaultTipX = 450;
      const defaultTipY = 300;
      const scaleX = 2.5;
      const scaleY = 1.8;
      const offsetX = (normX - 50) * scaleX;
      const offsetY = (normY - 50) * scaleY;
      let tipX = defaultTipX + offsetX;
      let tipY = defaultTipY + offsetY;
      const miniRight = document.getElementById('probeForceps');
      const miniRightInner = document.getElementById('probeForcepsInner');
      miniRight.setAttribute('x2', tipX);
      miniRightInner.setAttribute('x2', tipX);
      miniRight.setAttribute('y2', tipY);
      miniRightInner.setAttribute('y2', tipY);
    }
    /* Inicialización de Joysticks */
    function initJoystick(joystickElement, updateCallback) {
      const handle = joystickElement.querySelector('.joystick-handle');
      const rect = joystickElement.getBoundingClientRect();
      const centerX = rect.width / 2;
      const centerY = rect.height / 2;
      const maxDistance = rect.width / 2;
      let dragging = false;
      function pointerDown(e) {
        dragging = true;
        joystickElement.setPointerCapture(e.pointerId);
      }
      function pointerMove(e) {
        if (!dragging) return;
        const bounds = joystickElement.getBoundingClientRect();
        const x = e.clientX - bounds.left;
        const y = e.clientY - bounds.top;
        let deltaX = x - centerX;
        let deltaY = y - centerY;
        const distance = Math.sqrt(deltaX * deltaX + deltaY * deltaY);
        if (distance > maxDistance) {
          const angle = Math.atan2(deltaY, deltaX);
          deltaX = Math.cos(angle) * maxDistance;
          deltaY = Math.sin(angle) * maxDistance;
        }
        handle.style.transform = `translate(${deltaX}px, ${deltaY}px)`;
        let normalizedX = ((deltaX + maxDistance) / (2 * maxDistance)) * 100;
        let normalizedY = ((deltaY + maxDistance) / (2 * maxDistance)) * 100;
        updateCallback(normalizedX, normalizedY);
      }
      function pointerUp(e) {
        dragging = false;
        handle.style.transform = `translate(0px, 0px)`;
        updateCallback(50, 50);
      }
      joystickElement.addEventListener('pointerdown', pointerDown);
      joystickElement.addEventListener('pointermove', pointerMove);
      joystickElement.addEventListener('pointerup', pointerUp);
      joystickElement.addEventListener('pointerleave', pointerUp);
    }
    const joystickVitrectomo = document.getElementById('joystick-vitrectomo');
    initJoystick(joystickVitrectomo, updateVitrectomoPosition);
    const joystickLight = document.getElementById('joystick-light');
    initJoystick(joystickLight, updateEndoLightEffect);
    document.getElementById('vitrectomo-z-slider').addEventListener('input', function(){
      currentDepth = parseInt(this.value);
      updateVitrectomoPosition(vitrectomoJoystickX, vitrectomoJoystickX);
    });
    document.getElementById('endo-z-slider').addEventListener('input', function(){
      updateEndoLightEffect(lightJoystickX, lightJoystickY);
    });

    /* FUNCIONALIDADES ADICIONALES (integrando práctica6) */
    // Evento global en la retina para instrumentos que usan clic (láser y vitrectomía)
    document.getElementById('retina').addEventListener('click', function(e){
      if(activeInstrument === 'laser-probe') {
        laserFunction(e);
      } else if(activeInstrument === 'vitrectome') {
        vitrectomyFunction(e);
      }
    });
    // Función para la fotocoagulación láser
    function laserFunction(e) {
      const retina = document.getElementById('retina');
      const rect = retina.getBoundingClientRect();
      const laserSpot = document.createElement('div');
      laserSpot.className = 'laser-spot';
      laserSpot.style.left = (e.clientX - rect.left - 12) + 'px';
      laserSpot.style.top = (e.clientY - rect.top - 12) + 'px';
      retina.appendChild(laserSpot);
      const burnMark = document.createElement('div');
      burnMark.className = 'laser-burn';
      burnMark.style.left = (e.clientX - rect.left - 3) + 'px';
      burnMark.style.top = (e.clientY - rect.top - 3) + 'px';
      retina.appendChild(burnMark);
      setTimeout(() => {
        laserSpot.remove();
      }, 2500);
    }
    // Función para simular la vitrectomía
    function vitrectomyFunction(e) {
      const retina = document.getElementById('retina');
      const rect = retina.getBoundingClientRect();
      const particleCount = 10;
      for (let i = 0; i < particleCount; i++) {
        const particle = document.createElement('div');
        particle.className = 'vitreous-particle';
        particle.style.left = (e.clientX - rect.left + (Math.random()*20 - 10)) + 'px';
        particle.style.top = (e.clientY - rect.top + (Math.random()*20 - 10)) + 'px';
        particle.style.setProperty('--tx', Math.random()*40 - 20);
        particle.style.setProperty('--ty', Math.random()*40 - 20);
        retina.appendChild(particle);
        setTimeout(() => particle.remove(), 1500);
      }
      vitreousRemoved = Math.min(100, vitreousRemoved + 0.8);
      document.getElementById('vitreous-progress').innerText = `${vitreousRemoved.toFixed(0)}%`;
    }
    // Función para realizar la inyección (según práctica6)
    function performInjection() {
      showAlert("FASE 4: Inyección", "Inyectando solución salina equilibrada...");
      const retina = document.getElementById('retina');
      retina.style.background = `
        radial-gradient(circle at 35% 45%, rgba(100,150,255,0.2) 0%, rgba(50,100,255,0.3) 70%, rgba(20,50,255,0.2) 100%),
        repeating-linear-gradient(45deg, rgba(100,150,255,0.1) 0px, rgba(100,150,255,0.1) 1px, transparent 1px, transparent 10px)`;
      for (let i = 0; i < 30; i++) {
        setTimeout(() => {
          const bubble = document.createElement('div');
          bubble.className = 'injection-bubble';
          bubble.style.left = `${50 + Math.random()*20}%`;
          bubble.style.top = `${50 + Math.random()*20}%`;
          bubble.style.width = `${5 + Math.random()*15}px`;
          bubble.style.height = bubble.style.width;
          bubble.style.setProperty('--tx', Math.random()*100 - 50);
          bubble.style.setProperty('--ty', Math.random()*100 - 50);
          retina.appendChild(bubble);
          setTimeout(() => bubble.remove(), 4000);
        }, i*150);
      }
      setTimeout(() => {
        showAlert("Procedimiento Completado", "Vitrectomía posterior finalizada con éxito");
      }, 5000);
    }
    /* FUNCIONALIDAD DE PEELING (se activa con el botón “Peeling”) */
    let isPeelingInitialized = false;
    let tearPoints = [];
    let peelAccumulated = 0;
    const peelStep = 30;
    function initPeeling() {
      if(document.getElementById('membrane')) return;
      const retina = document.getElementById('retina');
      // Se crea el overlay de la membrana con tamaño 50%
      const membrane = document.createElement('div');
      membrane.id = 'membrane';
      membrane.style.position = 'absolute';
      membrane.style.top = '50%';
      membrane.style.left = '50%';
      membrane.style.width = '50%';
      membrane.style.height = '50%';
      /* Fondo similar al de práctica6, con tonos que simulan el color de la retina central */
      membrane.style.background = 'radial-gradient(circle at center, rgba(255,255,255,0.3) 0%, rgba(255,255,255,0.1) 60%, transparent 100%)';
      membrane.style.borderRadius = '50%';
      membrane.style.transform = 'translate(-50%, -50%)';
      membrane.style.cursor = 'grab';
      retina.appendChild(membrane);
      // Se crea el SVG que servirá para dibujar el tearPath y gestionar la máscara
      const peelCanvas = document.createElementNS("http://www.w3.org/2000/svg", "svg");
      peelCanvas.setAttribute('id', 'peelCanvas');
      peelCanvas.setAttribute('style', 'position:absolute; top:0; left:0; width:100%; height:100%; pointer-events:auto;');
      membrane.appendChild(peelCanvas);
      // Se crea un defs con un mask que cubre toda la membrana
      const defs = document.createElementNS("http://www.w3.org/2000/svg", "defs");
      const mask = document.createElementNS("http://www.w3.org/2000/svg", "mask");
      mask.setAttribute("id", "peelMask");
      // Fondo blanco (todo visible)
      const fullCircle = document.createElementNS("http://www.w3.org/2000/svg", "circle");
      fullCircle.setAttribute("cx", "50%");
      fullCircle.setAttribute("cy", "50%");
      fullCircle.setAttribute("r", "50%");
      fullCircle.setAttribute("fill", "white");
      mask.appendChild(fullCircle);
      defs.appendChild(mask);
      peelCanvas.appendChild(defs);
      // Se aplica la máscara al overlay
      membrane.style.webkitMaskImage = "url(#peelMask)";
      membrane.style.maskImage = "url(#peelMask)";
      // Se añade el path que se irá dibujando durante el peeling
      const tearPath = document.createElementNS("http://www.w3.org/2000/svg", "path");
      tearPath.setAttribute('id', 'tearPath');
      peelCanvas.appendChild(tearPath);
      // Se añade la “pinza” (punto de control) sin forzar que inicie siempre en el centro
      const pinza = document.createElementNS("http://www.w3.org/2000/svg", "circle");
      pinza.setAttribute('id', 'pinza');
      pinza.setAttribute('cx', '50%');
      pinza.setAttribute('cy', '50%');
      pinza.setAttribute('r', '7');
      pinza.setAttribute('fill', 'green');
      peelCanvas.appendChild(pinza);
      let isDragging = false;
      tearPoints = [];
      pinza.addEventListener('mousedown', (event) => {
        if(event.button !== 0) return;
        isDragging = true;
        tearPoints = [];
        const pt = getSVGPoint(event, peelCanvas);
        tearPoints.push([pt.x, pt.y]);
        pinza.style.cursor = 'grabbing';
      });
      peelCanvas.addEventListener('mousemove', (event) => {
        if(!isDragging) return;
        const pt = getSVGPoint(event, peelCanvas);
        pinza.setAttribute('cx', pt.x);
        pinza.setAttribute('cy', pt.y);
        tearPoints.push([pt.x, pt.y]);
        updateTearPath(tearPoints, tearPath);
        const memRect = membrane.getBoundingClientRect();
        const center = { x: memRect.width / 2, y: memRect.height / 2 };
        const current = { x: pt.x, y: pt.y };
        const dist = Math.hypot(current.x - center.x, current.y - center.y);
        const radius = memRect.width / 2;
        if(dist >= radius) {
          activarDesgarroPeel(membrane, pinza, peelCanvas);
          isDragging = false;
        }
      });
      document.addEventListener('mouseup', () => {
        if(isDragging) {
          isDragging = false;
          pinza.style.cursor = 'grab';
          tearPoints = [];
          updateTearPath([], tearPath);
        }
      });
      isPeelingInitialized = true;
    }
    function getSVGPoint(event, svg) {
      let pt = svg.createSVGPoint();
      pt.x = event.clientX;
      pt.y = event.clientY;
      return pt.matrixTransform(svg.getScreenCTM().inverse());
    }
    function updateTearPath(points, pathElement) {
      if(points.length === 0) {
        pathElement.setAttribute('d', '');
        return;
      }
      let d = `M ${points[0][0]} ${points[0][1]}`;
      for(let i = 1; i < points.length; i++){
        d += ` L ${points[i][0]} ${points[i][1]}`;
      }
      pathElement.setAttribute('d', d);
    }
    // Cuando se completa el corte, se añade el trayecto como “hueco” en la máscara para que se vea el fondo de la retina en esa zona
    function activarDesgarroPeel(membrane, pinza, peelCanvas) {
      peelAccumulated += peelStep;
      const tearPathEl = document.getElementById('tearPath');
      const d = tearPathEl.getAttribute('d');
      if(d) {
        const mask = peelCanvas.querySelector("mask#peelMask");
        const hole = document.createElementNS("http://www.w3.org/2000/svg", "path");
        hole.setAttribute("d", d);
        hole.setAttribute("fill", "black");
        mask.appendChild(hole);
        updateTearPath([], tearPathEl);
        tearPoints = [];
        if(peelAccumulated >= 100) {
          showAlert("Peeling Completado", "La membrana ha sido removida. Proceda al siguiente paso.");
        }
      } else {
        membrane.remove();
        showAlert("Peeling Completado", "La membrana ha sido removida. Proceda al siguiente paso.");
      }
      peelAccumulated = 0;
      pinza.style.cursor = 'grab';
    }
  </script>
</body>
</html>
