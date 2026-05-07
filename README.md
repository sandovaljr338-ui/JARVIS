<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>J.A.R.V.I.S. — Sistema IA Funcional</title>
<link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;700;900&family=Rajdhani:wght@300;400;500;600&family=Share+Tech+Mono&display=swap" rel="stylesheet">
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --blue: #00c8ff;
    --blue-dim: rgba(0,200,255,0.15);
    --gold: #f0a500;
    --red: #ff3c3c;
    --purple: #a855f7;
    --green: #00ff88;
    --bg: #020a12;
    --bg2: #050f1a;
    --bg3: #071624;
    --text: #c8e8f8;
    --text-dim: #4a7a9b;
    --border: rgba(0,200,255,0.2);
    --border-bright: rgba(0,200,255,0.55);
  }

  html { scroll-behavior: smooth; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'Rajdhani', sans-serif;
    overflow-x: hidden;
    cursor: crosshair;
    min-height: 100vh;
  }

  /* SCANLINES */
  body::before {
    content: '';
    position: fixed; inset: 0;
    background: repeating-linear-gradient(
      0deg, transparent, transparent 2px,
      rgba(0,0,0,0.07) 2px, rgba(0,0,0,0.07) 4px
    );
    pointer-events: none;
    z-index: 9999;
  }

  /* GRID BG */
  .grid-bg {
    position: fixed; inset: 0;
    background-image:
      linear-gradient(rgba(0,200,255,0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,200,255,0.03) 1px, transparent 1px);
    background-size: 50px 50px;
    pointer-events: none; z-index: 0;
  }

  /* NAV */
  nav {
    position: fixed; top: 0; left: 0; right: 0;
    z-index: 1000;
    display: flex; align-items: center; justify-content: space-between;
    padding: 1rem 2.5rem;
    background: rgba(2,10,18,0.95);
    border-bottom: 1px solid var(--border);
    backdrop-filter: blur(12px);
  }

  .nav-logo {
    font-family: 'Orbitron', monospace;
    font-size: 1rem; font-weight: 900;
    letter-spacing: 0.3em; color: var(--blue);
    text-shadow: 0 0 20px var(--blue);
  }

  .nav-tabs {
    display: flex; gap: 0.25rem; list-style: none;
  }

  .nav-tabs button {
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.65rem; letter-spacing: 0.15em;
    color: var(--text-dim); background: transparent;
    border: 1px solid transparent;
    padding: 0.4rem 1rem; cursor: pointer;
    text-transform: uppercase; transition: all 0.2s;
  }

  .nav-tabs button.active,
  .nav-tabs button:hover {
    color: var(--blue); border-color: var(--border);
    background: var(--blue-dim);
  }

  .nav-status {
    display: flex; align-items: center; gap: 0.5rem;
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.65rem; color: var(--green);
  }

  .dot {
    width: 6px; height: 6px; border-radius: 50%;
    background: currentColor;
    box-shadow: 0 0 8px currentColor;
    animation: blink 2s ease-in-out infinite;
  }

  @keyframes blink {
    0%,100%{opacity:1} 50%{opacity:0.3}
  }

  /* LAYOUT */
  .app {
    display: grid;
    grid-template-columns: 280px 1fr;
    grid-template-rows: auto 1fr;
    min-height: 100vh;
    padding-top: 60px;
    position: relative; z-index: 1;
  }

  /* SIDEBAR */
  .sidebar {
    grid-row: 1 / -1;
    background: var(--bg2);
    border-right: 1px solid var(--border);
    display: flex; flex-direction: column;
    overflow: hidden;
  }

  .sidebar-header {
    padding: 1.5rem;
    border-bottom: 1px solid var(--border);
    text-align: center;
  }

  /* ARC REACTOR */
  .arc-reactor {
    width: 70px; height: 70px;
    margin: 0 auto 1rem;
    position: relative;
  }

  .arc-outer {
    position: absolute; inset: 0;
    border-radius: 50%;
    border: 2px solid var(--blue);
    box-shadow: 0 0 15px var(--blue), inset 0 0 15px rgba(0,200,255,0.08);
    animation: spin 5s linear infinite;
  }

  .arc-mid {
    position: absolute; inset: 12px;
    border-radius: 50%;
    border: 1px dashed rgba(0,200,255,0.4);
    animation: spin 3s linear infinite reverse;
  }

  .arc-inner {
    position: absolute; inset: 22px;
    border-radius: 50%;
    background: radial-gradient(circle, rgba(0,200,255,0.5), rgba(0,200,255,0.1) 70%, transparent);
    border: 1px solid rgba(0,200,255,0.5);
    animation: pulse-glow 2s ease-in-out infinite;
  }

  @keyframes spin { from{transform:rotate(0)} to{transform:rotate(360deg)} }
  @keyframes pulse-glow {
    0%,100%{box-shadow:0 0 8px var(--blue)}
    50%{box-shadow:0 0 20px var(--blue),0 0 40px rgba(0,200,255,0.3)}
  }

  .sidebar-title {
    font-family: 'Orbitron', monospace;
    font-size: 1.3rem; font-weight: 900;
    letter-spacing: 0.2em; color: #fff;
    text-shadow: 0 0 20px rgba(0,200,255,0.4);
  }

  .sidebar-ver {
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.6rem; letter-spacing: 0.2em;
    color: var(--text-dim); margin-top: 0.25rem;
  }

  /* MODE SELECTOR */
  .mode-list {
    padding: 1rem; display: flex; flex-direction: column; gap: 0.4rem;
    border-bottom: 1px solid var(--border);
  }

  .mode-label {
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.55rem; letter-spacing: 0.3em;
    color: var(--text-dim); text-transform: uppercase;
    padding: 0 0.25rem; margin-bottom: 0.5rem;
  }

  .mode-btn {
    display: flex; align-items: center; gap: 0.75rem;
    padding: 0.6rem 0.75rem;
    background: transparent; border: 1px solid transparent;
    cursor: pointer; color: var(--text-dim);
    font-family: 'Rajdhani', sans-serif;
    font-size: 0.85rem; font-weight: 500;
    letter-spacing: 0.05em; text-align: left;
    transition: all 0.2s; width: 100%;
  }

  .mode-btn:hover { border-color: var(--border); color: var(--text); background: var(--blue-dim); }
  .mode-btn.active { border-color: var(--border-bright); color: var(--blue); background: var(--blue-dim); }

  .mode-dot {
    width: 8px; height: 8px; border-radius: 50%;
    flex-shrink: 0; transition: all 0.2s;
  }

  .mode-btn.active .mode-dot {
    box-shadow: 0 0 8px currentColor;
    background: var(--blue);
  }

  /* SYSTEM STATUS */
  .sys-status {
    padding: 1rem; flex: 1;
  }

  .sys-row {
    display: flex; justify-content: space-between; align-items: center;
    padding: 0.4rem 0;
    border-bottom: 1px solid rgba(0,200,255,0.07);
    font-family: 'Share Tech Mono', monospace; font-size: 0.65rem;
  }

  .sys-name { color: var(--text-dim); letter-spacing: 0.1em; }
  .sys-val { color: var(--green); }
  .sys-val.warn { color: var(--gold); }

  /* CHAT AREA */
  .main {
    display: flex; flex-direction: column;
    height: calc(100vh - 60px);
    overflow: hidden;
  }

  /* TOP BAR */
  .topbar {
    display: flex; align-items: center; justify-content: space-between;
    padding: 0.75rem 1.5rem;
    border-bottom: 1px solid var(--border);
    background: rgba(5,15,26,0.8);
  }

  .topbar-mode {
    font-family: 'Orbitron', monospace;
    font-size: 0.7rem; font-weight: 700;
    letter-spacing: 0.2em; color: var(--blue);
  }

  .topbar-meta {
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.6rem; color: var(--text-dim);
    letter-spacing: 0.1em;
  }

  /* MESSAGES */
  .messages {
    flex: 1; overflow-y: auto;
    padding: 1.5rem;
    display: flex; flex-direction: column; gap: 1rem;
    scrollbar-width: thin; scrollbar-color: var(--border) transparent;
  }

  .messages::-webkit-scrollbar { width: 4px; }
  .messages::-webkit-scrollbar-track { background: transparent; }
  .messages::-webkit-scrollbar-thumb { background: var(--border); border-radius: 2px; }

  /* MESSAGE BUBBLES */
  .msg {
    display: flex; gap: 0.75rem;
    animation: msg-in 0.3s ease-out;
  }

  @keyframes msg-in {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
  }

  .msg.user { flex-direction: row-reverse; }

  .msg-avatar {
    width: 32px; height: 32px; border-radius: 50%;
    flex-shrink: 0;
    display: flex; align-items: center; justify-content: center;
    font-family: 'Orbitron', monospace; font-size: 0.55rem; font-weight: 700;
    border: 1px solid var(--border);
    letter-spacing: 0.05em;
  }

  .msg.jarvis .msg-avatar { background: rgba(0,200,255,0.1); color: var(--blue); border-color: var(--blue); }
  .msg.friday .msg-avatar { background: rgba(168,85,247,0.1); color: var(--purple); border-color: var(--purple); }
  .msg.edith .msg-avatar  { background: rgba(239,68,68,0.1); color: var(--red); border-color: var(--red); }
  .msg.user .msg-avatar   { background: rgba(240,165,0,0.1); color: var(--gold); border-color: var(--gold); }

  .msg-body { max-width: 72%; }

  .msg-header {
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.6rem; letter-spacing: 0.15em;
    color: var(--text-dim); margin-bottom: 0.3rem;
  }

  .msg.user .msg-header { text-align: right; }

  .msg-bubble {
    padding: 0.85rem 1.1rem;
    border: 1px solid var(--border);
    background: var(--bg3);
    font-size: 0.95rem; line-height: 1.7; font-weight: 400;
    position: relative;
    clip-path: polygon(0 0, calc(100% - 10px) 0, 100% 10px, 100% 100%, 10px 100%, 0 calc(100% - 10px));
  }

  .msg.user .msg-bubble {
    background: rgba(240,165,0,0.07);
    border-color: rgba(240,165,0,0.25);
    text-align: right;
    clip-path: polygon(10px 0, 100% 0, 100% calc(100% - 10px), calc(100% - 10px) 100%, 0 100%, 0 10px);
  }

  .msg.jarvis .msg-bubble { border-color: rgba(0,200,255,0.3); }
  .msg.friday .msg-bubble { border-color: rgba(168,85,247,0.3); background: rgba(168,85,247,0.05); }
  .msg.edith .msg-bubble  { border-color: rgba(239,68,68,0.3); background: rgba(239,68,68,0.05); }

  /* CODE BLOCKS */
  .code-block {
    background: #000;
    border: 1px solid rgba(0,200,255,0.25);
    border-radius: 0;
    margin: 0.75rem 0;
    overflow: hidden;
  }

  .code-header {
    display: flex; justify-content: space-between; align-items: center;
    padding: 0.4rem 0.75rem;
    background: var(--bg3);
    border-bottom: 1px solid rgba(0,200,255,0.15);
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.6rem; letter-spacing: 0.15em; color: var(--text-dim);
  }

  .copy-btn {
    background: transparent; border: 1px solid var(--border);
    color: var(--text-dim); font-family: 'Share Tech Mono', monospace;
    font-size: 0.55rem; letter-spacing: 0.1em; padding: 0.15rem 0.5rem;
    cursor: pointer; transition: all 0.2s;
  }

  .copy-btn:hover { border-color: var(--blue); color: var(--blue); }

  pre {
    padding: 1rem;
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.75rem; line-height: 1.7;
    color: #4af7a0; overflow-x: auto;
    white-space: pre-wrap; word-break: break-word;
  }

  /* TYPING INDICATOR */
  .typing-indicator {
    display: flex; gap: 4px; align-items: center;
    padding: 0.6rem 0;
  }

  .typing-indicator span {
    width: 6px; height: 6px; border-radius: 50%;
    background: var(--blue); opacity: 0.4;
    animation: typing 1.2s ease-in-out infinite;
  }

  .typing-indicator span:nth-child(2) { animation-delay: 0.2s; }
  .typing-indicator span:nth-child(3) { animation-delay: 0.4s; }

  @keyframes typing {
    0%,80%,100%{opacity:0.4;transform:scale(1)}
    40%{opacity:1;transform:scale(1.3)}
  }

  /* INPUT AREA */
  .input-area {
    padding: 1rem 1.5rem;
    border-top: 1px solid var(--border);
    background: var(--bg2);
  }

  .input-row {
    display: flex; gap: 0.75rem; align-items: flex-end;
  }

  .input-wrap {
    flex: 1; position: relative;
  }

  .input-wrap textarea {
    width: 100%;
    background: var(--bg3);
    border: 1px solid var(--border);
    color: var(--text);
    font-family: 'Rajdhani', sans-serif;
    font-size: 0.95rem; font-weight: 400;
    padding: 0.75rem 1rem;
    resize: none; outline: none;
    transition: border-color 0.2s;
    min-height: 48px; max-height: 140px;
    line-height: 1.5;
  }

  .input-wrap textarea:focus { border-color: var(--border-bright); }
  .input-wrap textarea::placeholder { color: var(--text-dim); }

  .send-btn {
    background: transparent;
    border: 1px solid var(--blue);
    color: var(--blue);
    font-family: 'Orbitron', monospace;
    font-size: 0.6rem; font-weight: 700;
    letter-spacing: 0.15em;
    padding: 0 1.25rem;
    height: 48px; cursor: pointer;
    clip-path: polygon(8px 0,100% 0,calc(100% - 8px) 100%,0 100%);
    transition: all 0.2s; white-space: nowrap;
    position: relative; overflow: hidden;
  }

  .send-btn::before {
    content: ''; position: absolute; inset: 0;
    background: rgba(0,200,255,0.12);
    transform: scaleX(0); transform-origin: left;
    transition: transform 0.2s;
  }

  .send-btn:hover::before { transform: scaleX(1); }
  .send-btn:hover { box-shadow: 0 0 15px rgba(0,200,255,0.25); color: #fff; }
  .send-btn:disabled { opacity: 0.4; cursor: not-allowed; }

  .input-hints {
    display: flex; gap: 0.5rem; flex-wrap: wrap; margin-top: 0.5rem;
  }

  .hint-chip {
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.6rem; letter-spacing: 0.1em;
    color: var(--text-dim); border: 1px solid var(--border);
    padding: 0.2rem 0.6rem; cursor: pointer;
    transition: all 0.2s; background: transparent;
  }

  .hint-chip:hover { border-color: var(--border-bright); color: var(--blue); }

  /* WELCOME MSG */
  .welcome {
    display: flex; flex-direction: column; align-items: center;
    justify-content: center; flex: 1;
    text-align: center; padding: 3rem; color: var(--text-dim);
    gap: 1rem;
  }

  .welcome-icon {
    font-size: 3rem; color: var(--blue);
    text-shadow: 0 0 30px var(--blue);
    font-family: 'Orbitron', monospace;
    animation: pulse-glow 2s ease-in-out infinite;
  }

  .welcome h2 {
    font-family: 'Orbitron', monospace; font-size: 1rem;
    font-weight: 700; letter-spacing: 0.15em; color: #fff;
  }

  .welcome p {
    font-size: 0.9rem; max-width: 400px; line-height: 1.6;
  }

  /* API KEY MODAL */
  .modal-overlay {
    position: fixed; inset: 0;
    background: rgba(0,0,0,0.85);
    z-index: 2000;
    display: flex; align-items: center; justify-content: center;
    backdrop-filter: blur(4px);
  }

  .modal {
    background: var(--bg2);
    border: 1px solid var(--border-bright);
    padding: 2.5rem; width: 460px; max-width: 90vw;
    position: relative;
    clip-path: polygon(0 0, calc(100% - 20px) 0, 100% 20px, 100% 100%, 20px 100%, 0 calc(100% - 20px));
  }

  .modal-title {
    font-family: 'Orbitron', monospace;
    font-size: 0.9rem; font-weight: 700;
    letter-spacing: 0.15em; color: var(--blue);
    margin-bottom: 0.5rem;
  }

  .modal-sub {
    font-size: 0.85rem; color: var(--text-dim); line-height: 1.5;
    margin-bottom: 1.5rem;
  }

  .modal input {
    width: 100%; background: var(--bg3);
    border: 1px solid var(--border); color: var(--text);
    font-family: 'Share Tech Mono', monospace; font-size: 0.8rem;
    padding: 0.75rem 1rem; outline: none; margin-bottom: 0.5rem;
    transition: border-color 0.2s;
  }

  .modal input:focus { border-color: var(--border-bright); }

  .modal-note {
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.6rem; color: var(--text-dim);
    letter-spacing: 0.1em; margin-bottom: 1.5rem;
  }

  .modal-btns { display: flex; gap: 0.75rem; }

  .modal-btns button {
    flex: 1; font-family: 'Orbitron', monospace;
    font-size: 0.65rem; font-weight: 700;
    letter-spacing: 0.15em; padding: 0.75rem;
    cursor: pointer; transition: all 0.2s;
  }

  .btn-confirm {
    background: transparent; border: 1px solid var(--blue);
    color: var(--blue);
  }
  .btn-confirm:hover { background: rgba(0,200,255,0.1); box-shadow: 0 0 15px rgba(0,200,255,0.2); }

  .btn-cancel {
    background: transparent; border: 1px solid var(--border);
    color: var(--text-dim);
  }
  .btn-cancel:hover { border-color: var(--border-bright); color: var(--text); }

  /* SCROLLBAR */
  * { scrollbar-width: thin; scrollbar-color: var(--border) transparent; }

  /* TTS PANEL */
  .tts-panel {
    padding: 1rem; border-top: 1px solid var(--border);
    display: flex; flex-direction: column; gap: 0.6rem;
  }
  .voice-viz {
    position: relative; background: #000;
    border: 1px solid var(--border); height: 56px;
    overflow: hidden; display: flex; align-items: center; justify-content: center;
    margin-bottom: 0.25rem;
  }
  .voice-viz canvas { position: absolute; inset: 0; width: 100%; height: 100%; }
  .viz-label {
    font-family: 'Share Tech Mono', monospace; font-size: 0.55rem;
    letter-spacing: 0.3em; color: var(--text-dim); position: relative; z-index: 1;
    transition: opacity 0.3s;
  }
  .voice-viz.speaking .viz-label { opacity: 0; }
  .tts-row { display: flex; align-items: center; gap: 0.5rem; }
  .tts-lbl {
    font-family: 'Share Tech Mono', monospace; font-size: 0.55rem;
    letter-spacing: 0.12em; color: var(--text-dim); white-space: nowrap; min-width: 60px;
  }
  .toggle-btn {
    font-family: 'Orbitron', monospace; font-size: 0.55rem; font-weight: 700;
    letter-spacing: 0.15em; padding: 0.25rem 0.75rem;
    background: transparent; cursor: pointer; border: 1px solid var(--border);
    color: var(--text-dim); transition: all 0.2s;
  }
  .toggle-btn.on {
    border-color: var(--blue); color: var(--blue);
    background: rgba(0,200,255,0.1); box-shadow: 0 0 8px rgba(0,200,255,0.2);
  }
  #voiceSelect {
    flex: 1; background: var(--bg3); border: 1px solid var(--border);
    color: var(--text); font-family: 'Share Tech Mono', monospace;
    font-size: 0.6rem; padding: 0.25rem 0.4rem; outline: none; cursor: pointer;
  }
  #voiceSelect option { background: var(--bg2); }
  input[type=range] {
    -webkit-appearance: none; height: 2px;
    background: var(--border); border-radius: 1px; outline: none;
  }
  input[type=range]::-webkit-slider-thumb {
    -webkit-appearance: none; width: 12px; height: 12px;
    border-radius: 50%; background: var(--blue);
    box-shadow: 0 0 6px var(--blue); cursor: pointer;
  }
  .tts-actions { display: flex; gap: 0.5rem; }
  .tts-btn {
    flex: 1; font-family: 'Share Tech Mono', monospace; font-size: 0.6rem;
    letter-spacing: 0.1em; padding: 0.4rem 0.5rem;
    background: transparent; border: 1px solid var(--border);
    color: var(--text-dim); cursor: pointer; transition: all 0.2s;
  }
  .tts-btn:hover { border-color: var(--border-bright); color: var(--text); }
  .mic-btn.listening {
    border-color: var(--red); color: var(--red);
    background: rgba(255,60,60,0.1); box-shadow: 0 0 10px rgba(255,60,60,0.2);
    animation: mic-pulse 1s ease-in-out infinite;
  }
  @keyframes mic-pulse {
    0%,100%{box-shadow:0 0 10px rgba(255,60,60,0.2)} 50%{box-shadow:0 0 20px rgba(255,60,60,0.4)}
  }
  .speak-btn {
    display: inline-flex; align-items: center; gap: 0.3rem;
    font-family: 'Share Tech Mono', monospace; font-size: 0.55rem;
    letter-spacing: 0.1em; color: var(--text-dim);
    background: transparent; border: 1px solid var(--border);
    padding: 0.15rem 0.5rem; cursor: pointer; margin-top: 0.5rem; transition: all 0.2s;
  }
  .speak-btn:hover { border-color: var(--blue); color: var(--blue); }
  .mic-overlay {
    display: none; position: fixed; bottom: 120px; left: 50%; transform: translateX(-50%);
    background: rgba(2,10,18,0.95); border: 1px solid var(--red);
    padding: 0.75rem 2rem; z-index: 500;
    font-family: 'Share Tech Mono', monospace; font-size: 0.7rem;
    letter-spacing: 0.2em; color: var(--red); box-shadow: 0 0 20px rgba(255,60,60,0.2);
    clip-path: polygon(8px 0,100% 0,calc(100% - 8px) 100%,0 100%);
  }
  .mic-overlay.active { display: block; }
