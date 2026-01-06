# Quowt-
Um editor comum

script abaixo
(obs você deve criar um:index.html, você deve colocar o .html)


<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Quow! Ultra Edit</title>
    <style>
        :root {
            --bg: #000; --panel: #111; --accent: #00f2ff;
            --gold: #ffcc00; --text: #fff; --danger: #ff4757;
            --clip-v: #3d3d40; --clip-a: #1e3a5f;
        }
        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
        html, body {
            margin: 0; padding: 0; background: var(--bg); color: var(--text);
            font-family: 'Segoe UI', sans-serif; height: 100svh; width: 100vw;
            overflow: hidden;
        }

        /* --- LOBBY --- */
        #lobby {
            position: fixed; inset: 0; background: var(--bg); z-index: 2000;
            display: flex; flex-direction: column; padding: 30px; align-items: center;
        }
        .logo-q { font-size: 60px; font-weight: 900; color: var(--accent); margin-bottom: 40px; }
        .btn-new { background: var(--accent); color: #000; padding: 20px 50px; border-radius: 40px; font-weight: 900; cursor: pointer; transition: 0.3s; }

        /* --- EDITOR --- */
        #editor { display: none; flex-direction: column; height: 100svh; }

        header {
            height: 45px; padding: 0 15px; background: var(--panel);
            display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #222;
        }

        /* TELA DE PREVIEW */
        .preview-container { flex: 1; background: #080808; display: flex; align-items: center; justify-content: center; position: relative; }
        .canvas { 
            width: 90%; height: 80%; background: #000; position: relative; 
            overflow: hidden; border: 1px solid #222;
        }

        /* --- TIMELINE PROFISSIONAL --- */
        .timeline-section { height: 200px; background: #0a0a0a; border-top: 1px solid #222; display: flex; flex-direction: column; }
        
        .timeline-tools {
            height: 40px; display: flex; align-items: center; justify-content: space-between; padding: 0 15px; border-bottom: 1px solid #1a1a1a;
        }
        .play-btn { font-size: 24px; color: var(--accent); cursor: pointer; }

        .timeline-scroll { flex: 1; overflow-x: auto; position: relative; padding-left: 50%; display: flex; flex-direction: column; gap: 4px; padding-top: 10px; }
        .playhead-line { position: absolute; left: 50%; top: 0; bottom: 0; width: 2px; background: #fff; z-index: 100; pointer-events: none; }
        
        /* CLIPES NA TIMELINE */
        .track { height: 40px; display: flex; align-items: center; position: relative; min-width: 1000px; }
        .clip { 
            height: 34px; background: var(--clip-v); border: 1px solid var(--accent); 
            border-radius: 4px; display: flex; align-items: center; padding: 0 10px;
            font-size: 10px; position: absolute; cursor: pointer; overflow: hidden;
            resize: horizontal; /* Permite esticar/encolher no PC */
        }
        .clip-audio { background: var(--clip-a); border-color: #a200ff; }
        .clip.active { border-color: #fff; box-shadow: 0 0 10px var(--accent); }

        /* --- MENU PRINCIPAL --- */
        .main-menu { background: var(--panel); display: flex; overflow-x: auto; height: 75px; align-items: center; padding: 0 10px; gap: 15px; border-top: 1px solid #222; }
        .menu-item { display: flex; flex-direction: column; align-items: center; min-width: 60px; color: #888; cursor: pointer; }
        .menu-item i { font-size: 20px; color: #fff; margin-bottom: 5px; font-style: normal; }
        .menu-item span { font-size: 10px; }

        /* ELEMENTOS NO CANVAS */
        .layer { position: absolute; transform-origin: center; display: flex; align-items: center; justify-content: center; }
        .layer.selected { border: 2px solid var(--accent); }
        .layer img, .layer video { max-width: 100%; max-height: 100%; object-fit: contain; }
        .resize-dot { position: absolute; width: 16px; height: 16px; background: #fff; border: 2px solid var(--accent); border-radius: 50%; bottom: -8px; right: -8px; }

        /* ANIMAÇÃO DE KEYFRAME */
        .key-btn { color: var(--gold); border: 1px solid var(--gold); padding: 2px 5px; border-radius: 4px; font-size: 10px; }
    </style>
</head>
<body onclick="earn(0.1)">

<div id="lobby">
    <div class="logo-q">Quow!</div>
    <div class="btn-new" onclick="startEditor()">NOVO PROJETO</div>
    <p style="margin-top:20px; color: var(--gold)">🪙 <span id="l-coins">0</span></p>
</div>

<div id="editor">
    <header>
        <span onclick="location.reload()">🏠</span>
        <div class="key-btn" onclick="addKeyframe()">+ KEYFRAME</div>
        <button onclick="alert('Exportando MP4...')" style="background:var(--accent); border:none; border-radius:10px; padding:5px 10px; font-weight:bold;">SALVAR</button>
    </header>

    <div class="preview-container">
        <div class="canvas" id="canvas"></div>
    </div>

    <div class="timeline-section">
        <div class="timeline-tools">
            <span onclick="splitClip()" style="font-size:12px;">✂️ CORTAR</span>
            <div class="play-btn" id="play-btn" onclick="togglePlay()">▶️</div>
            <span id="time-display" style="font-size:10px;">00:00:00</span>
        </div>
        
        <div class="timeline-scroll">
            <div class="playhead-line"></div>
            <div class="track" id="v-track"></div>
            <div class="track" id="a-track"></div>
        </div>
    </div>

    <div class="main-menu">
        <div class="menu-item" onclick="importMedia('video')"><i>🎥</i><span>Camada</span></div>
        <div class="menu-item" onclick="importMedia('audio')"><i>🎵</i><span>Música</span></div>
        <div class="menu-item" onclick="alert('Texto')"><i>Aa</i><span>Texto</span></div>
        <div class="menu-item" onclick="alert('Dublando')"><i>🎙️</i><span>Dublar</span></div>
        <div class="menu-item" onclick="alert('Filtros')"><i>✨</i><span>Filtros</span></div>
        <div class="menu-item" onclick="deleteSelected()" style="color:var(--danger)"><i>🗑️</i><span>Excluir</span></div>
    </div>
</div>

<input type="file" id="inp" hidden>

<script>
let coins = localStorage.getItem('q_c') ? parseFloat(localStorage.getItem('q_c')) : 0;
let isPlaying = false;
let selectedClip = null;
let activeLayer = null;

function earn(v) {
    coins += v;
    document.getElementById('l-coins').innerText = Math.floor(coins);
    localStorage.setItem('q_c', coins);
}

function startEditor() {
    document.getElementById('lobby').style.display = 'none';
    document.getElementById('editor').style.display = 'flex';
}

function importMedia(type) {
    const inp = document.getElementById('inp');
    inp.accept = type === 'audio' ? "audio/*" : "video/*,image/*";
    inp.click();
}

document.getElementById('inp').onchange = function(e) {
    const file = e.target.files[0];
    if(!file) return;
    const url = URL.createObjectURL(file);

    if(file.type.startsWith('audio')) {
        addClipToTimeline(file.name, 'audio');
    } else {
        // Criar Layer no Canvas (Com Fit automático)
        const layer = document.createElement('div');
        layer.className = 'layer';
        layer.style.width = '100%'; // Fit automático
        layer.style.height = '100%';
        
        const content = file.type.startsWith('video') ? document.createElement('video') : document.createElement('img');
        content.src = url;
        if(file.type.startsWith('video')) { content.loop = true; content.muted = true; content.play(); }
        
        layer.appendChild(content);
        layer.innerHTML += `<div class="resize-dot"></div>`;
        document.getElementById('canvas').appendChild(layer);
        
        makeDraggable(layer);
        addClipToTimeline(file.name, 'video', layer);
    }
    earn(50);
};

function addClipToTimeline(name, type, linkedLayer) {
    const clip = document.createElement('div');
    clip.className = 'clip ' + (type === 'audio' ? 'clip-audio' : '');
    clip.innerText = name.substring(0,6);
    clip.style.left = "0px";
    clip.style.width = "150px"; // Duração padrão
    
    clip.onclick = () => {
        document.querySelectorAll('.clip').forEach(c => c.classList.remove('active'));
        clip.classList.add('active');
        selectedClip = clip;
        if(linkedLayer) {
            document.querySelectorAll('.layer').forEach(l => l.classList.remove('selected'));
            linkedLayer.classList.add('selected');
            activeLayer = linkedLayer;
        }
    };

    document.getElementById(type === 'audio' ? 'a-track' : 'v-track').appendChild(clip);
}

function splitClip() {
    if(!selectedClip) return alert("Selecione um clipe na timeline!");
    const w = parseInt(selectedClip.style.width);
    selectedClip.style.width = (w/2) + "px";
    
    const newClip = selectedClip.cloneNode(true);
    newClip.style.left = (parseInt(selectedClip.style.left) + w/2) + "px";
    selectedClip.parentNode.appendChild(newClip);
    earn(10);
}

function togglePlay() {
    isPlaying = !isPlaying;
    document.getElementById('play-btn').innerText = isPlaying ? "⏸️" : "▶️";
}

function makeDraggable(el) {
    let d=false, r=false, sx, sy, sw, sh;
    el.onmousedown = el.ontouchstart = (e) => {
        const cx = e.clientX || e.touches[0].clientX;
        const cy = e.clientY || e.touches[0].clientY;
        if(e.target.className === 'resize-dot') {
            r = true; sw = el.offsetWidth; sh = el.offsetHeight; sx = cx; sy = cy;
        } else {
            d = true; sx = cx - el.offsetLeft; sy = cy - el.offsetTop;
        }
    };
    window.onmousemove = window.ontouchmove = (e) => {
        const cx = e.clientX || (e.touches ? e.touches[0].clientX : null);
        const cy = e.clientY || (e.touches ? e.touches[0].clientY : null);
        if(!cx) return;
        if(d) { el.style.left = (cx - sx) + 'px'; el.style.top = (cy - sy) + 'px'; }
        if(r) { 
            el.style.width = (sw + (cx - sx)) + 'px';
            el.style.height = (sh + (cy - sy)) + 'px';
        }
    };
    window.onmouseup = window.ontouchend = () => { d = r = false; };
}

function deleteSelected() {
    if(selectedClip) { selectedClip.remove(); selectedClip = null; }
    if(activeLayer) { activeLayer.remove(); activeLayer = null; }
}

function addKeyframe() {
    if(!activeLayer) return alert("Selecione algo na tela primeiro!");
    activeLayer.style.transition = "all 1s ease";
    alert("Keyframe marcado! Mova a imagem para criar animação.");
    earn(30);
}

earn(0);
</script>
</body>
</html>
