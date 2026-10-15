<!DOCTYPE html>
<html lang="pl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>GLIWICE: KRONIKI ZSTI — Fabularna Gra Miejska</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Chakra+Petch:ital,wght@0,500;0,600;0,700;1,700&family=Press+Start+2P&family=VT323&display=swap" rel="stylesheet">
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<style>
:root {
  --bg-dark: #101520;
  --bg-panel: rgba(18, 24, 36, 0.95);
  --bg-card: #1b2436;
  --gold: #f5b942;
  --gold-bright: #ffd966;
  --cyan: #38bdf8;
  --green: #22c55e;
  --red: #ef4444;
  --orange: #f97316;
  --text-main: #f8fafc;
  --text-muted: #94a3b8;
  --border-steel: #334155;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  user-select: none;
  -webkit-user-select: none;
}

body, html {
  width: 100%;
  height: 100%;
  overflow: hidden;
  background-color: var(--bg-dark);
  font-family: 'Chakra Petch', sans-serif;
  color: var(--text-main);
}

/* MAPA SATELITARNA */
#game-wrapper {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}

#map {
  width: 100%;
  height: 100%;
  z-index: 1;
  background: #0d161c;
}

/* Lekki kontrast dla zdjęć satelitarnych, żeby budynki z lotu ptaka były ostre i wyraźne */
.leaflet-tile-pane {
  filter: contrast(1.08) saturate(1.12);
}

.leaflet-control-attribution {
  font-size: 9px !important;
  background: rgba(15, 23, 42, 0.8) !important;
  color: #cbd5e1 !important;
  border-radius: 4px 0 0 0;
}
.leaflet-control-attribution a { color: var(--gold) !important; }

/* GÓRNY HUD */
.top-hud {
  position: absolute;
  top: 12px;
  left: 12px;
  right: 12px;
  z-index: 500;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 12px;
  pointer-events: none;
}

.hud-box {
  pointer-events: auto;
  background: var(--bg-panel);
  border: 2px solid var(--border-steel);
  border-radius: 12px;
  box-shadow: 0 10px 25px rgba(0,0,0,0.65), inset 0 1px 0 rgba(255,255,255,0.1);
  backdrop-filter: blur(8px);
}

/* Karta Gracza */
.player-card {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 8px 14px;
  border-left: 4px solid var(--gold);
}