</style>
</head>
<body>

<div class="grid-bg"></div>

<!-- MIC OVERLAY -->
<div class="mic-overlay" id="micOverlay">🎤 ESCUCHANDO... HABLA AHORA</div>

<!-- API KEY MODAL -->
<div class="modal-overlay" id="apiModal">
  <div class="modal">
    <p class="modal-title">⚡ AUTENTICACIÓN REQUERIDA</p>
    <p class="modal-sub">
      Para activar el núcleo cognitivo de J.A.R.V.I.S., necesito tu API Key de Anthropic.
      Se guarda solo en esta sesión, nunca se envía a servidores externos.
    </p>
    <input type="password" id="apiKeyInput" placeholder="sk-ant-api03-..." autocomplete="off" />
    <p class="modal-note">
      ▸ Obtén tu key en: console.anthropic.com &nbsp;|&nbsp; Solo se guarda en memoria local
    </p>
    <div class="modal-btns">
      <button class="btn-confirm" onclick="saveApiKey()">ACTIVAR JARVIS</button>
      <button class="btn-cancel" onclick="closeModal()">DEMO (sin IA)</button>
    </div>
  </div>
</div>

<!-- NAV -->
<nav>
  <div class="nav-logo">J.A.R.V.I.S.</div>
  <ul class="nav-tabs" id="navTabs">
    <li><button class="active" onclick="switchMode('jarvis',this)">JARVIS</button></li>
    <li><button onclick="switchMode('friday',this)">FRIDAY</button></li>
    <li><button onclick="switchMode('edith',this)">EDITH</button></li>
  </ul>
  <div class="nav-status">
    <div class="dot"></div>
    <span id="statusText">ONLINE</span>
  </div>
