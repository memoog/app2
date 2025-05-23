<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Simulador de Extracción de Hemorragia Vítrea</title>
  <link rel="stylesheet" href="d.css">
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/animejs/3.2.1/anime.min.js"></script>
 <style>
/* Estilos base */
* {
  box-sizing: border-box;
  -webkit-tap-highlight-color: transparent;
  touch-action: manipulation;
}

body {
  margin: 0;
  padding: 0;
  font-family: 'Roboto', 'Segoe UI', system-ui, sans-serif;
  background: #0a0a12;
  color: #e0e0e0;
  overflow: hidden;
  height: 100vh;
  width: 100vw;
}

#container {
  position: relative;
  width: 100%;
  height: 100%;
  perspective: 1200px;
  background: radial-gradient(circle at center, #1a1a2a 0%, #0a0a12 100%);
}

/* Sistema de alertas profesional */
#alert-system {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 10000;
  width: 90%;
  max-width: 600px;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.alert-container {
  display: none;
  background: rgba(20, 20, 30, 0.95);
  border-left: 5px solid;
  border-radius: 8px;
  padding: 15px 20px;
  align-items: center;
  box-shadow: 0 10px 25px rgba(0,0,0,0.5);
  backdrop-filter: blur(8px);
  animation: alertSlideIn 0.3s ease-out forwards;
  transform-origin: top center;
}

@keyframes alertSlideIn {
  0% { opacity: 0; transform: translateY(-20px); }
  100% { opacity: 1; transform: translateY(0); }
}

@keyframes alertSlideOut {
  0% { opacity: 1; transform: translateY(0); }
  100% { opacity: 0; transform: translateY(-20px); }
}

/* Estilos específicos para cada tipo de alerta */
.alert-container#wall-collision-alert {
  border-left-color: #ff9933;
  background: rgba(40, 30, 10, 0.95);
  animation: alertShake 0.5s ease-out forwards;
}

@keyframes alertShake {
  0%, 100% { transform: translateX(0) translateY(0); }
  20% { transform: translateX(-10px) translateY(0); }
  40% { transform: translateX(10px) translateY(0); }
  60% { transform: translateX(-10px) translateY(0); }
  80% { transform: translateX(10px) translateY(0); }
}

.alert-container#vessel-damage-alert {
  border-left-color: #ff3333;
  background: rgba(40, 10, 10, 0.95);
}

.alert-container#retina-detachment-alert {
  border-left-color: #ff5555;
  background: rgba(30, 10, 20, 0.95);
}

.alert-container#lens-contact-alert {
  border-left-color: #55aaff;
  background: rgba(10, 20, 40, 0.95);
}

.alert-container#high-iop-alert {
  border-left-color: #ffcc00;
  background: rgba(40, 30, 10, 0.95);
}

.alert-container#retina-fixed-alert {
  border-left-color: #00cc66;
  background: rgba(10, 40, 20, 0.95);
}

.alert-container#vitreous-removed-alert {
  border-left-color: #55ff55;
  background: rgba(10, 40, 10, 0.95);
}

.alert-container#hole-located-alert {
  border-left-color: #ffff55;
  background: rgba(40, 40, 10, 0.95);
}

.alert-container#gas-injected-alert {
  border-left-color: #55aaff;
  background: rgba(10, 20, 40, 0.95);
}

.alert-icon {
  font-size: 1.8rem;
  margin-right: 15px;
  flex-shrink: 0;
}

.alert-content {
  flex-grow: 1;
}

.alert-content h3 {
  margin: 0 0 5px 0;
  color: white;
  font-size: 1.1rem;
  font-weight: 600;
}

.alert-content p {
  margin: 0;
  color: #ddd;
  font-size: 0.9rem;
  line-height: 1.4;
}

.alert-timer {
  height: 3px;
  background: rgba(255,255,255,0.3);
  margin-top: 8px;
  border-radius: 2px;
  overflow: hidden;
}

.alert-timer::after {
  content: '';
  display: block;
  height: 100%;
  width: 100%;
  background: white;
}

.alert-dismiss {
  background: none;
  border: none;
  color: white;
  font-size: 1.5rem;
  cursor: pointer;
  padding: 0 0 0 15px;
  opacity: 0.7;
  transition: opacity 0.2s;
  flex-shrink: 0;
}

.alert-dismiss:hover {
  opacity: 1;
}

/* Retina central - mejorada con texturas realistas */
#eye-chamber {
  position: absolute;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  touch-action: none;
  background: radial-gradient(circle at center, rgba(0,0,0,0.3) 0%, transparent 70%);
}

.retina-container {
  position: relative;
  width: 80vmin;
  height: 80vmin;
  max-width: 600px;
  max-height: 600px;
  transform-style: preserve-3d;
  perspective: 1200px;
  border-radius: 50%;
  overflow: hidden;
  transition: transform 0.3s ease;
  filter: contrast(1.2) brightness(0.9) saturate(1.1);
  will-change: transform, filter;
  box-shadow: 
    inset 0 0 100px rgba(100, 20, 20, 0.3),
    0 0 80px rgba(0, 0, 0, 0.8);
}

.retina-sphere {
  position: absolute;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: radial-gradient(circle at center, 
    #a76a01 0%, 
    #993d03 20%, 
    #712c02d5 40%,
    #843c06 70%,
    #832801 100%);
  transform: rotateX(15deg) translateZ(50px);
  box-shadow: 
    inset 0 0 200px rgba(19, 12, 12, 0.3),
    inset 0 0 80px rgba(166, 78, 5, 0.2),
    0 0 120px rgba(153, 11, 11, 0.9);
  overflow: hidden;
}

.retina-sphere::after {
  content: '';
  position: absolute;
  width: 200%;
  height: 200%;
  left: -50%;
  top: -50%;
  background-image: 
    radial-gradient(circle at 50% 50%, 
      rgba(255, 180, 180, 0.08) 0.5%, 
      transparent 1.2%),
    repeating-linear-gradient(
      45deg,
      transparent 0px,
      transparent 2px,
      rgba(255, 255, 255, 0.03) 2px,
      rgba(255, 255, 255, 0.03) 3px
    ),
    repeating-linear-gradient(
      -45deg,
      transparent 0px,
      transparent 3px,
      rgba(150, 50, 50, 0.05) 3px,
      rgba(150, 50, 50, 0.05) 5px
    );
  background-size: 
    60px 60px,
    100% 100%,
    100% 100%;
  transform: rotate(15deg);
  animation: textureMove 120s linear infinite;
  pointer-events: none;
}

@keyframes textureMove {
  0% { transform: rotate(15deg) translate(0,0); }
  100% { transform: rotate(15deg) translate(200px,200px); }
}

.retina-texture {
  position: absolute;
  width: 100%;
  height: 100%;
  background: 
    radial-gradient(circle at 50% 50%, rgba(255,200,200,0.2) 0%, rgba(255,150,150,0.15) 80%),
    url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="100" height="100" viewBox="0 0 100 100"><path d="M${Array.from({length:50},(_,i)=>`${Math.random()*100},${Math.random()*100}`).join(" ")}" stroke="rgba(180,80,80,0.08)" fill="none" stroke-width="0.5"/></svg>');
  background-size: 100%, 15px 15px;
  opacity: 0.8;
  border-radius: 50%;
  mix-blend-mode: overlay;
  filter: contrast(1.3);
  will-change: transform;
}

.macula {
  position: absolute;
  width: 100px;
  height: 100px;
  background: radial-gradient(circle at center, 
    rgba(255,230,200,0.7) 0%, 
    rgba(255,210,170,0.5) 60%, 
    transparent 100%);
  border-radius: 50%;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 2;
  box-shadow: 
    0 0 40px rgba(255, 220, 150, 0.4),
    inset 0 0 15px rgba(255, 200, 100, 0.5);
  animation: pulse 4s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 0.9; transform: translate(-50%, -50%) scale(1); }
  50% { opacity: 1; transform: translate(-50%, -50%) scale(1.05); }
}

.blood-vessels {
  position: absolute;
  width: 100%;
  height: 100%;
  z-index: 3;
  pointer-events: none;
  filter: drop-shadow(0 0 3px rgba(80, 0, 0, 0.7));
  will-change: transform;
}

.blood-vessels-animated {
  animation: vesselPulse 3s ease-in-out infinite;
}

@keyframes vesselPulse {
  0%, 100% { opacity: 0.7; transform: scale(1); }
  50% { opacity: 0.9; transform: scale(1.01); }
}

.retina-nerve {
  position: absolute;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="300" height="300"><path d="M0,0 Q50,50 100,0 T200,0" stroke="rgba(255,255,255,0.06)" stroke-width="1.5" fill="none"/></svg>') center/cover;
  pointer-events: none;
  opacity: 0.6;
  z-index: 4;
}

.optic-disc {
  position: absolute;
  width: 80px;
  height: 80px;
  background: radial-gradient(circle at center, 
    rgba(220,120,120,0.5) 0%, 
    rgba(200,100,100,0.4) 100%);
  border-radius: 50%;
  top: 50%;
  left: 25%;
  transform: translate(-50%, -50%);
  box-shadow: 
    inset 0 0 20px rgba(220,80,80,0.6), 
    0 0 25px rgba(220,80,80,0.5);
  z-index: 5;
  animation: discPulse 5s ease-in-out infinite;
}

@keyframes discPulse {
  0%, 100% { transform: translate(-50%, -50%) scale(1); }
  50% { transform: translate(-50%, -50%) scale(1.03); }
}

.optic-cup {
  position: absolute;
  width: 35px;
  height: 35px;
  background: radial-gradient(circle at center, 
    rgba(220,80,80,0.6) 0%, 
    rgba(200,60,60,0.5) 100%);
  border-radius: 50%;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  box-shadow: inset 0 0 15px rgba(180,50,50,0.7);
  z-index: 6;
}

/* Luz endoiluminador - efectos dinámicos mejorados */
#light-mask {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.95);
  mask-image: radial-gradient(
    circle at var(--light-x, 50%) var(--light-y, 50%), 
    transparent var(--light-size, 80px), 
    black calc(var(--light-size, 80px) + 80px)
  );
  -webkit-mask-image: radial-gradient(
    circle at var(--light-x, 50%) var(--light-y, 50%), 
    transparent var(--light-size, 80px), 
    black calc(var(--light-size, 80px) + 80px)
  );
  transition: all 0.15s ease-out;
  pointer-events: none;
  z-index: 7;
  will-change: mask-image, -webkit-mask-image;
}

#light-reflection {
  position: absolute;
  width: 100%;
  height: 100%;
  background: radial-gradient(
    circle at var(--light-x, 50%) var(--light-y, 50%), 
    rgba(255,255,240,0.95) calc(var(--light-size, 80px)*0.1), 
    rgba(255,220,180,0.6) calc(var(--light-size, 80px)*0.3), 
    transparent var(--light-size, 80px)
  );
  opacity: 0;
  transition: opacity 0.25s ease, transform 0.25s ease;
  border-radius: 50%;
  pointer-events: none;
  z-index: 8;
  mix-blend-mode: soft-light;
  filter: blur(1.5px);
  will-change: opacity, transform;
}

.light-scatter {
  position: absolute;
  width: 300%;
  height: 300%;
  left: -100%;
  top: -100%;
  background: radial-gradient(
    circle at var(--light-x, 50%) var(--light-y, 50%),
    rgba(255, 230, 200, 0.15) 0%,
    transparent 80%
  );
  pointer-events: none;
  z-index: 9;
  opacity: 0;
  transition: opacity 0.3s ease;
  will-change: background, opacity;
}

.light-scatter.active {
  opacity: 0.7;
}

.specular-highlight {
  position: absolute;
  width: 100%;
  height: 100%;
  background: radial-gradient(
    circle at var(--light-x, 50%) var(--light-y, 50%),
    rgba(255, 255, 255, 0.5) 0%,
    transparent 70%
  );
  pointer-events: none;
  z-index: 10;
  mix-blend-mode: overlay;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.specular-highlight.active {
  opacity: 0.4;
}

/* Instrumentos quirúrgicos 3D mejorados */
.instrument {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%) translateZ(0);
  display: none;
  z-index: 10;
  transition: transform 0.1s ease-out;
  will-change: transform;
  transform-style: preserve-3d;
  perspective: 1000px;
}

.instrument-body {
  position: relative;
  width: 100%;
  height: 100%;
  transform-style: preserve-3d;
  transition: all 0.2s ease;
}

.instrument-shadow {
  position: absolute;
  width: 120%;
  height: 120%;
  left: -10%;
  top: -10%;
  background: radial-gradient(
    ellipse at center,
    rgba(0,0,0,0.4) 0%,
    transparent 70%
  );
  transform: rotateX(75deg) translateZ(-10px);
  pointer-events: none;
  z-index: -1;
  filter: blur(5px);
  opacity: 0.7;
  transition: all 0.3s ease;
}

/* Vitrectomo 3D mejorado */
#vitrectome {
  width: 2.5mm;
  height: 15mm;
  transform-style: preserve-3d;
}

#vitrectome .instrument-body {
  background: linear-gradient(to bottom, 
    #999 0%, 
    #ccc 10%, 
    #eee 20%, 
    #fff 40%, 
    #ddd 60%, 
    #aaa 80%,
    #888 100%);
  border: 1px solid #777;
  border-radius: 1.2mm;
  box-shadow: 
    0 0 1.5mm rgba(0,0,0,0.7),
    0 0 3mm rgba(255,255,255,0.15),
    inset 0 0 5px rgba(255,255,255,0.3);
  transform: rotateX(10deg);
}

#vitrectome::before {
  content: '';
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%) translateZ(1px);
  width: 3mm;
  height: 1mm;
  background: linear-gradient(to right, #aaa, #ddd, #aaa);
  border-radius: 0.5mm;
  box-shadow: 0 1px 2px rgba(0,0,0,0.5);
  z-index: 2;
}

#vitrectome::after {
  content: '';
  position: absolute;
  bottom: -1mm;
  left: 50%;
  transform: translateX(-50%) translateZ(-1px);
  width: 1.8mm;
  height: 1.8mm;
  background: radial-gradient(circle, 
    #fff 0%, 
    #bbb 50%, 
    #888 100%);
  border-radius: 50%;
  box-shadow: 
    0 0 0.8mm rgba(0,0,0,0.7),
    0 0 1.5mm rgba(255,255,255,0.3),
    inset 0 0 5px rgba(255,255,255,0.4);
  z-index: 1;
}

#vitrectome .instrument-tip {
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%) translateZ(0);
  width: 1.2mm;
  height: 3mm;
  background: linear-gradient(to bottom, #ccc, #999);
  border-radius: 0.3mm;
  box-shadow: 
    inset 0 -1px 2px rgba(0,0,0,0.5),
    0 0 5px rgba(255,255,255,0.2);
}

#vitrectome .instrument-side {
  position: absolute;
  width: 100%;
  height: 100%;
  background: linear-gradient(to bottom, #888, #aaa, #888);
  transform: rotateY(90deg) translateZ(1.25mm);
  border-radius: 1.2mm;
  box-shadow: inset 0 0 5px rgba(0,0,0,0.5);
}