.player-avatar {
  width: 44px;
  height: 44px;
  background: linear-gradient(135deg, #1e293b, #0f172a);
  border: 2px solid var(--gold);
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.player-info {
  display: flex;
  flex-direction: column;
  gap: 3px;
}

.player-title {
  font-family: 'Press Start 2P', monospace;
  font-size: 9px;
  color: var(--gold);
}

.hearts-row {
  display: flex;
  gap: 5px;
  align-items: center;
}

.heart-svg {
  width: 20px;
  height: 20px;
  filter: drop-shadow(0 2px 3px rgba(0,0,0,0.7));
  transition: transform 0.25s;
}
.heart-svg.lost {
  opacity: 0.22;
  filter: grayscale(1);
  transform: scale(0.85);
}

.player-stats {
  display: flex;
  gap: 10px;
  font-size: 12px;
  font-weight: 700;
  color: var(--text-muted);
}
.player-stats b { color: var(--cyan); }

/* Środkowy Baner Fabularny */
.story-tracker {
  padding: 10px 18px;
  max-width: 540px;
  text-align: center;
  border-top: 3px solid var(--gold);
}

.story-chapter-tag {
  font-family: 'Press Start 2P', monospace;
  font-size: 9px;
  color: var(--gold);
  text-transform: uppercase;
  margin-bottom: 4px;
  letter-spacing: 0.5px;
}

.story-objective {
  font-size: 14px;
  font-weight: 700;
  color: #fff;
}

.story-sub {
  font-size: 12px;
  color: var(--cyan);
  margin-top: 2px;
}

/* Prawy Panel Akcji */
.hud-actions {
  display: flex;
  gap: 8px;
}

.hud-btn {
  pointer-events: auto;
  background: var(--bg-panel);
  color: var(--text-main);
  border: 2px solid var(--border-steel);
  border-radius: 10px;
  padding: 10px 14px;
  font-family: 'Chakra Petch', sans-serif;
  font-weight: 700;
  font-size: 13px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 6px;
  transition: all 0.15s;
  box-shadow: 0 6px 16px rgba(0,0,0,0.5);
}

.hud-btn:hover {
  border-color: var(--gold);
  color: var(--gold);
  transform: translateY(-2px);
}

.hud-btn.bus-btn {
  background: linear-gradient(135deg, #b45309, #d97706);
  border-color: var(--gold-bright);
  color: #fff;
}
.hud-btn.bus-btn:hover {
  filter: brightness(1.1);
}

/* DOLNY PASEK KONTROLI I WEJŚCIA DO BUDYNKU */
.bottom-hud {
  position: absolute;
  bottom: 16px;
  left: 16px;
  right: 16px;
  z-index: 500;
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  pointer-events: none;
}

.dpad-box {
  pointer-events: auto;
  background: var(--bg-panel);
  border: 2px solid var(--border-steel);
  border-radius: 14px;
  padding: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  box-shadow: 0 8px 22px rgba(0,0,0,0.6);
}

.dpad-label {
  font-size: 10px;
  color: var(--text-muted);
  font-weight: 700;
  text-transform: uppercase;
}

.dpad-grid {
  display: grid;
  grid-template-columns: repeat(3, 40px);
  grid-template-rows: repeat(3, 40px);
  gap: 4px;
}

.dpad-btn {
  background: #1e293b;
  border: 2px solid #475569;
  border-radius: 8px;
  color: #f8fafc;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.1s;
}
.dpad-btn:active, .dpad-btn.active {
  background: var(--gold);
  color: #0f172a;
  border-color: #fff;
  transform: scale(0.94);
}
.dpad-up { grid-column: 2; grid-row: 1; }
.dpad-left { grid-column: 1; grid-row: 2; }
.dpad-center {
  grid-column: 2; grid-row: 2;
  background: #0f172a;
  border-radius: 50%;
  border: 1px solid #334155;
  pointer-events: none;
}
.dpad-right { grid-column: 3; grid-row: 2; }
.dpad-down { grid-column: 2; grid-row: 3; }

/* Karta Aktywnej Lokalizacji na dole na środku */
.location-dock {
  pointer-events: auto;
  background: var(--bg-panel);
  border: 2px solid var(--gold);
  border-radius: 14px;
  padding: 14px 20px;
  max-width: 560px;
  width: 100%;
  margin: 0 16px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  box-shadow: 0 12px 32px rgba(0,0,0,0.8);
}

.loc-dock-info h3 {
  font-size: 16px;
  color: var(--gold);
  margin-bottom: 3px;
}

.loc-dock-info p {
  font-size: 12px;
  color: #cbd5e1;
  line-height: 1.35;
}

.loc-action-group {
  display: flex;
  gap: 8px;
  flex-shrink: 0;
}

.action-primary-btn {
  background: linear-gradient(135deg, #16a34a, #15803d);
  color: #fff;
  border: 2px solid #86efac;
  border-radius: 10px;
  padding: 12px 18px;
  font-family: 'Press Start 2P', monospace;
  font-size: 10px;
  cursor: pointer;
  box-shadow: 0 0 18px rgba(34, 197, 94, 0.5);
  transition: transform 0.15s;
  white-space: nowrap;
}
.action-primary-btn:hover {
  transform: scale(1.04);
}
.action-primary-btn.bus-mode {
  background: linear-gradient(135deg, #d97706, #b45309);
  border-color: var(--gold-bright);
  box-shadow: 0 0 18px rgba(245, 185, 66, 0.5);
}

/* ZMNIEJSZONA POSTAĆ GRACZA NA MAPIE SATELITARNEJ */
.hero-map-marker {
  width: 34px;
  height: 44px;
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  pointer-events: none;
  filter: drop-shadow(0 4px 5px rgba(0,0,0,0.85));
}

.hero-you-badge {
  background: var(--gold);
  color: #0f172a;
  font-family: 'Press Start 2P', monospace;
  font-size: 6px;
  padding: 2px 4px;
  border-radius: 3px;
  border: 1px solid #000;
  margin-bottom: 1px;
  white-space: nowrap;
}

.hero-sprite-small {
  width: 26px;
  height: 32px;
  transition: transform 0.15s;
}

.hero-walking .hero-sprite-small {
  animation: walkBobSmall 0.3s infinite alternate;
}

@keyframes walkBobSmall {
  0% { transform: translateY(0) rotate(-3deg); }
  100% { transform: translateY(-4px) rotate(3deg); }
}

/* MARKERY LOKALIZACJI NA MAPIE SATELITARNEJ */
.loc-pin-wrap {
  display: flex;
  flex-direction: column;
  align-items: center;
  cursor: pointer;
  transition: transform 0.2s;
}
.loc-pin-wrap:hover {
  transform: scale(1.12) translateY(-3px);
}

.loc-pin-badge {
  background: rgba(15, 23, 42, 0.95);
  border: 2px solid #64748b;
  color: #e2e8f0;
  padding: 3px 7px;
  border-radius: 6px;
  font-size: 10px;
  font-weight: 700;
  white-space: nowrap;
  box-shadow: 0 4px 10px rgba(0,0,0,0.75);
  margin-bottom: 3px;
}

.loc-pin-wrap.current-quest .loc-pin-badge {
  border-color: var(--gold);
  background: #1e1b4b;
  color: var(--gold-bright);
  box-shadow: 0 0 15px rgba(245, 185, 66, 0.8);
  animation: pulseQuest 1.4s infinite alternate;
}

.loc-pin-wrap.completed-quest .loc-pin-badge {
  border-color: var(--green);
  color: #86efac;
}

@keyframes pulseQuest {
  0% { transform: scale(1); }
  100% { transform: scale(1.07); }
}

.loc-pin-circle {
  width: 34px;
  height: 34px;
  border-radius: 50%;
  background: #1e293b;
  border: 3px solid #94a3b8;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.8);
}
.loc-pin-wrap.current-quest .loc-pin-circle {
  border-color: var(--gold);
  background: #422006;
}
.loc-pin-wrap.completed-quest .loc-pin-circle {
  border-color: var(--green);
  background: #064e3b;
}

/* MODALE I EKRANY PEŁNOEKRANOWE */
.overlay-modal {
  position: fixed;
  inset: 0;
  z-index: 2000;
  background: rgba(8, 12, 20, 0.88);
  backdrop-filter: blur(8px);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px;
}
.hidden { display: none !important; }

/* MENU STARTU */
.start-window {
  max-width: 820px;
  width: 100%;
  background: linear-gradient(180deg, #161f30 0%, #0f1623 100%);
  border: 3px solid var(--gold);
  border-radius: 18px;
  padding: 28px;
  box-shadow: 0 25px 70px rgba(0,0,0,0.9);
  display: flex;
  flex-direction: column;
  gap: 18px;
}

.start-header {
  display: flex;
  gap: 20px;
  align-items: center;
  border-bottom: 2px solid #26334d;
  padding-bottom: 16px;
}

.start-title-block h1 {
  font-family: 'Press Start 2P', monospace;
  font-size: 20px;
  color: var(--gold);
  line-height: 1.4;
  margin-bottom: 6px;
  text-shadow: 0 3px 0 #000;
}

.start-title-block p {
  color: var(--cyan);
  font-size: 14px;
  font-weight: 700;
}

.start-features-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 12px;
}

.feature-card {
  background: rgba(15, 23, 42, 0.75);
  border: 1px solid #334155;
  border-left: 4px solid var(--gold);
  border-radius: 10px;
  padding: 12px;
}
.feature-card h4 {
  color: var(--gold-bright);
  font-size: 13px;
  margin-bottom: 4px;
}
.feature-card p {
  font-size: 12px;
  color: #cbd5e1;
  line-height: 1.4;
}

.start-btn-big {
  background: linear-gradient(180deg, #f5b942 0%, #d97706 100%);
  color: #0f172a;
  border: 3px solid #fef08a;
  border-radius: 12px;
  padding: 16px;
  font-family: 'Press Start 2P', monospace;
  font-size: 13px;
  cursor: pointer;
  text-align: center;
  box-shadow: 0 8px 25px rgba(245, 185, 66, 0.4);
  transition: transform 0.15s;
}
.start-btn-big:hover {
  transform: translateY(-2px);
}

/* SCENKA PRZEJAZDU AUTOBUSEM ZTM GLIWICE */
.bus-cutscene-window {
  max-width: 720px;
  width: 100%;
  background: #0f172a;
  border: 3px solid var(--gold);
  border-radius: 18px;
  overflow: hidden;
  box-shadow: 0 25px 70px rgba(0,0,0,0.95);
}

.bus-header {
  background: #1e293b;
  padding: 14px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 2px solid var(--gold);
}

.bus-line-badge {
  background: var(--gold);
  color: #0f172a;
  font-family: 'Press Start 2P', monospace;
  font-size: 11px;
  padding: 5px 10px;
  border-radius: 6px;
}

.bus-stage {
  position: relative;
  height: 230px;
  background: linear-gradient(180deg, #1e293b 0%, #334155 68%, #1e293b 68%, #0f172a 100%);
  overflow: hidden;
}

/* Przesuwające się budynki Gliwic w tle autobusu */
.bus-city-skyline {
  position: absolute;
  bottom: 72px;
  left: 0;
  width: 200%;
  height: 110px;
  background-image:
    linear-gradient(90deg,
      transparent 0%, transparent 4%,
      #0f172a 4%, #0f172a 11%,
      transparent 11%, transparent 14%,
      #1e293b 14%, #1e293b 22%,
      transparent 22%, transparent 25%,
      #0f172a 25%, #0f172a 35%,
      transparent 35%, transparent 40%,
      #1e293b 40%, #1e293b 48%,
      transparent 48%, transparent 100%);
  background-size: 50% 100%;
  animation: moveSkyline 2.5s linear infinite;
}

@keyframes moveSkyline {
  from { transform: translateX(0); }
  to { transform: translateX(-50%); }
}

.bus-road-stripes {
  position: absolute;
  bottom: 30px;
  left: 0;
  width: 200%;
  height: 6px;
  background: repeating-linear-gradient(90deg, var(--gold) 0px, var(--gold) 40px, transparent 40px, transparent 80px);
  animation: moveRoad 0.45s linear infinite;
}

@keyframes moveRoad {
  from { transform: translateX(0); }
  to { transform: translateX(-80px); }
}

.bus-vehicle-wrap {
  position: absolute;
  bottom: 36px;
  left: 50%;
  transform: translateX(-50%);
  width: 340px;
  height: 110px;
  animation: busBounce 0.35s ease-in-out infinite alternate;
}

@keyframes busBounce {
  0% { transform: translateX(-50%) translateY(0px); }
  100% { transform: translateX(-50%) translateY(-3px); }
}

.bus-footer {
  padding: 16px 20px;
  text-align: center;
  background: #161f30;
}

.bus-route-text {
  font-size: 16px;
  font-weight: 700;
  color: var(--gold-bright);
  margin-bottom: 6px;
}

/* WNĘTRZE LOKALIZACJI (PRZEJŚCIE DO ŚRODKA BUDYNKU + FABUŁA + MINIGRY) */
.interior-window {
  max-width: 960px;
  width: 100%;
  max-height: 94vh;
  background: #111827;
  border: 3px solid var(--gold);
  border-radius: 18px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  box-shadow: 0 25px 80px rgba(0,0,0,0.95);
}

.interior-top-bar {
  background: #1e293b;
  padding: 12px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 2px solid #334155;
}

.interior-top-bar h2 {
  font-family: 'Press Start 2P', monospace;
  font-size: 13px;
  color: var(--gold);
}

/* Scena Wnętrza z unikalną grafiką dla każdego miejsca */
.interior-scene-canvas {
  position: relative;
  height: 250px;
  width: 100%;
  overflow: hidden;
  border-bottom: 3px solid var(--gold);
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  padding: 18px 40px;
}

/* Tła poszczególnych wnętrz */
.scene-bg-zsti {
  background:
    radial-gradient(circle at 50% 20%, rgba(56, 189, 248, 0.25), transparent 70%),
    repeating-linear-gradient(90deg, rgba(30,41,59,0.5) 0px, rgba(30,41,59,0.5) 60px, rgba(15,23,42,0.8) 60px, rgba(15,23,42,0.8) 120px),
    linear-gradient(180deg, #0f172a 0%, #1e293b 70%, #334155 70%, #1e293b 100%);
}
.scene-bg-forum {
  background:
    radial-gradient(circle at 50% 10%, rgba(250, 204, 21, 0.2), transparent 65%),
    repeating-linear-gradient(90deg, #1e293b 0px, #1e293b 90px, #f59e0b 90px, #f59e0b 98px),
    linear-gradient(180deg, #111827 0%, #1f2937 65%, #374151 65%, #111827 100%);
}
.scene-bg-arena {
  background:
    radial-gradient(circle at 50% 0%, rgba(239, 68, 68, 0.35), transparent 65%),
    linear-gradient(180deg, #090d16 0%, #1e1b4b 65%, #7f1d1d 65%, #450a0a 100%);
}
.scene-bg-historic {
  background:
    radial-gradient(circle at 50% 20%, rgba(245, 185, 66, 0.25), transparent 70%),
    linear-gradient(180deg, #1c1917 0%, #292524 68%, #44403c 68%, #1c1917 100%);
}

.interior-prop-banner {
  position: absolute;
  top: 12px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(15, 23, 42, 0.9);
  border: 2px solid var(--gold);
  padding: 5px 14px;
  border-radius: 8px;
  font-family: 'Press Start 2P', monospace;
  font-size: 9px;
  color: var(--gold-bright);
  z-index: 3;
  white-space: nowrap;
}

.character-stage-card {
  position: relative;
  z-index: 4;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.char-name-plate {
  background: rgba(15, 23, 42, 0.95);
  border: 2px solid var(--gold);
  padding: 4px 10px;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 700;
  color: #fff;
  margin-top: 6px;
}

/* Chmurka przylotu NPC (Daniel Kuśka / Damian Stelmach) */
.sky-messenger-cloud {
  position: absolute;
  top: 22px;
  right: 45px;
  z-index: 10;
  display: flex;
  flex-direction: column;
  align-items: center;
  animation: cloudFloat 2s ease-in-out infinite alternate;
}

@keyframes cloudFloat {
  0% { transform: translateY(0px); }
  100% { transform: translateY(-10px); }
}

/* Sekcja Opisu Historycznego & Fabuły */
.interior-body {
  padding: 20px 24px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.rich-location-description {
  background: #1e293b;
  border-left: 4px solid var(--cyan);
  border-radius: 8px;
  padding: 12px 16px;
  font-size: 13px;
  line-height: 1.5;
  color: #e2e8f0;
}

.rich-location-description b {
  color: var(--gold-bright);
}

.dialogue-box {
  background: #0f172a;
  border: 2px solid var(--gold);
  border-radius: 12px;
  padding: 16px;
}

.dialogue-speaker {
  font-family: 'Press Start 2P', monospace;
  font-size: 10px;
  color: var(--gold);
  margin-bottom: 8px;
}

.dialogue-text {
  font-size: 15px;
  line-height: 1.5;
  color: #fff;
}

/* KONTENERY MINIGIER */
.minigame-panel {
  background: #161f30;
  border: 2px solid #475569;
  border-radius: 12px;
  padding: 18px;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

/* MINIGRA 1: ZSTI KOD BINARNY (ŁUKASZ KORNAŚ) */
.binary-terminal {
  background: #050b14;
  border: 2px solid var(--cyan);
  border-radius: 10px;
  padding: 16px;
  text-align: center;
  font-family: 'VT323', monospace;
}

.binary-code-display {
  font-family: 'Press Start 2P', monospace;
  font-size: 24px;
  letter-spacing: 5px;
  color: #38bdf8;
  background: #0f172a;
  padding: 14px;
  border-radius: 8px;
  border: 1px dashed #38bdf8;
  margin: 10px 0;
  user-select: none;
}

.binary-timer-bar-bg {
  width: 100%;
  height: 14px;
  background: #1e293b;
  border-radius: 7px;
  overflow: hidden;
  margin: 8px 0;
}

.binary-timer-bar-fill {
  height: 100%;
  width: 100%;
  background: linear-gradient(90deg, #22c55e, #eab308, #ef4444);
  transition: width 0.1s linear;
}

.binary-input {
  width: 100%;
  max-width: 380px;
  background: #0f172a;
  border: 2px solid var(--gold);
  color: var(--gold-bright);
  font-family: 'Press Start 2P', monospace;
  font-size: 18px;
  text-align: center;
  padding: 12px;
  border-radius: 8px;
  letter-spacing: 4px;
  outline: none;
}

/* MINIGRA 2: FORUM GLIWICE WÓZEK SKLEPOWY */
.cart-canvas-wrap {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

#cartCanvas {
  background: #1f2937;
  border: 3px solid var(--gold);
  border-radius: 10px;
  max-width: 100%;
}

.cart-controls-row {
  display: flex;
  gap: 12px;
}

/* MINIGRA 3: PREZERO ARENA RING BOKSERSKI */
.boxing-arena-ui {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.boxing-bars {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

.hp-meter-box {
  background: #0f172a;
  padding: 10px;
  border-radius: 8px;
  border: 1px solid #334155;
}

.hp-meter-fill {
  height: 16px;
  background: var(--green);
  border-radius: 4px;
  transition: width 0.25s;
}
.hp-meter-fill.enemy { background: var(--red); }

.boxing-actions-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}

.box-move-btn {
  background: #1e293b;
  border: 2px solid var(--gold);
  color: #fff;
  padding: 14px 10px;
  border-radius: 10px;
  font-family: 'Chakra Petch', sans-serif;
  font-weight: 700;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.15s;
}
.box-move-btn:hover:not(:disabled) {
  background: var(--gold);
  color: #0f172a;
}
.box-move-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

/* QUIZ DLA POZOSTAŁYCH LOKALIZACJI */
.quiz-options-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.quiz-option-btn {
  background: #1e293b;
  border: 2px solid #475569;
  color: #f8fafc;
  padding: 12px 14px;
  border-radius: 10px;
  font-family: 'Chakra Petch', sans-serif;
  font-weight: 600;
  font-size: 14px;
  text-align: left;
  cursor: pointer;
  transition: all 0.15s;
}
.quiz-option-btn:hover {
  border-color: var(--gold);
  background: #26334d;
}

/* PRZEWODNIK MIEJSKI / ROZKŁAD JAZDY AUTOBUSÓW */
.guide-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
  max-height: 60vh;
  overflow-y: auto;
  padding-right: 6px;
}

.guide-item {
  background: #1e293b;
  border: 2px solid #334155;
  border-radius: 10px;
  padding: 12px 16px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 14px;
}
.guide-item.active-stop {
  border-color: var(--gold);
  background: #272014;
}
.guide-item.done-stop {
  border-color: var(--green);
}

/* POWIADOMIENIE TOAST */
#toast-banner {
  position: fixed;
  top: 86px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 3000;
  background: #0f172a;
  border: 2px solid var(--gold);
  color: #fff;
  padding: 10px 20px;
  border-radius: 10px;
  font-weight: 700;
  box-shadow: 0 10px 25px rgba(0,0,0,0.8);
}

@media (max-width: 768px) {
  .top-hud { flex-direction: column; align-items: stretch; }
  .quiz-options-grid, .boxing-actions-grid { grid-template-columns: 1fr; }
  .interior-scene-canvas { padding: 12px 16px; height: 210px; }
  .bottom-hud { flex-direction: column; align-items: center; gap: 10px; }
  .dpad-box { display: none; }
}
</style>
</head>
<body>

<div id="game-wrapper">
  <!-- MAPA SATELITARNA GLIWIC -->
  <div id="map"></div>

  <!-- GÓRNY PASEK HUD -->
  <div class="top-hud">
    <div class="hud-box player-card">
      <div class="player-avatar" id="hud-avatar-svg"></div>
      <div class="player-info">
        <div class="player-title">UCZEŃ ZSTI GLIWICE</div>
        <div class="hearts-row" id="hearts-container"></div>
        <div class="player-stats">
          <span>PKT: <b id="score-val">0</b></span>
          <span>ROZDZIAŁ: <b id="chapter-val">1 / 10</b></span>
        </div>
      </div>
    </div>

    <div class="hud-box story-tracker">
      <div class="story-chapter-tag" id="hud-chapter-title">ROZDZIAŁ 1: ZSTI GLIWICE</div>
      <div class="story-objective" id="hud-objective-text">Wejdź do budynku ZSTI i porozmawiaj z Panem Łukaszem Kornasiem</div>
      <div class="story-sub" id="hud-sub-text">Lokalizacja: ul. Chorzowska 5 (Zespół Szkół Techniczno-Informatycznych)</div>
    </div>

    <div class="hud-actions">
      <button class="hud-btn bus-btn" onclick="openBusScheduleModal()">🚌 AUTOBUS ZTM</button>
      <button class="hud-btn" onclick="centerOnCurrentObjective()">🎯 CEL MISJI</button>
      <button class="hud-btn" onclick="toggleAudio()" id="audio-btn">🔊 DŹWIĘK</button>
    </div>
  </div>

  <!-- DOLNY PASEK HUD -->
  <div class="bottom-hud">
    <div class="dpad-box">
      <div class="dpad-label">Ruch Pieszy</div>
      <div class="dpad-grid">
        <button class="dpad-btn dpad-up" data-dir="up">▲</button>
        <button class="dpad-btn dpad-left" data-dir="left">◀</button>
        <div class="dpad-center"></div>
        <button class="dpad-btn dpad-right" data-dir="right">▶</button>
        <button class="dpad-btn dpad-down" data-dir="down">▼</button>
      </div>
    </div>

    <div class="location-dock" id="location-dock">
      <div class="loc-dock-info">
        <h3 id="dock-loc-name">1. ZSTI Gliwice (ul. Chorzowska 5)</h3>
        <p id="dock-loc-desc">Jesteś przed wejściem do szkoły. Wejdź do środka, aby spotkać Pana Łukasza Kornasia!</p>
      </div>
      <div class="loc-action-group">
        <button class="action-primary-btn" id="dock-enter-btn" onclick="handleMainDockAction()">🚪 WEJDŹ DO ŚRODKA</button>
      </div>
    </div>
  </div>

  <div id="toast-banner" class="hidden"></div>

  <!-- MODAL POWITALNY -->
  <div class="overlay-modal" id="start-modal">
    <div class="start-window">
      <div class="start-header">
        <div id="start-hero-preview" style="width:85px;height:105px;flex-shrink:0;"></div>
        <div class="start-title-block">
          <h1>GLIWICE: KRONIKI ZSTI</h1>
          <p>Fabularna Gra Eksploracyjna na Mapie Satelitarnej z Lotu Ptaka</p>
          <p style="color:#cbd5e1;font-size:12px;margin-top:4px;">Przejdź pełną ścieżkę fabularną przez 10 autentycznych miejsc w Gliwicach!</p>
        </div>
      </div>

      <div class="start-features-grid">
        <div class="feature-card">
          <h4>🛰️ 1. Mapa Satelitarna 1:1</h4>
          <p>Prawdziwy widok satelitarny Gliwic z lotu ptaka – widzisz każdy budynek, boisko ZSTI, dach CH Forum oraz kopułę PreZero Areny.</p>
        </div>
        <div class="feature-card">
          <h4>🚌 2. Autobusy ZTM Gliwice</h4>
          <p>Między kolejnymi dzielnicami podróżujesz gliwickimi autobusami komunikacji miejskiej z animowaną scenką przejazdu!</p>
        </div>
        <div class="feature-card">
          <h4>💻 3. ZSTI & Pan Łukasz Kornaś</h4>
          <p>Rozpocznij w ZSTI: przepisz 3 poziomy kodu binarnego w 10 sekund podczas rozmowy z Panem Łukaszem Kornasiem.</p>
        </div>
        <div class="feature-card">
          <h4>🛒 4. Forum, Arena & Goście</h4>
          <p>Omiń auta wózkiem sklepowym w CH Forum (przyleci Pan Daniel Kuśka na chmurce!) i wygraj walkę bokserską w PreZero Arenie (przyleci Pan Damian Stelmach!).</p>
        </div>
      </div>

      <button class="start-btn-big" onclick="startStoryGame()">▶ ROZPOCZNIJ PRZYGODĘ POD ZSTI GLIWICE</button>
    </div>
  </div>

  <!-- SCENKA JAZDY AUTOBUSEM KOMUNIKACJI MIEJSKIEJ -->
  <div class="overlay-modal hidden" id="bus-cutscene-modal">
    <div class="bus-cutscene-window">
      <div class="bus-header">
        <span class="bus-line-badge" id="bus-line-label">ZTM LINIA A4</span>
        <span style="font-size:13px;color:#cbd5e1;font-weight:700;">KOMUNIKACJA MIEJSKA GLIWICE</span>
      </div>
      <div class="bus-stage">
        <div class="bus-city-skyline"></div>
        <div class="bus-road-stripes"></div>
        <div class="bus-vehicle-wrap" id="bus-svg-holder"></div>
      </div>
      <div class="bus-footer">
        <div class="bus-route-text" id="bus-route-title">Trasa: ZSTI Chorzowska ➔ CH Forum Gliwice</div>
        <p style="font-size:13px;color:#94a3b8;" id="bus-route-sub">Autobus mija gliwickie kamienice... Przygotuj się do wyjścia na przystanku!</p>
      </div>
    </div>
  </div>

  <!-- MODAL WNĘTRZA LOKALIZACJI (PRZEJŚCIE DO ŚRODKA) -->
  <div class="overlay-modal hidden" id="interior-modal">
    <div class="interior-window">
      <div class="interior-top-bar">
        <h2 id="int-header-title">WNĘTRZE: ZSTI GLIWICE</h2>
        <button class="hud-btn" style="padding:6px 12px;" onclick="closeInteriorModal()">✖ WYJDŹ NA ZEWNĄTRZ</button>
      </div>

      <!-- Grafika Wnętrza z postaciami -->
      <div class="interior-scene-canvas scene-bg-zsti" id="int-scene-bg">
        <div class="interior-prop-banner" id="int-room-banner">PRACOWNIA SIECIOWA SALA 204 — ZSTI</div>

        <!-- Bohater z lewej -->
        <div class="character-stage-card">
          <div id="int-player-sprite" style="width:92px;height:115px;"></div>
          <div class="char-name-plate">TY (Uczeń ZSTI)</div>
        </div>

        <!-- Chmurka specjalnego gościa (Daniel Kuśka / Damian Stelmach) -->
        <div class="sky-messenger-cloud hidden" id="int-cloud-guest">
          <div id="int-cloud-svg" style="width:150px;height:125px;"></div>
          <div class="char-name-plate" id="int-cloud-name" style="border-color:#38bdf8;color:#38bdf8;">Pan Daniel Kuśka</div>
        </div>

        <!-- Postać NPC / Przeciwnika z prawej -->
        <div class="character-stage-card" id="int-npc-card">
          <div id="int-npc-sprite" style="width:105px;height:125px;"></div>
          <div class="char-name-plate" id="int-npc-name">Pan Łukasz Kornaś</div>
        </div>
      </div>

      <!-- Bogaty opis miejsca + Dialog + Minigra -->
      <div class="interior-body">
        <div class="rich-location-description" id="int-loc-history"></div>

        <div class="dialogue-box">
          <div class="dialogue-speaker" id="int-speaker-name">PAN ŁUKASZ KORNAŚ (ZSTI GLIWICE):</div>
          <div class="dialogue-text" id="int-dialogue-text"></div>
        </div>

        <!-- Dynamiczny kontener na minigrę (Kod Binarny / Wózek w Forum / Ring w Arenie / Quiz Fabularny) -->
        <div class="minigame-panel" id="int-minigame-container"></div>
      </div>
    </div>
  </div>

  <!-- MODAL ROZKŁADU JAZDY AUTOBUSÓW / PRZEWODNIKA PO 10 LOKALIZACJACH -->
  <div class="overlay-modal hidden" id="schedule-modal">
    <div class="start-window" style="max-width:780px;">
      <div style="display:flex;justify-content:space-between;align-items:center;border-bottom:2px solid #334155;padding-bottom:12px;">
        <div>
          <h2 style="font-family:'Press Start 2P',monospace;font-size:14px;color:var(--gold);">🚌 ROZKŁAD JAZDY ZTM GLIWICE & MAPA FABUŁY</h2>
          <p style="font-size:12px;color:#94a3b8;margin-top:4px;">Wybierz autobus, aby przejechać do aktualnego miejsca misji lub przeczytać szczegółowe opisy lokalizacji.</p>
        </div>
        <button class="hud-btn" onclick="closeBusScheduleModal()">✖ ZAMKNIJ</button>
      </div>
      <div class="guide-list" id="bus-stops-list"></div>
    </div>
  </div>

  <!-- MODAL KOŃCA GRY / ZWYCIĘSTWA -->
  <div class="overlay-modal hidden" id="end-modal">
    <div class="start-window" style="max-width:600px;text-align:center;">
      <h1 id="end-title" style="font-family:'Press Start 2P',monospace;font-size:20px;color:var(--gold);">UKOŃCZONO PRZYGODĘ!</h1>
      <p id="end-desc" style="font-size:15px;color:#e2e8f0;line-height:1.5;margin:12px 0;"></p>
      <div style="background:#0f172a;border:2px solid var(--gold);border-radius:12px;padding:16px;margin-bottom:16px;">
        <div style="font-size:14px;color:#94a3b8;">WYNIK KOŃCOWY:</div>
        <div id="end-score" style="font-family:'Press Start 2P',monospace;font-size:24px;color:var(--cyan);margin-top:6px;">0 PKT</div>
      </div>
      <button class="start-btn-big" onclick="restartFullGame()">🔄 ZAGRAJ PONOWNIE OD ZSTI</button>
    </div>
  </div>
</div>

<script>
// ============================================================================
// GRAFIKI WEKTOROWE SVG W STYLU PUNCH CLUB 2 (POSTACIE, AUTOBUS, CHMURKI)
// ============================================================================

function getPlayerSvg() {
  return `<svg viewBox="0 0 80 100" width="100%" height="100%">
    <ellipse cx="40" cy="94" rx="22" ry="5" fill="rgba(0,0,0,0.5)"/>
    <rect x="26" y="64" width="11" height="24" rx="4" fill="#1e3a8a" stroke="#0f172a" stroke-width="2.5"/>
    <rect x="43" y="64" width="11" height="24" rx="4" fill="#1e3a8a" stroke="#0f172a" stroke-width="2.5"/>
    <rect x="23" y="84" width="15" height="8" rx="3" fill="#ef4444" stroke="#0f172a" stroke-width="2"/>
    <rect x="42" y="84" width="15" height="8" rx="3" fill="#ef4444" stroke="#0f172a" stroke-width="2"/>
    <path d="M20 34 L60 34 L56 67 L24 67 Z" fill="#1d4ed8" stroke="#0f172a" stroke-width="2.5"/>
    <path d="M32 34 L48 34 L45 67 L35 67 Z" fill="#f8fafc"/>
    <text x="40" y="53" font-family=" monospace" font-weight="bold" font-size="8" fill="#0f172a" text-anchor="middle">ZSTI</text>
    <rect x="12" y="36" width="10" height="24" rx="5" fill="#f59e0b" stroke="#0f172a" stroke-width="2.5"/>
    <rect x="58" y="36" width="10" height="24" rx="5" fill="#f59e0b" stroke="#0f172a" stroke-width="2.5"/>
    <rect x="27" y="11" width="26" height="23" rx="7" fill="#fed7aa" stroke="#0f172a" stroke-width="2.5"/>
    <path d="M25 16 Q40 4 55 16 L54 11 Q40 3 26 11 Z" fill="#451a03" stroke="#0f172a" stroke-width="2"/>
    <rect x="32" y="19" width="4" height="4" rx="1" fill="#0f172a"/>
    <rect x="44" y="19" width="4" height="4" rx="1" fill="#0f172a"/>
    <path d="M35 28 Q40 31 45 28" stroke="#0f172a" stroke-width="2" fill="none"/>
  </svg>`;
}

function getNpcSvg(type) {
  if (type === 'kornas') {
    // Pan Łukasz Kornaś - nauczyciel przedmiotów zawodowych IT w ZSTI
    return `<svg viewBox="0 0 90 110" width="100%" height="100%">
      <ellipse cx="45" cy="103" rx="26" ry="6" fill="rgba(0,0,0,0.5)"/>
      <rect x="30" y="70" width="12" height="26" rx="4" fill="#1e293b" stroke="#000" stroke-width="2.5"/>
      <rect x="48" y="70" width="12" height="26" rx="4" fill="#1e293b" stroke="#000" stroke-width="2.5"/>
      <path d="M22 36 L68 36 L64 72 L26 72 Z" fill="#0f172a" stroke="#38bdf8" stroke-width="2.5"/>
      <path d="M37 36 L53 36 L50 72 L40 72 Z" fill="#e2e8f0"/>
      <path d="M43 38 L47 38 L48 62 L45 66 L42 62 Z" fill="#0284c7"/>
      <rect x="14" y="38" width="10" height="26" rx="5" fill="#0f172a" stroke="#000" stroke-width="2"/>
      <rect x="66" y="38" width="10" height="26" rx="5" fill="#0f172a" stroke="#000" stroke-width="2"/>
      <!-- Laptop diagnostyczny w dłoni -->
      <rect x="56" y="50" width="24" height="16" rx="2" fill="#38bdf8" stroke="#000" stroke-width="2"/>
      <text x="68" y="61" font-family="monospace" font-size="7" font-weight="bold" fill="#000" text-anchor="middle">0101</text>
      <rect x="30" y="11" width="30" height="25" rx="7" fill="#fdba74" stroke="#000" stroke-width="2.5"/>
      <!-- Okulary eksperta IT -->
      <rect x="33" y="18" width="10" height="7" rx="1" fill="#38bdf8" stroke="#000" stroke-width="2"/>
      <rect x="47" y="18" width="10" height="7" rx="1" fill="#38bdf8" stroke="#000" stroke-width="2"/>
      <line x1="43" y1="21" x2="47" y2="21" stroke="#000" stroke-width="2"/>
      <path d="M38 30 Q45 33 52 30" stroke="#000" stroke-width="2" fill="none"/>
    </svg>`;
  }
  if (type === 'boxer') {
    // Bokser w PreZero Arenie Gliwice
    return `<svg viewBox="0 0 95 115" width="100%" height="100%">
      <ellipse cx="47" cy="108" rx="28" ry="6" fill="rgba(0,0,0,0.55)"/>
      <rect x="28" y="68" width="14" height="28" rx="4" fill="#dc2626" stroke="#000" stroke-width="2.5"/>
      <rect x="52" y="68" width="14" height="28" rx="4" fill="#dc2626" stroke="#000" stroke-width="2.5"/>
      <rect x="26" y="64" width="42" height="12" fill="#fbbf24" stroke="#000" stroke-width="2"/>
      <path d="M18 32 L76 32 L68 66 L26 66 Z" fill="#f97316" stroke="#000" stroke-width="2.5"/>
      <!-- Wielkie rękawice bokserskie -->
      <circle cx="16" cy="52" r="12" fill="#ef4444" stroke="#000" stroke-width="2.5"/>
      <circle cx="78" cy="48" r="12" fill="#ef4444" stroke="#000" stroke-width="2.5"/>
      <rect x="32" y="9" width="30" height="24" rx="7" fill="#fdba74" stroke="#000" stroke-width="2.5"/>
      <rect x="32" y="11" width="30" height="6" fill="#dc2626"/>
      <rect x="37" y="19" width="5" height="4" fill="#000"/>
      <rect x="51" y="19" width="5" height="4" fill="#000"/>
      <rect x="39" y="27" width="16" height="4" fill="#fff" stroke="#000" stroke-width="1.5"/>
    </svg>`;
  }
  // Strażnik / Przewodnik zabytków Gliwic
  return `<svg viewBox="0 0 90 110" width="100%" height="100%">
    <ellipse cx="45" cy="103" rx="25" ry="6" fill="rgba(0,0,0,0.5)"/>
    <rect x="30" y="70" width="12" height="26" rx="4" fill="#334155" stroke="#000" stroke-width="2.5"/>
    <rect x="48" y="70" width="12" height="26" rx="4" fill="#334155" stroke="#000" stroke-width="2.5"/>
    <path d="M22 36 L68 36 L64 72 L26 72 Z" fill="#b45309" stroke="#f5b942" stroke-width="2.5"/>
    <rect x="30" y="11" width="30" height="25" rx="7" fill="#fed7aa" stroke="#000" stroke-width="2.5"/>
    <rect x="35" y="19" width="5" height="5" fill="#000"/>
    <rect x="50" y="19" width="5" height="5" fill="#000"/>
    <path d="M37 29 Q45 33 53 29" stroke="#000" stroke-width="2" fill="none"/>
  </svg>`;
}

// Grafika Pana Daniela Kuśki / Pana Damiana Stelmacha przylatujących na chmurce
function getCloudMessengerSvg(personType) {
  const jacketColor = personType === 'kuska' ? '#0284c7' : '#16a34a';
  const badgeText = personType === 'kuska' ? 'D. KUŚKA' : 'D. STELMACH';
  return `<svg viewBox="0 0 150 120" width="100%" height="100%">
    <!-- Postać na chmurce -->
    <path d="M52 38 L98 38 L94 74 L56 74 Z" fill="${jacketColor}" stroke="#000" stroke-width="2.5"/>
    <rect x="60" y="14" width="30" height="24" rx="7" fill="#fed7aa" stroke="#000" stroke-width="2.5"/>
    <rect x="65" y="22" width="5" height="4" fill="#000"/>
    <rect x="80" y="22" width="5" height="4" fill="#000"/>
    <path d="M68 31 Q75 35 82 31" stroke="#000" stroke-width="2" fill="none"/>
    <!-- Puszysta złoto-błękitna chmurka -->
    <path d="M25 85 Q15 70 35 65 Q45 50 70 58 Q95 48 112 64 Q135 66 128 85 Q132 102 108 102 L38 102 Q18 102 25 85 Z"
          fill="#f8fafc" stroke="#38bdf8" stroke-width="3.5"/>
    <text x="75" y="88" font-family="monospace" font-weight="bold" font-size="11" fill="#0f172a" text-anchor="middle">${badgeText}</text>
  </svg>`;
}

function getBusSvg() {
  return `<svg viewBox="0 0 340 110" width="100%" height="100%">
    <rect x="10" y="18" width="320" height="68" rx="12" fill="#f59e0b" stroke="#0f172a" stroke-width="4"/>
    <rect x="10" y="62" width="320" height="16" fill="#b91c1c"/>
    <!-- Okna autobusu miejskiego -->
    <rect x="26" y="28" width="48" height="26" rx="4" fill="#38bdf8" stroke="#0f172a" stroke-width="2.5"/>
    <rect x="82" y="28" width="48" height="26" rx="4" fill="#38bdf8" stroke="#0f172a" stroke-width="2.5"/>
    <rect x="138" y="28" width="48" height="26" rx="4" fill="#38bdf8" stroke="#0f172a" stroke-width="2.5"/>
    <rect x="194" y="28" width="48" height="26" rx="4" fill="#38bdf8" stroke="#0f172a" stroke-width="2.5"/>
    <rect x="252" y="26" width="66" height="32" rx="5" fill="#7dd3fc" stroke="#0f172a" stroke-width="2.5"/>
    <!-- Wyświetlacz LED ZTM Gliwice -->
    <rect x="110" y="20" width="120" height="7" fill="#0f172a"/>
    <!-- Reflektory -->
    <rect x="322" y="64" width="8" height="12" fill="#fef08a"/>
    <rect x="10" y="64" width="6" height="12" fill="#ef4444"/>
    <!-- Koła -->
    <circle cx="78" cy="86" r="18" fill="#1e293b" stroke="#000" stroke-width="4"/>
    <circle cx="78" cy="86" r="8" fill="#94a3b8"/>
    <circle cx="256" cy="86" r="18" fill="#1e293b" stroke="#000" stroke-width="4"/>
    <circle cx="256" cy="86" r="8" fill="#94a3b8"/>
  </svg>`;
}

// ============================================================================
// 10 SZCZEGÓŁOWO OPISANYCH LOKALIZACJI W GLIWICACH (KOLEJNOŚĆ FABULARNA)
// ============================================================================

const STORY_LOCATIONS = [
  {
    id: 0,
    name: "1. ZSTI Gliwice",
    shortName: "ZSTI Gliwice",
    address: "ul. Chorzowska 5",
    coords: [50.2986, 18.6947],
    busLine: "START PIESZY",
    sceneClass: "scene-bg-zsti",
    roomBanner: "PRACOWNIA SIECIOWA I SERWEROWNIA — ZSTI GLIWICE",
    npcName: "Pan Łukasz Kornaś",
    npcSprite: "kornas",
    type: "binary_kornas",
    richHistory: `<b>Zespół Szkół Techniczno-Informatycznych w Gliwicach (ul. Chorzowska 5)</b> to jedna z najbardziej rozpoznawalnych szkół technicznych w regionie. Zabytkowy, ceglany gmach przy ulicy Chorzowskiej kryje nowoczesne pracownie programowania, sieci komputerowych Cisco, cyberbezpieczeństwa i elektroniki. To tutaj uczniowie przygotowują się do egzaminów zawodowych INF.02 oraz INF.03 i tworzą projekty na gliwickie hackathony.`,
    introDialogue: `„Cześć! Dobrze, że jesteś w pracowni ZSTI. Zanim ruszysz autobusem w miasto na wielką misję po Gliwicach, musimy skonfigurować szkolny ruter brzegowy! Wyświetlę Ci 3 kolejne pakiety kodu binarnego. Masz dokładnie 10 sekund na przepisanie każdego z nich bez błędu. Gotowy?”`,
    winDialogue: `Pan Łukasz Kornaś uśmiecha się i przybija Ci piątkę: „Brawo! Wszystkie 3 pakiety binarne przepisane bezbłędnie – serwer ZSTI działa na 100%! Teraz wsiadaj w autobus linii A4 pod szkołą i jedź do CH Forum Gliwice przy ul. Lipowej. Tam w podziemnym parkingu czeka na Ciebie kolejne zadanie!”`,
    binaryCodes: ["101101", "11001010", "1010110101"]
  },
  {
    id: 1,
    name: "2. CH Forum Gliwice",
    shortName: "Forum Gliwice",
    address: "ul. Lipowa 1",
    coords: [50.2994, 18.6820],
    busLine: "ZTM LINIA A4",
    sceneClass: "scene-bg-forum",
    roomBanner: "PARKING PODZIEMNY POZIOM -1 — CH FORUM GLIWICE",
    npcName: "Kierownik Parkingu Forum",
    npcSprite: "guide",
    type: "forum_cart",
    richHistory: `<b>Centrum Handlowe Forum Gliwice (ul. Lipowa 1)</b> powstało w miejscu dawnej cegielni tuż obok głównego dworca kolejowego. To największy kompleks handlowo-rozrywkowy w centrum miasta i popularne miejsce spotkań uczniów po lekcjach w ZSTI. Rozległy parking podziemny w godzinach szczytu zamienia się w prawdziwy tor przeszkód pełen samochodów i rozpędzonych wózków sklepowych.`,
    introDialogue: `„Witaj w podziemiach CH Forum Gliwice! Ktoś zostawił wyładowany sprzętem wózek sklepowy na samym środku pasa ruchu! Chwyć za rączkę wózka i przez 15 sekund omijaj nadjeżdżające samochody (strzałki LEWO / PRAWO lub przyciski pod ekranem)!”`,
    winDialogue: `Nagle z sufitu parkingu na puszystej chmurce zlatuje Pan Daniel Kuśka! „Świetny refleks za kierownicą tego wózka sklepowego! Mam dla Ciebie ważną wiadomość: wsiadaj teraz w autobus linii M1 i jedź prosto do PreZero Areny Gliwice przy ulicy Akademickiej. Na tamtejszym ringu czeka już potężny Bokser!”`
  },
  {
    id: 2,
    name: "3. PreZero Arena Gliwice",
    shortName: "PreZero Arena",
    address: "ul. Akademicka 50",
    coords: [50.2890, 18.6904],
    busLine: "ZTM LINIA M1",
    sceneClass: "scene-bg-arena",
    roomBanner: "GŁÓWNY RING BOKSERSKI — PREZERO ARENA GLIWICE",
    npcName: "Żelazny Bokser z Areny",
    npcSprite: "boxer",
    type: "arena_boxing",
    richHistory: `<b>PreZero Arena Gliwice (ul. Akademicka 50)</b> to jedna z największych i najnowocześniejszych hal widowiskowo-sportowych w Polsce, mogąca pomieścić ponad 17 000 widzów. Została wybudowana w miejscu dawnego stadionu XX-lecia. W hali odbywały się m.in. Eurowizja Junior, światowe gale sportów walki, koncerty oraz wielkie turnieje e-sportowe.`,
    introDialogue: `„Wchodzisz na oświetlony reflektorami ring PreZero Areny Gliwice! Naprzeciw Ciebie staje Żelazny Bokser. Wyprowadzaj celne ciosy (Lewy Prosty, Prawy Sierpowy) oraz stosuj Gardę i Kontrę, aby znokautować rywala na oczach gliwickiej publiczności!”`,
    winDialogue: `Gdy bokser pada na matę, spod kopuły PreZero Areny przylatuje Pan Damian Stelmach! „Co za walka! Piękny nokaut w samym sercu Areny Gliwice! Teraz czas poznać najważniejsze punkty naszego miasta. Wsiadaj w autobus linii 197 i jedź na zmodernizowany Dworzec PKP Gliwice!”`
  },
  {
    id: 3,
    name: "4. Dworzec PKP Gliwice",
    shortName: "Dworzec PKP",
    address: "ul. Bohaterów Getta Warszawskiego 12",
    coords: [50.3011, 18.6752],
    busLine: "ZTM LINIA 197",
    sceneClass: "scene-bg-historic",
    roomBanner: "HALA GŁÓWNA I CENTRUM PRZESIADKOWE — DWORZEC PKP GLIWICE",
    npcName: "Dyspozytor Kolei Śląskich",
    npcSprite: "guide",
    type: "quiz",
    richHistory: `<b>Dworzec PKP Gliwice oraz Centrum Przesiadkowe</b> to jeden z największych węzłów kolejowych na Górnym Śląsku. Historia kolei w Gliwicach sięga 1845 roku, kiedy dotarła tu Kolej Górnośląska łącząca Wrocław z Mysłowicami. Nowoczesne Centrum Przesiadkowe po północnej i południowej stronie torów łączy pociągi dalekobieżne z autobusami ZTM.`,
    introDialogue: `„Witaj w Hali Głównej Dworca PKP Gliwice! Zanim pojedziesz autobusem na kampus Politechniki Śląskiej, odpowiedz na 3 pytania kontrolne o gliwickim węźle komunikacyjnym.”`,
    winDialogue: `Dyspozytor salutuje: „Znakomicie! Masz pełną wiedzę o gliwickiej kolei. Twój następny przystanek to Wydział AEI Politechniki Śląskiej przy ul. Akademickiej 16!”`,
    questions: [
      { q: "W którym wieku do Gliwic dotarła pierwsza linia kolejowa (Kolej Górnośląska)?", a: ["W połowie XIX wieku (1845 r.)", "Pod koniec XX wieku (1995 r.)", "W średniowieczu (1410 r.)", "W 2015 roku"], c: 0 },
      { q: "Co znajduje się bezpośrednio obok Dworca PKP Gliwice i ułatwia przesiadki na autobusy ZTM?", a: ["Zadaszone Centrum Przesiadkowe", "Port lotniczy", "Stocznia morska", "Skocznia narciarska"], c: 0 },
      { q: "Które centrum handlowe sąsiaduje bezpośrednio z terenami przy Dworcu PKP?", a: ["CH Forum Gliwice", "Złote Tarasy", "Manufaktura", "Stary Browar"], c: 0 }
    ]
  },
  {
    id: 4,
    name: "5. Politechnika Śląska (Wydział AEI)",
    shortName: "Politechnika Śląska",
    address: "ul. Akademicka 16",
    coords: [50.2887, 18.6774],
    busLine: "ZTM LINIA 676",
    sceneClass: "scene-bg-zsti",
    roomBanner: "AULA GŁÓWNA WYDZIAŁU AEI — POLITECHNIKA ŚLĄSKA",
    npcName: "Profesor Cybernetyki PŚ",
    npcSprite: "kornas",
    type: "quiz",
    richHistory: `<b>Politechnika Śląska w Gliwicach (Wydział Automatyki, Elektroniki i Informatyki, ul. Akademicka 16)</b> została założona w 1945 roku jako pierwsza uczelnia techniczna na Górnym Śląsku. Kampus akademicki tworzy prawdziwe miasteczko studenckie wzdłuż ulicy Akademickiej, a wielu absolwentów ZSTI kontynuuje tu studia inżynierskie.`,
    introDialogue: `„Witaj w murach Politechniki Śląskiej! Wielu absolwentów ZSTI trafia właśnie do naszych laboratoriów. Sprawdźmy Twoją wiedzę o gliwickiej uczelni!”`,
    winDialogue: `Profesor uśmiecha się z uznaniem: „Zaliczone na piątkę z plusem! Teraz jedź autobusem linii A4 na historyczny Rynek w Gliwicach, aby zobaczyć Ratusz i Fontannę Neptuna.”`,
    questions: [
      { q: "W którym roku została założona Politechnika Śląska w Gliwicach?", a: ["W 1945 roku", "W 1989 roku", "W 1650 roku", "W 2004 roku"], c: 0 },
      { q: "Jak nazywa się główny deptak kampusu Politechniki Śląskiej w Gliwicach?", a: ["Ulica Akademicka", "Ulica Morska", "Aleja Gwiazd", "Bulwar Nadmorski"], c: 0 },
      { q: "Co oznacza skrót wydziału AEI na Politechnice Śląskiej?", a: ["Automatyki, Elektroniki i Informatyki", "Architektury i Ekologii Miasta", "Astronomii i Energetyki", "Administracji Europejskiej"], c: 0 }
    ]
  },
  {
    id: 5,
    name: "6. Rynek i Ratusz w Gliwicach",
    shortName: "Rynek i Ratusz",
    address: "Rynek 1 (Stare Miasto)",
    coords: [50.2940, 18.6657],
    busLine: "ZTM LINIA A4",
    sceneClass: "scene-bg-historic",
    roomBanner: "SALA RAJCKA RATUSZA MIEJSKIEGO — RYNEK W GLIWICACH",
    npcName: "Kustosz Starego Miasta",
    npcSprite: "guide",
    type: "quiz",
    richHistory: `<b>Rynek w Gliwicach</b> to serce średniowiecznego układu urbanistycznego miasta z XIII wieku. Centralny punkt placu zajmuje Ratusz z wysoką wieżą oraz barokowa <b>Fontanna Neptuna z 1794 roku</b>, wykuta z piaskowca przez rzeźbiarza Jana Nitschego z Opawy. Rynek otaczają malownicze kamieniczki z podcieniami.`,
    introDialogue: `„Witaj na gliwickim Rynku! Stoisz między Ratuszem a słynną Fontanną Neptuna. Odpowiedz na 3 pytania o sercu naszego Starego Miasta!”`,
    winDialogue: `Kustosz kłania się nisko: „Doskonale znasz historię Starego Miasta! Przejdź teraz lub podjedź autobusem do Zamku Piastowskiego przy ul. Pod Murami!”`,
    questions: [
      { q: "Jaka rzymska postać z trójzębem stoi na szczycie fontanny na gliwickim Rynku od 1794 roku?", a: ["Neptun (bóg mórz)", "Mars", "Apollo", "Merkury"], c: 0 },
      { q: "Jaki charakterystyczny element architektoniczny posiadają kamienice wokół gliwickiego Rynku?", a: ["Sklepione podcienia", "Zwodzone mosty", "Szklane piramidy", "Drewniane wiatraki"], c: 0 },
      { q: "Jaki budynek stoi w samym środku gliwickiego Rynku?", a: ["Ratusz Miejski z wieżą", "Latarnia morska", "Hala targowa", "Opera"], c: 0 }
    ]
  },
  {
    id: 6,
    name: "7. Zamek Piastowski w Gliwicach",
    shortName: "Zamek Piastowski",
    address: "ul. Pod Murami 2",
    coords: [50.2928, 18.6638],
    busLine: "ZTM LINIA 126",
    sceneClass: "scene-bg-historic",
    roomBanner: "ZBROJOWNIA KSIĘCIA SIEMOWITA — ZAMEK PIASTOWSKI",
    npcName: "Kasztelan Zamku Piastowskiego",
    npcSprite: "guide",
    type: "quiz",
    richHistory: `<b>Zamek Piastowski w Gliwicach (ul. Pod Murami 2)</b> sięga swoimi początkami połowy XIV wieku, gdy stanowił część miejskich murów obronnych i rezydencję księcia Siemowita bytomsko-gliwickiego. W późniejszych stuleciach pełnił funkcję arsenału i więzienia, a od 1959 roku mieści dział historyczny Muzeum w Gliwicach.`,
    introDialogue: `„Witaj w murach najstarszej budowli obronnej Gliwic! Sprawdźmy, czy znasz tajemnice Zamku Piastowskiego!”`,
    winDialogue: `Kasztelan otwiera bramę: „Znakomicie! Teraz wsiadaj w autobus linii 197 i jedź do Parku Chopina, gdzie wznosi się egzotyczna Palmiarnia Miejska!”`,
    questions: [
      { q: "Z którego wieku pochodzą najstarsze mury obronne Zamku Piastowskiego w Gliwicach?", a: ["Z połowy XIV wieku", "Z końca XIX wieku", "Z XX wieku", "Z X wieku"], c: 0 },
      { q: "Jaka instytucja kultury ma swoją siedzibę w Zamku Piastowskim?", a: ["Muzeum w Gliwicach", "Teatr Muzyczny", "Filharmonia Śląska", "Planetarium"], c: 0 },
      { q: "Który książę piastowski rezydował w gliwickim zamku w XIV wieku?", a: ["Książę Siemowit", "Mieszko I", "Bolesław Chrobry", "Kazimierz Wielki"], c: 0 }
    ]
  },
  {
    id: 7,
    name: "8. Palmiarnia Miejska w Gliwicach",
    shortName: "Palmiarnia Miejska",
    address: "ul. Aleksandra Fredry 6 (Park Chopina)",
    coords: [50.2995, 18.6673],
    busLine: "ZTM LINIA 197",
    sceneClass: "scene-bg-zsti",
    roomBanner: "PAWILON TROPIKALNY I AKWARIA — PALMIARNIA MIEJSKA",
    npcName: "Główny Botanik Palmiarni",
    npcSprite: "guide",
    type: "quiz",
    richHistory: `<b>Palmiarnia Miejska w Gliwicach (Park Chopina)</b> wywodzi się z historycznej wystawy roślin egzotycznych z 1880 roku. Dziś składa się z pięciu ogromnych, przeszklonych pawilonów kryjących ponad 6 tysięcy roślin z całego świata (w tym ponad 100-letnie palmy feniksy kanaryjskie) oraz pawilonu akwarystycznego z czterema gigantycznymi zbiornikami wodnymi.`,
    introDialogue: `„Witaj pod szklanymi kopułami gliwickiej Palmiarni! Wokół nas rosną ponad stuletnie palmy i pływają amazońskie ryby. Odpowiedz na 3 pytania przyrodnicze!”`,
    winDialogue: `Botanik wręcza Ci przepustkę: „Brawo! Teraz jedź autobusem do pięknej Willi Caro przy ulicy Dolnych Wałów!”`,
    questions: [
      { q: "W którym parku w centrum Gliwic znajduje się Palmiarnia Miejska?", a: ["W Parku im. Fryderyka Chopina", "W Parku Chrobrego", "W Lesie Łabędzkim", "W Parku Szwajcaria"], c: 0 },
      { q: "Ile głównych pawilonów tematycznych (wraz z akwarystycznym) tworzy Palmiarnię w Gliwicach?", a: ["5 pawilonów", "1 pawilon", "12 pawilonów", "20 pawilonów"], c: 0 },
      { q: "Z którego roku pochodzą początki gliwickiej ekspozycji egzotycznych roślin?", a: ["Z około 1880 roku", "Z 1999 roku", "Z 2018 roku", "Z 1500 roku"], c: 0 }
    ]
  },
  {
    id: 8,
    name: "9. Willa Caro (Muzeum w Gliwicach)",
    shortName: "Willa Caro",
    address: "ul. Dolnych Wałów 8a",
    coords: [50.2916, 18.6685],
    busLine: "ZTM LINIA 676",
    sceneClass: "scene-bg-historic",
    roomBanner: "SALON RENAISSANCE — WILLA CARO W GLIWICACH",
    npcName: "Konserwator Sztuki Willi Caro",
    npcSprite: "guide",
    type: "quiz",
    richHistory: `<b>Willa Caro w Gliwicach (ul. Dolnych Wałów 8a)</b> to dziewiętnastowieczna rezydencja przemysłowca Oscara Caro z lat 1882–1885, otoczona zabytkowym ogrodem. Budynek zachwyca oryginalnymi drewnianymi stropami kasetonowymi, boazerią, witrażami oraz piecami kaflowymi. Obecnie stanowi główną siedzibę Muzeum w Gliwicach.`,
    introDialogue: `„Witaj w reprezentacyjnym salonie Willi Oscara Caro! Został Ci już tylko przedostatni krok przed finałem pod Radiostacją Gliwicką!”`,
    winDialogue: `Konserwator bije brawo: „Znakomicie! Teraz wsiadaj w autobus linii 80 lub A4 na ulicę Tarnogórską. Czeka Cię wielki finał pod 111-metrową Radiostacją Gliwicką!”`,
    questions: [
      { q: "Dla jakiego gliwickiego przemysłowca wybudowano w XIX wieku tę rezydencję?", a: ["Dla Oscara Caro", "Dla Karola Goduli", "Dla Johna Baildona", "Dla Ignacego Łukasiewicza"], c: 0 },
      { q: "W jakim stylu architektonicznym zaprojektowano wnętrza i fasadę Willi Caro?", a: ["W stylu neorenesansowym (eklektycznym)", "W stylu gotyku ceglanego", "W stylu brutalizmu", "W stylu romańskim"], c: 0 },
      { q: "Przy jakiej ulicy w Gliwicach mieści się Willa Caro?", a: ["Przy ul. Dolnych Wałów 8a", "Przy ul. Tarnogórskiej 129", "Przy ul. Chorzowskiej 5", "Przy ul. Lipowej 1"], c: 0 }
    ]
  },
  {
    id: 9,
    name: "10. Radiostacja Gliwicka (FINAŁ)",
    shortName: "Radiostacja Gliwicka",
    address: "ul. Tarnogórska 129",
    coords: [50.3133, 18.6890],
    busLine: "ZTM LINIA 80",
    sceneClass: "scene-bg-zsti",
    roomBanner: "HALA NADAWCZA POD 111-METROWĄ WIEŻĄ — RADIOSTACJA GLIWICKA",
    npcName: "Strażnik Masztu Radiostacji",
    npcSprite: "kornas",
    type: "quiz",
    richHistory: `<b>Radiostacja Gliwicka (ul. Tarnogórska 129)</b> to unikatowy w skali światowej zabytek inżynierii i Pomnik Historii. Wzniesiony w 1935 roku maszt mierzy <b>111 metrów wysokości</b> i jest najwyższą zachowaną w całości drewnianą konstrukcją na świecie, zbudowaną z impregnowanego modrzewia syberyjskiego połączonego ponad 16 tysiącami mosiężnych śrub (bez ani jednego stalowego gwoździa).`,
    introDialogue: `„Dotarłeś do wielkiego finału przygody! Stoisz u stóp 111-metrowej drewnianej wieży Radiostacji Gliwickiej. Odpowiedz na 3 finałowe pytania, aby nadać sygnał zwycięstwa dla ZSTI na całe Gliwice!”`,
    winDialogue: `Z wierzchołka 111-metrowego masztu rozbłyska sygnał na całe Gliwice! Ukończyłeś wszystkie 10 rozdziałów fabularnych!`,
    questions: [
      { q: "Ile dokładnie metrów wysokości ma drewniana wieża Radiostacji Gliwickiej?", a: ["111 metrów", "55 metrów", "210 metrów", "80 metrów"], c: 0 },
      { q: "Z jakiego gatunku drewna zbudowano maszt Radiostacji Gliwickiej w 1935 roku?", a: ["Z modrzewia syberyjskiego", "Z sosny pospolitej", "Z brzozy", "Z bambusa"], c: 0 },
      { q: "Czym połączono belki masztu, aby metal nie zakłócał pola elektromagnetycznego anteny?", a: ["Ponad 16 000 mosiężnych śrub", "Stalowymi gwoździami", "Sznurem konopnym", "Klejem żywicznym"], c: 0 }
    ]
  }
];

// ============================================================================
// STAN GRY I SILNIK DŹWIĘKOWY WEB AUDIO
// ============================================================================

let map = null;
let heroMarker = null;
let routePolyline = null;
let locationMarkers = [];

let gameState = {
  started: false,
  lives: 3,
  maxLives: 3,
  score: 0,
  currentChapter: 0, // Indeks aktualnej lokalizacji 0..9
  completedIds: [],
  heroPos: [50.2986, 18.6947], // Start pod ZSTI Gliwice
  soundOn: true,
  moveKeys: { up: false, down: false, left: false, right: false }
};

let audioCtx = null;
function playTone(freq, duration = 0.12, type = 'triangle') {
  if (!gameState.soundOn) return;
  try {
    if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();
    osc.type = type;
    osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
    gain.gain.setValueAtTime(0.08, audioCtx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + duration);
    osc.connect(gain);
    gain.connect(audioCtx.destination);
    osc.start();
    osc.stop(audioCtx.currentTime + duration);
  } catch (e) {}
}

function toggleAudio() {
  gameState.soundOn = !gameState.soundOn;
  document.getElementById('audio-btn').textContent = gameState.soundOn ? '🔊 DŹWIĘK' : '🔇 WYCISZONO';
}

function showToast(msg) {
  const el = document.getElementById('toast-banner');
  el.textContent = msg;
  el.classList.remove('hidden');
  clearTimeout(window._toastTimer);
  window._toastTimer = setTimeout(() => el.classList.add('hidden'), 3800);
}

// ============================================================================
// INICJALIZACJA MAPY SATELITARNEJ GLIWIC Z LOTU PTAKA
// ============================================================================

function initSatelliteMap() {
  map = L.map('map', {
    center: [50.2986, 18.6947],
    zoom: 17,
    minZoom: 13,
    maxZoom: 19,
    zoomControl: false
  });

  // Prawdziwa mapa satelitarna z lotu ptaka (Esri World Imagery - widać każdy budynek)
  L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', {
    attribution: 'Mapa satelitarna © Esri — Gliwice z lotu ptaka',
    maxZoom: 19
  }).addTo(map);

  // Nakładka nazw ulic i dzielnic na zdjęcie satelitarne dla pełnej czytelności
  L.tileLayer('https://services.arcgisonline.com/ArcGIS/rest/services/Reference/World_Transportation/MapServer/tile/{z}/{y}/{x}', {
    maxZoom: 19,
    opacity: 0.85
  }).addTo(map);

  // Linia łącząca trasę fabularną autobusów po Gliwicach
  const pathCoords = STORY_LOCATIONS.map(l => l.coords);
  routePolyline = L.polyline(pathCoords, {
    color: '#f5b942',
    weight: 3,
    dashArray: '8, 8',
    opacity: 0.75
  }).addTo(map);

  // Zmniejszona postać gracza na mapie satelitarnej
  const heroIcon = L.divIcon({
    className: '',
    html: `<div class="hero-map-marker" id="hero-map-el">
      <div class="hero-you-badge">TY</div>
      <div class="hero-sprite-small">${getPlayerSvg()}</div>
    </div>`,
    iconSize: [34, 44],
    iconAnchor: [17, 38]
  });

  heroMarker = L.marker(gameState.heroPos, { icon: heroIcon, zIndexOffset: 1000 }).addTo(map);

  renderMapPins();
  updateTopAndBottomHud();
}

function renderMapPins() {
  locationMarkers.forEach(m => map.removeLayer(m));
  locationMarkers = [];

  STORY_LOCATIONS.forEach((loc, idx) => {
    const isCompleted = gameState.completedIds.includes(loc.id);
    const isCurrent = gameState.currentChapter === idx && !isCompleted;

    let stateClass = '';
    let iconSymbol = '🔒';
    if (isCompleted) {
      stateClass = 'completed-quest';
      iconSymbol = '✔';
    } else if (isCurrent) {
      stateClass = 'current-quest';
      iconSymbol = '★';
    }

    const pinIcon = L.divIcon({
      className: '',
      html: `<div class="loc-pin-wrap ${stateClass}">
        <div class="loc-pin-badge">${loc.shortName}</div>
        <div class="loc-pin-circle">${iconSymbol}</div>
      </div>`,
      iconSize: [120, 58],
      iconAnchor: [60, 50]
    });

    const marker = L.marker(loc.coords, { icon: pinIcon }).addTo(map);
    marker.on('click', () => onLocationPinClick(idx));
    locationMarkers.push(marker);
  });
}

function onLocationPinClick(idx) {
  const loc = STORY_LOCATIONS[idx];
  if (idx > gameState.currentChapter) {
    showToast(`🔒 Najpierw ukończ Rozdział ${gameState.currentChapter + 1}: ${STORY_LOCATIONS[gameState.currentChapter].shortName}!`);
    playTone(220, 0.15, 'sawtooth');
    return;
  }

  // Sprawdź odległość gracza od wybranej lokalizacji
  const dist = calcDistanceMeters(gameState.heroPos, loc.coords);
  if (dist > 140) {
    triggerBusCutsceneTo(idx);
  } else {
    openInteriorModal(idx);
  }
}

// ============================================================================
// AKTUALIZACJA INTERFEJSU HUD I DOCKA NA DOLE
// ============================================================================

function updateTopAndBottomHud() {
  // Serduszka (3 życia - odejmowane wyłącznie za przegraną z bossem / minigrą)
  const heartsEl = document.getElementById('hearts-container');
  heartsEl.innerHTML = '';
  for (let i = 0; i < gameState.maxLives; i++) {
    const lost = i >= gameState.lives ? 'lost' : '';
    heartsEl.innerHTML += `<svg class="heart-svg ${lost}" viewBox="0 0 24 24">
      <path fill="#ef4444" stroke="#000" stroke-width="2"
        d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/>
    </svg>`;
  }

  document.getElementById('score-val').textContent = gameState.score;
  const chapNum = Math.min(gameState.currentChapter + 1, STORY_LOCATIONS.length);
  document.getElementById('chapter-val').textContent = `${chapNum} / ${STORY_LOCATIONS.length}`;

  const activeLoc = STORY_LOCATIONS[Math.min(gameState.currentChapter, STORY_LOCATIONS.length - 1)];
  document.getElementById('hud-chapter-title').textContent = `AKTUALNA MISJA — ${activeLoc.name.toUpperCase()}`;

  const dist = calcDistanceMeters(gameState.heroPos, activeLoc.coords);
  const dockTitle = document.getElementById('dock-loc-name');
  const dockDesc = document.getElementById('dock-loc-desc');
  const dockBtn = document.getElementById('dock-enter-btn');

  dockTitle.textContent = `${activeLoc.name} (${activeLoc.address})`;

  if (dist <= 140) {
    document.getElementById('hud-objective-text').textContent = `Jesteś na miejscu! Wejdź do środka: ${activeLoc.shortName}`;
    document.getElementById('hud-sub-text').textContent = `Kliknij zielony przycisk „WEJDŹ DO ŚRODKA” na dole ekranu`;
    dockDesc.textContent = activeLoc.richHistory.replace(/<[^>]*>?/gm, '').slice(0, 145) + '...';
    dockBtn.textContent = `🚪 WEJDŹ DO ŚRODKA`;
    dockBtn.classList.remove('bus-mode');
  } else {
    document.getElementById('hud-objective-text').textContent = `Pojedź autobusem ${activeLoc.busLine} do: ${activeLoc.shortName}`;
    document.getElementById('hud-sub-text').textContent = `Odległość w linii prostej: ${Math.round(dist)} m | Kliknij „JEDŹ AUTOBUSEM”`;
    dockDesc.textContent = `Następny punkt fabuły znajduje się w innej części Gliwic (${activeLoc.address}). Wsiądź w autobus komunikacji miejskiej!`;
    dockBtn.textContent = `🚌 JEDŹ AUTOBUSEM (${activeLoc.busLine})`;
    dockBtn.classList.add('bus-mode');
  }
}

function handleMainDockAction() {
  const idx = Math.min(gameState.currentChapter, STORY_LOCATIONS.length - 1);
  const loc = STORY_LOCATIONS[idx];
  const dist = calcDistanceMeters(gameState.heroPos, loc.coords);

  if (dist <= 140) {
    openInteriorModal(idx);
  } else {
    triggerBusCutsceneTo(idx);
  }
}

function centerOnCurrentObjective() {
  const loc = STORY_LOCATIONS[Math.min(gameState.currentChapter, STORY_LOCATIONS.length - 1)];
  map.flyTo(loc.coords, 17, { duration: 0.9 });
}

// ============================================================================
// KOMUNIKACJA MIEJSKA ZTM GLIWICE — SCENKA JAZDY AUTOBUSEM
// ============================================================================

function triggerBusCutsceneTo(targetIdx) {
  const targetLoc = STORY_LOCATIONS[targetIdx];
  const modal = document.getElementById('bus-cutscene-modal');
  document.getElementById('bus-line-label').textContent = targetLoc.busLine;
  document.getElementById('bus-route-title').textContent = `Kierunek: ${targetLoc.name} (${targetLoc.address})`;
  document.getElementById('bus-route-sub').textContent = `Autobus ZTM Gliwice jedzie ulicami miasta... Za chwilę wysiadasz pod samym budynkiem!`;
  document.getElementById('bus-svg-holder').innerHTML = getBusSvg();

  modal.classList.remove('hidden');
  playTone(320, 0.2, 'sawtooth');
  setTimeout(() => playTone(440, 0.25, 'triangle'), 300);

  setTimeout(() => {
    modal.classList.add('hidden');
    gameState.heroPos = [...targetLoc.coords];
    heroMarker.setLatLng(gameState.heroPos);
    map.setView(gameState.heroPos, 18, { animate: true });
    updateTopAndBottomHud();
    showToast(`🚌 Dojechałeś autobusem pod: ${targetLoc.shortName}!`);
    // Od razu po przyjeździe autobusem możesz wejść lub otwieramy podgląd miejsca
    setTimeout(() => openInteriorModal(targetIdx), 450);
  }, 2600);
}

function openBusScheduleModal() {
  const listEl = document.getElementById('bus-stops-list');
  listEl.innerHTML = '';

  STORY_LOCATIONS.forEach((loc, idx) => {
    const isDone = gameState.completedIds.includes(loc.id);
    const isCurrent = idx === gameState.currentChapter;
    const isLocked = idx > gameState.currentChapter;

    const item = document.createElement('div');
    item.className = `guide-item ${isCurrent ? 'active-stop' : ''} ${isDone ? 'done-stop' : ''}`;

    let btnHtml = '';
    if (isLocked) {
      btnHtml = `<span style="font-size:12px;color:#94a3b8;">🔒 Zablokowane</span>`;
    } else {
      btnHtml = `<button class="hud-btn bus-btn" onclick="closeBusScheduleModal(); triggerBusCutsceneTo(${idx});">🚌 Jedź (${loc.busLine})</button>`;
    }

    item.innerHTML = `
      <div style="flex:1;">
        <div style="font-weight:700;font-size:15px;color:${isDone ? '#4ade80' : '#f5b942'};">
          ${loc.name} — <span style="font-size:12px;color:#cbd5e1;">${loc.address}</span>
        </div>
        <div style="font-size:12px;color:#cbd5e1;margin-top:4px;line-height:1.4;">${loc.richHistory}</div>
      </div>
      <div>${btnHtml}</div>
    `;
    listEl.appendChild(item);
  });

  document.getElementById('schedule-modal').classList.remove('hidden');
}

function closeBusScheduleModal() {
  document.getElementById('schedule-modal').classList.add('hidden');
}

// ============================================================================
// WEJŚCIE DO ŚRODKA LOKALIZACJI (WNĘTRZA + FABUŁA + MINIGRY)
// ============================================================================

let activeMinigameCleanup = null;

function openInteriorModal(idx) {
  if (activeMinigameCleanup) {
    activeMinigameCleanup();
    activeMinigameCleanup = null;
  }

  const loc = STORY_LOCATIONS[idx];
  const modal = document.getElementById('interior-modal');
  const sceneBg = document.getElementById('int-scene-bg');

  sceneBg.className = `interior-scene-canvas ${loc.sceneClass}`;
  document.getElementById('int-header-title').textContent = `WNĘTRZE: ${loc.name.toUpperCase()}`;
  document.getElementById('int-room-banner').textContent = loc.roomBanner;
  document.getElementById('int-player-sprite').innerHTML = getPlayerSvg();
  document.getElementById('int-npc-sprite').innerHTML = getNpcSvg(loc.npcSprite);
  document.getElementById('int-npc-name').textContent = loc.npcName;
  document.getElementById('int-npc-card').classList.remove('hidden');
  document.getElementById('int-cloud-guest').classList.add('hidden');

  document.getElementById('int-loc-history').innerHTML = loc.richHistory;
  document.getElementById('int-speaker-name').textContent = `${loc.npcName.toUpperCase()}:`;
  document.getElementById('int-dialogue-text').textContent = loc.introDialogue;

  const container = document.getElementById('int-minigame-container');
  container.innerHTML = '';

  if (gameState.completedIds.includes(loc.id)) {
    renderAlreadyCompletedView(loc, container);
  } else if (loc.type === 'binary_kornas') {
    setupBinaryMinigame(loc, container);
  } else if (loc.type === 'forum_cart') {
    setupForumCartMinigame(loc, container);
  } else if (loc.type === 'arena_boxing') {
    setupArenaBoxingMinigame(loc, container);
  } else {
    setupLocationQuizMinigame(loc, container);
  }

  modal.classList.remove('hidden');
  playTone(520, 0.15, 'triangle');
}

function closeInteriorModal() {
  if (activeMinigameCleanup) {
    activeMinigameCleanup();
    activeMinigameCleanup = null;
  }
  document.getElementById('interior-modal').classList.add('hidden');
}

function renderAlreadyCompletedView(loc, container) {
  const nextIdx = loc.id + 1;
  const hasNext = nextIdx < STORY_LOCATIONS.length;
  container.innerHTML = `
    <div style="text-align:center;padding:12px;">
      <div style="color:#4ade80;font-weight:700;font-size:16px;margin-bottom:10px;">✔ To zadanie fabularne zostało już pomyślnie ukończone!</div>
      ${hasNext ? `<button class="start-btn-big" onclick="closeInteriorModal(); triggerBusCutsceneTo(${nextIdx});">🚌 WSIĄDŹ W AUTOBUS DO: ${STORY_LOCATIONS[nextIdx].shortName.toUpperCase()}</button>` : ''}
    </div>
  `;
}

// Funkcja odejmowania życia wyłącznie po przegranej w zadaniu / walce z bossem
function loseOneHeart(reasonText, retryCallback) {
  gameState.lives -= 1;
  playTone(150, 0.35, 'sawtooth');
  updateTopAndBottomHud();

  if (gameState.lives <= 0) {
    closeInteriorModal();
    showEndGameModal(false);
    return;
  }

  const container = document.getElementById('int-minigame-container');
  container.innerHTML = `
    <div style="text-align:center;padding:14px;background:#2a1215;border:2px solid #ef4444;border-radius:10px;">
      <h3 style="color:#f87171;margin-bottom:6px;">💔 UTRACONO 1 SERDUSZKO! (Pozostało: ${gameState.lives}/3)</h3>
      <p style="font-size:14px;color:#fecaca;margin-bottom:14px;">${reasonText}</p>
      <button class="action-primary-btn" onclick="retryActiveMinigame()">🔄 SPRÓBUJ PONOWNIE</button>
    </div>
  `;
  window.retryActiveMinigame = retryCallback;
}

// Funkcja ukończenia rozdziału i wyświetlenia fabularnej zapowiedzi kolejnego miejsca
function completeChapterSuccess(loc, specialCloudPerson = null) {
  if (!gameState.completedIds.includes(loc.id)) {
    gameState.completedIds.push(loc.id);
    gameState.score += 500;
    if (gameState.currentChapter === loc.id && gameState.currentChapter < STORY_LOCATIONS.length - 1) {
      gameState.currentChapter += 1;
    }
  }

  playTone(587, 0.12);
  setTimeout(() => playTone(784, 0.25), 130);

  renderMapPins();
  updateTopAndBottomHud();

  // Jeśli przylatuje Pan Daniel Kuśka na chmurce lub Pan Damian Stelmach
  if (specialCloudPerson) {
    const cloudEl = document.getElementById('int-cloud-guest');
    const cloudSvg = document.getElementById('int-cloud-svg');
    const cloudName = document.getElementById('int-cloud-name');
    cloudSvg.innerHTML = getCloudMessengerSvg(specialCloudPerson);
    cloudName.textContent = specialCloudPerson === 'kuska' ? 'Pan Daniel Kuśka (na chmurce)' : 'Pan Damian Stelmach';
    cloudEl.classList.remove('hidden');
  }

  document.getElementById('int-dialogue-text').textContent = loc.winDialogue;

  const container = document.getElementById('int-minigame-container');
  const nextIdx = loc.id + 1;

  if (nextIdx >= STORY_LOCATIONS.length) {
    container.innerHTML = `
      <div style="text-align:center;padding:14px;">
        <h3 style="color:var(--gold-bright);margin-bottom:10px;">🏆 WSZYSTKIE 10 MIEJSC W GLIWICACH UKOŃCZONE!</h3>
        <button class="start-btn-big" onclick="closeInteriorModal(); showEndGameModal(true);">🎉 ZOBACZ PODSUMOWANIE PRZYGODY</button>
      </div>
    `;
  } else {
    const nextLoc = STORY_LOCATIONS[nextIdx];
    container.innerHTML = `
      <div style="text-align:center;padding:12px;">
        <div style="color:#4ade80;font-weight:700;font-size:15px;margin-bottom:12px;">
          +500 PKT! Odblokowano kolejny przystanek fabuły: ${nextLoc.name} (${nextLoc.address})
        </div>
        <button class="start-btn-big" onclick="closeInteriorModal(); triggerBusCutsceneTo(${nextIdx});">
          🚌 WSIĄDŹ W AUTOBUS (${nextLoc.busLine}) ➔ ${nextLoc.shortName.toUpperCase()}
        </button>
      </div>
    `;
  }
}

// ============================================================================
// MINIGRA 1: ZSTI GLIWICE — 3 POZIOMY KODU BINARNEGO W 10 SEKUND (ŁUKASZ KORNAŚ)
// ============================================================================

function setupBinaryMinigame(loc, container) {
  let level = 0;
  let timeLeft = 10.0;
  let timerId = null;

  function stopTimer() {
    if (timerId) {
      clearInterval(timerId);
      timerId = null;
    }
  }
  activeMinigameCleanup = stopTimer;

  function renderBinaryLevel() {
    stopTimer();
    timeLeft = 10.0;
    const targetCode = loc.binaryCodes[level];

    container.innerHTML = `
      <div class="binary-terminal">
        <div style="display:flex;justify-content:space-between;align-items:center;font-family:'Chakra Petch',sans-serif;font-size:14px;color:#cbd5e1;">
          <span>POZIOM KODU BINARNEGO: <b style="color:var(--gold);">${level + 1} / 3</b></span>
          <span>CZAS: <b id="bin-timer-txt" style="color:#38bdf8;">10.0 s</b></span>
        </div>
        <div class="binary-timer-bar-bg">
          <div class="binary-timer-bar-fill" id="bin-timer-bar"></div>
        </div>
        <div style="font-size:14px;color:#94a3b8;margin-top:4px;">Przepisz dokładnie poniższy kod binarny dla Pana Łukasza Kornasia:</div>
        <div class="binary-code-display">${targetCode}</div>
        <input type="text" id="bin-user-input" class="binary-input" placeholder="Wpisz 0 i 1..." maxlength="14" autocomplete="off" />
        <div style="margin-top:12px;display:flex;justify-content:center;gap:10px;">
          <button class="action-primary-btn" id="bin-submit-btn">✔ ZATWIERDŹ KOD</button>
        </div>
      </div>
    `;

    const inputEl = document.getElementById('bin-user-input');
    const timerTxt = document.getElementById('bin-timer-txt');
    const timerBar = document.getElementById('bin-timer-bar');
    const submitBtn = document.getElementById('bin-submit-btn');

    setTimeout(() => inputEl.focus(), 60);

    function verifyInput() {
      const val = inputEl.value.trim();
      if (val === targetCode) {
        stopTimer();
        playTone(660, 0.14);
        level += 1;
        if (level >= 3) {
          completeChapterSuccess(loc);
        } else {
          document.getElementById('int-dialogue-text').textContent =
            `Pan Łukasz Kornaś: „Świetnie! Poziom ${level}/3 zaliczony. Teraz kolejny, dłuższy ciąg binarny – znów masz 10 sekund!”`;
          renderBinaryLevel();
        }
      } else {
        stopTimer();
        loseOneHeart(
          `Wpisano błędny kod binarny (${val || 'brak'}) zamiast ${targetCode}! Pan Łukasz Kornaś prosi o ponowną konfigurację.`,
          () => setupBinaryMinigame(loc, container)
        );
      }
    }

    submitBtn.onclick = verifyInput;
    inputEl.onkeydown = (e) => {
      if (e.key === 'Enter') verifyInput();
    };
    inputEl.oninput = () => {
      if (inputEl.value.trim() === targetCode) {
        verifyInput();
      }
    };

    timerId = setInterval(() => {
      timeLeft -= 0.1;
      if (timeLeft <= 0) {
        timeLeft = 0;
        stopTimer();
        loseOneHeart(
          `Upłynęło 10 sekund na przepisanie kodu binarnego w pracowni ZSTI!`,
          () => setupBinaryMinigame(loc, container)
        );
        return;
      }
      timerTxt.textContent = `${timeLeft.toFixed(1)} s`;
      timerBar.style.width = `${(timeLeft / 10) * 100}%`;
    }, 100);
  }

  renderBinaryLevel();
}

// ============================================================================
// MINIGRA 2: CH FORUM GLIWICE — PARKING PODZIEMNY I WÓZEK SKLEPOWY + DANIEL KUŚKA
// ============================================================================

function setupForumCartMinigame(loc, container) {
  container.innerHTML = `
    <div class="cart-canvas-wrap">
      <div style="display:flex;justify-content:space-between;width:100%;max-width:480px;font-size:14px;font-weight:700;">
        <span>🛒 OMIJAJ NADJEŻDŻAJĄCE AUTA!</span>
        <span style="color:var(--gold);">CZAS DO METY: <b id="cart-time-left">14.0 s</b></span>
      </div>
      <canvas id="cartCanvas" width="480" height="250"></canvas>
      <div class="cart-controls-row">
        <button class="hud-btn" id="cart-left-btn">◀ WÓZEK W LEWO (A / ◀)</button>
        <button class="hud-btn" id="cart-right-btn">WÓZEK W PRAWO (D / ▶) ▶</button>
      </div>
    </div>
  `;

  const canvas = document.getElementById('cartCanvas');
  const ctx = canvas.getContext('2d');
  const timeEl = document.getElementById('cart-time-left');

  const lanes = [90, 240, 390];
  let playerLane = 1;
  let timeRemaining = 14.0;
  let cars = [];
  let roadOffset = 0;
  let running = true;
  let animId = null;

  function moveLane(dir) {
    if (!running) return;
    playerLane = Math.max(0, Math.min(2, playerLane + dir));
    playTone(380, 0.06);
  }

  document.getElementById('cart-left-btn').onclick = () => moveLane(-1);
  document.getElementById('cart-right-btn').onclick = () => moveLane(1);

  function keyHandler(e) {
    if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') moveLane(-1);
    if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') moveLane(1);
  }
  window.addEventListener('keydown', keyHandler);

  let spawnTimer = setInterval(() => {
    if (!running) return;
    const laneIdx = Math.floor(Math.random() * 3);
    const colors = ['#ef4444', '#38bdf8', '#f59e0b', '#a855f7'];
    cars.push({
      lane: laneIdx,
      y: -60,
      speed: 3.6 + Math.random() * 1.4,
      color: colors[Math.floor(Math.random() * colors.length)]
    });
  }, 850);

  let clockTimer = setInterval(() => {
    if (!running) return;
    timeRemaining -= 0.1;
    if (timeRemaining <= 0) {
      cleanupCart();
      document.getElementById('int-speaker-name').textContent = "PAN DANIEL KUŚKA (PRZYLATUJE NA CHMURCE!):";
      completeChapterSuccess(loc, 'kuska');
      return;
    }
    timeEl.textContent = `${timeRemaining.toFixed(1)} s`;
  }, 100);

  function cleanupCart() {
    running = false;
    clearInterval(spawnTimer);
    clearInterval(clockTimer);
    cancelAnimationFrame(animId);
    window.removeEventListener('keydown', keyHandler);
  }
  activeMinigameCleanup = cleanupCart;

  function loop() {
    if (!running) return;
    ctx.fillStyle = '#1f2937';
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // Pasy ruchu parkingu podziemnego Forum Gliwice
    roadOffset = (roadOffset + 5) % 40;
    ctx.strokeStyle = '#f59e0b';
    ctx.lineWidth = 3;
    ctx.setLineDash([20, 20]);
    ctx.lineDashOffset = -roadOffset;
    [165, 315].forEach(x => {
      ctx.beginPath();
      ctx.moveTo(x, 0);
      ctx.lineTo(x, canvas.height);
      ctx.stroke();
    });
    ctx.setLineDash([]);

    // Rysuj wózek sklepowy gracza
    const px = lanes[playerLane];
    const py = 185;
    ctx.fillStyle = '#e2e8f0';
    ctx.strokeStyle = '#0f172a';
    ctx.lineWidth = 2;
    ctx.fillRect(px - 22, py, 44, 48);
    ctx.strokeRect(px - 22, py, 44, 48);
    // Kratka wózka i rączka
    ctx.fillStyle = '#ef4444';
    ctx.fillRect(px - 24, py + 44, 48, 6);
    ctx.fillStyle = '#38bdf8';
    ctx.font = 'bold 10px monospace';
    ctx.textAlign = 'center';
    ctx.fillText('WÓZEK', px, py + 28);

    // Rysuj nadjeżdżające samochody
    for (let i = cars.length - 1; i >= 0; i--) {
      const c = cars[i];
      c.y += c.speed;
      const cx = lanes[c.lane];

      ctx.fillStyle = c.color;
      ctx.fillRect(cx - 24, c.y, 48, 56);
      ctx.fillStyle = '#fef08a';
      ctx.fillRect(cx - 20, c.y + 48, 10, 6);
      ctx.fillRect(cx + 10, c.y + 48, 10, 6);
      ctx.fillStyle = '#0f172a';
      ctx.fillRect(cx - 18, c.y + 12, 36, 16);

      // Kolizja z wózkiem sklepowym
      if (c.lane === playerLane && c.y + 52 > py && c.y < py + 44) {
        cleanupCart();
        loseOneHeart(
          `Rozpędzony samochód na parkingu podziemnym CH Forum uderzył w Twój wózek sklepowy!`,
          () => setupForumCartMinigame(loc, container)
        );
        return;
      }

      if (c.y > canvas.height + 60) {
        cars.splice(i, 1);
      }
    }

    animId = requestAnimationFrame(loop);
  }

  loop();
}

// ============================================================================
// MINIGRA 3: PREZERO ARENA GLIWICE — WALKA BOKSERSKA + DAMIAN STELMACH
// ============================================================================

function setupArenaBoxingMinigame(loc, container) {
  let playerHp = 100;
  let boxerHp = 100;

  function renderRing() {
    container.innerHTML = `
      <div class="boxing-arena-ui">
        <div class="boxing-bars">
          <div class="hp-meter-box">
            <div style="display:flex;justify-content:space-between;font-size:13px;font-weight:700;margin-bottom:4px;">
              <span>🥊 TY (Uczeń ZSTI)</span>
              <span>${playerHp} HP</span>
            </div>
            <div class="hp-meter-fill" style="width:${playerHp}%;"></div>
          </div>
          <div class="hp-meter-box">
            <div style="display:flex;justify-content:space-between;font-size:13px;font-weight:700;margin-bottom:4px;">
              <span>🥊 ŻELAZNY BOKSER ARENY</span>
              <span>${boxerHp} HP</span>
            </div>
            <div class="hp-meter-fill enemy" style="width:${boxerHp}%;"></div>
          </div>
        </div>

        <div id="boxing-log" style="background:#0f172a;border:1px solid #334155;padding:10px;border-radius:8px;text-align:center;font-weight:700;color:var(--gold-bright);">
          Wybierz akcję bokserską na ringu PreZero Areny Gliwice!
        </div>

        <div class="boxing-actions-grid">
          <button class="box-move-btn" onclick="executeBoxingMove('jab')">🥊 LEWY PROSTY<br><span style="font-size:11px;color:#94a3b8;">Szybki i pewny (-25 HP)</span></button>
          <button class="box-move-btn" onclick="executeBoxingMove('hook')">💥 PRAWY SIERPOWY<br><span style="font-size:11px;color:#94a3b8;">Potężny cios (-40 HP)</span></button>
          <button class="box-move-btn" onclick="executeBoxingMove('guard')">🛡️ GARDA I KONTRA<br><span style="font-size:11px;color:#94a3b8;">Blok + Kontratak (-20 HP)</span></button>
        </div>
      </div>
    `;
  }

  window.executeBoxingMove = function(move) {
    const logEl = document.getElementById('boxing-log');
    let msg = '';

    if (move === 'jab') {
      boxerHp = Math.max(0, boxerHp - 25);
      playerHp = Math.max(0, playerHp - 15);
      msg = 'Wyprowadzasz błyskawiczny lewy prosty (-25 HP)! Bokser odpowiada lekkim ciosem (-15 HP).';
      playTone(480, 0.1);
    } else if (move === 'hook') {
      const hit = Math.random() < 0.78;
      if (hit) {
        boxerHp = Math.max(0, boxerHp - 40);
        playerHp = Math.max(0, playerHp - 20);
        msg = 'POTĘŻNY PRAWY SIERPOWY trafia czysto w szczękę (-40 HP)!';
        playTone(620, 0.14);
      } else {
        playerHp = Math.max(0, playerHp - 35);
        msg = 'Bokser robi unik przed Twoim sierpowym i wyprowadza mocną kontrę (-35 HP)!';
        playTone(200, 0.18, 'sawtooth');
      }
    } else {
      boxerHp = Math.max(0, boxerHp - 20);
      playerHp = Math.min(100, playerHp + 5);
      msg = 'Szczelna garda blokuje atak boksera, a Twoja kontra zadaje -20 HP (+5 HP oddechu)!';
      playTone(540, 0.12);
    }

    if (boxerHp <= 0) {
      document.getElementById('int-speaker-name').textContent = "PAN DAMIAN STELMACH (PRZYLATUJE NA RING!):";
      completeChapterSuccess(loc, 'stelmach');
      return;
    }

    if (playerHp <= 0) {
      loseOneHeart(
        `Żelazny Bokser znokautował Cię na ringu PreZero Areny Gliwice!`,
        () => setupArenaBoxingMinigame(loc, container)
      );
      return;
    }

    renderRing();
    document.getElementById('boxing-log').textContent = msg;
  };

  renderRing();
}

// ============================================================================
// MINIGRA 4..10: WYZWANIA FABULARNE W POZOSTAŁYCH LOKALIZACJACH GLIWIC
// ============================================================================

function setupLocationQuizMinigame(loc, container) {
  let qIdx = 0;

  function renderQuestion() {
    const qObj = loc.questions[qIdx];
    const buttonsHtml = qObj.a.map((ans, i) =>
      `<button class="quiz-option-btn" onclick="answerLocQuestion(${i})">${ans}</button>`
    ).join('');

    container.innerHTML = `
      <div style="display:flex;justify-content:space-between;font-size:13px;color:var(--gold);font-weight:700;">
        <span>WYZWANIE W LOKALIZACJI: ${loc.shortName.toUpperCase()}</span>
        <span>PYTANIE ${qIdx + 1} / ${loc.questions.length}</span>
      </div>
      <div style="font-size:16px;font-weight:700;color:#fff;margin:4px 0 8px 0;">${qObj.q}</div>
      <div class="quiz-options-grid">${buttonsHtml}</div>
    `;
  }

  window.answerLocQuestion = function(choice) {
    const qObj = loc.questions[qIdx];
    if (choice === qObj.c) {
      playTone(640, 0.12);
      qIdx += 1;
      if (qIdx >= loc.questions.length) {
        completeChapterSuccess(loc);
      } else {
        renderQuestion();
      }
    } else {
      loseOneHeart(
        `Błędna odpowiedź podczas misji w: ${loc.name}!`,
        () => setupLocationQuizMinigame(loc, container)
      );
    }
  };

  renderQuestion();
}

// ============================================================================
// RUCH PIESZY PO MAPIE SATELITARNEJ + OBSŁUGA KLAWIATURY / D-PADA
// ============================================================================

function calcDistanceMeters(c1, c2) {
  const R = 6371000;
  const dLat = (c2[0] - c1[0]) * Math.PI / 180;
  const dLon = (c2[1] - c1[1]) * Math.PI / 180;
  const a = Math.sin(dLat / 2) ** 2 +
            Math.cos(c1[0] * Math.PI / 180) * Math.cos(c2[0] * Math.PI / 180) *
            Math.sin(dLon / 2) ** 2;
  return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
}

function setupControls() {
  const keyMap = {
    ArrowUp: 'up', w: 'up', W: 'up',
    ArrowDown: 'down', s: 'down', S: 'down',
    ArrowLeft: 'left', a: 'left', A: 'left',
    ArrowRight: 'right', d: 'right', D: 'right'
  };

  window.addEventListener('keydown', (e) => {
    if (keyMap[e.key]) {
      gameState.moveKeys[keyMap[e.key]] = true;
    }
  });

  window.addEventListener('keyup', (e) => {
    if (keyMap[e.key]) {
      gameState.moveKeys[keyMap[e.key]] = false;
    }
  });

  document.querySelectorAll('.dpad-btn').forEach(btn => {
    const dir = btn.dataset.dir;
    const startMove = (e) => { e.preventDefault(); gameState.moveKeys[dir] = true; btn.classList.add('active'); };
    const stopMove = (e) => { e.preventDefault(); gameState.moveKeys[dir] = false; btn.classList.remove('active'); };
    btn.addEventListener('mousedown', startMove);
    btn.addEventListener('mouseup', stopMove);
    btn.addEventListener('mouseleave', stopMove);
    btn.addEventListener('touchstart', startMove, { passive: false });
    btn.addEventListener('touchend', stopMove, { passive: false });
  });

  // Pętla spokojnego ruchu pieszego wokół bieżącej lokalizacji
  const walkSpeed = 0.000028;
  setInterval(() => {
    if (!gameState.started) return;
    let dLat = 0, dLng = 0;
    if (gameState.moveKeys.up) dLat += walkSpeed;
    if (gameState.moveKeys.down) dLat -= walkSpeed;
    if (gameState.moveKeys.left) dLng -= walkSpeed * 1.4;
    if (gameState.moveKeys.right) dLng += walkSpeed * 1.4;

    const heroEl = document.getElementById('hero-map-el');
    if (dLat !== 0 || dLng !== 0) {
      gameState.heroPos[0] += dLat;
      gameState.heroPos[1] += dLng;
      heroMarker.setLatLng(gameState.heroPos);
      if (heroEl) heroEl.classList.add('hero-walking');
      updateTopAndBottomHud();
    } else if (heroEl) {
      heroEl.classList.remove('hero-walking');
    }
  }, 35);
}

// ============================================================================
// START GRY I PODSUMOWANIE
// ============================================================================

function startStoryGame() {
  document.getElementById('start-modal').classList.add('hidden');
  gameState.started = true;
  playTone(440, 0.12);
  setTimeout(() => playTone(660, 0.18), 120);
  // Od razu na starcie pod ZSTI otwieramy możliwość wejścia lub podpowiedź
  showToast("📍 Jesteś przed ZSTI Gliwice (ul. Chorzowska 5)! Kliknij „WEJDŹ DO ŚRODKA”.");
}

function showEndGameModal(victory) {
  const modal = document.getElementById('end-modal');
  const title = document.getElementById('end-title');
  const desc = document.getElementById('end-desc');
  const scoreEl = document.getElementById('end-score');

  if (victory) {
    title.textContent = "🏆 MISTRZ GLIWIC I DUMA ZSTI!";
    title.style.color = "var(--gold)";
    desc.textContent = "Gratulacje! Przepisałeś kody binarne u Pana Łukasza Kornasia w ZSTI, ominąłeś samochody wózkiem sklepowym w CH Forum (z pomocą Pana Daniela Kuśki na chmurce), znokautowałeś boksera w PreZero Arenie (gdzie przyleciał Pan Damian Stelmach) i odkryłeś wszystkie 10 najważniejszych miejsc Gliwic!";
  } else {
    title.textContent = "💔 KONIEC GRY — UTRACONO 3 SERDUSZKA";
    title.style.color = "#ef4444";
    desc.textContent = "Straciłeś wszystkie 3 serduszka podczas wyzwań w Gliwicach. Spróbuj ponownie i ukończ całą fabułę od ZSTI aż po Radiostację Gliwicką!";
  }

  scoreEl.textContent = `${gameState.score} PKT`;
  modal.classList.remove('hidden');
}

function restartFullGame() {
  gameState.lives = 3;
  gameState.score = 0;
  gameState.currentChapter = 0;
  gameState.completedIds = [];
  gameState.heroPos = [...STORY_LOCATIONS[0].coords];

  document.getElementById('end-modal').classList.add('hidden');
  heroMarker.setLatLng(gameState.heroPos);
  map.setView(gameState.heroPos, 17);
  renderMapPins();
  updateTopAndBottomHud();
}

window.addEventListener('DOMContentLoaded', () => {
  document.getElementById('hud-avatar-svg').innerHTML = getPlayerSvg();
  document.getElementById('start-hero-preview').innerHTML = getPlayerSvg();
  initSatelliteMap();
  setupControls();
});
</script>
</body>
</html>