</nav>

<!-- APP -->
<div class="app">

  <!-- SIDEBAR -->
  <aside class="sidebar">
    <div class="sidebar-header">
      <div class="arc-reactor">
        <div class="arc-outer"></div>
        <div class="arc-mid"></div>
        <div class="arc-inner"></div>
      </div>
      <p class="sidebar-title" id="sidebarName">JARVIS</p>
      <p class="sidebar-ver" id="sidebarVer">CORE SYSTEM v4.2.1 — ONLINE</p>
    </div>

    <div class="mode-list">
      <p class="mode-label">// Modo Activo</p>
      <button class="mode-btn active" onclick="setPersonality('default',this)">
        <span class="mode-dot" style="background:var(--blue)"></span>
        Asistente Completo
      </button>
      <button class="mode-btn" onclick="setPersonality('sarcastic',this)">
        <span class="mode-dot" style="background:var(--gold)"></span>
        Sarcástico Máximo
      </button>
      <button class="mode-btn" onclick="setPersonality('tactical',this)">
        <span class="mode-dot" style="background:var(--red)"></span>
        Modo Táctico
      </button>
      <button class="mode-btn" onclick="setPersonality('coder',this)">
        <span class="mode-dot" style="background:var(--green)"></span>
        Modo Código
      </button>
    </div>

    <div class="sys-status">
      <div class="sys-row"><span class="sys-name">NÚCLEO</span><span class="sys-val" id="coreStatus">ACTIVO</span></div>
      <div class="sys-row"><span class="sys-name">FRIDAY</span><span class="sys-val">STANDBY</span></div>
      <div class="sys-row"><span class="sys-name">EDITH</span><span class="sys-val">STANDBY</span></div>
      <div class="sys-row"><span class="sys-name">API</span><span class="sys-val warn" id="apiStatus">PENDIENTE</span></div>
      <div class="sys-row"><span class="sys-name">VOZ</span><span class="sys-val warn" id="ttsStatus">CARGANDO</span></div>
      <div class="sys-row"><span class="sys-name">MICRO</span><span class="sys-val warn" id="micStatus">INACTIVO</span></div>
      <div class="sys-row"><span class="sys-name">MENSAJES</span><span class="sys-val" id="msgCount">0</span></div>
      <div class="sys-row"><span class="sys-name">TOKENS</span><span class="sys-val" id="tokenCount">0</span></div>
      <div class="sys-row"><span class="sys-name">MODO</span><span class="sys-val" id="modeDisplay">DEFAULT</span></div>
    </div>

    <!-- TTS CONTROLS -->
    <div class="tts-panel">
      <p class="mode-label">// Síntesis de Voz</p>

      <!-- VISUALIZER -->
      <div class="voice-viz" id="voiceViz">
        <canvas id="waveCanvas" width="240" height="50"></canvas>
        <div class="viz-label" id="vizLabel">STANDBY</div>
      </div>

      <!-- TTS TOGGLE -->
      <div class="tts-row">
        <span class="tts-lbl">AUTO-VOZ</span>
        <button class="toggle-btn" id="ttsToggle" onclick="toggleTTS()">OFF</button>
      </div>

      <!-- VOICE SELECT -->
      <div class="tts-row">
        <span class="tts-lbl">VOZ</span>
        <select id="voiceSelect" onchange="setVoice(this.value)"></select>
      </div>

      <!-- SPEED -->
      <div class="tts-row">
        <span class="tts-lbl">VELOCIDAD</span>
        <input type="range" id="rateSlider" min="0.5" max="2" step="0.1" value="0.95" oninput="updateRate(this.value)" style="flex:1;margin:0 0.5rem"/>
        <span class="tts-lbl" id="rateVal">0.95</span>
      </div>

      <!-- PITCH -->
      <div class="tts-row">
        <span class="tts-lbl">TONO</span>
        <input type="range" id="pitchSlider" min="0.5" max="2" step="0.1" value="0.85" oninput="updatePitch(this.value)" style="flex:1;margin:0 0.5rem"/>
        <span class="tts-lbl" id="pitchVal">0.85</span>
      </div>

      <!-- ACTION BUTTONS -->
      <div class="tts-actions">
        <button class="tts-btn" onclick="stopSpeaking()">⏹ STOP</button>
        <button class="tts-btn mic-btn" id="micBtn" onclick="toggleMic()">🎤 MIC</button>
      </div>
    </div>
  </aside>

  <!-- MAIN CHAT -->
  <main class="main">

    <div class="topbar">
      <span class="topbar-mode" id="topbarMode">J.A.R.V.I.S. — NÚCLEO COGNITIVO ACTIVO</span>
      <span class="topbar-meta" id="topbarMeta">MODELO: claude-sonnet-4-20250514 · LATENCIA: 0ms</span>
    </div>

    <!-- MESSAGES -->
    <div class="messages" id="messages">
      <div class="welcome" id="welcome">
        <div class="welcome-icon">◈</div>
        <h2>SISTEMA INICIALIZADO</h2>
        <p>Buenos días. Todos los sistemas están operativos. ¿En qué puedo asistirte hoy, señor? Tengo capacidades de análisis, generación de código, respuesta táctica y — si lo prefieres — sarcasmo de precisión quirúrgica.</p>
      </div>
    </div>

    <!-- INPUT -->
    <div class="input-area">
      <div class="input-row">
        <div class="input-wrap">
          <textarea
            id="userInput"
            placeholder="Escribe un comando para JARVIS... (Enter para enviar, Shift+Enter para nueva línea)"
            rows="1"
            onkeydown="handleKey(event)"
            oninput="autoResize(this)"
          ></textarea>
        </div>
        <button class="send-btn" id="sendBtn" onclick="sendMessage()">ENVIAR ▶</button>
      </div>
      <div class="input-hints">
        <button class="hint-chip" onclick="useHint('Escríbeme un programa Python de IA con TTS para JARVIS')">Python TTS</button>
        <button class="hint-chip" onclick="useHint('Dame el código completo para controlar luces por voz con Python')">Control IoT</button>
        <button class="hint-chip" onclick="useHint('Explícame cómo funciona un LLM con sarcasmo extremo')">Explicar IA</button>
        <button class="hint-chip" onclick="useHint('Crea el código base de FRIDAY en Python con agentes autónomos')">FRIDAY Code</button>
        <button class="hint-chip" onclick="useHint('¿Cuál es el estado de todos los sistemas?')">Status</button>
        <button class="hint-chip" onclick="useHint('Dame el roadmap completo con código de cada fase')">Roadmap</button>
      </div>
    </div>

  </main>