/* Sonda de Láser 3D mejorada */
#laser-probe {
  width: 0.5mm;
  height: 12mm;
  transform-style: preserve-3d;
}

#laser-probe .instrument-body {
  background: linear-gradient(to bottom, 
    #f00, 
    #f90 20%, 
    #ff0 30%, 
    #f90 70%, 
    #f00);
  border-radius: 0.3mm;
  box-shadow: 
    0 0 1.5mm rgba(255,0,0,0.9),
    0 0 3mm rgba(255,100,100,0.6),
    inset 0 0 5px rgba(255,255,255,0.4);
  transform: rotateX(10deg);
}

#laser-probe::before {
  content: '';
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%) translateZ(1px);
  width: 1.5mm;
  height: 1mm;
  background: linear-gradient(to right, #aaa, #ddd, #aaa);
  border-radius: 0.5mm;
  box-shadow: 0 1px 2px rgba(0,0,0,0.5);
  z-index: 2;
}

#laser-probe .instrument-tip {
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%) translateZ(0);
  width: 0.8mm;
  height: 1.5mm;
  background: radial-gradient(circle, #fff, #f99);
  border-radius: 50%;
  box-shadow: 
    0 0 5px rgba(255,100,100,0.8),
    inset 0 0 3px rgba(255,255,255,0.8);
}

/* Sonda de PFC */
#pfc-probe {
  width: 0.8mm;
  height: 13mm;
  transform-style: preserve-3d;
}

#pfc-probe .instrument-body {
  background: linear-gradient(to bottom, 
    #0af 0%, 
    #5bf 30%, 
    #8df 70%, 
    #0af 100%);
  border: 1px solid #07a;
  border-radius: 0.5mm;
  box-shadow: 
    0 0 1.5mm rgba(0,100,255,0.7),
    0 0 3mm rgba(100,200,255,0.4),
    inset 0 0 5px rgba(255,255,255,0.6);
  transform: rotateX(10deg);
}

#pfc-probe::before {
  content: '';
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%) translateZ(1px);
  width: 2mm;
  height: 1mm;
  background: linear-gradient(to right, #aaa, #ddd, #aaa);
  border-radius: 0.5mm;
  box-shadow: 0 1px 2px rgba(0,0,0,0.5);
  z-index: 2;
}

#pfc-probe .instrument-tip {
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%) translateZ(0);
  width: 1mm;
  height: 1.5mm;
  background: radial-gradient(circle, #fff, #5bf);
  border-radius: 50%;
  box-shadow: 
    0 0 5px rgba(0,150,255,0.8),
    inset 0 0 3px rgba(255,255,255,0.8);
}

/* Cauterio */
#cautery-probe {
  width: 1.5mm;
  height: 12mm;
  transform-style: preserve-3d;
}

#cautery-probe .instrument-body {
  background: linear-gradient(to bottom, 
    #ff3333 0%, 
    #ff6666 20%, 
    #ff9999 40%, 
    #ff6666 70%, 
    #ff3333 100%);
  border: 1px solid #cc0000;
  border-radius: 0.8mm;
  box-shadow: 
    0 0 1.5mm rgba(255,0,0,0.7),
    0 0 3mm rgba(255,100,100,0.4),
    inset 0 0 5px rgba(255,255,255,0.6);
  transform: rotateX(10deg);
}

#cautery-probe::before {
  content: '';
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%) translateZ(1px);
  width: 2mm;
  height: 1mm;
  background: linear-gradient(to right, #aaa, #ddd, #aaa);
  border-radius: 0.5mm;
  box-shadow: 0 1px 2px rgba(0,0,0,0.5);
  z-index: 2;
}

#cautery-probe .instrument-tip {
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%) translateZ(0);
  width: 1.2mm;
  height: 2mm;
  background: radial-gradient(circle, #ff9999, #ff3333);
  border-radius: 0.3mm;
  box-shadow: 
    0 0 5px rgba(255,100,100,0.8),
    inset 0 0 3px rgba(255,255,255,0.8);
  z-index: 1;
}

/* Efecto de cauterio */
.cautery-effect {
  position: absolute;
  width: 30px;
  height: 30px;
  background: radial-gradient(circle, 
    rgba(255,200,200,0.9) 0%, 
    rgba(255,100,100,0.7) 70%, 
    transparent 90%);
  border-radius: 50%;
  pointer-events: none;
  z-index: 18;
  animation: cauteryFade 1s ease-out forwards;
  filter: blur(1.5px);
  box-shadow: 0 0 20px rgba(255,100,100,0.8);
}

@keyframes cauteryFade {
  0% { transform: scale(0.8); opacity: 1; }
  50% { transform: scale(1.5); opacity: 0.9; }
  100% { transform: scale(1.5); opacity: 0; }
}

/* Burbujas de PFC */
.pfc-bubble {
  position: absolute;
  background: rgba(100, 255, 255, 0.7);
  border-radius: 50%;
  pointer-events: none;
  z-index: 5;
  filter: blur(1px);
  box-shadow: 
    0 0 10px rgba(100, 255, 255, 0.8),
    inset 0 0 5px rgba(255, 255, 255, 0.8);
  transform: translate(-50%, -50%);
  will-change: transform, opacity;
}

.instrument-active {
  animation: instrumentVibrate 0.1s linear infinite;
}

@keyframes instrumentVibrate {
  0% { transform: translate(-50%, -50%) rotate(0.5deg) translateZ(0); }
  50% { transform: translate(-50%, -50%) rotate(-0.5deg) translateZ(0); }
  100% { transform: translate(-50%, -50%) rotate(0.5deg) translateZ(0); }
}

/* Interfaz de usuario profesional */
.instrument-panel {
  position: absolute;
  top: 50%;
  left: 20px;
  transform: translateY(-50%);
  background: rgba(0,0,0,0.85);
  padding: 15px 10px;
  border-radius: 12px;
  z-index: 5000;
  display: flex;
  flex-direction: column;
  box-shadow: 
    0 6px 20px rgba(0,0,0,0.6),
    inset 0 0 15px rgba(255,255,255,0.15);
  backdrop-filter: blur(8px);
  border: 1px solid rgba(255,255,255,0.15);
}

.toggle-btn {
  background: #1a3a8a;
  color: white;
  padding: 12px 18px;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  font-size: clamp(0.85rem, 2.5vw, 1rem);
  margin: 8px 5px;
  min-width: 100px;
  transition: all 0.2s;
  font-weight: 500;
  box-shadow: 
    0 3px 8px rgba(0,0,0,0.4),
    inset 0 0 8px rgba(255,255,255,0.15);
  position: relative;
  overflow: hidden;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.toggle-btn::after {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 5px;
  height: 5px;
  background: rgba(255, 255, 255, 0.6);
  opacity: 0;
  border-radius: 100%;
  transform: scale(1, 1) translate(-50%, -50%);
  transform-origin: 50% 50%;
}

.toggle-btn:active::after {
  animation: ripple 0.6s ease-out;
}

@keyframes ripple {
  0% {
    transform: scale(0, 0);
    opacity: 1;
  }
  20% {
    transform: scale(25, 25);
    opacity: 1;
  }
  100% {
    opacity: 0;
    transform: scale(40, 40);
  }
}

.toggle-btn.active {
  background: #3b82f6;
  box-shadow: 
    0 0 20px #3b82f6,
    0 0 40px rgba(59, 130, 246, 0.4);
}

/* Panel de controles mejorado */
.control-panel {
  position: absolute;
  right: 20px;
  top: 190px;
  background: rgba(10,10,20,0.95);
  padding: 20px;
  border-radius: 50px;
  z-index: 5000;
  font-size: clamp(0.75rem, 2vw, 0.9rem);
  width: 180px;
  max-height: 80vh;
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
  box-shadow: 
    0 6px 20px rgba(18, 20, 41, 0.6),
    inset 0 0 15px rgba(255,255,255,0.15);
  border: 1px solid rgba(255,255,255,0.15);
  backdrop-filter: blur(8px);
}

.control-panel h3 {
  margin: 0 0 15px 0;
  color: #2990aa;
  font-size: 1.1rem;
  text-align: center;
  border-bottom: 1px solid #444;
  padding-bottom: 8px;
}

.vital-sign {
  display: flex;
  justify-content: space-between;
  margin: 15px 0;
  flex-wrap: wrap;
}

.vital-label {
  color: #aaa;
  font-size: 0.85em;
  width: 60%;
}

.vital-value {
  font-family: 'Courier New', monospace;
  font-weight: bold;
  font-size: 0.95em;
  width: 40%;
  text-align: right;
  transition: color 0.3s;
}

.normal { color: #2ecc71; text-shadow: 0 0 8px rgba(46, 204, 113, 0.4); }
.warning { color: #f39c12; text-shadow: 0 0 8px rgba(243, 156, 18, 0.4); }
.danger { color: #e74c3c; text-shadow: 0 0 8px rgba(231, 76, 60, 0.4); }

.gauge-container {
  width: 100%;
  height: 10px;
  background: #222;
  border-radius: 5px;
  margin: 8px 0 15px 0;
  overflow: hidden;
  box-shadow: inset 0 0 8px rgba(0,0,0,0.7);
}

.gauge-level {
  height: 100%;
  border-radius: 5px;
  transition: width 0.5s ease;
  box-shadow: inset 0 0 8px rgba(255,255,255,0.3);
}

#iop-gauge .gauge-level {
  background: linear-gradient(to right, #2ecc71, #f39c12, #e74c3c);
}

#perfusion-gauge .gauge-level {
  background: linear-gradient(to right, #3498db, #9b59b6);
}

#vitreous-gauge .gauge-level {
  background: linear-gradient(to right, #1abc9c, #f1c40f);
}

#pfc-gauge .gauge-level {
  background: linear-gradient(to right, #0af, #5bf);
}

/* Joysticks mejorados */
.joystick-container {
  position: absolute;
  bottom: 1px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  z-index: 3000;
}

#joystick-light-container {
  left: 200px;
}

#joystick-vitrectomo-container {
  right: 180px;
}

.joystick {
  width: 100px;
  height: 100px;
  background: rgba(255,255,255,0.1);
  border: 2px solid rgba(255,255,255,0.4);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  touch-action: none;
  position: relative;
  box-shadow: 
    0 6px 15px rgba(0,0,0,0.4),
    inset 0 0 25px rgba(255,255,255,0.15);
  backdrop-filter: blur(8px);
  will-change: transform;
}

.joystick .joystick-handle {
  width: 40px;
  height: 40px;
  background: rgba(255,255,255,0.8);
  border-radius: 50%;
  position: absolute;
  transition: transform 0.1s ease;
  box-shadow: 
    0 0 15px rgba(255,255,255,0.4),
    inset 0 0 8px rgba(255,255,255,0.7);
  will-change: transform;
}

/* Controles Z mejorados */
.slider-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  background: rgba(0,0,0,0.7);
  padding: 15px;
  border-radius: 12px;
  box-shadow: 
    0 4px 10px rgba(0,0,0,0.4),
    inset 0 0 8px rgba(255,255,255,0.15);
  backdrop-filter: blur(8px);
}

.slider-container label {
  font-size: 0.8rem;
  margin-bottom: 10px;
  text-align: center;
  color: #eee;
  font-weight: 500;
  text-shadow: 0 0 8px rgba(0,0,0,0.6);
}

input[type="range"] {
  width: 100%;
  -webkit-appearance: none;
  height: 10px;
  background: #333;
  border-radius: 5px;
  margin: 8px 0 12px 0;
  box-shadow: inset 0 0 8px rgba(0,0,0,0.7);
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 22px;
  height: 22px;
  background: #3b82f6;
  border-radius: 50%;
  cursor: pointer;
  border: 2px solid white;
  box-shadow: 
    0 0 8px rgba(0,0,0,0.7),
    0 0 15px rgba(59, 130, 246, 0.7);
  transition: all 0.2s;
}

input[type="range"]::-webkit-slider-thumb:active {
  transform: scale(1.2);
  box-shadow: 
    0 0 12px rgba(0,0,0,0.7),
    0 0 20px rgba(59, 130, 246, 1);
}

#btn-precionar {
  background: linear-gradient(to bottom, #dc2626, #b91c1c);
  color: white;
  padding: 12px 15px;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  font-size: 0.95rem;
  margin-top: 8px;
  width: 100%;
  font-weight: bold;
  box-shadow: 
    0 4px 12px rgba(0,0,0,0.4),
    0 0 15px rgba(220, 38, 38, 0.4);
  transition: all 0.2s;
  text-shadow: 0 0 8px rgba(0,0,0,0.4);
  text-transform: uppercase;
  letter-spacing: 1px;
}

#btn-precionar:active {
  transform: scale(0.95);
  box-shadow: 
    0 2px 6px rgba(0,0,0,0.4),
    0 0 8px rgba(220, 38, 38, 0.4);
}

/* Mini mapa mejorado */
#miniMapContainer {
  position: absolute;
  top: 40px;
  right: 30px;
  width: 170px;
  height: 120px;
  overflow: hidden;
  border-radius: 5px;
  z-index: 2000;
  background: rgba(0,0,0,0.85);
  border: 1px solid rgba(255,255,255,0.15);
  box-shadow: 
    0 6px 15px rgba(0,0,0,0.5),
    inset 0 0 15px rgba(255,255,255,0.15);
  backdrop-filter: blur(8px);
}

#eyeCrossSection {
  width: 100%;
  height: 100%;
  display: block;
  filter: drop-shadow(0 0 8px rgba(0,0,0,0.7));
}

