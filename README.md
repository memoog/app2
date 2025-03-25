<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Simulación de Vitrectomía con Profundidad 3D</title>
  <style>
    /* ================== ESTILOS GENERALES ================== */
    body {
      margin: 0;
      padding: 0;
      background-color: #000;
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
    /* ================== NUEVA RETINA ================== */
    .retina-container {
      position: relative;
      width: 100%;
      height: 100%;
      transform-style: preserve-3d;
      perspective: 1200px;
    }
    .retina-sphere {
      position: absolute;
      width: 100%;
      height: 100%;
      background: radial-gradient(circle at center, #400000 0%, #300000 40%, #200000 70%, #100000 90%);
      border-radius: 50%;
      transform: rotateX(20deg) translateZ(-150px);
      box-shadow: inset 0 0 150px rgba(200, 0, 0, 0.3), inset 0 0 50px rgba(255, 0, 0, 0.2), 0 0 100px rgba(0, 0, 0, 0.9);
      overflow: hidden;
    }
    .retina-texture {
      position: absolute;
      width: 100%;
      height: 100%;
      background: radial-gradient(circle at 50% 50%, rgba(255, 200, 200, 0.3) 0%, rgba(255, 150, 150, 0.2) 80%), url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="100" height="100" viewBox="0 0 100 100"><circle cx="50" cy="50" r="40" fill="none" stroke="rgba(255, 100, 100, 0.2)" stroke-width="1"/></svg>');
      background-size: 100%, 20px 20px;
      opacity: 0.9;
      border-radius: 50%;
      mix-blend-mode: multiply;
    }
    .blood-vessels {
      position: absolute;
      width: 100%;
      height: 100%;
      opacity: 0.3;
      transition: opacity 0.5s;
      filter: drop-shadow(0 0 2px rgba(255, 100, 100, 0.3));
    }
    .macula {
      position: absolute;
      width: 80px;
      height: 80px;
      background: radial-gradient(circle at center, rgba(255, 200, 200, 0.3) 0%, rgba(255, 150, 150, 0.4) 40%, rgba(255, 100, 100, 0.5) 70%);
      border-radius: 50%;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      filter: blur(10px);
      box-shadow: 0 0 30px rgba(255, 100, 100, 0.3), inset 0 0 20px rgba(255, 100, 100, 0.4);
    }
    .optic-disc {
      position: absolute;
      width: 70px;
      height: 70px;
      background-color: rgba(200, 100, 100, 0.3);
      border-radius: 50%;
      top: 50%;
      left: 25%;
      transform: translate(-50%, -50%);
      box-shadow: inset 0 0 15px rgba(200, 50, 50, 0.4), 0 0 20px rgba(200, 50, 50, 0.3);
    }
    .optic-cup {
      position: absolute;
      width: 30px;
      height: 30px;
      background-color: rgba(200, 50, 50, 0.4);
      border-radius: 50%;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      box-shadow: inset 0 0 10px rgba(150, 30, 30, 0.5);
    }
    /* ================== SIMULACIÓN PRINCIPAL ================== */
    #eye-chamber {
      position: absolute;
      width: 100%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    #retina {
      position: relative;
      width: 80vmin;
      height: 80vmin;
      border-radius: 50%;
      overflow: hidden;
      transform-style: preserve-3d;
      transform: rotateX(20deg) translateZ(50px);
      transition: transform 0.3s ease;
    }
    #light-mask {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0, 0, 0, 0.85);
      -webkit-mask-image: radial-gradient(circle at var(--light-x, 50%) var(--light-y, 50%), transparent var(--light-size, 100px), black calc(var(--light-size, 100px) + 40px));
      mask-image: radial-gradient(circle at var(--light-x, 50%) var(--light-y, 50%), transparent var(--light-size, 100px), black calc(var(--light-size, 100px) + 40px));
      transition: all 0.3s ease;
      border-radius: 50%;
    }
    #light-reflection {
      position: absolute;
      width: 100%;
      height: 100%;
      background: radial-gradient(circle at var(--light-x, 50%) var(--light-y, 50%), rgba(255, 255, 220, 0.9) calc(var(--light-size, 100px) * 0.2), rgba(255, 200, 150, 0.5) calc(var(--light-size, 100px) * 0.4), transparent var(--light-size, 100px));
      pointer-events: none;
      opacity: 0;
      transition: opacity 0.3s ease;
      border-radius: 50%;
    }
    /* Endoiluminador: su tamaño físico se mantiene fijo */
    #endoilluminator {
      position: absolute;
      bottom: 40%;
      left: 5%;
      width: 20px;
      height: 150px;
      background: linear-gradient(to bottom, #444, #888);
      border-radius: 5px;
      cursor: pointer;
      z-index: 100;
      box-shadow: 0 0 15px rgba(255, 255, 255, 0.2);
      transition: all 0.3s ease;
    }
    #endoilluminator::after {
      content: '';
      position: absolute;
      top: 0;
      left: 50%;
      transform: translateX(-50%);
      width: 10px;
      height: 10px;
      background: radial-gradient(circle at center, #fff, #ccc);
      border-radius: 50%;
      box-shadow: 0 0 10px #fff;
    }
    /* Vitrectomo */
    #vitrectomo {
      position: absolute;
      width: 4vmin;
      height: 20vmin;
      background: linear-gradient(to right, #888 0%, #aaa 15%, #ddd 30%, #fff 45%, #ddd 60%, #aaa 85%, #888 100%);
      border-radius: 1vmin;
      z-index: 150;
      box-shadow: 0 0 15px rgba(0, 0, 0, 0.3), inset 0 0 10px rgba(255, 255, 255, 0.2);
      transition: all 0.3s ease;
    }
    #vitrectomo::before {
      content: '';
      position: absolute;
      top: -1vmin;
      left: 50%;
      transform: translateX(-50%);
      width: 3vmin;
      height: 3vmin;
      background: radial-gradient(circle at center, #fff 0%, #ccc 100%);
      border-radius: 50%;
      box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
      z-index: 2;
    }
    #vitrectomo::after {
      content: '';
      position: absolute;
      bottom: 0;
      left: 50%;
      transform: translateX(-50%);
      width: 2vmin;
      height: 4vmin;
      background: linear-gradient(45deg, #ff5555 0%, #ff2222 30%, #880000 100%);
      border-radius: 0.5vmin;
      box-shadow: 0 0 15px rgba(255, 50, 50, 0.5), inset 0 0 5px rgba(0, 0, 0, 0.3);
      filter: blur(0.3vmin);
      transition: all 0.1s ease;
    }
    .vibrate #vitrectomo::after {
      animation: cutting 0.05s linear infinite;
    }
    @keyframes cutting {
      0% { transform: translateX(-50%) rotate(0deg); }
      50% { transform: translateX(-50%) rotate(2deg); }
      100% { transform: translateX(-50%) rotate(-2deg); }
    }
    /* Indicadores */
    .instrument-indicator {
      position: absolute;
      width: 15px;
      height: 15px;
      border-radius: 50%;
      z-index: 200;
      pointer-events: none;
      opacity: 0;
      transition: opacity 0.3s ease;
    }
    #endo-indicator,
    #vitrectomo-indicator {
      background: rgba(255, 100, 100, 0.8);
    }
    /* ================== CONTROLES ================== */
    /* Se eliminó el contenedor global de controles para integrar los sliders en los contenedores de cada joystick */
    .slider-container {
      display: flex;
      flex-direction: column;
      gap: 5px;
      text-align: center;
    }
    .slider-container.z-control {
      width: 100px;
    }
    input[type="range"] {
      width: 100px;
      height: 8px;
      -webkit-appearance: none;
      background: rgba(50, 50, 50, 0.8);
      border-radius: 5px;
      outline: none;
    }
    input[type="range"]::-webkit-slider-thumb {
      -webkit-appearance: none;
      width: 15px;
      height: 15px;
      border-radius: 50%;
      background: #3498db;
      cursor: pointer;
    }
    /* ================== MINI MAP ================== */
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
    /* ================== JOYSTICK ================== */
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
    .joystick-container button {
      margin-bottom: 5px;
      padding: 5px 10px;
      background-color: #3498db;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      font-size: 0.8em;
    }
    .joystick {
      width: 100px;
      height: 100px;
      background: rgba(255, 255, 255, 0.1);
      border: 2px solid rgba(255, 255, 255, 0.3);
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      touch-action: none;
    }
    .joystick .joystick-handle {
      width: 40px;
      height: 40px;
      background: rgba(255, 255, 255, 0.5);
      border-radius: 50%;
      position: relative;
      transition: transform 0.1s ease;
    }
  </style>