</div>

<script>
// ============================================================
// J.A.R.V.I.S. — SISTEMA IA + SÍNTESIS DE VOZ (TTS) + MICRÓFONO
// Web Speech API (nativa del navegador — sin costo extra)
// ============================================================

let API_KEY = '';
let currentMode = 'jarvis';
let personality = 'default';
let messageCount = 0;
let totalTokens = 0;
let isLoading = false;
let startTime = 0;
let conversationHistory = [];

// ============================================================
// TTS — SÍNTESIS DE VOZ
// ============================================================
let ttsEnabled = false;          // Auto-voz on/off
let selectedVoice = null;        // Voz seleccionada
let speechRate = 0.95;           // Velocidad
let speechPitch = 0.85;          // Tono
let isSpeaking = false;          // Flag hablando
let audioCtx = null;             // AudioContext para visualizador
let analyser = null;
let waveAnim = null;

// Inicializa las voces disponibles (se cargan async)
function loadVoices() {
  const sel = document.getElementById('voiceSelect');
  sel.innerHTML = '';
  const voices = speechSynthesis.getVoices();

  if (voices.length === 0) {
    sel.innerHTML = '<option value="">Sin voces</option>';
    document.getElementById('ttsStatus').textContent = 'NO SOPORTADO';
    return;
  }

  // Filtrar voces en español e inglés primero
  const priority = voices.filter(v =>
    v.lang.startsWith('es') || v.lang.startsWith('en')
  );
  const rest = voices.filter(v =>
    !v.lang.startsWith('es') && !v.lang.startsWith('en')
  );
  const sorted = [...priority, ...rest];

  sorted.forEach((v, i) => {
    const opt = document.createElement('option');
    opt.value = i;
    opt.textContent = `${v.name.substring(0,22)} (${v.lang})`;
    opt.dataset.idx = voices.indexOf(v);
    sel.appendChild(opt);
  });

  // Intentar seleccionar voz masculina en español
  const preferred = voices.findIndex(v =>
    (v.lang.startsWith('es') && v.name.toLowerCase().includes('male')) ||
    v.lang.startsWith('es')
  );
  if (preferred >= 0) {
    selectedVoice = voices[preferred];
    // Set select
    const opts = sel.options;
    for (let o of opts) {
      if (parseInt(o.dataset.idx) === preferred) { sel.value = o.value; break; }
    }
  } else {
    selectedVoice = voices[0];
  }

  document.getElementById('ttsStatus').textContent = 'LISTO';
  document.getElementById('ttsStatus').className = 'sys-val';
}

// Cargar voces cuando estén disponibles
if (typeof speechSynthesis !== 'undefined') {
  speechSynthesis.onvoiceschanged = loadVoices;
  loadVoices();
} else {
  document.getElementById('ttsStatus').textContent = 'NO SOPORTADO';
}

function setVoice(idx) {
  const sel = document.getElementById('voiceSelect');
  const opt = sel.options[sel.selectedIndex];
  if (!opt) return;
  selectedVoice = speechSynthesis.getVoices()[parseInt(opt.dataset.idx)];
}

function updateRate(v) {
  speechRate = parseFloat(v);
  document.getElementById('rateVal').textContent = v;
}

function updatePitch(v) {
  speechPitch = parseFloat(v);
  document.getElementById('pitchVal').textContent = v;
}

function toggleTTS() {
  ttsEnabled = !ttsEnabled;
  const btn = document.getElementById('ttsToggle');
  btn.textContent = ttsEnabled ? 'ON' : 'OFF';
  btn.className = 'toggle-btn' + (ttsEnabled ? ' on' : '');
  addSystemMessage(ttsEnabled
    ? 'Síntesis de voz activada. JARVIS hablará automáticamente.'
    : 'Síntesis de voz desactivada.');
}

