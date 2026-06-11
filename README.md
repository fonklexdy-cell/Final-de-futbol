
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ken: Final en el Estadio</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Fredoka:wght@400;600&display=swap');
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Fredoka', sans-serif; }
        body { background-color: #E0FFFF; display: flex; justify-content: center; align-items: center; min-height: 100vh; padding: 15px; }
        .contenedor { background: white; width: 100%; max-width: 500px; border-radius: 30px; padding: 25px; box-shadow: 0 15px 35px rgba(0,0,0,0.1); border: 4px solid #00CED1; text-align: center; }
        .avatar { font-size: 80px; margin: 15px 0; animation: bounce 1.5s infinite; }
        @keyframes bounce { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-15px); } }
        .stats { display: flex; justify-content: space-around; margin-bottom: 10px; font-weight: 600; color: #008B8B; }
        .progreso { height: 10px; background: #eee; border-radius: 5px; margin-bottom: 20px; }
        .barra { height: 100%; background: #40E0D0; width: 0%; transition: 0.4s; }
        .pregunta { font-size: 20px; font-weight: 600; margin-bottom: 20px; color: #333; }
        .opcion { width: 100%; padding: 15px; margin-bottom: 10px; border: 2px solid #afeeee; border-radius: 15px; background: #fcfcfc; cursor: pointer; font-size: 18px; font-weight: 600; color: #008B8B; transition: 0.2s; }
        .opcion:hover { background: #e0ffff; transform: scale(1.02); }
        .msg-error { color: #d32f2f; font-weight: bold; margin-bottom: 10px; height: 20px; }
        .btn-inicio { width: 100%; padding: 18px; border: none; border-radius: 15px; background: #2e7d32; color: white; font-size: 20px; cursor: pointer; }
    </style>
</head>
<body>

<div class="contenedor">
    <div id="inicio">
        <h1>⚽ Final de Fútbol</h1>
        <div class="avatar">😎</div>
        <p>¡Ayuda a Ken a comprar todo en el Estadio Nacional!</p><br>
        <button class="btn-inicio" onclick="iniciarTodo()">Empezar Juego</button>
    </div>

    <div id="juego" style="display:none;">
        <div class="stats"><span>⏱️ <span id="timer">0</span>s</span><span>🏟️ <span id="n">1</span>/8</span></div>
        <div class="progreso"><div class="barra" id="barra"></div></div>
        <div class="avatar" id="emoji">⚽</div>
        <div class="msg-error" id="msg"></div>
        <div class="pregunta" id="pregunta"></div>
        <div id="opciones"></div>
    </div>

    <div id="final" style="display:none;">
        <div class="avatar" id="final-emoji">🏆</div>
        <h2 id="rango"></h2>
        <div style="font-size: 22px; margin: 20px 0;" id="puntaje"></div>
        <button class="btn-inicio" onclick="location.reload()">Jugar de nuevo</button>
    </div>
</div>

<script>
    let ctx, indice, aciertos, intentos, timer, juegoActual;
    
    const banco = [
        { q: "Vigorón en hoja a C$120. ¿2 platos?", r: 240, e: "🌿" },
        { q: "Refresco de cacao a C$30. ¿3 vasos?", r: 90, e: "🥤" },
        { q: "Bolsa de semillas a C$25. ¿4 bolsas?", r: 100, e: "🥜" },
        { q: "Entrada general C$150. ¿2 personas?", r: 300, e: "🎟️" },
        { q: "Tiste helado a C$40. ¿2 vasos?", r: 80, e: "🍹" },
        { q: "Gorra del equipo C$200. Pagas C$300. ¿Vuelto?", r: 100, e: "🧢" },
        { q: "Plato de carne asada C$250. ¿2 platos?", r: 500, e: "🥩" },
        { q: "4 paletas a C$15 c/u. ¿Total?", r: 60, e: "🍦" },
        { q: "Pipa helada a C$20. ¿3 pipas?", r: 60, e: "🥥" },
        { q: "Churro C$15. ¿5 churros?", r: 75, e: "🥨" }
    ];

    function playTone(f, t, d, v=0.3, del=0) {
        if (!ctx) return;
        const osc = ctx.createOscillator(), g = ctx.createGain();
        osc.connect(g); g.connect(ctx.destination);
        osc.type = t; osc.frequency.setValueAtTime(f, ctx.currentTime + del);
        g.gain.setValueAtTime(v, ctx.currentTime + del);
        g.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + del + d);
        osc.start(ctx.currentTime + del); osc.stop(ctx.currentTime + del + d);
    }

    function iniciarTodo() {
        ctx = new (window.AudioContext || window.webkitAudioContext)();
        indice = 0; aciertos = 0; intentos = 0;
        // Mezcla el banco completo cada vez que reinicias
        juegoActual = [...banco].sort(() => Math.random() - 0.5);
        document.getElementById('inicio').style.display = 'none';
        document.getElementById('juego').style.display = 'block';
        timer = setInterval(() => document.getElementById('timer').innerText++, 1000);
        mostrar();
    }

    function mostrar() {
        const p = juegoActual[indice];
        document.getElementById('n').innerText = indice + 1;
        document.getElementById('barra').style.width = (indice/8*100) + '%';
        document.getElementById('emoji').innerText = p.e;
        document.getElementById('pregunta').innerText = p.q;
        const box = document.getElementById('opciones');
        box.innerHTML = '';
        
        // Mezcla opciones para que el correcto no siempre esté igual
        [p.r, p.r+50, p.r-30].sort(() => Math.random()-0.5).forEach(o => {
            const b = document.createElement('button');
            b.className = 'opcion'; b.innerText = "C$ " + o;
            b.onclick = () => {
                intentos++;
                if(o === p.r) { 
                    aciertos++; playTone(500,'sine',0.2); 
                    document.getElementById('msg').innerText = "";
                    indice++; indice < 8 ? mostrar() : finalizar();
                } else { 
                    playTone(200,'sawtooth',0.3); 
                    document.getElementById('msg').innerText = "¡Vuelve a intentarlo!";
                }
            };
            box.appendChild(b);
        });
    }

    function finalizar() {
        clearInterval(timer);
        document.getElementById('juego').style.display = 'none';
        document.getElementById('final').style.display = 'block';
        const pct = Math.round((aciertos / intentos) * 100);
        document.getElementById('puntaje').innerText = `Tu precisión: ${pct}% (${aciertos} de ${intentos} intentos)`;
        document.getElementById('rango').innerText = pct >= 80 ? "¡Eres un Crack, sigue triunfando! 🏆" : "Debes mejorar, ¡tú puedes! ⚽";
        [400, 500, 600, 800, 1000].forEach((f,i) => playTone(f, 'sine', 0.5, 0.3, i*0.2));
    }
</script>
</body>
</html>