/* Indicador de profundidad */
.depth-indicator {
  position: absolute;
  bottom: 130px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(0,0,0,0.8);
  padding: 10px 20px;
  border-radius: 25px;
  color: white;
  font-size: 0.85rem;
  z-index: 2000;
  display: flex;
  align-items: center;
  backdrop-filter: blur(8px);
  box-shadow: 0 0 15px rgba(0,0,0,0.6);
  border: 1px solid var(--depth-color, #3b82f6);
  text-transform: uppercase;
  letter-spacing: 1px;
}

.depth-indicator::before {
  content: '';
  display: inline-block;
  width: 12px;
  height: 12px;
  border-radius: 50%;
  margin-right: 10px;
  background: var(--depth-color, #3b82f6);
  box-shadow: 0 0 8px var(--depth-color, #3b82f6);
}

/* Efectos visuales mejorados */
.laser-spot {
  width: 30px;
  height: 30px;
  background: radial-gradient(circle, 
    rgba(255,80,80,0.95), 
    rgba(255,0,0,0.8) 70%, 
    transparent 90%);
  border-radius: 50%;
  position: absolute;
  pointer-events: none;
  animation: laser-fade 2.5s ease-out forwards;
  z-index: 15;
  filter: blur(1px);
  will-change: transform, opacity;
}

@keyframes laser-fade {
  0% { opacity: 1; transform: scale(1); }
  100% { opacity: 0; transform: scale(0.3); }
}

.laser-burn-permanent {
  width: 8px;
  height: 8px;
  background: radial-gradient(circle, 
    white 0%, 
    rgba(255,255,255,0.9) 60%, 
    transparent 80%);
  border-radius: 50%;
  position: absolute;
  pointer-events: none;
  z-index: 16;
  box-shadow: 0 0 15px rgba(255,120,120,0.6);
  animation: burnPulse 3s infinite alternate;
}

.vitreous-particle {
  width: 6px;
  height: 6px;
  background: rgba(255,255,255,0.95);
  border-radius: 50%;
  position: absolute;
  pointer-events: none;
  animation: float-particle 1.5s ease-out forwards;
  z-index: 12;
  filter: blur(0.8px);
  box-shadow: 0 0 8px rgba(255,255,255,0.9);
  will-change: transform, opacity;
}

@keyframes float-particle {
  100% { transform: translate(var(--tx, 0px), var(--ty, 0px)); opacity: 0; }
}

.injection-bubble {
  position: absolute;
  background: rgba(200,230,255,0.9);
  border-radius: 50%;
  pointer-events: none;
  z-index: 3;
  filter: blur(1.5px);
  animation: bubble-float 4s ease-in-out forwards;
  box-shadow: 
    0 0 8px rgba(100,150,255,0.7),
    inset 0 0 5px rgba(255,255,255,0.7);
  will-change: transform, opacity;
}

@keyframes bubble-float {
  100% { 
    transform: translate(calc(var(--tx)*1px), calc(var(--ty)*1px));
    opacity: 0;
  }
}

/* Nuevos estilos para el desprendimiento mejorado */
#retinal-hole {
  display: none;
  position: absolute;
  width: 30px;
  height: 30px;
  background: radial-gradient(circle, 
    rgba(255,255,255,0.8) 0%, 
    rgba(200,0,0,0.6) 70%, 
    transparent 100%);
  border-radius: 50%;
  z-index: 15;
  box-shadow: 0 0 15px rgba(255,255,255,0.5);
  border: 2px dashed rgba(255,255,255,0.7);
  pointer-events: none;
  animation: holePulse 2s infinite alternate;
}

@keyframes holePulse {
  0% { transform: scale(1); opacity: 0.8; }
  50% { transform: scale(1.1); opacity: 1; }
  100% { transform: scale(1); opacity: 0.8; }
}

/* Burbujas de gas mejoradas */
.gas-bubble {
  position: absolute;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 50%;
  pointer-events: none;
  z-index: 6;
  filter: blur(1px);
  box-shadow: 
    0 0 15px rgba(255, 255, 255, 0.9),
    inset 0 0 8px rgba(255, 255, 255, 0.8);
  transform: translate(-50%, -50%);
  will-change: transform, opacity;
  animation: bubbleFloat 4s ease-in-out forwards;
}

.gas-bubble-large {
  position: absolute;
  width: 40%;
  height: 40%;
  background: radial-gradient(circle, 
    rgba(255,255,255,0.95) 0%, 
    rgba(220,220,255,0.8) 70%, 
    transparent 100%);
  border-radius: 50%;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  z-index: 5;
  filter: blur(2px);
  box-shadow: 
    0 0 30px rgba(255,255,255,0.9),
    inset 0 0 20px rgba(255,255,255,0.8);
  animation: largeBubblePulse 3s infinite alternate;
}

@keyframes bubbleFloat {
  0% { transform: translate(-50%, -50%) scale(0.8); opacity: 0; }
  50% { transform: translate(-50%, -50%) scale(1.2); opacity: 0.9; }
  100% { transform: translate(calc(var(--tx)*1px), calc(var(--ty)*1px)) scale(1); opacity: 0; }
}

@keyframes largeBubblePulse {
  0% { transform: translate(-50%, -50%) scale(1); }
  50% { transform: translate(-50%, -50%) scale(1.1); }
  100% { transform: translate(-50%, -50%) scale(1); }
}

/* Caso clínico */
#clinical-case {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.9);
  z-index: 20000;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  color: white;
  padding: 20px;
  box-sizing: border-box;
  backdrop-filter: blur(5px);
}

#clinical-case-content {
  max-width: 800px;
  background: rgba(20, 20, 40, 0.95);
  padding: 30px;
  border-radius: 15px;
  border-left: 5px solid #3b82f6;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.7);
}

#clinical-case h2 {
  color: #3b82f6;
  margin-top: 0;
  text-align: center;
  font-size: 1.8rem;
  margin-bottom: 20px;
}

#clinical-case h3 {
  color: #10b981;
  margin-bottom: 10px;
  font-size: 1.3rem;
}

#clinical-case ul {
  padding-left: 20px;
}

#clinical-case li {
  margin-bottom: 8px;
  line-height: 1.5;
}

#start-simulation-btn {
  background: #3b82f6;
  color: white;
  border: none;
  padding: 15px 30px;
  font-size: 1.2rem;
  border-radius: 8px;
  cursor: pointer;
  margin-top: 30px;
  transition: all 0.3s;
  box-shadow: 0 5px 15px rgba(59, 130, 246, 0.4);
}

#start-simulation-btn:hover {
  background: #2563eb;
  transform: translateY(-2px);
  box-shadow: 0 7px 20px rgba(59, 130, 246, 0.6);
}

#start-simulation-btn:active {
  transform: translateY(0);
  box-shadow: 0 5px 15px rgba(59, 130, 246, 0.4);
}

/* ================== AJUSTES RESPONSIVOS ================== */
@media (max-width: 768px) {
  /* Ajustes para tabletas y móviles grandes */
  .instrument-panel {
    top: 10px;
    left: 10px;
    right: auto;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
    padding: 8px;
    width: calc(100% - 20px);
    transform: none;
  }
  
  .toggle-btn {
    padding: 10px 12px;
    min-width: auto;
    font-size: 0.8rem;
    margin: 4px;
    flex: 1 1 calc(33% - 8px);
    max-width: calc(33% - 8px);
  }
  
  .control-panel {
    top: 120px;
    right: 10px;
    width: 150px;
    padding: 12px;
  }
  
  #miniMapContainer {
    top: 120px;
    left: 10px;
    width: 150px;
    height: 110px;
  }
  
  .joystick-container {
    bottom: 20px;
    width: 45%;
  }
  
  #joystick-light-container {
    left: 10px;
  }
  
  #joystick-vitrectomo-container {
    right: 10px;
  }
  
  .joystick {
    width: 80px;
    height: 80px;
  }
  
  .joystick .joystick-handle {
    width: 35px;
    height: 35px;
  }
  
  .slider-container {
    padding: 10px;
  }
  
  #btn-precionar {
    padding: 10px;
    font-size: 0.9rem;
  }
  
  .depth-indicator {
    bottom: 110px;
    font-size: 0.8rem;
  }
}

@media (max-width: 480px) {
  /* Ajustes para móviles pequeños */
  .retina-container {
    width: 95vmin;
    height: 95vmin;
  }
  
  .toggle-btn {
    font-size: 0.7rem;
    padding: 8px 10px;
  }
  
  .joystick {
    width: 70px;
    height: 70px;
  }
  
  .joystick .joystick-handle {
    width: 30px;
    height: 30px;
  }
  
  .control-panel {
    width: 130px;
    padding: 10px;
  }
  
  #miniMapContainer {
    width: 130px;
    height: 95px;
  }
  
  .depth-indicator {
    bottom: 100px;
    font-size: 0.75rem;
    padding: 8px 15px;
  }
  
  /* Ajustar posición de los joysticks para móviles */
  .joystick-container {
    bottom: 10px;
    width: 40%;
  }
  
  /* Ajustar tamaño de la retina para mejor visualización */
  .retina-container {
    width: 90vmin;
    height: 90vmin;
  }
  
  /* Ajustar tamaño de texto en alertas */
  .alert-content h3 {
    font-size: 1rem;
  }
  
  .alert-content p {
    font-size: 0.8rem;
  }
  
  /* Ajustar panel de controles para mejor acceso */
  .control-panel {
    top: 100px;
    right: 5px;
    width: 120px;
    padding: 8px;
  }
  
  /* Ajustar botones del panel de instrumentos */
  .toggle-btn {
    padding: 6px 8px;
    font-size: 0.65rem;
    margin: 3px;
  }
}

#miniMapContainer {
  position: absolute;
  top: 40px;
  left: 30px; /* Cambiado de right a left */
  width: 170px;
  height: 120px;
  overflow: hidden;
  border-radius: 5px;
  z-index: 2000;
  background: rgba(0,0,0,0.85);
  border: 1px solid rgba(255,255,255,0.15);
  box-shadow: 
    0 6px 15px rgba(0,0,0,0.5),
    inset 0 0 15px rgba(255,255,255,0.15);
  backdrop-filter: blur(8px);
}
/* Sistema de alertas */
#alert-system {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 10000;
  width: 90%;
  max-width: 600px;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.alert-container {
  display: none;
  background: rgba(20, 20, 30, 0.95);
  border-left: 5px solid;
  border-radius: 8px;
  padding: 15px 20px;
  align-items: center;
  box-shadow: 0 10px 25px rgba(0,0,0,0.5);
  backdrop-filter: blur(8px);
  transform-origin: top center;
  border-left-color: #ff5555;
}

.alert-icon {
  font-size: 1.8rem;
  margin-right: 15px;
  flex-shrink: 0;
}

.alert-content {
  flex-grow: 1;
}

.alert-content h3 {
  margin: 0 0 5px 0;
  color: white;
  font-size: 1.1rem;
  font-weight: 600;
}

.alert-content p {
  margin: 0;
  color: #ddd;
  font-size: 0.9rem;
  line-height: 1.4;
}

.alert-timer {
  height: 3px;
  background: rgba(255,255,255,0.3);
  margin-top: 8px;
  border-radius: 2px;
  overflow: hidden;
}

.alert-timer::after {
  content: '';
  display: block;
  height: 100%;
  width: 100%;
  background: white;
}

.alert-dismiss {
  background: none;
  border: none;
  color: white;
  font-size: 1.5rem;
  cursor: pointer;
  padding: 0 0 0 15px;
  opacity: 0.7;
  transition: opacity 0.2s;
  flex-shrink: 0;
}

.alert-dismiss:hover {
  opacity: 1;
}

.toggle-btn {
  background: #1a3a8a;
  color: white;
  padding: 12px 18px;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  font-size: clamp(0.85rem, 2.5vw, 1rem);
  margin: 8px 5px;
  min-width: 100px;
  transition: all 0.2s;
  font-weight: 500;
  box-shadow: 
    0 3px 8px rgba(0,0,0,0.4),
    inset 0 0 8px rgba(255,255,255,0.15);
  position: relative;
  overflow: hidden;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.toggle-btn.active {
  background: #3b82f6;
  box-shadow: 
    0 0 20px #3b82f6,
    0 0 40px rgba(59, 130, 246, 0.4);
}

/* Panel de parámetros */
.control-panel {
  position: absolute;
  right: 20px;
  top: 190px;
  background: rgba(10,10,20,0.95);
  padding: 20px;
  border-radius: 50px;
  z-index: 5000;
  font-size: clamp(0.75rem, 2vw, 0.9rem);
  width: 180px;
  max-height: 80vh;
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
  box-shadow: 
    0 6px 20px rgba(18, 20, 41, 0.6),
    inset 0 0 15px rgba(255,255,255,0.15);
  border: 1px solid rgba(255,255,255,0.15);
  backdrop-filter: blur(8px);
}

.vital-sign {
  display: flex;
  justify-content: space-between;
  margin: 15px 0;
  flex-wrap: wrap;
}

.vital-label {
  color: #aaa;
  font-size: 0.85em;
  width: 60%;
}

.vital-value {
  font-family: 'Courier New', monospace;
  font-weight: bold;
  font-size: 0.95em;
  width: 40%;
  text-align: right;
  transition: color 0.3s;
}

.normal { color: #2ecc71; text-shadow: 0 0 8px rgba(46, 204, 113, 0.4); }
.warning { color: #f39c12; text-shadow: 0 0 8px rgba(243, 156, 18, 0.4); }
.danger { color: #e74c3c; text-shadow: 0 0 8px rgba(231, 76, 60, 0.4); }

.gauge-container {
  width: 100%;
  height: 10px;
  background: #222;
  border-radius: 5px;
  margin: 8px 0 15px 0;
  overflow: hidden;
  box-shadow: inset 0 0 8px rgba(0,0,0,0.7);
}

.gauge-level {
  height: 100%;
  border-radius: 5px;
  transition: width 0.5s ease;
  box-shadow: inset 0 0 8px rgba(255,255,255,0.3);
}

/* Joysticks */
.joystick-container {
  position: absolute;
  bottom: 1px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  z-index: 3000;
}

#joystick-light-container {
  left: 200px;
}

#joystick-vitrectomo-container {
  right: 180px;
}

.joystick {
  width: 100px;
  height: 100px;
  background: rgba(255,255,255,0.1);
  border: 2px solid rgba(255,255,255,0.4);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  touch-action: none;
  position: relative;
  box-shadow: 
    0 6px 15px rgba(0,0,0,0.4),
    inset 0 0 25px rgba(255,255,255,0.15);
  backdrop-filter: blur(8px);
  will-change: transform;
}

.joystick .joystick-handle {
  width: 40px;
  height: 40px;
  background: rgba(255,255,255,0.8);
  border-radius: 50%;
  position: absolute;
  transition: transform 0.1s ease;
  box-shadow: 
    0 0 15px rgba(255,255,255,0.4),
    inset 0 0 8px rgba(255,255,255,0.7);
  will-change: transform;
}

/* Controles Z */
.slider-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  background: rgba(0,0,0,0.7);
  padding: 15px;
  border-radius: 12px;
  box-shadow: 
    0 4px 10px rgba(0,0,0,0.4),
    inset 0 0 8px rgba(255,255,255,0.15);
  backdrop-filter: blur(8px);
}

.slider-container label {
  font-size: 0.8rem;
  margin-bottom: 10px;
  text-align: center;
  color: #eee;
  font-weight: 500;
  text-shadow: 0 0 8px rgba(0,0,0,0.6);
}

input[type="range"] {
  width: 100%;
  -webkit-appearance: none;
  height: 10px;
  background: #333;
  border-radius: 5px;
  margin: 8px 0 12px 0;
  box-shadow: inset 0 0 8px rgba(0,0,0,0.7);
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 22px;
  height: 22px;
  background: #3b82f6;
  border-radius: 50%;
  cursor: pointer;
  border: 2px solid white;
  box-shadow: 
    0 0 8px rgba(0,0,0,0.7),
    0 0 15px rgba(59, 130, 246, 0.7);
  transition: all 0.2s;
}

input[type="range"]::-webkit-slider-thumb:active {
  transform: scale(1.2);
  box-shadow: 
    0 0 12px rgba(0,0,0,0.7),
    0 0 20px rgba(59, 130, 246, 1);
}

#btn-precionar {
  background: linear-gradient(to bottom, #dc2626, #b91c1c);
  color: white;
  padding: 12px 15px;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  font-size: 0.95rem;
  margin-top: 8px;
  width: 100%;
  font-weight: bold;
  box-shadow: 
    0 4px 12px rgba(0,0,0,0.4),
    0 0 15px rgba(220, 38, 38, 0.4);
  transition: all 0.2s;
  text-shadow: 0 0 8px rgba(0,0,0,0.4);
  text-transform: uppercase;
  letter-spacing: 1px;
}

#btn-precionar:active {
  transform: scale(0.95);
  box-shadow: 
    0 2px 6px rgba(0,0,0,0.4),
    0 0 8px rgba(220, 38, 38, 0.4);
}