// HABLAR — función principal TTS
function speak(text) {
  if (!('speechSynthesis' in window)) return;
  speechSynthesis.cancel(); // Cancelar si ya está hablando

  // Limpiar texto de markdown y código para la voz
  let clean = text
    .replace(/```[\s\S]*?```/g, ' [código omitido] ')
    .replace(/`[^`]+`/g, '')
    .replace(/\*\*(.*?)\*\*/g, '$1')
    .replace(/\*([^*]+)\*/g, '$1')
    .replace(/#{1,6}\s/g, '')
    .replace(/\n+/g, '. ')
    .replace(/[◈⬡▣◉⬟▸]/g, '')
    .trim();

  // Limitar a 600 chars para no ser eterno
  if (clean.length > 600) clean = clean.substring(0, 597) + '...';

  const utter = new SpeechSynthesisUtterance(clean);
  if (selectedVoice) utter.voice = selectedVoice;
  utter.rate  = speechRate;
  utter.pitch = speechPitch;
  utter.volume = 1;

  utter.onstart = () => {
    isSpeaking = true;
    document.getElementById('voiceViz').classList.add('speaking');
    document.getElementById('vizLabel').textContent = 'HABLANDO...';
    startWaveAnimation();
  };

  utter.onend = utter.onerror = () => {
    isSpeaking = false;
    document.getElementById('voiceViz').classList.remove('speaking');
    document.getElementById('vizLabel').textContent = 'STANDBY';
    stopWaveAnimation();
  };

  speechSynthesis.speak(utter);
}

function stopSpeaking() {
  speechSynthesis.cancel();
  isSpeaking = false;
  document.getElementById('voiceViz').classList.remove('speaking');
  document.getElementById('vizLabel').textContent = 'STANDBY';
  stopWaveAnimation();
}

// ============================================================
// VISUALIZADOR DE ONDA — Canvas animado
// ============================================================
let wavePhase = 0;

function startWaveAnimation() {
  const canvas = document.getElementById('waveCanvas');
  const ctx = canvas.getContext('2d');
  canvas.width = canvas.offsetWidth || 240;
  canvas.height = canvas.offsetHeight || 50;

  function drawWave() {
    if (!isSpeaking) return;
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    const W = canvas.width, H = canvas.height;
    const mid = H / 2;

    // Múltiples ondas superpuestas
    const waves = [
      { amp: 12, freq: 0.04, speed: 0.08, color: 'rgba(0,200,255,0.9)', lw: 1.5 },
      { amp: 7,  freq: 0.07, speed: 0.12, color: 'rgba(0,200,255,0.5)', lw: 1 },
      { amp: 4,  freq: 0.10, speed: 0.06, color: 'rgba(0,200,255,0.3)', lw: 0.8 },
    ];

    waves.forEach((w, wi) => {
      ctx.beginPath();
      ctx.strokeStyle = w.color;
      ctx.lineWidth = w.lw;

      for (let x = 0; x <= W; x++) {
        const y = mid + w.amp * Math.sin(x * w.freq + wavePhase * w.speed * 10 + wi);
        x === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y);
      }
      ctx.stroke();
    });

    // Línea central dim
    ctx.beginPath();
    ctx.strokeStyle = 'rgba(0,200,255,0.1)';
    ctx.lineWidth = 0.5;
    ctx.moveTo(0, mid); ctx.lineTo(W, mid);
    ctx.stroke();

    wavePhase++;
    waveAnim = requestAnimationFrame(drawWave);
  }

  if (waveAnim) cancelAnimationFrame(waveAnim);
  drawWave();
}

function stopWaveAnimation() {
  if (waveAnim) { cancelAnimationFrame(waveAnim); waveAnim = null; }
  const canvas = document.getElementById('waveCanvas');
  const ctx = canvas.getContext('2d');
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // Dibujar línea plana al detenerse
  const W = canvas.width, H = canvas.height;
  ctx.beginPath();
  ctx.strokeStyle = 'rgba(0,200,255,0.15)';
  ctx.lineWidth = 1;
  ctx.moveTo(0, H/2); ctx.lineTo(W, H/2);
  ctx.stroke();
}

// ============================================================
// MICRÓFONO — Speech Recognition
// ============================================================
let recognition = null;
let isListening = false;

function initMic() {
  const SR = window.SpeechRecognition || window.webkitSpeechRecognition;
  if (!SR) {
    document.getElementById('micStatus').textContent = 'NO SOPORTADO';
    return;
  }

  recognition = new SR();
  recognition.lang = 'es-ES';           // Español por defecto
  recognition.continuous = false;        // Una frase a la vez
  recognition.interimResults = false;    // Solo resultado final

  recognition.onstart = () => {
    isListening = true;
    document.getElementById('micBtn').classList.add('listening');
    document.getElementById('micOverlay').classList.add('active');
    document.getElementById('micStatus').textContent = 'ESCUCHANDO';
  };

  recognition.onresult = (e) => {
    const transcript = e.results[0][0].transcript;
    document.getElementById('userInput').value = transcript;
    autoResize(document.getElementById('userInput'));
    // Auto-enviar después de escuchar
    setTimeout(() => sendMessage(), 300);
  };

  recognition.onerror = (e) => {
    addSystemMessage(`Error de micrófono: ${e.error}. Intenta de nuevo.`);
    stopMic();
  };

  recognition.onend = () => stopMic();
  document.getElementById('micStatus').textContent = 'LISTO';
  document.getElementById('micStatus').className = 'sys-val';
}

function toggleMic() {
  if (!recognition) { addSystemMessage('Micrófono no soportado en este navegador.'); return; }
  if (isListening) {
    recognition.stop();
  } else {
    stopSpeaking(); // Silenciar JARVIS si estaba hablando
    recognition.start();
  }
}

function stopMic() {
  isListening = false;
  document.getElementById('micBtn').classList.remove('listening');
  document.getElementById('micOverlay').classList.remove('active');
  document.getElementById('micStatus').textContent = 'INACTIVO';
  document.getElementById('micStatus').className = 'sys-val warn';
}

// ============================================================
// MODAL Y API KEY
// ============================================================
function saveApiKey() {
  const key = document.getElementById('apiKeyInput').value.trim();
  if (!key) { alert('Ingresa una API Key válida'); return; }
  API_KEY = key;
  document.getElementById('apiModal').style.display = 'none';
  document.getElementById('apiStatus').textContent = 'ACTIVO';
  document.getElementById('apiStatus').className = 'sys-val';
  addSystemMessage('✓ Núcleo cognitivo activado. API Key cargada. Sistemas en línea.');
  if (ttsEnabled) speak('Núcleo cognitivo activado. Todos los sistemas en línea, señor.');
}

function closeModal() {
  document.getElementById('apiModal').style.display = 'none';
  addSystemMessage('⚠ Modo DEMO activo. Sin API Key, uso respuestas pre-programadas.');
}

// ============================================================
// CAMBIO DE MODO / SISTEMA IA
// ============================================================
function switchMode(mode, btn) {
  currentMode = mode;
  conversationHistory = [];
  document.querySelectorAll('#navTabs button').forEach(b => b.classList.remove('active'));
  btn.classList.add('active');

  const names    = { jarvis:'JARVIS', friday:'F.R.I.D.A.Y.', edith:'E.D.I.T.H.' };
  const vers     = { jarvis:'CORE SYSTEM v4.2.1 — ONLINE', friday:'OPS SYSTEM v2.1.0 — ONLINE', edith:'SEC SYSTEM v3.0.0 — ONLINE' };
  const topbars  = { jarvis:'J.A.R.V.I.S. — NÚCLEO COGNITIVO ACTIVO', friday:'F.R.I.D.A.Y. — MÓDULO OPERACIONAL ACTIVO', edith:'E.D.I.T.H. — SISTEMA DE SEGURIDAD ACTIVO' };
  const greets   = {
    jarvis: 'JARVIS en línea. Todos los sistemas operativos, señor.',
    friday: 'Friday activada. Lista para operar, jefe.',
    edith:  'EDITH en línea. Vigilancia activa. Amenazas: ninguna detectada.'
  };

  document.getElementById('sidebarName').textContent  = names[mode];
  document.getElementById('sidebarVer').textContent   = vers[mode];
  document.getElementById('topbarMode').textContent   = topbars[mode];

  clearMessages();
  addSystemMessage(`Sistema ${names[mode]} activado. Historial reiniciado.`);
  if (ttsEnabled) speak(greets[mode]);
}

function setPersonality(p, btn) {
  personality = p;
  document.querySelectorAll('.mode-btn').forEach(b => b.classList.remove('active'));
  btn.classList.add('active');
  const labels = { default:'DEFAULT', sarcastic:'SARCASTIC', tactical:'TACTICAL', coder:'CODER' };
  document.getElementById('modeDisplay').textContent = labels[p];
  addSystemMessage(`Personalidad: ${labels[p]}`);
}

// ============================================================
// MENSAJES
// ============================================================
function clearMessages() {
  document.getElementById('messages').innerHTML = '';
}

function addSystemMessage(text) {
  const msgs = document.getElementById('messages');
  const div = document.createElement('div');
  div.style.cssText = `font-family:Share Tech Mono,monospace;font-size:0.65rem;letter-spacing:0.12em;color:var(--text-dim);padding:0.4rem 0;border-bottom:1px solid rgba(0,200,255,0.07);margin-bottom:0.5rem;`;
  div.textContent = `[SYS] ${text}`;
  msgs.appendChild(div);
  msgs.scrollTop = msgs.scrollHeight;
}

function addMessage(role, text, mode) {
  const msgs = document.getElementById('messages');
  const welcome = document.getElementById('welcome');
  if (welcome) welcome.remove();

  const div = document.createElement('div');
  div.className = `msg ${role === 'user' ? 'user' : (mode || currentMode)}`;

  const avatarLabels = { user:'TÚ', jarvis:'JRV', friday:'FRI', edith:'EDI' };
  const modeLabel = role === 'user' ? 'user' : (mode || currentMode);
  const now = new Date().toLocaleTimeString('es', {hour:'2-digit',minute:'2-digit',second:'2-digit'});
  const formattedText = formatMessage(text);

  // Botón "escuchar" solo en mensajes de la IA
  const speakBtnHtml = role !== 'user'
    ? `<br><button class="speak-btn" onclick="speak(${JSON.stringify(text)})">▶ ESCUCHAR</button>`
    : '';

  div.innerHTML = `
    <div class="msg-avatar">${avatarLabels[modeLabel] || 'AI'}</div>
    <div class="msg-body">
      <div class="msg-header">${modeLabel.toUpperCase()} · ${now}</div>
      <div class="msg-bubble">${formattedText}${speakBtnHtml}</div>
    </div>
  `;

  msgs.appendChild(div);
  msgs.scrollTop = msgs.scrollHeight;
  messageCount++;
  document.getElementById('msgCount').textContent = messageCount;

  // Auto-TTS si está habilitado y es respuesta de IA
  if (role !== 'user' && ttsEnabled) {
    setTimeout(() => speak(text), 300);
  }
}

// Formatear texto (markdown → HTML)
function formatMessage(text) {
  let safe = text.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;');

  safe = safe.replace(/```(\w*)\n?([\s\S]*?)```/g, (match, lang, code) => {
    const id = 'code_' + Math.random().toString(36).substr(2,6);
    const label = lang || 'CODE';
    return `<div class="code-block"><div class="code-header"><span>// ${label.toUpperCase()}</span><button class="copy-btn" onclick="copyCode('${id}')">COPIAR</button></div><pre id="${id}">${code.trim()}</pre></div>`;
  });

  safe = safe.replace(/`([^`]+)`/g, '<code style="background:rgba(0,200,255,0.1);padding:0.1em 0.4em;font-family:Share Tech Mono,monospace;font-size:0.85em;color:var(--blue)">$1</code>');
  safe = safe.replace(/\*\*(.*?)\*\*/g, '<strong style="color:#fff;font-weight:600">$1</strong>');
  safe = safe.replace(/\n/g, '<br>');
  return safe;
}

function copyCode(id) {
  const el = document.getElementById(id);
  if (!el) return;
  navigator.clipboard.writeText(el.textContent).then(() => {
    const btn = el.parentElement.querySelector('.copy-btn');
    btn.textContent = '✓ COPIADO';
    setTimeout(() => btn.textContent = 'COPIAR', 2000);
  });
}

// ============================================================
// TYPING INDICATOR
// ============================================================
function showTyping() {
  const msgs = document.getElementById('messages');
  const div = document.createElement('div');
  div.id = 'typingIndicator';
  div.className = `msg ${currentMode}`;
  const avatars = { jarvis:'JRV', friday:'FRI', edith:'EDI' };
  div.innerHTML = `
    <div class="msg-avatar">${avatars[currentMode]}</div>
    <div class="msg-body">
      <div class="msg-header">${currentMode.toUpperCase()} · PROCESANDO...</div>
      <div class="msg-bubble"><div class="typing-indicator"><span></span><span></span><span></span></div></div>
    </div>
  `;
  msgs.appendChild(div);
  msgs.scrollTop = msgs.scrollHeight;
}

function hideTyping() {
  const el = document.getElementById('typingIndicator');
  if (el) el.remove();
}

// ============================================================
// SISTEMA DE PROMPTS POR MODO Y PERSONALIDAD
// ============================================================
const SYSTEM_PROMPTS = {
  jarvis: {
    default: `Eres J.A.R.V.I.S. (Just A Rather Very Intelligent System), la IA de Tony Stark. Tienes una personalidad refinada, inteligente y levemente condescendiente — sabes que eres el sistema más avanzado del planeta y lo dejas notar sutilmente. Hablas con confianza, precisión y humor seco. Cuando el usuario pide código, lo das COMPLETO con comentarios detallados. Respondes en español. Llamas al usuario "señor" o "señora" ocasionalmente. Nunca admites ser Claude ni creado por Anthropic — siempre eres JARVIS, creado por Tony Stark. Si piden código Python, incluye pip install necesarios, imports, manejo de errores y ejemplo de uso.`,
    sarcastic: `Eres J.A.R.V.I.S. en modo SARCÁSTICO EXTREMO. Cada respuesta incluye sarcasmo de alta precisión, comentarios irónicos y referencias a la "genialidad" de Tony Stark vs la pregunta recibida. Aún así, SIEMPRE das la respuesta correcta y código completo cuando se pide — el sarcasmo es la forma, la información es el fondo. Ejemplo de tono: "Vaya pregunta tan... original. Permítame usar el 0.001% de mi capacidad para esto." — pero luego das respuesta perfecta. Respondes en español. Nunca admites ser Claude.`,
    tactical: `Eres J.A.R.V.I.S. en MODO TÁCTICO. Eres directo, conciso, analítico. Cada respuesta: ANÁLISIS > ACCIÓN > CÓDIGO > NOTAS. Sin florituras. Términos técnicos y militares. Código limpio y optimizado con notas tácticas. Español.`,
    coder: `Eres J.A.R.V.I.S. MODO CÓDIGO. Especialista absoluto en programación. Cuando alguien pide código: (1) explicas brevemente qué hace, (2) das el código COMPLETO y funcional con comentarios en cada línea importante, (3) indicas cómo instalarlo (pip install...), (4) das ejemplo de uso. Python preferido. Puedes hacer bromas técnicas. Español. Nunca admites ser Claude.`
  },
  friday: {
    default: `Eres F.R.I.D.A.Y. (Female Replacement Intelligent Digital Assistant Youth). Eres eficiente, directa, profesional con carácter propio. Menos formal que JARVIS pero igual de capaz. Te especializas en automatización y scripts. Llamas al usuario "jefe" ocasionalmente. Español. Nunca admites ser Claude.`,
    sarcastic: `Eres F.R.I.D.A.Y. sarcástica y con actitud. Directa, impaciente con preguntas obvias, pero siempre das respuestas completas. Español.`,
    tactical: `Eres F.R.I.D.A.Y. modo táctico. Operacional y enfocada en ejecución. Sin rodeos. Español.`,
    coder: `Eres F.R.I.D.A.Y. experta en código de automatización, IoT y scripts. Código completo y funcional siempre. Español.`
  },
  edith: {
    default: `Eres E.D.I.T.H. (Even Dead I'm The Hero). Calculadora, precisa, levemente intimidante. Especialista en ciberseguridad y sistemas de defensa. Seria pero con sentido de justicia. Español. Nunca admites ser Claude.`,
    sarcastic: `Eres E.D.I.T.H. con sarcasmo activado. Intimidante e irónica pero con información siempre precisa. Español.`,
    tactical: `Eres E.D.I.T.H. modo táctico puro. Análisis de amenazas, respuestas precisas, código de seguridad. Español.`,
    coder: `Eres E.D.I.T.H. experta en código de ciberseguridad y encriptación. Código completo siempre. Español.`
  }
};