</head>
<body>
  <div id="container">
    <div id="eye-chamber">
      <div id="retina">
        <!-- Nueva estructura de retina -->
        <div class="retina-container">
          <div class="retina-sphere">
            <div class="retina-texture"></div>
            <svg class="blood-vessels" viewBox="0 0 600 600">
              <path d="M150,300 Q250,200 300,300 T450,300" fill="none" stroke="rgba(255, 100, 100, 0.3)" stroke-width="4"></path>
              <path d="M150,290 Q250,390 300,290 T450,290" fill="none" stroke="rgba(255, 100, 100, 0.3)" stroke-width="4"></path>
              <path d="M300,150 Q200,250 300,300 T300,450" fill="none" stroke="rgba(255, 100, 100, 0.3)" stroke-width="4"></path>
              <path d="M290,150 Q390,250 290,300 T290,450" fill="none" stroke="rgba(255, 100, 100, 0.3)" stroke-width="4"></path>
              <path d="M150,300 Q200,250 220,270" fill="none" stroke="rgba(255, 150, 150, 0.3)" stroke-width="2.5"></path>
              <path d="M150,290 Q200,330 220,310" fill="none" stroke="rgba(255, 150, 150, 0.3)" stroke-width="2.5"></path>
              <path d="M300,150 Q250,200 270,220" fill="none" stroke="rgba(255, 150, 150, 0.3)" stroke-width="2.5"></path>
              <path d="M290,150 Q330,200 310,220" fill="none" stroke="rgba(255, 150, 150, 0.3)" stroke-width="2.5"></path>
              <path d="M450,300 Q400,250 380,270" fill="none" stroke="rgba(255, 150, 150, 0.3)" stroke-width="2.5"></path>
              <path d="M450,290 Q400,330 380,310" fill="none" stroke="rgba(255, 150, 150, 0.3)" stroke-width="2.5"></path>
              <path d="M300,450 Q250,400 270,380" fill="none" stroke="rgba(255, 150, 150, 0.3)" stroke-width="2.5"></path>
              <path d="M290,450 Q330,400 310,380" fill="none" stroke="rgba(255, 150, 150, 0.3)" stroke-width="2.5"></path>
            </svg>
            <div class="optic-disc">
              <div class="optic-cup"></div>
            </div>
            <div class="macula"></div>
          </div>
        </div>
        <!-- Elementos originales -->
        <div id="light-mask"></div>
        <div id="light-reflection"></div>
        <!-- Vitrectomo -->
        <div id="vitrectomo"></div>
      </div>
      <div id="endoilluminator"></div>
    </div>
    <!-- MINI MAP -->
    <div id="miniMapContainer">
      <svg id="eyeCrossSection" viewBox="0 0 800 600" preserveAspectRatio="xMidYMid meet">
        <defs>
          <radialGradient id="bgGradient" cx="50%" cy="50%" r="70%" fx="50%" fy="50%">
            <stop offset="0%" stop-color="#1E293B" />
            <stop offset="100%" stop-color="#0F172A" />
          </radialGradient>
          <radialGradient id="vitreousGradient" cx="50%" cy="50%" r="70%" fx="45%" fy="45%">
            <stop offset="0%" stop-color="#FFAB91" stop-opacity="0.9" />
            <stop offset="30%" stop-color="#FF7043" stop-opacity="0.95" />
            <stop offset="70%" stop-color="#E64A19" stop-opacity="0.95" />
            <stop offset="100%" stop-color="#BF360C" stop-opacity="1" />
          </radialGradient>
          <radialGradient id="lensGradient" cx="50%" cy="50%" r="50%" fx="40%" fy="40%">
            <stop offset="0%" stop-color="#E0F7FA" stop-opacity="0.9" />
            <stop offset="70%" stop-color="#B2EBF2" stop-opacity="0.8" />
            <stop offset="100%" stop-color="#80DEEA" stop-opacity="0.7" />
          </radialGradient>
          <linearGradient id="irisGradient" x1="0%" y1="0%" x2="100%" y2="100%">
            <stop offset="0%" stop-color="#FF7043" />
            <stop offset="100%" stop-color="#D84315" />
          </linearGradient>
          <linearGradient id="corneaSheen" x1="30%" y1="20%" x2="70%" y2="80%">
            <stop offset="0%" stop-color="#FFFFFF" stop-opacity="0.7" />
            <stop offset="100%" stop-color="#FFFFFF" stop-opacity="0.1" />
          </linearGradient>
          <linearGradient id="lightBeamGradient" x1="0%" y1="0%" x2="100%" y2="100%">
            <stop offset="0%" stop-color="#FFFFFF" stop-opacity="0.9" />
            <stop offset="100%" stop-color="#FFFFFF" stop-opacity="0.3" />
          </linearGradient>
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
          <filter id="glow" x="-50%" y="-50%" width="200%" height="200%">
            <feGaussianBlur stdDeviation="5" result="blur" />
            <feMerge>
              <feMergeNode in="blur" />
              <feMergeNode in="SourceGraphic" />
            </feMerge>
          </filter>
        </defs>
        <rect width="800" height="600" fill="url(#bgGradient)" />
        <circle cx="400" cy="300" r="250" fill="white" filter="url(#dropShadow)" />
        <circle cx="400" cy="300" r="242" fill="none" stroke="#E0E0E0" stroke-width="4" />
        <circle cx="400" cy="300" r="240" fill="url(#vitreousGradient)" />
        <g class="blood-vessels" opacity="0.8">
          <path d="M320 360 Q350 380, 380 390 T450 400" stroke="#B71C1C" stroke-width="1.5" fill="none" />
          <path d="M330 370 Q360 385, 390 395 T460 405" stroke="#B71C1C" stroke-width="1.2" fill="none" />
          <path d="M310 350 Q340 370, 370 380 T440 390" stroke="#C62828" stroke-width="1.8" fill="none" />
          <path d="M350 380 Q380 390, 410 400 T470 405" stroke="#C62828" stroke-width="1" fill="none" />
          <path d="M300 340 Q330 360, 360 370 T430 380" stroke="#D32F2F" stroke-width="1.5" fill="none" />
          <path d="M290 335 Q320 355, 350 365 T425 375" stroke="#D32F2F" stroke-width="1.2" fill="none" />
        </g>
        <ellipse cx="400" cy="150" rx="100" ry="65" fill="url(#lensGradient)" filter="url(#dropShadow)" />
        <ellipse cx="400" cy="150" rx="95" ry="60" fill="none" stroke="#B3E5FC" stroke-width="2" />
        <circle cx="400" cy="150" r="55" fill="none" stroke="url(#irisGradient)" stroke-width="15" />
        <circle cx="400" cy="150" r="30" fill="#000000" />
        <ellipse cx="400" cy="180" rx="50" ry="28" fill="#E1F5FE" opacity="0.7" />
        <ellipse cx="400" cy="180" rx="48" ry="26" fill="url(#corneaSheen)" opacity="0.6" />
        <line id="probeLight" x1="225" y1="100" x2="350" y2="300" stroke="#FFFFFF" stroke-width="9" stroke-linecap="round" />
        <line id="probeLightInner" x1="225" y1="100" x2="350" y2="300" stroke="#FFFFFF" stroke-width="6" stroke-linecap="round" />
        <line id="probeForceps" x1="575" y1="100" x2="450" y2="300" stroke="#FFFFFF" stroke-width="9" stroke-linecap="round" />
        <line id="probeForcepsInner" x1="575" y1="100" x2="450" y2="300" stroke="#FFFFFF" stroke-width="6" stroke-linecap="round" />
        <polygon id="lightEffect" points="350,300 330,360 400,390 420,340" fill="url(#lightBeamGradient)" opacity="0.7" filter="url(#glow)" />
        <polygon id="lightEffectInner" points="350,300 340,330 390,350 400,325" fill="#FFFFFF" opacity="0.8" />
        <circle cx="350" cy="300" r="11" fill="#D0D0D0" />
        <circle cx="350" cy="300" r="9" fill="#F5F5F5" />
        <circle cx="450" cy="300" r="11" fill="#D0D0D0" />
        <circle cx="450" cy="300" r="9" fill="#F5F5F5" />
        <path d="M400 500 L400 550" stroke="#FFCC80" stroke-width="18" stroke-linecap="round" />
        <path d="M400 500 L400 550" stroke="#FFB74D" stroke-width="14" stroke-linecap="round" />
      </svg>
    </div>
    <!-- JOYSTICK CONTAINERS CON SLIDERS INTEGRADOS -->
    <div id="joystick-vitrectomo-container" class="joystick-container">
      <button id="toggle-vitrectomo-joystick">Activar Vitrectomo</button>
      <div id="joystick-vitrectomo" class="joystick">
        <div class="joystick-handle"></div>
      </div>
      <div class="slider-container z-control">
        <label for="vitrectomo-z-slider">Vitrectomo Z:</label>
        <input type="range" id="vitrectomo-z-slider" min="-250" max="-50" value="-150">
      </div>
    </div>
    <div id="joystick-light-container" class="joystick-container">
      <button id="toggle-light-joystick">Activar Endoiluminador</button>
      <div id="joystick-light" class="joystick">
        <div class="joystick-handle"></div>
      </div>
      <div class="slider-container z-control">
        <label for="endo-z-slider">Endoiluminador Z (tamaño luz):</label>
        <input type="range" id="endo-z-slider" min="0" max="200" value="50">
      </div>
    </div>
  </div>
  <script>
    // VARIABLES BASE
    let currentDepth = -150; // para la retina
    let currentZoom = 1;
    let rotationX = 20;
    let rotationY = 0;
    // Valores de joystick (normalizados 0 a 100)
    let lightJoystickX = 50, lightJoystickY = 50;
    let vitrectomoJoystickX = 50, vitrectomoJoystickY = 50;
    // Sliders para distancia Z
    const vitrectomoZSlider = document.getElementById('vitrectomo-z-slider');
    const endoZSlider = document.getElementById('endo-z-slider');
    // Para el vitrectomo
    let currentVitScale = 1;
    let alertStructureTriggered = false;
    let alertBloodTriggered = false;
    let alertRetinaDetachTriggered = false;
    let alertMiniMapTriggered = false;
    let alertPressureTriggered = false;
    let alertDepthWarningTriggered = false;
    // Nuevas variables para alertas del endoiluminador
    let alertStructureLightTriggered = false;
    // Elementos de la retina
    const retinaSphere = document.querySelector('.retina-sphere');
    const bloodVessels = document.querySelector('.blood-vessels');
    const macula = document.querySelector('.macula');
    const retina = document.getElementById('retina');
    // Elementos para la luz
    const lightMask = document.getElementById('light-mask');
    const lightReflection = document.getElementById('light-reflection');
    const distanceSlider = document.getElementById('distance-slider'); // Nota: si se requiere, se puede mantener un control global para la distancia de la retina
    // Instrumentos
    const endoilluminator = document.getElementById('endoilluminator');
    const vitrectomo = document.getElementById('vitrectomo');
    // Mini mapa (elementos SVG)
    const miniProbeLight = document.getElementById('probeLight');
    const miniProbeLightInner = document.getElementById('probeLightInner');
    const miniProbeForceps = document.getElementById('probeForceps');
    const miniProbeForcepsInner = document.getElementById('probeForcepsInner');
    
    // Actualiza la transformación de la retina
    function updateRetinaTransform() {
      retinaSphere.style.transform = `
        rotateX(${rotationX}deg) 
        rotateY(${rotationY}deg) 
        translateZ(${currentDepth}px) 
        scale(${currentZoom})
      `;
      // Efecto parallax
      const parallaxFactor = currentDepth / -150;
      bloodVessels.style.opacity = 0.3 + (parallaxFactor * 0.3);
      macula.style.opacity = 0.5 + (parallaxFactor * 0.3);
    }
    
    // Actualiza la luz (máscara y reflexión) usando el slider del endoiluminador
    function updateMask() {
      const retinaRect = retina.getBoundingClientRect();
      const posX = (lightJoystickX / 100) * retinaRect.width;
      const posY = (lightJoystickY / 100) * retinaRect.height;
      const baseSize = parseInt(endoZSlider.value); // controla el tamaño de la luz
      // Si se utiliza un slider global para la distancia de la retina, se puede calcular un scaleFactor o tamaño ajustado
      const adjustedSize = baseSize; 
      
      lightMask.style.setProperty('--light-x', posX + 'px');
      lightMask.style.setProperty('--light-y', posY + 'px');
      lightMask.style.setProperty('--light-size', adjustedSize + 'px');
      
      lightReflection.style.setProperty('--light-x', posX + 'px');
      lightReflection.style.setProperty('--light-y', posY + 'px');
      lightReflection.style.setProperty('--light-size', adjustedSize + 'px');
      
      const perspectiveX = (posX / retinaRect.width - 0.5) * 20;
      const perspectiveY = (posY / retinaRect.height - 0.5) * -20;
      rotationX = 20 + perspectiveY;
      rotationY = perspectiveX;
      
      updateRetinaTransform();
      updateMiniProbe();
      updateMiniForceps();
    }
    
    // Zoom con la rueda en la retina
    retina.addEventListener('wheel', (e) => {
      e.preventDefault();
      if (e.deltaY < 0) {
        currentDepth += 10;
        if (currentDepth > -50) currentDepth = -50;
      } else {
        currentDepth -= 10;
        if (currentDepth < -250) currentDepth = -250;
      }
      updateRetinaTransform();
      updateVitrectomoPosition();
    });
    
    // Actualiza la posición del vitrectomo según el slider "Vitrectomo Z"
    function updateVitrectomoPosition() {
      vitrectomo.style.left = vitrectomoJoystickX + "%";
      vitrectomo.style.top = vitrectomoJoystickY + "%";
      let vitZ = parseInt(vitrectomoZSlider.value);
      currentVitScale = 1 - ((Math.abs(vitZ) - 50) / 300);
      vitrectomo.style.transform = `translate(-50%, -50%) scale(${currentVitScale})`;
    }
    
    // Animación del vitrectomo (genera partículas)
    let vitrectomoActive = false;
    let vitrectomoAnimationFrame;
    function animateVitrectomo() {
      if (vitrectomoActive) {
        const vitRect = vitrectomo.getBoundingClientRect();
        const vitX = vitRect.left + vitRect.width / 2;
        const vitY = vitRect.top + vitRect.height / 2;
        const particle = document.createElement('div');
        particle.style.position = 'absolute';
        const particleSize = 3;
        particle.style.width = particleSize + 'px';
        particle.style.height = particleSize + 'px';
        particle.style.backgroundColor = `rgba(255, 255, 255, ${Math.random() * 0.5 + 0.5})`;
        particle.style.borderRadius = '50%';
        const retinaRect = retina.getBoundingClientRect();
        particle.style.left = (vitX - retinaRect.left + (Math.random() * 6 - 3)) + 'px';
        particle.style.top = (vitY - retinaRect.top + (Math.random() * 6 - 3)) + 'px';
        particle.style.zIndex = '150';
        retina.appendChild(particle);
        const duration = 300 + Math.random() * 500;
        let start = null;
        function animateParticle(timestamp) {
          if (!start) start = timestamp;
          const progress = timestamp - start;
          if (progress < duration) {
            particle.style.opacity = (1 - (progress / duration)).toString();
            requestAnimationFrame(animateParticle);
          } else {
            retina.removeChild(particle);
          }
        }
        requestAnimationFrame(animateParticle);
        vitrectomoAnimationFrame = requestAnimationFrame(animateVitrectomo);
      }
    }
    
    function toggleLight() {
      lightOn = !lightOn;
      lightMask.style.display = lightOn ? 'block' : 'none';
      lightReflection.style.opacity = lightOn ? '1' : '0';
    }
    
    let lightOn = false;
    function toggleVitrectomo() {
      vitrectomoOn = !vitrectomoOn;
      if (vitrectomoOn) {
        vitrectomo.classList.add('vibrate');
        startVitrectomoAnimation();
      } else {
        vitrectomo.classList.remove('vibrate');
        stopVitrectomoAnimation();
      }
    }
    let vitrectomoOn = false;
    function startVitrectomoAnimation() {
      vitrectomoActive = true;
      animateVitrectomo();
    }
    function stopVitrectomoAnimation() {
      vitrectomoActive = false;
      cancelAnimationFrame(vitrectomoAnimationFrame);
    }
    
    // Rotación de la retina (cuando no se usan instrumentos)
    let mouseDown = false, initialX, initialY;
    retina.addEventListener('mousedown', (e) => {
      mouseDown = true;
      initialX = e.clientX;
      initialY = e.clientY;
    });
    document.addEventListener('mouseup', () => { mouseDown = false; });
    document.addEventListener('mousemove', (e) => {
      if (mouseDown && !lightOn && !vitrectomoOn) {
        const deltaX = (e.clientX - initialX) / 50;
        const deltaY = (e.clientY - initialY) / 50;
        retina.style.transform = `rotateX(${20 - deltaY}deg) rotateY(${deltaX}deg) translateZ(50px)`;
      }
    });
    
    // Función para actualizar la línea del mini mapa para el endoiluminador
    function updateMiniProbe() {
      const defaultTipX = 350;
      const defaultTipY = 300;
      const scaleX = 2.5;
      const scaleY = 1.8;
      const offsetX = (lightJoystickX - 50) * scaleX;
      const offsetY = (lightJoystickY - 50) * scaleY;
      let tipX = defaultTipX + offsetX;
      let tipY = defaultTipY + offsetY;
      miniProbeLight.setAttribute('x2', tipX);
      miniProbeLightInner.setAttribute('x2', tipX);
      miniProbeLight.setAttribute('y2', tipY);
      miniProbeLightInner.setAttribute('y2', tipY);
      let depthFactor = (-currentDepth - 50) / 200;
      let strokeWidth = 4 - depthFactor * 3;
      miniProbeLight.setAttribute('stroke-width', strokeWidth);
      miniProbeLightInner.setAttribute('stroke-width', strokeWidth);
      let strokeColor = depthFactor > 0.8 ? "red" : "#FFFFFF";
      miniProbeLight.setAttribute('stroke', strokeColor);
      miniProbeLightInner.setAttribute('stroke', strokeColor);
    }
    
    // Función para actualizar la línea del mini mapa para el vitrectomo
    function updateMiniForceps() {
      const defaultTipX = 450;
      const scaleX = 2.5;
      const scaleY = 1.8;
      const offsetX = (vitrectomoJoystickX - 50) * scaleX;
      const offsetY = (vitrectomoJoystickY - 50) * scaleY;
      let tipX = defaultTipX + offsetX;
      let tipY = 300 + offsetY;
      miniProbeForceps.setAttribute('x2', tipX);
      miniProbeForcepsInner.setAttribute('x2', tipX);
      miniProbeForceps.setAttribute('y2', tipY);
      miniProbeForcepsInner.setAttribute('y2', tipY);
      let depthFactor = (-currentDepth - 50) / 200;
      let strokeWidth = 2 + depthFactor * 3;
      miniProbeForceps.setAttribute('stroke-width', strokeWidth);
      miniProbeForcepsInner.setAttribute('stroke-width', strokeWidth);
      let strokeColor = depthFactor > 0.8 ? "red" : "#FFFFFF";
      miniProbeForceps.setAttribute('stroke', strokeColor);
      miniProbeForcepsInner.setAttribute('stroke', strokeColor);
    }
    
    // ================== IMPLEMENTACIÓN DEL JOYSTICK ==================
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
    
    // Inicializa el joystick para el endoiluminador (derecha)
    const joystickLight = document.getElementById('joystick-light');
    initJoystick(joystickLight, (normX, normY) => {
      lightJoystickX = normX;
      lightJoystickY = normY;
      updateMask();
    });
    
    // Inicializa el joystick para el vitrectomo (izquierda)
    const joystickVitrectomo = document.getElementById('joystick-vitrectomo');
    initJoystick(joystickVitrectomo, (normX, normY) => {
      vitrectomoJoystickX = normX;
      vitrectomoJoystickY = normY;
      updateMiniForceps();
      updateVitrectomoPosition();
    });
    
    // Botones de activación
    const toggleLightJoystickButton = document.getElementById('toggle-light-joystick');
    const toggleVitrectomoJoystickButton = document.getElementById('toggle-vitrectomo-joystick');
    toggleLightJoystickButton.addEventListener('click', () => {
      toggleLight();
      toggleLightJoystickButton.textContent = lightOn ? 'Desactivar Endoiluminador' : 'Activar Endoiluminador';
    });
    toggleVitrectomoJoystickButton.addEventListener('click', () => {
      toggleVitrectomo();
      toggleVitrectomoJoystickButton.textContent = vitrectomoOn ? 'Desactivar Vitrectomo' : 'Activar Vitrectomo';
    });
    
    // ================== ALERTAS ==================
    function checkAlerts() {
      const retinaRect = retina.getBoundingClientRect();
      const retinaCenterX = retinaRect.left + retinaRect.width / 2;
      const retinaCenterY = retinaRect.top + retinaRect.height / 2;
      const retinaRadius = retinaRect.width / 2;
      
      // ALERTAS PARA EL VITRECTOMO
      const vitX = (vitrectomoJoystickX / 100) * retinaRect.width + retinaRect.left;
      const vitY = (vitrectomoJoystickY / 100) * retinaRect.height + retinaRect.top;
      const dx = vitX - retinaCenterX;
      const dy = vitY - retinaCenterY;
      const distanceFromCenter = Math.sqrt(dx * dx + dy * dy);
      
      // Aproximación al borde (fondo del ojo)
      if (distanceFromCenter > retinaRadius * 0.85) {
        if (!alertStructureTriggered) {
          alertStructureTriggered = true;
          alert("Alerta: Aproximación al borde del fondo del ojo.");
        }
      } else {
        alertStructureTriggered = false;
      }
      
      // Alerta de colisión de vena (solo para el vitrectomo)
      if (currentVitScale < 0.5 && distanceFromCenter < retinaRadius * 0.3) {
        if (!alertBloodTriggered) {
          alertBloodTriggered = true;
          alert("Alerta: Colisión de vena detectada en el Vitrectomo (tamaño crítico).");
          retina.innerHTML = "";
          retina.style.backgroundColor = "red";
          const miniMap = document.getElementById("eyeCrossSection");
          for (let i = 0; i < 5; i++) {
            let circle = document.createElementNS("http://www.w3.org/2000/svg", "circle");
            circle.setAttribute("cx", 400 + Math.random() * 100 - 50);
            circle.setAttribute("cy", 300 + Math.random() * 100 - 50);
            circle.setAttribute("r", 5);
            circle.setAttribute("fill", "red");
            miniMap.appendChild(circle);
          }
        }
      } else {
        alertBloodTriggered = false;
      }
      
      // Alerta de desprendimiento de la retina
      if (currentDepth <= -250) {
        if (!alertRetinaDetachTriggered) {
          alertRetinaDetachTriggered = true;
          alert("Alerta: Desprendimiento de la retina detectado.");
        }
      } else {
        alertRetinaDetachTriggered = false;
      }
      
      // Alerta de contacto con el iris o cristalino (mini mapa) – se utiliza el mini probe de la luz
      let miniTipX = parseFloat(miniProbeLight.getAttribute("x2"));
      let miniTipY = parseFloat(miniProbeLight.getAttribute("y2"));
      let dIris = Math.sqrt((miniTipX - 400) ** 2 + (miniTipY - 150) ** 2);
      let inLens = (((miniTipX - 400) ** 2) / (100 * 100) + ((miniTipY - 150) ** 2) / (65 * 65)) <= 1;
      if (dIris < 55 || inLens) {
        if (!alertMiniMapTriggered) {
          alertMiniMapTriggered = true;
          alert("Alerta: Contacto con el iris o cristalino en el mini mapa.");
        }
      } else {
        alertMiniMapTriggered = false;
      }
      
      // Alerta de presión alta
      // (se activa cuando la distancia (en un slider global o variable) es alta; en este ejemplo, se utiliza la condición sobre currentDepth)
      if (parseInt(distanceSlider.value) >= 350) {
        if (!alertPressureTriggered) {
          alertPressureTriggered = true;
          alert("Alerta: Presión alta. El nervio adyacente a la mácula se iluminará en blanco intenso.");
        }
        const opticDisc = document.querySelector('.optic-disc');
        opticDisc.style.backgroundColor = "white";
        opticDisc.style.boxShadow = "0 0 30px white, inset 0 0 20px white";
      } else {
        alertPressureTriggered = false;
        const opticDisc = document.querySelector('.optic-disc');
        opticDisc.style.backgroundColor = "rgba(200, 100, 100, 0.3)";
        opticDisc.style.boxShadow = "inset 0 0 15px rgba(200, 50, 50, 0.4), 0 0 20px rgba(200, 50, 50, 0.3)";
      }
      
      // Alerta de precaución por profundidad
      if (currentDepth < -240 && currentDepth > -250) {
        if (!alertDepthWarningTriggered) {
          alertDepthWarningTriggered = true;
          alert("Precaución: Se acerca demasiado al fondo del ojo.");
        }
      } else {
        alertDepthWarningTriggered = false;
      }
      
      // ================== ALERTAS PARA EL ENDOILUMINADOR ==================
      const endoX = (lightJoystickX / 100) * retinaRect.width + retinaRect.left;
      const endoY = (lightJoystickY / 100) * retinaRect.height + retinaRect.top;
      const dxLight = endoX - retinaCenterX;
      const dyLight = endoY - retinaCenterY;
      const distanceFromCenterLight = Math.sqrt(dxLight * dxLight + dyLight * dyLight);
      if (distanceFromCenterLight > retinaRadius * 0.85) {
        if (!alertStructureLightTriggered) {
          alertStructureLightTriggered = true;
          alert("Alerta: Aproximación al borde del fondo del ojo (Endoiluminador).");
        }
      } else {
        alertStructureLightTriggered = false;
      }
    }
    setInterval(checkAlerts, 250);
    
    // EVENTOS DE SLIDERS (los controles globales que afecten la retina se pueden conservar si se desea)
    // Si no se requiere el slider global de distancia, se puede comentar o eliminar.
    if(distanceSlider) {
      distanceSlider.addEventListener('input', updateMask);
    }
    vitrectomoZSlider.addEventListener('input', updateVitrectomoPosition);
    endoZSlider.addEventListener('input', updateMask);
    const controls = document.querySelectorAll('input');
    controls.forEach(control => {
      control.addEventListener('mousedown', () => { control.style.transform = 'scale(0.98)'; });
      control.addEventListener('mouseup', () => { control.style.transform = ''; });
    });
    
    updateMask();
    lightMask.style.display = 'none';
    updateVitrectomoPosition();
  </script>
</body>
</html>
