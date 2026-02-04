# Good-morning-2-4
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Pro Archer: Stormy Morning - Giselle ❤️</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; touch-action: none; }
        html, body { width: 100%; height: 100%; overflow: hidden; background: #000; position: fixed; font-family: -apple-system, system-ui, sans-serif; }

        #sky { position: fixed; inset: 0; background: linear-gradient(to bottom, #020111 0%, #000000 100%); transition: background 2s; z-index: -2; }
        #sun { position: absolute; bottom: -150px; left: 50%; transform: translateX(-50%); width: 140px; height: 140px; background: radial-gradient(circle, #fff700, #ff8c00); border-radius: 50%; box-shadow: 0 0 80px #ff8c00; z-index: -1; transition: bottom 2s; }
        
        #basket { 
            position: absolute; bottom: -20px; width: 120%; left: -10%; height: 25%; 
            background: #5d4037; border-top: 10px solid #3e2723; border-radius: 50% 50% 0 0; 
            z-index: 10; transition: transform 1.5s cubic-bezier(0.6, -0.28, 0.735, 0.045);
        }

        #bow-container { position: absolute; bottom: 5%; left: 50%; transform: translateX(-50%); width: 100px; height: 150px; z-index: 20; transition: transform 1.5s; }
        #bow-visual { font-size: 80px; text-align: center; pointer-events: none; }

        .target { position: absolute; z-index: 5; transition: font-size 0.3s; }
        .projectile { position: absolute; width: 4px; height: 40px; background: gold; z-index: 15; border-radius: 2px; box-shadow: 0 0 10px orange; }

        #hud { position: absolute; top: 60px; width: 100%; display: flex; justify-content: space-around; z-index: 100; color: white; font-weight: bold; text-shadow: 0 0 10px #000; }
        .screen { position: fixed; inset: 0; z-index: 200; display: flex; flex-direction: column; justify-content: center; align-items: center; text-align: center; padding: 30px; }
        #start-screen { background: rgba(0,0,0,0.9); color: white; }
        #win-screen { display: none; background: rgba(255, 255, 255, 0.98); color: #333; }
        #fail-screen { display: none; background: rgba(0,0,0,0.95); color: white; }
        .btn { padding: 15px 35px; border-radius: 50px; border: none; background: #fbc02d; font-weight: bold; margin-top: 15px; color: #000; }
    </style>
</head>
<body>

<div id="sky"></div>
<div id="sun"></div>
<div id="basket"></div>

<div id="bow-container">
    <div id="bow-visual">🏹</div>
</div>

<div id="start-screen" class="screen">
    <h1 style="color: #fbc02d;">Pro Archer Mode ❤️</h1>
    <p style="margin: 15px 0;">Sunbeams shrink as you climb!<br>Shoot 10 to clear the storm.</p>
    <button class="btn" onclick="startGame()">Fly Balloon 🎈</button>
</div>

<div id="hud">
    <div>Sunbeams: <span id="score">0</span>/10</div>
    <div>HP: <span id="lives-ui">❤️❤️❤️</span></div>
</div>

<div id="fail-screen" class="screen">
    <h1 style="color: #ff4d4d;">THE BALLOON FELL!</h1>
    <p style="margin: 15px 0;">The storm was too much...</p>
    <button class="btn" onclick="location.reload()">Try Again</button>
</div>

<div id="win-screen" class="screen">
    <h1 style="color: #f57c00;">Sunrise Secured! ☀️</h1>
    <p>Good morning, my love! I hope you have a perfect day.</p>
    <div id="timerText" style="margin: 15px 0; font-weight: bold; color: #f57c00;"></div>
    <button class="btn" onclick="showLetter()" id="letterBtn">Open My Letter 💌</button>
    <div id="secretLetter" style="display:none; padding: 15px; font-style: italic;">"Giselle, you are beautiful and perfect. I love you!"</div>
</div>

<audio id="music" loop><source src="https://www.bensound.com/bensound-music/bensound-tomorrow.mp3" type="audio/mpeg"></audio>

<script>
    let score = 0, lives = 3, gameActive = false, touchStartY = 0;
    const bowContainer = document.getElementById('bow-container');
    const basket = document.getElementById('basket');
    const sky = document.getElementById('sky');
    const sun = document.getElementById('sun');

    function startGame() {
        document.getElementById('start-screen').style.display = 'none';
        gameActive = true;
        updateCountdown();
        spawnLoop();
    }

    bowContainer.addEventListener('touchstart', (e) => { if (!gameActive) return; touchStartY = e.touches[0].clientY; });
    bowContainer.addEventListener('touchend', (e) => {
        if (!gameActive) return;
        const touchEndY = e.changedTouches[0].clientY;
        if (touchEndY > touchStartY + 25) fireArrow();
    });

    function fireArrow() {
        const proj = document.createElement('div');
        proj.className = 'projectile';
        proj.style.left = '50%';
        proj.style.bottom = '15%';
        document.body.appendChild(proj);
        let projPos = 15;
        const fly = setInterval(() => {
            projPos += 8;
            proj.style.bottom = projPos + '%';
            checkCollision(proj, fly);
            if (projPos > 110) { clearInterval(fly); proj.remove(); }
        }, 10);
    }

    function spawnItem() {
        if (!gameActive) return;
        const item = document.createElement('div');
        item.className = 'target';
        const rand = Math.random();
        
        let type = 'rain';
        if (rand > 0.82) type = 'sun';
        else if (rand > 0.6) type = 'cloud';

        item.innerHTML = (type === 'sun') ? '☀️' : (type === 'cloud' ? '☁️' : '💧');
        item.dataset.type = type;
        
        // Shrink the sunbeams as score increases
        const sunSize = type === 'sun' ? Math.max(25, 50 - (score * 3)) : 45;
        item.style.fontSize = sunSize + 'px';
        
        let xPos = Math.random() * 80 + 10;
        item.style.left = xPos + '%';
        item.style.top = '-50px';
        document.body.appendChild(item);

        let yPos = -50;
        const fall = setInterval(() => {
            yPos += (type === 'sun' ? 3 : 6);
            item.style.top = yPos + 'px';
            if (yPos > window.innerHeight) { clearInterval(fall); item.remove(); }
        }, 20);
        item.dataset.intervalId = fall;
    }

    function checkCollision(proj, flyInterval) {
        const targets = document.querySelectorAll('.target');
        const pRect = proj.getBoundingClientRect();
        targets.forEach(t => {
            const tRect = t.getBoundingClientRect();
            if (pRect.left < tRect.right && pRect.right > tRect.left && pRect.top < tRect.bottom && pRect.bottom > tRect.top) {
                if (t.dataset.type === 'sun') {
                    score++;
                    updateEnv();
                } else {
                    lives--;
                    updateLivesUI();
                    if (lives <= 0) gameFail();
                }
                document.getElementById('score').innerText = score;
                clearInterval(t.dataset.intervalId);
                t.remove(); clearInterval(flyInterval); proj.remove();
                if (score >= 10) win();
            }
        });
    }

    function updateLivesUI() {
        document.getElementById('lives-ui').innerText = "❤️".repeat(Math.max(0, lives));
    }

    function updateEnv() {
        const colors = ['#020111', '#1a1a40', '#4d2c5e', '#a34d5d', '#ff9a44', '#ffcf71'];
        sky.style.background = colors[Math.floor(score/2)] || '#ffcf71';
        sun.style.bottom = (-150 + (score * 35)) + 'px';
    }

    function gameFail() {
        gameActive = false;
        basket.style.transform = "translateY(600px) rotate(20deg)";
        bowContainer.style.transform = "translateY(600px)";
        setTimeout(() => { document.getElementById('fail-screen').style.display = 'flex'; }, 800);
    }

    function win() {
        gameActive = false;
        document.getElementById('win-screen').style.display = 'flex';
        document.querySelectorAll('.target').forEach(t => t.remove());
    }

    function showLetter() {
        document.getElementById('secretLetter').style.display = 'block';
        document.getElementById('letterBtn').style.display = 'none';
    }

    function updateCountdown() {
        const trip = new Date("February 14, 2026 00:00:00").getTime();
        const diff = trip - new Date().getTime();
        const d = Math.floor(diff / (1000*60*60*24));
        const h = Math.floor((diff % (1000*60*60*24)) / (1000*60*60));
        document.getElementById('timerText').innerText = `Seattle Trip: ${d}d ${h}h Left!`;
    }

    function spawnLoop() { 
        if(gameActive && score < 10) { 
            spawnItem(); 
            setTimeout(() => { if(gameActive) spawnItem(); }, 200);
            setTimeout(spawnLoop, 750); 
        } 
    }
</script>
</body>
</html>