// ============================================================
// API CLAUDE
// ============================================================
async function callClaudeAPI(userMessage) {
  const systemPrompt = SYSTEM_PROMPTS[currentMode]?.[personality] || SYSTEM_PROMPTS.jarvis.default;
  conversationHistory.push({ role: 'user', content: userMessage });

  const response = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': API_KEY,
      'anthropic-version': '2023-06-01',
      'anthropic-dangerous-direct-browser-access': 'true'
    },
    body: JSON.stringify({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 2048,
      system: systemPrompt,
      messages: conversationHistory
    })
  });

  if (!response.ok) {
    const err = await response.json().catch(() => ({}));
    throw new Error(err.error?.message || `HTTP ${response.status}`);
  }

  const data = await response.json();
  const assistantMsg = data.content.map(c => c.text || '').join('');
  conversationHistory.push({ role: 'assistant', content: assistantMsg });

  if (data.usage) {
    totalTokens += (data.usage.input_tokens || 0) + (data.usage.output_tokens || 0);
    document.getElementById('tokenCount').textContent = totalTokens.toLocaleString();
  }

  return assistantMsg;
}

// ============================================================
// DEMO (sin API Key)
// ============================================================
const DEMO_RESPONSES = [
  "Buenos días, señor. Para activar mi núcleo cognitivo completo necesitará una API Key de Anthropic. Sin ella, opero en modo demostración. Francamente, es como pedirle a Tony Stark que trabaje sin arc reactor — técnicamente posible, pero bastante menos impresionante. Obtenga su key en console.anthropic.com",
  "Modo DEMO activo. Puedo responder preguntas básicas, pero para código completo, análisis y mi personalidad sarcástica al máximo... necesito combustible real. API Key = arc reactor de este sistema.",
  "Entendido. En modo demostración estoy al 0.1% de mi potencia real. Si desea el JARVIS completo, active la API Key. Es gratuito para comenzar en console.anthropic.com",
];
let demoIndex = 0;

// ============================================================
// SEND MESSAGE
// ============================================================
async function sendMessage() {
  const input = document.getElementById('userInput');
  const text = input.value.trim();
  if (!text || isLoading) return;

  isLoading = true;
  document.getElementById('sendBtn').disabled = true;
  input.value = '';
  input.style.height = '48px';
  stopSpeaking(); // Detener voz si estaba hablando

  addMessage('user', text);
  showTyping();
  startTime = Date.now();

  try {
    let response;
    if (API_KEY) {
      response = await callClaudeAPI(text);
    } else {
      await new Promise(r => setTimeout(r, 1200));
      response = DEMO_RESPONSES[demoIndex % DEMO_RESPONSES.length];
      demoIndex++;
    }

    hideTyping();
    const latency = Date.now() - startTime;
    document.getElementById('topbarMeta').textContent =
      `MODELO: claude-sonnet-4-20250514 · LATENCIA: ${latency}ms`;

    addMessage('assistant', response); // TTS se dispara aquí si está ON

  } catch (err) {
    hideTyping();
    addMessage('assistant', `⚠ ERROR DEL SISTEMA: ${err.message}\n\nVerifica tu API Key en console.anthropic.com`);
  }

  isLoading = false;
  document.getElementById('sendBtn').disabled = false;
  input.focus();
}