.laser-spot {
  position: absolute;
  width: 24px;
  height: 24px;
  background: radial-gradient(circle, rgba(255,50,50,0.8) 0%, rgba(255,0,0,0.6) 70%, transparent 100%);
  border-radius: 50%;
  box-shadow: 0 0 15px rgba(255,100,100,0.7);
  pointer-events: none;
  z-index: 20;
}

.laser-burn-permanent {
  position: absolute;
  width: 6px;
  height: 6px;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 50%;
  box-shadow: 0 0 5px rgba(255, 255, 255, 0.8);
  pointer-events: none;
  z-index: 15;
}

.cautery-effect {
  position: absolute;
  width: 30px;
  height: 30px;
  background: radial-gradient(circle, rgba(255,200,0,0.8) 0%, rgba(255,150,0,0.6) 70%, transparent 100%);
  border-radius: 50%;
  box-shadow: 0 0 20px rgba(255,200,0,0.7);
  pointer-events: none;
  z-index: 20;
}

.blood-clot {
  position: absolute;
  background: radial-gradient(circle, rgba(180,0,0,0.8) 0%, rgba(120,0,0,0.6) 70%, transparent 100%);
  border-radius: 50%;
  pointer-events: none;
  z-index: 10;
}

.instrument-action-indicator {
  position: absolute;
  top: -30px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(0,0,0,0.7);
  color: white;
  padding: 5px 10px;
  border-radius: 15px;
  font-size: 0.8rem;
  white-space: nowrap;
  z-index: 100;
  opacity: 0;
  pointer-events: none;
}

.instrument-action-indicator.active {
  opacity: 1;
}

/* Efectos de vitrectomía mejorados */
.vitrectomy-effect {
  position: absolute;
  width: 60px;
  height: 60px;
  background: radial-gradient(circle, 
    rgba(255,255,255,0.2) 0%, 
    rgba(255,255,255,0.1) 40%, 
    transparent 70%);
  border-radius: 50%;
  pointer-events: none;
  z-index: 14;
  transform: scale(0);
  opacity: 0;
}

.suction-particle {
  position: absolute;
  width: 4px;
  height: 4px;
  background: rgba(200, 200, 255, 0.8);
  border-radius: 50%;
  pointer-events: none;
  z-index: 12;
  box-shadow: 0 0 5px rgba(255,255,255,0.6);
}

/* Efecto de sangre mejorado */
.blood-extraction {
  position: absolute;
  width: 40px;
  height: 40px;
  background: radial-gradient(circle, 
    rgba(200,0,0,0.8) 0%, 
    rgba(150,0,0,0.6) 50%, 
    transparent 80%);
  border-radius: 50%;
  pointer-events: none;
  z-index: 13;
  transform: scale(0);
  filter: blur(2px);
}

/* Estilos para el instrumento único - PALO GRIS */
.instrument-container {
  position: absolute;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 30;
}

.instrument-rod {
  position: absolute;
  height: 8px;
  background: linear-gradient(to right, #888, #ccc);
  border-radius: 4px;
  transform-origin: left center;
  z-index: 30;
  box-shadow: 0 0 10px rgba(0,0,0,0.5);
}

.instrument-tip {
  position: absolute;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #ccc;
  z-index: 31;
  box-shadow: 0 0 10px rgba(0,0,0,0.5);
  transform: translate(-50%, -50%);
}

.instrument-base {
  position: absolute;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background: #555;
  z-index: 29;
  box-shadow: 0 0 15px rgba(0,0,0,0.7);
}

.instrument-tip.active-vitrectomo {
  background: #3b82f6;
  box-shadow: 0 0 15px rgba(59, 130, 246, 0.7);
}

.instrument-tip.active-laser {
  background: #ff5555;
  box-shadow: 0 0 15px rgba(255, 85, 85, 0.7);
}

.instrument-tip.active-cautery {
  background: #ff9900;
  box-shadow: 0 0 15px rgba(255, 153, 0, 0.7);
}

/* Retina y efectos visuales */
.retina-container {
  position: relative;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  overflow: hidden;
  background: radial-gradient(circle at center, #400000 0%, #200000 100%);
}

.blood-vessels {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  z-index: 5;
}

.macula {
  position: absolute;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background: rgba(200, 200, 100, 0.3);
  top: 50%;
  left: 50%;
  transform: translate(calc(-50% + 100px), translateY(-50%);
  box-shadow: 0 0 20px rgba(200, 200, 100, 0.5);
  z-index: 6;
}

.optic-disc {
  position: absolute;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: rgba(220, 80, 80, 0.5);
  top: 50%;
  left: 50%;
  transform: translate(calc(-50% - 80px), translateY(-50%));
  box-shadow: inset 0 0 20px rgba(220,80,80,0.6), 0 0 25px rgba(220,80,80,0.5);
  z-index: 6;
}

.optic-cup {
  position: absolute;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: rgba(180, 60, 60, 0.7);
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 7;
}

.retina-nerve {
  position: absolute;
  width: 15px;
  height: 80px;
  background: rgba(220, 80, 80, 0.5);
  top: 50%;
  left: 50%;
  transform: translate(calc(-50% - 80px), translateY(-50%)) rotate(30deg);
  z-index: 6;
}

#hemorrhage-overlay {
  position: absolute;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: radial-gradient(circle at center, rgba(180,0,0,0.8) 0%, rgba(120,0,0,0.7) 50%, rgba(80,0,0,0.6) 100%);
  z-index: 13;
}

#light-reflection {
  position: absolute;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: radial-gradient(circle at var(--light-x) var(--light-y), 
              rgba(255,255,255,0.1) 0%, 
              transparent calc(var(--light-size) / 2));
  z-index: 11;
}

.light-scatter {
  position: absolute;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: radial-gradient(circle at var(--light-x) var(--light-y), 
              rgba(255,255,255,0.03) 0%, 
              transparent 70%);
  z-index: 10;
  opacity: 0;
}

.light-scatter.active {
  opacity: 0.5;
}

.specular-highlight {
  position: absolute;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background: white;
  filter: blur(5px);
  transform: translate(calc(var(--light-x) - 15px), calc(var(--light-y) - 15px));
  z-index: 12;
  opacity: 0;
}

.specular-highlight.active {
  opacity: 0.3;
}

/* Indicador de profundidad */
.depth-indicator {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(0,0,0,0.7);
  color: var(--depth-color, white);
  padding: 8px 15px;
  border-radius: 20px;
  font-size: 0.9rem;
  z-index: 1000;
  box-shadow: 0 0 10px rgba(0,0,0,0.5);
}

/* Efecto de hemorragia */
.hemorrhage-overlay {
  position: absolute;
  width: 100%;
  height: 100%;
  background: radial-gradient(circle, rgba(200,0,0,0.7) 0%, transparent 70%);
  z-index: 25;
  opacity: 0;
  display: none;
}
 </style>
</head>
<body>
  <!-- CASO CLÍNICO -->
  <div id="clinical-case">
    <div id="clinical-case-content">
      <h2>CASO CLÍNICO: HEMORRAGIA VÍTREA</h2>
      
      <h3>Paciente:</h3>
      <p>Varón de 62 años, diabético</p>
      
      <h3>Síntomas:</h3>
      <p>Pérdida súbita e indolora de visión en ojo izquierdo</p>
      
      <h3>Hallazgos:</h3>
      <ul>
        <li>Hemorragia vítrea densa</li>
        <li>No se visualiza retina por hemorragia</li>
        <li>Retinopatía diabética proliferativa</li>
      </ul>
      
      <h3>OBJETIVOS QUIRÚRGICOS:</h3>
      <ol>
        <li>Realizar vitrectomía posterior para limpieza de hemorragia</li>
        <li>Identificar y tratar fuentes de sangrado</li>
        <li>Aplicar endoláser en áreas de neovascularización</li>
      </ol>
      
      <button id="start-simulation-btn">Iniciar Simulación</button>
    </div>
  </div>
  <div id="container">
    <!-- SISTEMA DE ALERTAS -->
    <div id="alert-system">
      <div class="alert-container" id="retina-detachment-alert">
        <div class="alert-icon">❗</div>
        <div class="alert-content">
          <h3>DESPRENDIMIENTO DE RETINA</h3>
          <p>¡Emergencia! Se ha detectado desprendimiento retiniano. Suspender procedimiento e inyectar PFC inmediatamente.</p>
          <div class="alert-timer"></div>
        </div>
        <button class="alert-dismiss">×</button>
      </div>
      
      <div class="alert-container" id="high-iop-alert">
        <div class="alert-icon">📈</div>
        <div class="alert-content">
          <h3>PRESIÓN INTRAOCULAR ELEVADA</h3>
          <p>PIO > 25 mmHg. Riesgo de daño al nervio óptico. Reducir flujo de irrigación y considerar paracentesis.</p>
          <div class="alert-timer"></div>
        </div>
        <button class="alert-dismiss">×</button>
      </div>
      
      <div class="alert-container" id="vessel-damage-alert">
        <div class="alert-icon">🩸</div>
        <div class="alert-content">
          <h3>HEMORRAGIA RETINIANA</h3>
          <p>¡Daño vascular detectado! Aplicar presión suave con PFC y considerar coagulación con cauterio.</p>
          <div class="alert-timer"></div>
        </div>
        <button class="alert-dismiss">×</button>
      </div>
      
      <div class="alert-container" id="wall-collision-alert">
        <div class="alert-icon">⚠️</div>
        <div class="alert-content">
          <h3>COLISIÓN CON ESTRUCTURA OCULAR</h3>
          <p>¡Advertencia! El instrumento ha contactado con la pared ocular. Retire inmediatamente y reevalúe la posición.</p>
          <div class="alert-timer"></div>
        </div>
        <button class="alert-dismiss">×</button>
      </div>
      
      <div class="alert-container" id="vision-loss-alert">
        <div class="alert-icon">👁️</div>
        <div class="alert-content">
          <h3>PÉRDIDA DE VISIÓN</h3>
          <p>¡Precaución! Se ha aplicado cauterio en la mácula. Riesgo de pérdida visual permanente.</p>
          <div class="alert-timer"></div>
        </div>
        <button class="alert-dismiss">×</button>
      </div>
      
      <div class="alert-container" id="surgery-complete-alert">
        <div class="alert-icon">🏥</div>
        <div class="alert-content">
          <h3>CIRUGÍA COMPLETADA</h3>
          <p>Procedimiento finalizado con éxito. La hemorragia ha sido completamente removida y las áreas de neovascularización han sido tratadas.</p>
          <div class="alert-timer"></div>
        </div>
        <button class="alert-dismiss" id="back-to-menu-btn">Volver al Menú</button>
      </div>
    </div>
    
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
          <filter id="bloodFilter">
            <feTurbulence type="fractalNoise" baseFrequency="0.05" numOctaves="3" result="turbulence" />
            <feDisplacementMap in2="turbulence" in="SourceGraphic" scale="5" xChannelSelector="R" yChannelSelector="G" />
          </filter>
        </defs>
        <rect width="800" height="600" fill="url(#bgGradient)" />
        <ellipse id="lens-minimap" cx="400" cy="150" rx="100" ry="65" fill="url(#lensGradient)" filter="url(#dropShadow)" />
        <ellipse cx="400" cy="150" rx="95" ry="60" fill="none" stroke="#B3E5FC" stroke-width="2" />
        <circle id="iris-minimap" cx="400" cy="150" r="55" fill="none" stroke="url(#irisGradient)" stroke-width="15" />
        <circle cx="400" cy="150" r="30" fill="#000000" />
        <circle id="retina-wall" cx="400" cy="300" r="250" fill="none" stroke="#E0E0E0" stroke-width="4" />
        <circle id="retina-minimap" cx="400" cy="300" r="240" fill="#400000" opacity="0.8" filter="url(#bloodFilter)" />
        <circle id="miniCenter" cx="400" cy="300" r="8" fill="#00f7ff" />
        <line id="probeLight" x1="225" y1="100" x2="350" y2="300" stroke="#FFFFFF" stroke-width="9" stroke-linecap="round" />
        <line id="probeLightInner" x1="225" y1="100" x2="350" y2="300" stroke="#FFFFFF" stroke-width="6" stroke-linecap="round" />
        <line id="probeForceps" x1="575" y1="100" x2="450" y2="300" stroke="#FFFFFF" stroke-width="9" stroke-linecap="round" />
        <line id="probeForcepsInner" x1="575" y1="100" x2="450" y2="300" stroke="#FFFFFF" stroke-width="6" stroke-linecap="round" />
      </svg>
    </div>

    <!-- PANEL DE INSTRUMENTOS -->
    <div class="instrument-panel">
      <button class="toggle-btn" id="btn-vitrectomo">Vitrectomo</button>
      <button class="toggle-btn" id="btn-laser">Láser</button>
      <button class="toggle-btn" id="btn-cautery">Cauterio</button>
    </div>

    <!-- PANEL DE PARÁMETROS -->
    <div class="control-panel">
      <div class="vital-sign">
        <span class="vital-label">Presión Intraocular:</span>
        <span class="vital-value normal" id="iop-value">20 mmHg</span>
      </div>
      <div id="iop-gauge" class="gauge-container">
        <div class="gauge-level" id="iop-level"></div>
      </div>
      
      <div class="vital-sign">
        <span class="vital-label">Perfusión Retiniana:</span>
        <span class="vital-value normal" id="perfusion-value">95%</span>
      </div>
      <div id="perfusion-gauge" class="gauge-container">
        <div class="gauge-level" id="perfusion-level"></div>
      </div>
      
      <div class="vital-sign">
        <span class="vital-label">Hemorragia Removida:</span>
        <span class="vital-value" id="hemorrhage-value">0%</span>
      </div>
      <div id="hemorrhage-gauge" class="gauge-container">
        <div class="gauge-level" id="hemorrhage-level"></div>
      </div>
      
      <div class="vital-sign">
        <span class="vital-label">Puntos de Cauterio:</span>
        <span class="vital-value" id="cautery-value">0/15</span>
      </div>
      
      <div class="vital-sign">
        <span class="vital-label">Puntos de Láser:</span>
        <span class="vital-value" id="laser-value">0/30</span>
      </div>
    </div>

    <!-- JOYSTICKS -->
    <div id="joystick-light-container" class="joystick-container">
      <div id="joystick-light" class="joystick">
        <div class="joystick-handle"></div>
      </div>
      <div class="slider-container">
        <label>Luz (Z)</label>
        <input type="range" id="endo-z-slider" min="0" max="200" value="50">
      </div>
    </div>

    <div id="joystick-vitrectomo-container" class="joystick-container">
      <div id="joystick-vitrectomo" class="joystick">
        <div class="joystick-handle"></div>
      </div>
      <div class="slider-container">
        <label>Instrumento (Z)</label>
        <input type="range" id="vitrectomo-z-slider" min="-250" max="-50" value="-150">
        <button id="btn-precionar">Accionar</button>
      </div>
    </div>

    <!-- RETINA CENTRAL -->
    <div id="eye-chamber">
      <div class="retina-container" id="retina">
        <div class="retina-sphere">
          <div class="retina-texture"></div>
          <div class="retina-nerve"></div>
          <div class="optic-disc">
            <div class="optic-cup"></div>
          </div>

          <div class="macula"></div>
          <div id="hemorrhage-overlay"></div>
        </div>

        <div id="light-mask"></div>
        <div id="light-reflection"></div>
        <div id="light-scatter" class="light-scatter"></div>
        <div id="specular-highlight" class="specular-highlight"></div>
        <!-- Dentro del div retina-container en SimuladorHV.html -->
<div class="blood-vessels">
  <svg viewBox="0 0 1024 1024" preserveAspectRatio="xMidYMid meet">
    <defs>
      <style type="text/css">
        .blood-vessel {
          stroke: #8B0000;
          stroke-width: 1.5;
          fill: #8b0808;
          fill-opacity: 0.7;
        }
        .small-details {
          stroke: #8B0000;
          stroke-width: 1;
          fill: none;
        }
      </style>
    </defs>
    
    <!-- Vasos sanguíneos principales -->
    <path d="M 558.00,952.00 L 552.00,938.00 L 538.00,922.00 L 499.00,901.00 L 477.00,882.00 L 448.00,821.00 L 432.00,801.00 L 415.00,791.00 L 366.00,773.00 L 346.00,760.00 L 331.00,745.00 L 317.00,721.00 L 301.00,676.00 L 281.00,651.00 L 301.00,722.00 L 338.00,790.00 L 364.00,863.00 L 368.00,881.00 L 368.00,905.00 L 364.00,923.00 L 361.00,925.00 L 365.00,895.00 L 357.00,857.00 L 331.00,789.00 L 297.00,729.00 L 275.00,654.00 L 260.00,630.00 L 239.00,607.00 L 226.00,585.00 L 218.00,554.00 L 215.00,514.00 L 208.00,544.00 L 210.00,574.00 L 253.00,694.00 L 251.00,721.00 L 236.00,759.00 L 238.00,811.00 L 229.00,837.00 L 227.00,835.00 L 234.00,811.00 L 231.00,757.00 L 246.00,716.00 L 247.00,700.00 L 242.00,677.00 L 205.00,586.00 L 194.00,615.00 L 164.00,653.00 L 154.00,686.00 L 144.00,704.00 L 125.00,723.00 L 110.00,733.00 L 101.00,735.00 L 124.00,720.00 L 141.00,701.00 L 162.00,645.00 L 188.00,614.00 L 196.00,599.00 L 200.00,584.00 L 199.00,544.00 L 207.00,512.00 L 166.00,542.00 L 125.00,561.00 L 83.00,601.00 L 66.00,608.00 L 58.00,607.00 L 82.00,597.00 L 120.00,557.00 L 166.00,534.00 L 205.00,505.00 L 209.00,501.00 L 209.00,482.00 L 203.00,458.00 L 195.00,449.00 L 153.00,424.00 L 124.00,415.00 L 101.00,404.00 L 71.00,382.00 L 116.00,407.00 L 151.00,418.00 L 134.00,393.00 L 122.00,354.00 L 111.00,334.00 L 87.00,310.00 L 57.00,296.00 L 65.00,296.00 L 83.00,304.00 L 115.00,332.00 L 125.00,348.00 L 135.00,381.00 L 149.00,408.00 L 169.00,428.00 L 193.00,442.00 L 177.00,403.00 L 174.00,366.00 L 167.00,345.00 L 154.00,320.00 L 108.00,255.00 L 97.00,227.00 L 92.00,198.00 L 94.00,184.00 L 98.00,216.00 L 108.00,245.00 L 160.00,320.00 L 178.00,360.00 L 180.00,320.00 L 161.00,248.00 L 163.00,210.00 L 176.00,166.00 L 167.00,213.00 L 166.00,246.00 L 186.00,323.00 L 181.00,391.00 L 186.00,412.00 L 210.00,458.00 L 214.00,474.00 L 217.00,450.00 L 209.00,401.00 L 219.00,350.00 L 218.00,282.00 L 229.00,238.00 L 229.00,221.00 L 222.00,193.00 L 222.00,171.00 L 236.00,122.00 L 260.00,84.00 L 236.00,134.00 L 227.00,174.00 L 228.00,194.00 L 235.00,220.00 L 235.00,240.00 L 224.00,283.00 L 225.00,351.00 L 216.00,390.00 L 216.00,416.00 L 223.00,445.00 L 223.00,464.00 L 217.00,496.00 L 229.00,461.00 L 228.00,412.00 L 237.00,388.00 L 267.00,346.00 L 292.00,284.00 L 332.00,238.00 L 353.00,180.00 L 407.00,121.00 L 433.00,73.00 L 426.00,97.00 L 410.00,126.00 L 359.00,182.00 L 337.00,242.00 L 300.00,284.00 L 282.00,330.00 L 307.00,304.00 L 333.00,283.00 L 414.00,236.00 L 432.00,222.00 L 465.00,183.00 L 489.00,135.00 L 504.00,113.00 L 531.00,85.00 L 546.00,74.00 L 548.00,75.00 L 509.00,115.00 L 463.00,198.00 L 433.00,229.00 L 481.00,208.00 L 526.00,173.00 L 545.00,163.00 L 564.00,158.00 L 615.00,158.00 L 651.00,153.00 L 697.00,134.00 L 735.00,123.00 L 750.00,115.00 L 765.00,99.00 L 763.00,106.00 L 742.00,124.00 L 698.00,139.00 L 660.00,156.00 L 698.00,161.00 L 725.00,161.00 L 767.00,154.00 L 764.00,156.00 L 718.00,166.00 L 693.00,166.00 L 649.00,160.00 L 621.00,164.00 L 560.00,166.00 L 528.00,180.00 L 499.00,204.00 L 471.00,220.00 L 522.00,222.00 L 583.00,237.00 L 609.00,235.00 L 653.00,225.00 L 673.00,226.00 L 702.00,238.00 L 729.00,261.00 L 747.00,269.00 L 766.00,273.00 L 798.00,272.00 L 838.00,254.00 L 863.00,250.00 L 867.00,252.00 L 837.00,258.00 L 809.00,273.00 L 791.00,278.00 L 765.00,278.00 L 738.00,271.00 L 764.00,298.00 L 801.00,313.00 L 820.00,334.00 L 830.00,339.00 L 846.00,339.00 L 865.00,330.00 L 885.00,325.00 L 908.00,327.00 L 876.00,330.00 L 843.00,344.00 L 821.00,341.00 L 832.00,357.00 L 851.00,375.00 L 874.00,387.00 L 906.00,398.00 L 921.00,406.00 L 923.00,410.00 L 901.00,399.00 L 867.00,388.00 L 847.00,377.00 L 827.00,358.00 L 801.00,319.00 L 762.00,303.00 L 723.00,262.00 L 688.00,237.00 L 673.00,232.00 L 654.00,231.00 L 606.00,242.00 L 578.00,243.00 L 577.00,242.00 L 568.00,241.00 L 557.00,237.00 L 554.00,237.00 L 550.00,235.00 L 547.00,235.00 L 543.00,233.00 L 540.00,233.00 L 539.00,232.00 L 536.00,232.00 L 535.00,231.00 L 533.00,231.00 L 573.00,262.00 L 616.00,272.00 L 636.00,281.00 L 653.00,299.00 L 672.00,333.00 L 693.00,350.00 L 739.00,362.00 L 759.00,376.00 L 766.00,391.00 L 765.00,396.00 L 759.00,381.00 L 747.00,369.00 L 707.00,359.00 L 720.00,377.00 L 732.00,422.00 L 752.00,456.00 L 756.00,472.00 L 754.00,475.00 L 749.00,455.00 L 726.00,416.00 L 713.00,372.00 L 703.00,361.00 L 681.00,348.00 L 667.00,335.00 L 649.00,303.00 L 636.00,288.00 L 619.00,279.00 L 586.00,273.00 L 567.00,266.00 L 519.00,229.00 L 497.00,226.00 L 464.00,227.00 L 415.00,246.00 L 348.00,284.00 L 408.00,286.00 L 458.00,269.00 L 484.00,272.00 L 496.00,279.00 L 509.00,292.00 L 536.00,342.00 L 547.00,355.00 L 589.00,373.00 L 607.00,384.00 L 657.00,438.00 L 642.00,428.00 L 602.00,385.00 L 555.00,365.00 L 572.00,390.00 L 586.00,436.00 L 599.00,462.00 L 586.00,444.00 L 566.00,388.00 L 533.00,348.00 L 507.00,300.00 L 489.00,282.00 L 468.00,275.00 L 453.00,277.00 L 407.00,293.00 L 351.00,291.00 L 330.00,297.00 L 292.00,331.00 L 262.00,367.00 L 247.00,390.00 L 238.00,410.00 L 237.00,415.00 L 237.00,430.00 L 255.00,407.00 L 300.00,385.00 L 336.00,362.00 L 359.00,353.00 L 388.00,351.00 L 412.00,359.00 L 449.00,387.00 L 470.00,399.00 L 514.00,407.00 L 490.00,408.00 L 467.00,402.00 L 449.00,392.00 L 415.00,366.00 L 397.00,358.00 L 373.00,356.00 L 351.00,361.00 L 325.00,375.00 L 346.00,374.00 L 366.00,381.00 L 409.00,425.00 L 456.00,448.00 L 407.00,429.00 L 364.00,386.00 L 343.00,379.00 L 325.00,380.00 L 264.00,408.00 L 250.00,420.00 L 238.00,443.00 L 238.00,463.00 L 231.00,489.00 L 242.00,487.00 L 270.00,495.00 L 287.00,495.00 L 338.00,474.00 L 392.00,478.00 L 420.00,469.00 L 402.00,479.00 L 390.00,482.00 L 339.00,479.00 L 303.00,496.00 L 285.00,501.00 L 269.00,501.00 L 247.00,494.00 L 234.00,494.00 L 229.00,499.00 L 229.00,510.00 L 239.00,532.00 L 251.00,544.00 L 283.00,566.00 L 360.00,567.00 L 376.00,564.00 L 391.00,557.00 L 434.00,515.00 L 456.00,505.00 L 434.00,519.00 L 402.00,555.00 L 380.00,568.00 L 363.00,572.00 L 311.00,570.00 L 288.00,572.00 L 298.00,588.00 L 314.00,629.00 L 326.00,647.00 L 360.00,679.00 L 384.00,691.00 L 414.00,690.00 L 445.00,676.00 L 461.00,659.00 L 484.00,613.00 L 500.00,599.00 L 525.00,588.00 L 491.00,611.00 L 462.00,667.00 L 446.00,682.00 L 428.00,692.00 L 469.00,699.00 L 503.00,694.00 L 515.00,687.00 L 554.00,644.00 L 596.00,614.00 L 605.00,601.00 L 611.00,584.00 L 612.00,589.00 L 607.00,603.00 L 596.00,619.00 L 557.00,647.00 L 521.00,689.00 L 503.00,700.00 L 486.00,704.00 L 461.00,704.00 L 427.00,697.00 L 388.00,698.00 L 369.00,693.00 L 388.00,723.00 L 409.00,741.00 L 460.00,762.00 L 490.00,779.00 L 512.00,784.00 L 577.00,756.00 L 593.00,742.00 L 602.00,722.00 L 606.00,688.00 L 615.00,673.00 L 628.00,661.00 L 610.00,687.00 L 605.00,729.00 L 594.00,749.00 L 574.00,764.00 L 514.00,790.00 L 547.00,801.00 L 580.00,800.00 L 604.00,791.00 L 654.00,749.00 L 674.00,724.00 L 696.00,665.00 L 697.00,626.00 L 707.00,590.00 L 703.00,545.00 L 706.00,522.00 L 711.00,592.00 L 703.00,622.00 L 703.00,656.00 L 700.00,670.00 L 723.00,644.00 L 769.00,611.00 L 779.00,594.00 L 795.00,542.00 L 808.00,527.00 L 836.00,505.00 L 856.00,471.00 L 880.00,454.00 L 905.00,449.00 L 886.00,455.00 L 866.00,467.00 L 855.00,480.00 L 837.00,512.00 L 805.00,538.00 L 796.00,553.00 L 784.00,597.00 L 776.00,611.00 L 762.00,625.00 L 731.00,645.00 L 714.00,661.00 L 700.00,681.00 L 675.00,735.00 L 659.00,753.00 L 618.00,789.00 L 588.00,805.00 L 549.00,808.00 L 527.00,803.00 L 492.00,788.00 L 519.00,822.00 L 536.00,831.00 L 557.00,836.00 L 601.00,835.00 L 647.00,819.00 L 676.00,803.00 L 711.00,773.00 L 751.00,749.00 L 766.00,731.00 L 791.00,678.00 L 804.00,665.00 L 838.00,642.00 L 856.00,623.00 L 894.00,554.00 L 922.00,530.00 L 968.00,505.00 L 917.00,538.00 L 900.00,554.00 L 888.00,571.00 L 861.00,625.00 L 835.00,652.00 L 796.00,681.00 L 770.00,736.00 L 757.00,752.00 L 793.00,739.00 L 830.00,712.00 L 856.00,699.00 L 876.00,695.00 L 917.00,694.00 L 939.00,685.00 L 960.00,668.00 L 957.00,674.00 L 939.00,689.00 L 921.00,697.00 L 876.00,700.00 L 850.00,707.00 L 795.00,745.00 L 781.00,752.00 L 747.00,761.00 L 725.00,772.00 L 707.00,784.00 L 686.00,803.00 L 685.00,803.00 L 682.00,806.00 L 681.00,806.00 L 675.00,811.00 L 672.00,812.00 L 671.00,813.00 L 684.00,810.00 L 721.00,812.00 L 771.00,832.00 L 801.00,840.00 L 826.00,840.00 L 845.00,834.00 L 847.00,836.00 L 827.00,843.00 L 798.00,844.00 L 776.00,839.00 L 723.00,818.00 L 693.00,815.00 L 663.00,820.00 L 600.00,842.00 L 558.00,843.00 L 544.00,840.00 L 581.00,865.00 L 603.00,876.00 L 624.00,881.00 L 665.00,882.00 L 684.00,887.00 L 742.00,929.00 L 727.00,923.00 L 682.00,891.00 L 664.00,887.00 L 616.00,886.00 L 582.00,873.00 L 518.00,830.00 L 474.00,780.00 L 455.00,769.00 L 421.00,757.00 L 395.00,742.00 L 376.00,722.00 L 348.00,679.00 L 313.00,644.00 L 281.00,578.00 L 270.00,567.00 L 242.00,548.00 L 230.00,535.00 L 223.00,522.00 L 227.00,566.00 L 234.00,587.00 L 252.00,613.00 L 302.00,667.00 L 311.00,684.00 L 324.00,722.00 L 334.00,739.00 L 347.00,753.00 L 366.00,766.00 L 416.00,785.00 L 432.00,794.00 L 453.00,819.00 L 482.00,881.00 L 494.00,893.00 L 537.00,918.00 L 554.00,937.00 L 558.00,952.00" class="blood-vessel" />
  
    <!-- Pequeños detalles -->
    <path d="M 549.00,75.00 L 547.00,74.00 L 550.00,72.00 L 551.00,73.00 L 549.00,75.00" class="small-details" />
    <path d="M 262.00,84.00 L 261.00,83.00 L 262.00,82.00 L 263.00,83.00 L 262.00,84.00" class="small-details" />
    <path d="M 261.00,85.00 L 260.00,84.00 L 261.00,83.00 L 262.00,84.00 L 261.00,85.00" class="small-details" />
    <path d="M 766.00,100.00 L 765.00,99.00 L 766.00,97.00 L 767.00,98.00 L 766.00,100.00" class="small-details" />
    <path d="M 927.00,413.00 L 926.00,413.00 L 923.00,410.00 L 925.00,409.00 L 927.00,413.00" class="small-details" />
    <path d="M 929.00,415.00 L 927.00,413.00 L 928.00,412.00 L 930.00,413.00 L 929.00,415.00" class="small-details" />
    <path d="M 659.00,439.00 L 658.00,439.00 L 657.00,438.00 L 659.00,437.00 L 659.00,439.00" class="small-details" />
    <path d="M 600.00,463.00 L 599.00,462.00 L 600.00,461.00 L 601.00,462.00 L 600.00,463.00" class="small-details" />
    <path d="M 420.00,469.00 L 419.00,468.00 L 420.00,467.00 L 421.00,468.00 L 420.00,469.00" class="small-details" />
    <path d="M 526.00,589.00 L 525.00,588.00 L 526.00,587.00 L 527.00,588.00 L 526.00,589.00" class="small-details" />
    <path d="M 629.00,662.00 L 628.00,661.00 L 629.00,660.00 L 630.00,661.00 L 629.00,662.00" class="small-details" />
    <path d="M 99.00,738.00 L 96.00,737.00 L 102.00,736.00 L 100.00,737.00 L 99.00,738.00" class="small-details" />
    <path d="M 848.00,835.00 L 847.00,834.00 L 848.00,833.00 L 849.00,834.00 L 848.00,835.00" class="small-details" />
    <path d="M 542.00,839.00 L 542.00,839.00 L 542.00,839.00 L 542.00,839.00 L 542.00,839.00" class="small-details" />
  </svg>
</div>
        <!-- Instrumento único completo -->
        <div class="instrument-container">
          <div id="instrument-base" class="instrument-base"></div>
          <div id="instrument-rod" class="instrument-rod"></div>
          <div id="instrument-tip" class="instrument-tip"></div>
          <div id="instrument-action-indicator" class="instrument-action-indicator">Vitrectomía</div>
        </div>
        
        <!-- Coágulos de sangre -->
        <div id="blood-clots"></div>
        
        <!-- Marcas permanentes de láser y cauterio -->
        <div id="permanent-marks"></div>
      </div>
    </div>

    <!-- INDICADOR DE PROFUNDIDAD -->
    <div class="depth-indicator" id="depth-indicator">
      Profundidad: Media
    </div>
    
    <!-- EFECTO DE HEMORRAGIA -->
    <div id="hemorrhage-effect" class="hemorrhage-overlay"></div>
  </div>
  <script>
/* ================== VARIABLES GLOBALES ================== */
let lightJoystickX = 50, lightJoystickY = 50;
let vitrectomoJoystickX = 50, vitrectomoJoystickY = 50;
let currentDepth = parseInt(document.getElementById('vitrectomo-z-slider').value);
let activeInstrument = null;
let iop = 20; // mmHg (valor inicial ajustado a 20)
let perfusion = 95; // %
let hemorrhageRemoved = 0; // %
let bloodClots = [];
let laserBurns = [];
let cauteryMarks = [];
let hemorrhageActive = true;
let wallCollisionActive = false;
let retinaDetached = false;
let procedureStep = 0; // 0: vitrectomía, 1: cauterio, 2: láser
let isTouchingLight = false;
let isTouchingVitrectomo = false;
let lastLightX = 50, lastLightY = 50;
let lastVitrectomoX = 50, lastVitrectomoY = 50;
let joystickSensitivity = 0.7;
let cauteryPoints = 0;
let laserPoints = 0;
let activeTouchId = null;
let isActionButtonPressed = false;
let vitrectomyInterval = null;
let bloodClotsRemoved = 0;
const TOTAL_BLOOD_CLOTS = 20; // Total de coágulos iniciales

// Nuevas variables para control multitouch
let lightTouchId = null;
let vitrectomoTouchId = null;
let actionTouchId = null;

/* ================== INICIALIZACIÓN ================== */
document.addEventListener('DOMContentLoaded', function() {
  // Mostrar caso clínico al inicio
  document.getElementById('start-simulation-btn').addEventListener('click', function() {
    anime({
      targets: '#clinical-case',
      opacity: 0,
      duration: 500,
      easing: 'easeInOutQuad',
      complete: () => {
        document.getElementById('clinical-case').style.display = 'none';
        initSimulation();
      }
    });
  });
  
  // Botón para volver al menú
  document.getElementById('back-to-menu-btn').addEventListener('click', function() {
    location.reload();
  });
});

function initSimulation() {
  // Configurar el instrumento único
  setupSingleInstrument();
  
  initJoysticks();
  setupEventListeners();
  setupAlertDismissButtons();
  updateVitals();
  updateEndoLightEffect(50, 50);
  
  // Crear hemorragia inicial
  createInitialHemorrhage();
  
  requestAnimationFrame(updateInstrumentPositions);
}

function setupSingleInstrument() {
  const retina = document.getElementById('retina');
  const retinaRect = retina.getBoundingClientRect();
  
  // Configurar la base del instrumento (fija en la parte inferior derecha)
  const base = document.getElementById('instrument-base');
  base.style.left = (retinaRect.width * 0.85 - 15) + 'px';
  base.style.top = (retinaRect.height * 0.85 - 15) + 'px';
  
  // Configurar la punta del instrumento (inicialmente en el centro)
  const tip = document.getElementById('instrument-tip');
  tip.style.left = (retinaRect.width / 2) + 'px';
  tip.style.top = (retinaRect.height / 2) + 'px';
  
  // Actualizar el palo para que conecte base y punta
  updateInstrumentRod();
}

function updateInstrumentRod() {
  const retina = document.getElementById('retina');
  const retinaRect = retina.getBoundingClientRect();
  const base = document.getElementById('instrument-base');
  const baseRect = base.getBoundingClientRect();
  const tip = document.getElementById('instrument-tip');
  const tipRect = tip.getBoundingClientRect();
  const rod = document.getElementById('instrument-rod');
  
  // Posiciones absolutas
  const baseX = baseRect.left - retinaRect.left + baseRect.width/2;
  const baseY = baseRect.top - retinaRect.top + baseRect.height/2;
  const tipX = tipRect.left - retinaRect.left + tipRect.width/2;
  const tipY = tipRect.top - retinaRect.top + tipRect.height/2;
  
  // Calcular longitud y ángulo
  const dx = tipX - baseX;
  const dy = tipY - baseY;
  const length = Math.sqrt(dx * dx + dy * dy);
  const angle = Math.atan2(dy, dx) * (180 / Math.PI);
  
  // Posicionar y rotar el palo
  rod.style.width = length + 'px';
  rod.style.left = baseX + 'px';
  rod.style.top = baseY + 'px';
  rod.style.transform = `rotate(${angle}deg)`;
}

function createInitialHemorrhage() {
  const retina = document.getElementById('retina');
  const overlay = document.getElementById('hemorrhage-overlay');
  
  overlay.style.position = 'absolute';
  overlay.style.width = '100%';
  overlay.style.height = '100%';
  overlay.style.borderRadius = '50%';
  overlay.style.background = 'radial-gradient(circle at center, rgba(180,0,0,0.8) 0%, rgba(120,0,0,0.7) 50%, rgba(80,0,0,0.6) 100%)';
  overlay.style.zIndex = '13';
  overlay.style.pointerEvents = 'none';
  
  // Crear coágulos iniciales
  for (let i = 0; i < TOTAL_BLOOD_CLOTS; i++) {
    setTimeout(() => {
      createBloodClots(1);
    }, i * 100);
  }
}

/* ================== SISTEMA DE ALERTAS ================== */
function setupAlertDismissButtons() {
  document.querySelectorAll('.alert-dismiss').forEach(btn => {
    btn.addEventListener('click', function() {
      const alert = this.parentElement;
      anime({
        targets: alert,
        opacity: 0,
        translateY: -20,
        duration: 300,
        easing: 'easeOutQuad',
        complete: () => {
          alert.style.display = 'none';
          alert.style.opacity = '1';
          alert.style.transform = 'translateY(0)';
          if(alert.id === 'wall-collision-alert') {
            wallCollisionActive = false;
          }
        }
      });
    });
  });
}

function showAlert(alertId, duration = 5000) {
  const alert = document.getElementById(alertId);
  if (!alert) return;
  
  alert.style.display = 'flex';
  anime({
    targets: alert,
    opacity: [0, 1],
    translateY: [-20, 0],
    duration: 300,
    easing: 'easeOutQuad'
  });
  
  const timer = alert.querySelector('.alert-timer');
  if (timer) {
    anime({
      targets: timer,
      width: ['100%', '0%'],
      duration: duration,
      easing: 'linear'
    });
  }
  
  if (duration > 0) {
    setTimeout(() => {
      anime({
        targets: alert,
        opacity: 0,
        translateY: -20,
        duration: 300,
        easing: 'easeOutQuad',
        complete: () => {
          alert.style.display = 'none';
          alert.style.opacity = '1';
          alert.style.transform = 'translateY(0)';
          if(alertId === 'wall-collision-alert') {
            wallCollisionActive = false;
          }
        }
      });
    }, duration);
  }
  
  if(alertId === 'wall-collision-alert') {
    wallCollisionActive = true;
  }
}

function checkWallCollision() {
  const miniMapRect = document.getElementById('miniMapContainer').getBoundingClientRect();
  const retinaRect = document.getElementById('retina').getBoundingClientRect();
  
  // Verificar colisión para la punta del instrumento
  const tip = document.getElementById('instrument-tip');
  const tipRect = tip.getBoundingClientRect();
  const tipCenterX = tipRect.left + tipRect.width/2;
  const tipCenterY = tipRect.top + tipRect.height/2;
  
  // Coordenadas relativas al mini mapa
  const relX = (tipCenterX - retinaRect.left) / retinaRect.width * miniMapRect.width;
  const relY = (tipCenterY - retinaRect.top) / retinaRect.height * miniMapRect.height;
  
  // Verificar contacto con retina en mini mapa
  const retinaWall = document.getElementById('retina-wall');
  const retinaX = 400 * (miniMapRect.width / 800);
  const retinaY = 300 * (miniMapRect.height / 600);
  const retinaRadius = 250 * (miniMapRect.width / 800);
  
  const distance = Math.sqrt(Math.pow(relX - retinaX, 2) + Math.pow(relY - retinaY, 2));
  
  // Solo mostrar alerta si estamos en el borde (90% del radio)
  if (distance > retinaRadius * 1.1 && !wallCollisionActive) {
    showAlert('wall-collision-alert');
    return true;
  }
  
  // Verificar colisión para el endoiluminador
  const lightPosX = retinaRect.left + retinaRect.width * lightJoystickX / 100;
  const lightPosY = retinaRect.top + retinaRect.height * lightJoystickY / 100;
  
  // Coordenadas relativas al mini mapa
  const lightRelX = (lightPosX - retinaRect.left) / retinaRect.width * miniMapRect.width;
  const lightRelY = (lightPosY - retinaRect.top) / retinaRect.height * miniMapRect.height;
  
  // Verificar contacto con retina para el endoiluminador
  const lightDistance = Math.sqrt(Math.pow(lightRelX - retinaX, 2) + Math.pow(lightRelY - retinaY, 2));
  
  if (lightDistance > retinaRadius * 1.1 && !wallCollisionActive) {
    showAlert('wall-collision-alert');
    return true;
  }
  
  return false;
}

function checkVesselDamage() {
  if (currentDepth > -200 || !hemorrhageActive) return false;
  
  const retinaRect = document.getElementById('retina').getBoundingClientRect();
  const tip = document.getElementById('instrument-tip');
  const tipRect = tip.getBoundingClientRect();
  
  const tipCenterX = tipRect.left + tipRect.width/2;
  const tipCenterY = tipRect.top + tipRect.height/2;
  
  const bloodVessels = document.querySelector('.blood-vessels');
  const bloodVesselsRect = bloodVessels.getBoundingClientRect();
  
  const relX = tipCenterX - bloodVesselsRect.left;
  const relY = tipCenterY - bloodVesselsRect.top;
  
  const proximityThreshold = 30;
  const vesselProximity = Math.abs(relX - bloodVesselsRect.width/2) < proximityThreshold && 
                        Math.abs(relY - bloodVesselsRect.height/2) < proximityThreshold;
  
  if (vesselProximity && hemorrhageActive) {
    showAlert('vessel-damage-alert', 0);
    
    document.getElementById('hemorrhage-effect').style.display = 'block';
    anime({
      targets: '#hemorrhage-effect',
      opacity: 0.7,
      duration: 1000,
      easing: 'easeOutQuad'
    });
    
    createBloodClots(5, {x: tipCenterX, y: tipCenterY});
    
    perfusion -= 15;
    iop += 5;
    updateVitals();
    
    return true;
  }
  return false;
}

function createBloodClots(count, position = null) {
  const retinaRect = document.getElementById('retina').getBoundingClientRect();
  const bloodClotsContainer = document.getElementById('blood-clots');
  
  for (let i = 0; i < count; i++) {
    const clot = document.createElement('div');
    clot.className = 'blood-clot';
    
    let x, y;
    if (position) {
      x = position.x - retinaRect.left + (Math.random() * 40 - 20);
      y = position.y - retinaRect.top + (Math.random() * 40 - 20);
    } else {
      const angle = Math.random() * Math.PI * 2;
      const distance = Math.random() * retinaRect.width * 0.4;
      x = retinaRect.width/2 + Math.cos(angle) * distance;
      y = retinaRect.height/2 + Math.sin(angle) * distance;
    }
    
    clot.style.left = `${x}px`;
    clot.style.top = `${y}px`;
    const size = 10 + Math.random() * 20;
    clot.style.width = `${size}px`;
    clot.style.height = `${size}px`;
    
    // Animación de aparición con anime.js
    anime({
      targets: clot,
      scale: [0, 1],
      opacity: [0, 1],
      duration: 500,
      easing: 'easeOutElastic'
    });
    
    bloodClotsContainer.appendChild(clot);
    bloodClots.push({
      element: clot,
      x: x,
      y: y,
      size: size
    });
  }
}

function removeBloodClot(index) {
  if (index >= 0 && index < bloodClots.length) {
    const clot = bloodClots[index];
    if (clot.element.parentNode) {
      // Efecto de extracción de sangre
      createBloodExtractionEffect(clot.x + clot.size/2, clot.y + clot.size/2, clot.size);
      
      anime({
        targets: clot.element,
        opacity: 0,
        scale: 0.5,
        duration: 500,
        easing: 'easeInOutQuad',
        complete: () => {
          if (clot.element.parentNode) {
            clot.element.parentNode.removeChild(clot.element);
          }
        }
      });
    }
    bloodClots.splice(index, 1);
    bloodClotsRemoved++;
    
    // Actualizar porcentaje de hemorragia removida
    hemorrhageRemoved = Math.min(100, Math.floor((bloodClotsRemoved / TOTAL_BLOOD_CLOTS) * 100));
    document.getElementById('hemorrhage-value').textContent = `${hemorrhageRemoved}%`;
    document.getElementById('hemorrhage-level').style.width = `${hemorrhageRemoved}%`;
    
    // Reducir la opacidad de la hemorragia
    const overlay = document.getElementById('hemorrhage-overlay');
    overlay.style.opacity = (1 - (hemorrhageRemoved / 100)).toString();
    
    if (bloodClots.length === 0) {
      hemorrhageActive = false;
      anime({
        targets: '#hemorrhage-overlay',
        opacity: 0,
        duration: 1000,
        easing: 'easeInOutQuad',
        complete: () => {
          document.getElementById('hemorrhage-overlay').style.display = 'none';
          // Avanzar al siguiente paso del procedimiento
          if (procedureStep === 0) {
            procedureStep = 1;
            document.getElementById('btn-cautery').classList.add('active');
            activeInstrument = 'cautery';
            updateInstrumentTipAppearance();
          }
        }
      });
    }
  }
}

function createBloodExtractionEffect(x, y, size) {
  const retina = document.getElementById('retina');
  const bloodEffect = document.createElement('div');
  bloodEffect.className = 'blood-extraction';
  bloodEffect.style.left = `${x - 20}px`;
  bloodEffect.style.top = `${y - 20}px`;
  
  // Ajustar tamaño según el coágulo
  const effectSize = Math.max(20, Math.min(60, size * 2));
  bloodEffect.style.width = `${effectSize}px`;
  bloodEffect.style.height = `${effectSize}px`;
  
  anime({
    targets: bloodEffect,
    scale: [0, 1.5],
    opacity: [1, 0],
    duration: 800,
    easing: 'easeOutQuad',
    complete: () => {
      if (bloodEffect.parentNode) {
        bloodEffect.parentNode.removeChild(bloodEffect);
      }
    }
  });
  
  retina.appendChild(bloodEffect);
}

function removeNearbyBloodClots() {
  const retinaRect = document.getElementById('retina').getBoundingClientRect();
  const tip = document.getElementById('instrument-tip');
  const tipRect = tip.getBoundingClientRect();
  
  // Calcular posición central de la punta relativa a la retina
  const tipCenterX = tipRect.left - retinaRect.left + tipRect.width/2;
  const tipCenterY = tipRect.top - retinaRect.top + tipRect.height/2;
  
  // Hacer una copia del array para evitar problemas al modificar durante la iteración
  const clotsToCheck = [...bloodClots];
  
  clotsToCheck.forEach((clot, index) => {
    const dx = tipCenterX - clot.x;
    const dy = tipCenterY - clot.y;
    const distance = Math.sqrt(dx * dx + dy * dy);
    
    if (distance < clot.size + tipRect.width/2) {
      // Encontrar el índice real en el array bloodClots
      const realIndex = bloodClots.findIndex(c => 
        c.x === clot.x && c.y === clot.y && c.size === clot.size
      );
      
      if (realIndex !== -1) {
        removeBloodClot(realIndex);
        
        // Crear partículas de succión
        for (let i = 0; i < 5; i++) {
          const particle = document.createElement('div');
          particle.className = 'suction-particle';
          particle.style.left = `${clot.x + (Math.random()*20 - 10)}px`;
          particle.style.top = `${clot.y + (Math.random()*20 - 10)}px`;
          
          anime({
            targets: particle,
            translateX: Math.random() * 40 - 20,
            translateY: Math.random() * 40 - 20,
            opacity: [1, 0],
            duration: 1500,
            easing: 'easeOutQuad',
            complete: () => {
              if (particle.parentNode) {
                particle.parentNode.removeChild(particle);
              }
            }
          });
          
          document.getElementById('retina').appendChild(particle);
        }
      }
    }
  });
}

function checkRetinaDetachment() {
  if (iop > 28 && !retinaDetached) {
    showAlert('retina-detachment-alert');
    retinaDetached = true;
    return true;
  }
  return false;
}

function checkHighIOP() {
  if (iop > 25) {
    showAlert('high-iop-alert', 0);
    document.querySelector('.retina-nerve').style.backgroundColor = 'rgba(255,255,255,0.8)';
    document.querySelector('.optic-disc').style.boxShadow = 'inset 0 0 20px rgba(255,255,255,0.6), 0 0 25px rgba(255,255,255,0.5)';
    return true;
  } else {
    document.querySelector('.retina-nerve').style.backgroundColor = '';
    document.querySelector('.optic-disc').style.boxShadow = 'inset 0 0 20px rgba(220,80,80,0.6), 0 0 25px rgba(220,80,80,0.5)';
    document.getElementById('high-iop-alert').style.display = 'none';
    return false;
  }
}

/* ================== CONTROL DE JOYSTICKS MEJORADO PARA MÓVIL ================== */
function initJoysticks() {
  const joystickVitrectomo = document.getElementById('joystick-vitrectomo');
  initJoystick(joystickVitrectomo, (x, y) => {
    vitrectomoJoystickX = x;
    vitrectomoJoystickY = y;
    lastVitrectomoX = x;
    lastVitrectomoY = y;
    updateInstrumentPosition(x, y);
    updateMiniLeftLine(x, y);
    checkWallCollision();
  }, 'vitrectomo');
  
  const joystickLight = document.getElementById('joystick-light');
  initJoystick(joystickLight, (x, y) => {
    lightJoystickX = x;
    lightJoystickY = y;
    lastLightX = x;
    lastLightY = y;
    updateEndoLightEffect(x, y);
    updateMiniRightLine(x, y);
    checkWallCollision();
  }, 'light');
}

function initJoystick(joystickElement, updateCallback, type) {
  const handle = joystickElement.querySelector('.joystick-handle');
  const rect = joystickElement.getBoundingClientRect();
  const centerX = rect.width / 2;
  const centerY = rect.height / 2;
  const maxDistance = rect.width / 2 * joystickSensitivity;
  
  let isDragging = false;
  let touchId = null;
  
  function handleStart(e) {
    e.preventDefault();
    if (isDragging) return;
    
    isDragging = true;
    const touch = getTouch(e, joystickElement);
    if (!touch) return;
    
    touchId = touch.identifier;
    if (type === 'vitrectomo') {
      vitrectomoTouchId = touchId;
    } else {
      lightTouchId = touchId;
    }
    
    handleMove(e);
  }
  
  function handleMove(e) {
    if (!isDragging) return;
    
    const touch = getTouch(e, joystickElement, touchId);
    if (!touch) return;
    
    const bounds = joystickElement.getBoundingClientRect();
    const x = touch.clientX - bounds.left;
    const y = touch.clientY - bounds.top;
    
    let deltaX = x - centerX;
    let deltaY = y - centerY;
    const distance = Math.sqrt(deltaX * deltaX + deltaY * deltaY);
    
    if (distance > maxDistance) {
      const angle = Math.atan2(deltaY, deltaX);
      deltaX = Math.cos(angle) * maxDistance;
      deltaY = Math.sin(angle) * maxDistance;
    }
    
    // Suavizar el movimiento con anime.js
    anime({
      targets: handle,
      translateX: deltaX,
      translateY: deltaY,
      duration: 100,
      easing: 'easeOutQuad'
    });
    
    // Normalizar las coordenadas (0-100)
    const normalizedX = ((deltaX + maxDistance) / (2 * maxDistance)) * 100;
    const normalizedY = ((deltaY + maxDistance) / (2 * maxDistance)) * 100;
    
    updateCallback(normalizedX, normalizedY);
  }
  
  function handleEnd(e) {
    if (!isDragging) return;
    
    const touch = getTouch(e, joystickElement, touchId, true);
    if (!touch) return;
    
    isDragging = false;
    touchId = null;
    if (type === 'vitrectomo') {
      vitrectomoTouchId = null;
    } else {
      lightTouchId = null;
    }
    
    // Suavizar el retorno a la posición central con anime.js
    anime({
      targets: handle,
      translateX: 0,
      translateY: 0,
      duration: 300,
      easing: 'easeOutElastic(1, 0.5)'
    });
    
    // Solo resetear la posición del joystick, no del instrumento
    if (type === 'light') {
      updateCallback(50, 50);
    }
  }
  
  // Eventos táctiles mejorados para multitouch
  joystickElement.addEventListener('touchstart', handleStart, { passive: false });
  document.addEventListener('touchmove', (e) => {
    if (isDragging) {
      e.preventDefault();
      handleMove(e);
    }
  }, { passive: false });
  
  document.addEventListener('touchend', handleEnd);
  document.addEventListener('touchcancel', handleEnd);
  
  // Eventos de ratón (para compatibilidad con desktop)
  joystickElement.addEventListener('mousedown', handleStart);
  document.addEventListener('mousemove', handleMove);
  document.addEventListener('mouseup', handleEnd);
}

// Función auxiliar para manejar eventos táctiles
function getTouch(e, element, touchId = null, isEndEvent = false) {
  if (e.touches) {
    const touches = Array.from(isEndEvent ? e.changedTouches : e.touches);
    if (touchId !== null) {
      return touches.find(t => t.identifier === touchId);
    }
    return touches.find(t => {
      const target = document.elementFromPoint(t.clientX, t.clientY);
      return target === element || target === element.querySelector('.joystick-handle');
    });
  }
  return { clientX: e.clientX, clientY: e.clientY };
}

/* ================== ACTUALIZACIÓN DE POSICIONES ================== */
function updateInstrumentPositions() {
  updateInstrumentPosition(vitrectomoJoystickX, vitrectomoJoystickY);
  requestAnimationFrame(updateInstrumentPositions);
}

function updateInstrumentPosition(normX, normY) {
  const retina = document.getElementById('retina');
  const retinaRect = retina.getBoundingClientRect();
  const base = document.getElementById('instrument-base');
  const tip = document.getElementById('instrument-tip');
  const rod = document.getElementById('instrument-rod');
  
  // Calcular posición de la punta basada en el joystick
  const maxOffset = retinaRect.width * 0.4; // Reducir el rango para que no salga de la retina
  const offsetX = (normX - 50) / 50 * maxOffset;
  const offsetY = (normY - 50) / 50 * maxOffset;
  
  // Posición absoluta de la punta
  const tipX = retinaRect.width / 2 + offsetX;
  const tipY = retinaRect.height / 2 + offsetY;
  
  // Posición de la base (fija en la parte inferior derecha)
  const baseX = retinaRect.width * 0.85;
  const baseY = retinaRect.height * 0.85;
  
  // Actualizar posición de la punta
  tip.style.left = tipX + 'px';
  tip.style.top = tipY + 'px';
  
  // Actualizar el palo para que conecte base y punta
  updateInstrumentRod();
  
  // Actualizar profundidad
  tip.style.transform = `translateZ(${currentDepth}px)`;
  
  checkWallCollision();
  
  if (currentDepth <= -200) {
    checkVesselDamage();
  }
  
  updateDepthIndicator();
}

function updateDepthIndicator() {
  const depthIndicator = document.getElementById('depth-indicator');
  
  if (currentDepth > -80) {
    depthIndicator.textContent = "Profundidad: Superficial";
    depthIndicator.style.setProperty('--depth-color', '#3b82f6');
  } else if (currentDepth > -150) {
    depthIndicator.textContent = "Profundidad: Media";
    depthIndicator.style.setProperty('--depth-color', '#10b981');
  } else {
    depthIndicator.textContent = "Profundidad: Profunda";
    depthIndicator.style.setProperty('--depth-color', '#ef4444');
  }
}

function updateEndoLightEffect(normX, normY) {
  document.documentElement.style.setProperty('--light-x', normX + '%');
  document.documentElement.style.setProperty('--light-y', normY + '%');
  
  const zVal = parseInt(document.getElementById('endo-z-slider').value);
  const retinaRect = document.getElementById('retina').getBoundingClientRect();
  const newLightSize = 20 + (retinaRect.width - 20) * (zVal / 200);
  
  document.documentElement.style.setProperty('--light-size', newLightSize + 'px');
  document.getElementById('light-reflection').style.opacity = 0.7;
  
  if (zVal > 100) {
    document.getElementById('light-scatter').classList.add('active');
    document.getElementById('specular-highlight').classList.add('active');
  } else {
    document.getElementById('light-scatter').classList.remove('active');
    document.getElementById('specular-highlight').classList.remove('active');
  }
}

function updateMiniLeftLine(normX, normY) {
  const defaultTipX = 350;
  const defaultTipY = 300;
  const scaleX = 2.5;
  const scaleY = 1.8;
  const offsetX = (normX - 50) * scaleX;
  const offsetY = (normY - 50) * scaleY;
  let tipX = defaultTipX + offsetX;
  let tipY = defaultTipY + offsetY;
  
  tipX = Math.max(50, Math.min(750, tipX));
  tipY = Math.max(50, Math.min(550, tipY));
  
  const miniLeft = document.getElementById('probeLight');
  const miniLeftInner = document.getElementById('probeLightInner');
  if(miniLeft && miniLeftInner) {
    miniLeft.setAttribute('x2', tipX);
    miniLeftInner.setAttribute('x2', tipX);
    miniLeft.setAttribute('y2', tipY);
    miniLeftInner.setAttribute('y2', tipY);
  }
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
  
  tipX = Math.max(50, Math.min(750, tipX));
  tipY = Math.max(50, Math.min(550, tipY));
  
  const miniRight = document.getElementById('probeForceps');
  const miniRightInner = document.getElementById('probeForcepsInner');
  if(miniRight && miniRightInner) {
    miniRight.setAttribute('x2', tipX);
    miniRightInner.setAttribute('x2', tipX);
    miniRight.setAttribute('y2', tipY);
    miniRightInner.setAttribute('y2', tipY);
  }
}

/* ================== GESTIÓN DE INSTRUMENTOS MEJORADAS PARA MÓVIL ================== */
function setupEventListeners() {
  document.getElementById('btn-vitrectomo').addEventListener('click', () => toggleInstrument('btn-vitrectomo', 'vitrectomo'));
  document.getElementById('btn-laser').addEventListener('click', () => toggleInstrument('btn-laser', 'laser'));
  document.getElementById('btn-cautery').addEventListener('click', () => toggleInstrument('btn-cautery', 'cautery'));
  
  // Mejorar los eventos para el botón de acción en móvil
  const actionButton = document.getElementById('btn-precionar');
  
  // Eventos táctiles mejorados para multitouch
  actionButton.addEventListener('touchstart', function(e) {
    e.preventDefault();
    const touch = e.touches[0];
    actionTouchId = touch.identifier;
    startAction();
  }, { passive: false });
  
  actionButton.addEventListener('touchend', function(e) {
    e.preventDefault();
    const touch = e.changedTouches[0];
    if (touch.identifier === actionTouchId) {
      stopAction();
      actionTouchId = null;
    }
  });
  
  actionButton.addEventListener('touchcancel', function(e) {
    e.preventDefault();
    const touch = e.changedTouches[0];
    if (touch.identifier === actionTouchId) {
      stopAction();
      actionTouchId = null;
    }
  });
  
  // Eventos de ratón (para compatibilidad con desktop)
  actionButton.addEventListener('mousedown', startAction);
  actionButton.addEventListener('mouseup', stopAction);
  actionButton.addEventListener('mouseleave', stopAction);
  
  document.getElementById('vitrectomo-z-slider').addEventListener('input', function() {
    currentDepth = parseInt(this.value);
    updateInstrumentPosition(vitrectomoJoystickX, vitrectomoJoystickY);
  });
  
  document.getElementById('endo-z-slider').addEventListener('input', function() {
    updateEndoLightEffect(lightJoystickX, lightJoystickY);
  });
}

function startAction() {
  if (!activeInstrument) return;
  
  isActionButtonPressed = true;
  
  // Mostrar indicador de acción
  const actionIndicator = document.getElementById('instrument-action-indicator');
  if (actionIndicator) {
    actionIndicator.classList.add('active');
    
    // Actualizar texto según el instrumento activo
    if (activeInstrument === 'vitrectomo') {
      actionIndicator.textContent = "Vitrectomía";
    } else if (activeInstrument === 'laser') {
      actionIndicator.textContent = "Aplicando láser";
    } else if (activeInstrument === 'cautery') {
      actionIndicator.textContent = "Coagulando";
    }
  }
  
  // Iniciar el efecto continuo
  vitrectomyInterval = setInterval(() => {
    if (!isActionButtonPressed || !activeInstrument) {
      clearInterval(vitrectomyInterval);
      return;
    }
    
    const retinaRect = document.getElementById('retina').getBoundingClientRect();
    const tip = document.getElementById('instrument-tip');
    const tipRect = tip.getBoundingClientRect();
    
    // Calcular posición relativa al centro de la retina
    const offsetX = tipRect.left - retinaRect.left + tipRect.width/2 - retinaRect.width/2;
    const offsetY = tipRect.top - retinaRect.top + tipRect.height/2 - retinaRect.height/2;
    
    // Crear efecto visual
    if (activeInstrument === 'vitrectomo') {
      createVitrectomyEffect(tipRect.left + tipRect.width/2, tipRect.top + tipRect.height/2);
      removeNearbyBloodClots();
      
      // Actualizar parámetros fisiológicos
      iop = Math.max(18, iop - 0.1);
      perfusion = Math.min(100, perfusion + 0.05);
    } else if (activeInstrument === 'laser' && procedureStep === 2) {
      laserFunction({ 
        clientX: tipRect.left + tipRect.width/2, 
        clientY: tipRect.top + tipRect.height/2,
        preventDefault: () => {}
      });
    } else if (activeInstrument === 'cautery' && procedureStep === 1) {
      cauteryFunction({ 
        clientX: tipRect.left + tipRect.width/2, 
        clientY: tipRect.top + tipRect.height/2,
        preventDefault: () => {}
      });
    }
    
    // Actualizar UI inmediatamente
    updateVitals();
    
  }, 100); // Ejecutar cada 100ms mientras se mantenga presionado
}

function stopAction() {
  isActionButtonPressed = false;
  
  // Ocultar indicador de acción
  const actionIndicator = document.getElementById('instrument-action-indicator');
  if (actionIndicator) {
    actionIndicator.classList.remove('active');
  }
  
  // Detener el efecto continuo
  if (vitrectomyInterval) {
    clearInterval(vitrectomyInterval);
    vitrectomyInterval = null;
  }
}

function createVitrectomyEffect(x, y) {
  const retina = document.getElementById('retina');
  const retinaRect = retina.getBoundingClientRect();
  
  // Efecto de vitrectomía
  const vitrectomyEffect = document.createElement('div');
  vitrectomyEffect.className = 'vitrectomy-effect';
  vitrectomyEffect.style.left = (x - retinaRect.left - 30) + 'px';
  vitrectomyEffect.style.top = (y - retinaRect.top - 30) + 'px';
  
  anime({
    targets: vitrectomyEffect,
    scale: [0, 1],
    opacity: [0.8, 0],
    duration: 800,
    easing: 'easeOutQuad',
    complete: () => {
      if (vitrectomyEffect.parentNode) {
        vitrectomyEffect.parentNode.removeChild(vitrectomyEffect);
      }
    }
  });
  
  retina.appendChild(vitrectomyEffect);
  
  // Partículas de succión
  for (let i = 0; i < 8; i++) {
    const particle = document.createElement('div');
    particle.className = 'suction-particle';
    particle.style.left = (x - retinaRect.left + (Math.random()*20 - 10)) + 'px';
    particle.style.top = (y - retinaRect.top + (Math.random()*20 - 10)) + 'px';
    
    anime({
      targets: particle,
      translateX: Math.random() * 40 - 20,
      translateY: Math.random() * 40 - 20,
      opacity: [1, 0],
      duration: 1500,
      easing: 'easeOutQuad',
      complete: () => {
        if (particle.parentNode) {
          particle.parentNode.removeChild(particle);
        }
      }
    });
    
    retina.appendChild(particle);
  }
}

function toggleInstrument(btnId, instrumentType) {
  const btn = document.getElementById(btnId);
  
  if(btn.classList.contains('active')) {
    btn.classList.remove('active');
    activeInstrument = null;
    isActionButtonPressed = false;
    
    // Resetear apariencia de la punta
    const tip = document.getElementById('instrument-tip');
    tip.className = 'instrument-tip';
    
    // Detener cualquier intervalo activo
    if (vitrectomyInterval) {
      clearInterval(vitrectomyInterval);
      vitrectomyInterval = null;
    }
  } else {
    document.querySelectorAll('.toggle-btn').forEach(b => b.classList.remove('active'));
    
    btn.classList.add('active');
    activeInstrument = instrumentType;
    
    // Actualizar apariencia de la punta según el instrumento
    updateInstrumentTipAppearance();
  }
}

function updateInstrumentTipAppearance() {
  const tip = document.getElementById('instrument-tip');
  tip.className = 'instrument-tip';
  
  if (activeInstrument === 'vitrectomo') {
    tip.classList.add('active-vitrectomo');
  } else if (activeInstrument === 'laser') {
    tip.classList.add('active-laser');
  } else if (activeInstrument === 'cautery') {
    tip.classList.add('active-cautery');
  }
}

/* ================== FUNCIONES DE INSTRUMENTOS MEJORADAS ================== */
function laserFunction(e) {
  const retina = document.getElementById('retina');
  const retinaRect = retina.getBoundingClientRect();
  
  // Verificar que no esté en la mácula, nervio óptico o vasos sanguíneos
  if (isInSensitiveArea(e.clientX, e.clientY)) {
    return;
  }
  
  // Verificar que no esté demasiado cerca de otro punto de láser
  const tooClose = laserBurns.some(burn => {
    const dx = e.clientX - retinaRect.left - burn.x;
    const dy = e.clientY - retinaRect.top - burn.y;
    return Math.sqrt(dx * dx + dy * dy) < 30;
  });
  
  if (tooClose) return;
  
  // Crear efecto de láser temporal
  const laserSpot = document.createElement('div');
  laserSpot.className = 'laser-spot';
  laserSpot.style.left = (e.clientX - retinaRect.left - 12) + 'px';
  laserSpot.style.top = (e.clientY - retinaRect.top - 12) + 'px';
  
  anime({
    targets: laserSpot,
    scale: [0.5, 1.2, 1],
    opacity: [0, 1, 0.8],
    duration: 500,
    easing: 'easeOutQuad'
  });
  
  retina.appendChild(laserSpot);
  
  // Crear marca permanente de láser (punto blanco)
  const burnMark = document.createElement('div');
  burnMark.className = 'laser-burn-permanent';
  burnMark.style.left = (e.clientX - retinaRect.left - 3) + 'px';
  burnMark.style.top = (e.clientY - retinaRect.top - 3) + 'px';
  
  // Añadir brillo extra
  burnMark.style.boxShadow = '0 0 8px rgba(255,255,255,0.9)';
  
  anime({
    targets: burnMark,
    scale: [0, 1],
    opacity: [0, 1],
    duration: 500,
    easing: 'easeOutElastic'
  });
  
  document.getElementById('permanent-marks').appendChild(burnMark);
  
  laserBurns.push({
    element: burnMark,
    x: e.clientX - retinaRect.left - 3,
    y: e.clientY - retinaRect.top - 3
  });
  
  laserPoints++;
  document.getElementById('laser-value').textContent = `${laserPoints}/30`;
  
  // Actualizar parámetros fisiológicos
  iop += 0.5;
  perfusion -= 0.2;
  updateVitals();
  
  // Eliminar efecto de láser después de 2.5 segundos
  setTimeout(() => {
    if (laserSpot.parentNode) {
      laserSpot.parentNode.removeChild(laserSpot);
    }
  }, 2500);
  
  // Completar cirugía si se alcanzan los 30 puntos
  if (laserPoints >= 30) {
    setTimeout(() => {
      showAlert('surgery-complete-alert');
    }, 1000);
  }
}

function cauteryFunction(e) {
  const retina = document.getElementById('retina');
  const retinaRect = retina.getBoundingClientRect();
  
  // Verificar que no esté en la mácula, nervio óptico o vasos sanguíneos
  if (isInSensitiveArea(e.clientX, e.clientY)) {
    return;
  }
  
  // Verificar que no esté demasiado cerca de otro punto de cauterio
  const tooClose = cauteryMarks.some(mark => {
    const dx = e.clientX - retinaRect.left - mark.x;
    const dy = e.clientY - retinaRect.top - mark.y;
    return Math.sqrt(dx * dx + dy * dy) < 30;
  });
  
  if (tooClose) return;
  
  // Efecto visual de cauterio temporal
  const cauteryEffect = document.createElement('div');
  cauteryEffect.className = 'cautery-effect';
  cauteryEffect.style.left = (e.clientX - retinaRect.left - 15) + 'px';
  cauteryEffect.style.top = (e.clientY - retinaRect.top - 15) + 'px';
  
  anime({
    targets: cauteryEffect,
    scale: [0.5, 1.3, 1],
    opacity: [0, 1, 0],
    duration: 1000,
    easing: 'easeOutQuad',
    complete: () => {
      if (cauteryEffect.parentNode) {
        cauteryEffect.parentNode.removeChild(cauteryEffect);
      }
    }
  });
  
  retina.appendChild(cauteryEffect);
  
  // Crear marca permanente de cauterio (punto blanco)
  const burnMark = document.createElement('div');
  burnMark.className = 'laser-burn-permanent';
  burnMark.style.left = (e.clientX - retinaRect.left - 3) + 'px';
  burnMark.style.top = (e.clientY - retinaRect.top - 3) + 'px';
  
  // Añadir brillo extra
  burnMark.style.boxShadow = '0 0 8px rgba(255,255,255,0.9)';
  
  anime({
    targets: burnMark,
    scale: [0, 1],
    opacity: [0, 1],
    duration: 500,
    easing: 'easeOutElastic'
  });
  
  document.getElementById('permanent-marks').appendChild(burnMark);
  
  cauteryMarks.push({
    element: burnMark,
    x: e.clientX - retinaRect.left - 3,
    y: e.clientY - retinaRect.top - 3
  });
  
  cauteryPoints++;
  document.getElementById('cautery-value').textContent = `${cauteryPoints}/15`;
  
  // Actualizar parámetros fisiológicos
  perfusion += 1;
  iop = Math.min(30, iop + 0.3);
  updateVitals();
  
  // Avanzar al siguiente paso si se completan los puntos de cauterio
  if (cauteryPoints >= 15 && procedureStep === 1) {
    procedureStep = 2;
    document.getElementById('btn-laser').classList.add('active');
    activeInstrument = 'laser';
    updateInstrumentTipAppearance();
  }
}

function isInSensitiveArea(x, y) {
  const retina = document.getElementById('retina');
  const retinaRect = retina.getBoundingClientRect();
  
  // Verificar mácula
  const macula = document.querySelector('.macula');
  const maculaRect = macula.getBoundingClientRect();
  const maculaDistance = Math.sqrt(
    Math.pow(x - (maculaRect.left + maculaRect.width/2), 2) + 
    Math.pow(y - (maculaRect.top + maculaRect.height/2), 2)
  );
  
  if (maculaDistance < maculaRect.width/2) {
    showAlert('vision-loss-alert');
    return true;
  }
  
  // Verificar nervio óptico
  const opticDisc = document.querySelector('.optic-disc');
  const opticDiscRect = opticDisc.getBoundingClientRect();
  const opticDiscDistance = Math.sqrt(
    Math.pow(x - (opticDiscRect.left + opticDiscRect.width/2), 2) + 
    Math.pow(y - (opticDiscRect.top + opticDiscRect.height/2), 2)
  );
  
  if (opticDiscDistance < opticDiscRect.width/2) {
    return true;
  }
  
  // Verificar vasos sanguíneos (simplificado)
  const bloodVessels = document.querySelector('.blood-vessels');
  const bloodVesselsRect = bloodVessels.getBoundingClientRect();
  const relX = x - bloodVesselsRect.left;
  const relY = y - bloodVesselsRect.top;
  
  const proximityThreshold = 15;
  const vesselProximity = Math.abs(relX - bloodVesselsRect.width/2) < proximityThreshold && 
                        Math.abs(relY - bloodVesselsRect.height/2) < proximityThreshold;
  
  if (vesselProximity) {
    showAlert('vessel-damage-alert');
    return true;
  }
  
  return false;
}

/* ================== ACTUALIZACIÓN DE PARÁMETROS ================== */
function updateVitals() {
  // Variación natural de los parámetros
  iop += (Math.random() - 0.5) * 0.1;
  perfusion += (Math.random() - 0.5) * 0.2;
  
  // Mantener la PIO alrededor de 20 mmHg
  if (iop < 19.5) iop += 0.1;
  if (iop > 20.5) iop -= 0.1;
  
  checkRetinaDetachment();
  checkHighIOP();
  
  // Limitar valores dentro de rangos razonables
  iop = Math.max(18, Math.min(30, iop));
  perfusion = Math.max(60, Math.min(100, perfusion));
  
  // Actualizar valores en la interfaz
  document.getElementById('iop-value').innerText = iop.toFixed(1) + " mmHg";
  document.getElementById('iop-level').style.width = ((iop - 10) / 20 * 100) + "%";
  
  document.getElementById('perfusion-value').innerText = perfusion.toFixed(0) + "%";
  document.getElementById('perfusion-level').style.width = perfusion + "%";
  
  // Actualizar clases de estado
  const iopElement = document.getElementById('iop-value');
  iopElement.classList.remove('normal', 'warning', 'danger');
  
  if(iop < 15 || iop > 25) {
    iopElement.classList.add('danger');
  } else if(iop < 18 || iop > 22) {
    iopElement.classList.add('warning');
  } else {
    iopElement.classList.add('normal');
  }
  
  setTimeout(updateVitals, 1000);
}
 </script>
</body>
</html>