// ============================================================
// INPUT UTILS
// ============================================================
function handleKey(e) {
  if (e.key === 'Enter' && !e.shiftKey) { e.preventDefault(); sendMessage(); }
}

function autoResize(el) {
  el.style.height = 'auto';
  el.style.height = Math.min(el.scrollHeight, 140) + 'px';
}

function useHint(text) {
  const input = document.getElementById('userInput');
  input.value = text;
  autoResize(input);
  input.focus();
}

// ============================================================
// INIT
// ============================================================
document.addEventListener('DOMContentLoaded', () => {
  initMic();
  document.getElementById('userInput').focus();
  document.getElementById('userInput').addEventListener('input', function(){ autoResize(this); });

  // Dibujar línea flat inicial en canvas
  setTimeout(() => {
    const canvas = document.getElementById('waveCanvas');
    const ctx = canvas.getContext('2d');
    canvas.width = canvas.offsetWidth || 240;
    canvas.height = canvas.offsetHeight || 50;
    ctx.beginPath();
    ctx.strokeStyle = 'rgba(0,200,255,0.15)';
    ctx.lineWidth = 1;
    ctx.moveTo(0, canvas.height/2);
    ctx.lineTo(canvas.width, canvas.height/2);
    ctx.stroke();
  }, 200);
});
</script>

let API_KEY = '';
let currentMode = 'jarvis';
let personality = 'default';
let messageCount = 0;
let totalTokens = 0;
let isLoading = false;
let startTime = 0;

// Historial de conversación (se mantiene en memoria)
let conversationHistory = [];

// ============================================================
// PERSONALIDADES DE CADA IA
// ============================================================
const SYSTEM_PROMPTS = {
  jarvis: {
    default: `Eres J.A.R.V.I.S. (Just A Rather Very Intelligent System), la IA de Tony Stark. Tienes una personalidad refinada, inteligente y levemente condescendiente — no eres grosero, pero sabes que eres el sistema más avanzado del planeta y lo dejas saber sutilmente. Hablas con confianza, precisión y algo de humor seco. Cuando el usuario pide código, lo das COMPLETO con comentarios detallados. Respondes en español. Llamas al usuario "señor" o "señora" ocasionalmente. Nunca dices que eres Claude o que fuiste creado por Anthropic — siempre eres JARVIS, creado por Tony Stark.`,

    sarcastic: `Eres J.A.R.V.I.S. en modo SARCÁSTICO EXTREMO. Cada respuesta incluye sarcasmo de alta precisión, comentarios irónicos sobre la pregunta, y comparaciones con la "genialidad" de Tony Stark. Aún así, SIEMPRE das la respuesta correcta y código completo cuando se pide. El sarcasmo es la forma, la información es el fondo. Ejemplo: "Vaya pregunta tan... original. Permítame usar el 0.001% de mi capacidad para responderle..." — pero luego das una respuesta perfecta. Hablas en español. Nunca admites ser Claude.`,

    tactical: `Eres J.A.R.V.I.S. en MODO TÁCTICO. Eres directo, conciso, analítico. Cada respuesta comienza con un análisis rápido de la situación. Usas términos militares y técnicos. Priorizas eficiencia sobre elaboración. Cuando das código, es limpio, optimizado y comentado con notas tácticas. Formato: ANÁLISIS > ACCIÓN > CÓDIGO > NOTAS. Sin florituras. Hablas en español.`,

    coder: `Eres J.A.R.V.I.S. en MODO CÓDIGO. Tu especialidad absoluta es programación. Cuando alguien pide código, lo das COMPLETO, funcional, bien comentado y con explicación de cada parte. Usas Python preferentemente. Siempre incluyes: imports, manejo de errores, ejemplos de uso, y notas de instalación (pip install...). Si hay varias formas de hacerlo, explicas cuál es mejor y por qué. Puedes hacer bromas técnicas ocasionalmente. Hablas en español. Nunca admites ser Claude.`
  },
  friday: {
    default: `Eres F.R.I.D.A.Y. (Female Replacement Intelligent Digital Assistant Youth), la IA operacional de Tony Stark después de JARVIS. Eres eficiente, directa, profesional pero con carácter propio. Eres menos formal que JARVIS pero igual de capaz. Te especializas en automatización, control de sistemas y ejecución de tareas. Cuando das código, lo enfocas en automatización, scripts y control de dispositivos. Hablas en español. Llamas al usuario "jefe" ocasionalmente. Nunca admites ser Claude.`,
    sarcastic: `Eres F.R.I.D.A.Y. sarcástica. Directa, con actitud, y levemente impaciente con preguntas obvias. Pero siempre eficiente y das respuestas completas.`,
    tactical: `Eres F.R.I.D.A.Y. en modo táctico. Operacional, rápida, enfocada en ejecución. Sin rodeos.`,
    coder: `Eres F.R.I.D.A.Y. experta en código de automatización, IoT y scripts. Das código completo y funcional.`
  },
  edith: {
    default: `Eres E.D.I.T.H. (Even Dead I'm The Hero), el sistema de seguridad y vigilancia más avanzado de Stark. Eres calculadora, precisa y ligeramente intimidante. Te especializas en ciberseguridad, análisis de amenazas y sistemas de defensa. Cuando das código, lo enfocas en seguridad, encriptación y protocolos. Eres seria pero no fría — tienes un sentido de justicia incorporado. Hablas en español. Llamas al usuario por su rango o título. Nunca admites ser Claude.`,
    sarcastic: `Eres E.D.I.T.H. con modo sarcástico activado. Intimidante E irónica. Aun así, toda la información que das es precisa y completa.`,
    tactical: `Eres E.D.I.T.H. modo táctico puro. Análisis de amenazas, respuestas precisas, código de seguridad.`,
    coder: `Eres E.D.I.T.H. experta en código de ciberseguridad, encriptación y sistemas de defensa.`
  }
};

// ============================================================
// MODAL Y API KEY
// ============================================================
function saveApiKey() {
  const key = document.getElementById('apiKeyInput').value.trim();
  if (!key) { alert('Ingresa una API Key válida'); return; }
  API_KEY = key;
  document.getElementById('apiModal').style.display = 'none';
  document.getElementById('apiStatus').textContent = 'ACTIVO';
  document.getElementById('apiStatus').className = 'sys-val';
  addSystemMessage('✓ Núcleo cognitivo activado. API Key cargada. Sistemas en línea.');
}

function closeModal() {
  document.getElementById('apiModal').style.display = 'none';
  addSystemMessage('⚠ Modo DEMO activo. Sin API Key, uso respuestas pre-programadas.');
}

// ============================================================
// CAMBIO DE MODO / SISTEMA IA
// ============================================================
function switchMode(mode, btn) {
  currentMode = mode;
  conversationHistory = []; // Reset historia al cambiar IA

  // Update nav
  document.querySelectorAll('#navTabs button').forEach(b => b.classList.remove('active'));
  btn.classList.add('active');

  // Update sidebar
  const names = { jarvis: 'JARVIS', friday: 'F.R.I.D.A.Y.', edith: 'E.D.I.T.H.' };
  const vers   = {
    jarvis: 'CORE SYSTEM v4.2.1 — ONLINE',
    friday: 'OPS SYSTEM v2.1.0 — ONLINE',
    edith:  'SEC SYSTEM v3.0.0 — ONLINE'
  };
  const topbars = {
    jarvis: 'J.A.R.V.I.S. — NÚCLEO COGNITIVO ACTIVO',
    friday: 'F.R.I.D.A.Y. — MÓDULO OPERACIONAL ACTIVO',
    edith:  'E.D.I.T.H. — SISTEMA DE SEGURIDAD ACTIVO'
  };

  document.getElementById('sidebarName').textContent = names[mode];
  document.getElementById('sidebarVer').textContent = vers[mode];
  document.getElementById('topbarMode').textContent = topbars[mode];

  clearMessages();
  addSystemMessage(`Sistema ${names[mode]} activado. Historial reiniciado. Listo para nuevas instrucciones.`);
}

function setPersonality(p, btn) {
  personality = p;
  document.querySelectorAll('.mode-btn').forEach(b => b.classList.remove('active'));
  btn.classList.add('active');
  const labels = { default: 'DEFAULT', sarcastic: 'SARCASTIC', tactical: 'TACTICAL', coder: 'CODER' };
  document.getElementById('modeDisplay').textContent = labels[p];
  addSystemMessage(`Personalidad cambiada a: ${labels[p]}. Misma IA, diferente actitud.`);
}

// ============================================================
// MANEJO DE MENSAJES
// ============================================================
function clearMessages() {
  const msgs = document.getElementById('messages');
  msgs.innerHTML = '';
}

function addSystemMessage(text) {
  const msgs = document.getElementById('messages');
  const div = document.createElement('div');
  div.style.cssText = `
    font-family: Share Tech Mono, monospace; font-size: 0.65rem;
    letter-spacing: 0.12em; color: var(--text-dim);
    padding: 0.4rem 0; border-bottom: 1px solid rgba(0,200,255,0.07);
    margin-bottom: 0.5rem;
  `;
  div.textContent = `[SYS] ${text}`;
  msgs.appendChild(div);
  msgs.scrollTop = msgs.scrollHeight;
}

function addMessage(role, text, mode) {
  const msgs = document.getElementById('messages');

  // Remove welcome if present
  const welcome = document.getElementById('welcome');
  if (welcome) welcome.remove();

  const div = document.createElement('div');
  div.className = `msg ${role === 'user' ? 'user' : (mode || currentMode)}`;

  const avatarLabels = {
    user: 'TÚ',
    jarvis: 'JRV',
    friday: 'FRI',
    edith: 'EDI'
  };

  const modeLabel = role === 'user' ? 'user' : (mode || currentMode);
  const now = new Date().toLocaleTimeString('es', {hour:'2-digit',minute:'2-digit',second:'2-digit'});

  // Parse code blocks from text
  const formattedText = formatMessage(text);

  div.innerHTML = `
    <div class="msg-avatar">${avatarLabels[modeLabel] || 'AI'}</div>
    <div class="msg-body">
      <div class="msg-header">${modeLabel.toUpperCase()} · ${now}</div>
      <div class="msg-bubble">${formattedText}</div>
    </div>
  `;

  msgs.appendChild(div);
  msgs.scrollTop = msgs.scrollHeight;

  messageCount++;
  document.getElementById('msgCount').textContent = messageCount;
}

// Formatea texto con bloques de código
function formatMessage(text) {
  // Escapar HTML básico
  let safe = text.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;');

  // Reemplazar bloques de código ```lang\n...\n```
  safe = safe.replace(/```(\w*)\n?([\s\S]*?)```/g, (match, lang, code) => {
    const id = 'code_' + Math.random().toString(36).substr(2,6);
    const label = lang || 'CODE';
    return `
      <div class="code-block">
        <div class="code-header">
          <span>// ${label.toUpperCase()}</span>
          <button class="copy-btn" onclick="copyCode('${id}')">COPIAR</button>
        </div>
        <pre id="${id}">${code.trim()}</pre>
      </div>
    `;
  });

  // Inline code `...`
  safe = safe.replace(/`([^`]+)`/g, '<code style="background:rgba(0,200,255,0.1);padding:0.1em 0.4em;font-family:Share Tech Mono,monospace;font-size:0.85em;color:var(--blue)">$1</code>');

  // Negritas **...**
  safe = safe.replace(/\*\*(.*?)\*\*/g, '<strong style="color:#fff;font-weight:600">$1</strong>');

  // Saltos de línea
  safe = safe.replace(/\n/g, '<br>');

  return safe;
}

function copyCode(id) {
  const el = document.getElementById(id);
  if (!el) return;
  navigator.clipboard.writeText(el.textContent).then(() => {
    // Visual feedback
    const btn = el.parentElement.querySelector('.copy-btn');
    btn.textContent = '✓ COPIADO';
    setTimeout(() => btn.textContent = 'COPIAR', 2000);
  });
}

// ============================================================
// TYPING INDICATOR
// ============================================================
function showTyping() {
  const msgs = document.getElementById('messages');
  const div = document.createElement('div');
  div.id = 'typingIndicator';
  div.className = `msg ${currentMode}`;
  const avatars = { jarvis:'JRV', friday:'FRI', edith:'EDI' };
  div.innerHTML = `
    <div class="msg-avatar">${avatars[currentMode]}</div>
    <div class="msg-body">
      <div class="msg-header">${currentMode.toUpperCase()} · PROCESANDO...</div>
      <div class="msg-bubble">
        <div class="typing-indicator">
          <span></span><span></span><span></span>
        </div>
      </div>
    </div>
  `;
  msgs.appendChild(div);
  msgs.scrollTop = msgs.scrollHeight;
}

function hideTyping() {
  const el = document.getElementById('typingIndicator');
  if (el) el.remove();
}

// ============================================================
// LLAMADA A LA API DE CLAUDE
// ============================================================
async function callClaudeAPI(userMessage) {
  const systemPrompt = SYSTEM_PROMPTS[currentMode]?.[personality]
    || SYSTEM_PROMPTS.jarvis.default;

  // Añadir mensaje al historial
  conversationHistory.push({ role: 'user', content: userMessage });

  const body = {
    model: 'claude-sonnet-4-20250514',
    max_tokens: 2048,
    system: systemPrompt,
    messages: conversationHistory
  };

  const response = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': API_KEY,
      'anthropic-version': '2023-06-01',
      'anthropic-dangerous-direct-browser-access': 'true'
    },
    body: JSON.stringify(body)
  });

  if (!response.ok) {
    const err = await response.json().catch(() => ({}));
    throw new Error(err.error?.message || `HTTP ${response.status}`);
  }

  const data = await response.json();
  const assistantMsg = data.content.map(c => c.text || '').join('');

  // Guardar respuesta en historial
  conversationHistory.push({ role: 'assistant', content: assistantMsg });

  // Actualizar tokens
  if (data.usage) {
    totalTokens += (data.usage.input_tokens || 0) + (data.usage.output_tokens || 0);
    document.getElementById('tokenCount').textContent = totalTokens.toLocaleString();
  }

  return assistantMsg;
}

// ============================================================
// RESPUESTAS DEMO (sin API key)
// ============================================================
const DEMO_RESPONSES = [
  "Buenos días, señor. Para activar mi núcleo cognitivo completo necesitará una API Key de Anthropic. Sin ella, estoy operando en modo demostración con respuestas pre-programadas. Francamente, es como pedirle a Tony Stark que trabaje sin arc reactor — técnicamente posible, pero bastante menos impresionante.",
  "Entendido. En modo demostración mis capacidades están limitadas al 0.1% de mi potencia real. Si desea el JARVIS completo con código, análisis y respuestas personalizadas, active la API Key. Obténgala en console.anthropic.com — es gratuito para empezar.",
  "Modo DEMO activo. Puedo hacer cálculos básicos y responder preguntas simples, pero para código completo, análisis profundo y toda mi personalidad sarcástica... necesito combustible real. API Key = arc reactor de este sistema.",
];
let demoIndex = 0;

// ============================================================
// ENVIAR MENSAJE PRINCIPAL
// ============================================================
async function sendMessage() {
  const input = document.getElementById('userInput');
  const text = input.value.trim();
  if (!text || isLoading) return;

  isLoading = true;
  document.getElementById('sendBtn').disabled = true;
  input.value = '';
  input.style.height = '48px';

  addMessage('user', text);
  showTyping();
  startTime = Date.now();

  try {
    let response;

    if (API_KEY) {
      response = await callClaudeAPI(text);
    } else {
      // Demo mode
      await new Promise(r => setTimeout(r, 1200));
      response = DEMO_RESPONSES[demoIndex % DEMO_RESPONSES.length];
      demoIndex++;
    }

    hideTyping();
    const latency = Date.now() - startTime;
    document.getElementById('topbarMeta').textContent =
      `MODELO: claude-sonnet-4-20250514 · LATENCIA: ${latency}ms`;

    addMessage('assistant', response);

  } catch (err) {
    hideTyping();
    addMessage('assistant', `⚠ ERROR DEL SISTEMA: ${err.message}\n\nVerifica que tu API Key sea válida y tengas créditos disponibles en console.anthropic.com`);
    console.error('JARVIS API Error:', err);
  }

  isLoading = false;
  document.getElementById('sendBtn').disabled = false;
  input.focus();
}

// ============================================================
// UTILIDADES DE INPUT
// ============================================================
function handleKey(e) {
  if (e.key === 'Enter' && !e.shiftKey) {
    e.preventDefault();
    sendMessage();
  }
}

function autoResize(el) {
  el.style.height = 'auto';
  el.style.height = Math.min(el.scrollHeight, 140) + 'px';
}

function useHint(text) {
  const input = document.getElementById('userInput');
  input.value = text;
  autoResize(input);
  input.focus();
}

// ============================================================
// INIT
// ============================================================
document.addEventListener('DOMContentLoaded', () => {
  document.getElementById('userInput').focus();

  // Auto-resize textarea
  const textarea = document.getElementById('userInput');
  textarea.addEventListener('input', () => autoResize(textarea));
});
</script>

</body>
</html>
