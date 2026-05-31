<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, maximum-scale=1.0, minimum-scale=1.0">
    <title>크롬 공룡 게임 만들기</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 100vh;
            width: 100vw;
            background-color: #202124 !important; /* 다크모드 배경 고정 */
            font-family: Arial, sans-serif;
            overflow: hidden;
            /* 🛠️ 모바일 터치 시 푸른색 선택 박스 현상 및 꾹 누를 때 팝업창 방지 */
            -webkit-touch-callout: none;
            -webkit-user-select: none;
            user-select: none;
        }
        #game-container {
            position: relative;
            width: 100%;
            height: 100%;
            max-width: 100vw;
            max-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        #score {
            position: absolute;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            font-size: 24px;
            font-weight: bold;
            color: #ffffff !important;
            z-index: 5;
        }
        #skill-hud {
            position: absolute;
            top: 60px;
            left: 50%;
            transform: translateX(-50%);
            font-size: 16px;
            font-weight: bold;
            color: #ff9900;
            background: rgba(0, 0, 0, 0.6);
            padding: 5px 15px;
            border-radius: 15px;
            z-index: 5;
            display: none;
        }
        #gameCanvas {
            border: 2px solid #535353;
            background-color: #f7f7f7 !important;
            box-shadow: 0px 4px 10px rgba(0,0,0,0.5);
            max-width: 100%;
            max-height: 100%;
            object-fit: contain;
        }
        #menu-ui {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: rgba(32, 33, 36, 0.9);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            border: 2px solid #535353;
            box-sizing: border-box;
            z-index: 10;
        }
        #menu-title {
            color: #ffffff;
            font-size: 36px;
            font-weight: bold;
            margin-bottom: 10px;
            letter-spacing: 2px;
            text-align: center;
        }
        #menu-subtitle {
            color: #aaa;
            font-size: 14px;
            margin-bottom: 25px;
            text-align: center;
            padding: 0 10px;
            line-height: 1.4;
        }
        .volume-group {
            display: flex;
            flex-direction: column;
            gap: 10px;
            margin-bottom: 30px;
        }
        .volume-container {
            display: flex;
            align-items: center;
            color: #ffffff;
            font-size: 14px;
            background: rgba(250, 250, 250, 0.1);
            padding: 8px 15px;
            border-radius: 20px;
        }
        .volume-container label {
            width: 70px; /* 라벨 너비 고정으로 줄맞춤 */
            font-weight: bold;
            color: #aaa;
        }
        .volume-slider {
            -webkit-appearance: none;
            width: 120px;
            height: 6px;
            background: #535353;
            border-radius: 3px;
            outline: none;
            cursor: pointer;
        }
        .volume-slider::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 16px;
            height: 16px;
            background: #8a2be2;
            border-radius: 50%;
            cursor: pointer;
            transition: transform 0.1s;
        }
        .volume-slider::-webkit-slider-thumb:hover {
            transform: scale(1.2);
        }
        .volume-text {
            margin-left: 10px;
            min-width: 35px;
            text-align: right;
            font-weight: bold;
        }
        #start-button {
            padding: 12px 35px;
            font-size: 18px;
            font-weight: bold;
            color: #202124;
            background-color: #8a2be2;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: all 0.2s ease;
            box-shadow: 0 4px 6px rgba(0,0,0,0.3);
        }
        #start-button:hover {
            background-color: #a044ff;
            transform: scale(1.05);
        }
        #start-button:active {
            transform: scale(0.95);
        }
    </style>
</head>
<body>
    <div id="score">Score: 0</div>
    <div id="skill-hud">⚡ SKILL: READY (Shift)</div>
    <div id="game-container">
        <div id="menu-ui">
            <div id="menu-title">KIM3SIU RUNNER</div>
            <div id="menu-subtitle">PC: Space (Jump) / Shift (Dash)<br>Mobile: Touch Left (Dash) / Touch Right (Jump)</div>
            <div class="volume-group">
                <div class="volume-container">
                    <label for="volume-bgm">🔊 BGM</label>
                    <input type="range" id="volume-bgm" class="volume-slider" min="0" max="100" value="50" oninput="updateBGMVolume(this.value)">
                    <span id="volume-bgm-text" class="volume-text">50%</span>
                </div>
                <div class="volume-container">
                    <label for="volume-sfx">🎵 SFX</label>
                    <input type="range" id="volume-sfx" class="volume-slider" min="0" max="100" value="50" oninput="updateSFXVolume(this.value)">
                    <span id="volume-sfx-text" class="volume-text">50%</span>
                </div>
            </div>
            <button id="start-button" onclick="startGame()">GAME START</button>
        </div>  
        <canvas id="gameCanvas" width="800" height="300"></canvas>
    </div>
    <audio id="bgm" src="audio/temp_1780239567828.-1448785110.m4a" loop></audio>
    <audio id="jump-sfx" src="audio/헤이.m4a"></audio>
    <audio id="dash-sfx" src="audio/오빠달린다.m4a"></audio>

<script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const scoreElement = document.getElementById("score");
    const skillHud = document.getElementById("skill-hud");
    const bgm = document.getElementById("bgm");
    const jumpSfx = document.getElementById("jump-sfx");
    const dashSfx = document.getElementById("dash-sfx"); 
    const menuUi = document.getElementById("menu-ui");
    const menuSubtitle = document.getElementById("menu-subtitle");
    const startButton = document.getElementById("start-button");
    
    const volumeBgmText = document.getElementById("volume-bgm-text");
    const volumeSfxText = document.getElementById("volume-sfx-text");

    let score = 0;
    let isGameOver = false;
    let gameSpeed = 4; 
    let isGameStarted = false; 

    // 배경 이미지 설정
    const bgImage = new Image();
    bgImage.src = 'img/20251012_150702.webp'; 
    
    let bgX = 0;
    let bgScrollSpeed = 1; 

    // 공룡 캐릭터 이미지 선언 및 경로 설정
    const dinoNormalImg = new Image();
    dinoNormalImg.src = 'img/20231031_1322471.png'; 

    const dinoDoubleImg = new Image();
    dinoDoubleImg.src = 'img/20231031_1338271.png'; 

    // 대시 스킬 사용 시 출력될 이미지 선언 및 경로 설정
    const dinoDashImg = new Image();
    dinoDashImg.src = 'img/MEITU_20260531_2330200381.png'; 

    // 장애물 이미지 선언 및 경로 설정
    const obsNormalImg = new Image();
    obsNormalImg.src = 'img/MEITU_20260531_2314035391.png'; 

    const obsHighImg = new Image();
    obsHighImg.src = 'img/20240516_1045241.png';

    // 게임오버 전용 이미지 선언 및 경로 설정
    const gameOverImg = new Image();
    gameOverImg.src = 'img/1762503875333.jpg'; 

    // 초기 개별 볼륨 세팅 (50%)
    bgm.volume = 0.5;
    jumpSfx.volume = 0.5;
    dashSfx.volume = 0.5; 

    // BGM 볼륨 전용 업데이트 함수
    function updateBGMVolume(val) {
        bgm.volume = val / 100; 
        volumeBgmText.innerText = val + "%";
    }

    // 효과음(SFX) 볼륨 전용 업데이트 함수
    function updateSFXVolume(val) {
        let volumeRatio = val / 100;
        jumpSfx.volume = volumeRatio; 
        dashSfx.volume = volumeRatio; 
        volumeSfxText.innerText = val + "%";
    }

    function resizeGame() {
        const canvasClientRect = canvas.getBoundingClientRect();
        menuUi.style.width = canvasClientRect.width + "px";
        menuUi.style.height = canvasClientRect.height + "px";
    }
    window.addEventListener("resize", resizeGame);

    // 공룡 데이터
    const dino = {
        baseX: 50,       
        x: 50,
        y: 230, 
        width: 45,        
        height: 50,
        vy: 0,
        gravity: 0.6, 
        jumpPower: -11.5,       
        doubleJumpPower: -9.5,  
        jumpCount: 0,          
        maxJumps: 2,           
        
        isDashing: false,      
        dashTimer: 0,          
        dashDuration: 24,      
        canDash: true,         
        dashCooldown: 180,     
        cooldownTimer: 0,      

        draw() {
            if (this.isDashing) {
                if (dinoDashImg.complete) {
                    ctx.drawImage(dinoDashImg, this.x, this.y, this.width, this.height);
                } else {
                    ctx.fillStyle = "#ff9900"; 
                    ctx.fillRect(this.x, this.y, this.width, this.height);
                }
            } else {
                let currentImg = this.jumpCount === 2 ? dinoDoubleImg : dinoNormalImg;
                
                if (currentImg.complete) {
                    ctx.drawImage(currentImg, this.x, this.y, this.width, this.height);
                } else {
                    ctx.fillStyle = this.jumpCount === 2 ? "#a044ff" : "#535353"; 
                    ctx.fillRect(this.x, this.y, this.width, this.height);
                }
            }
        }
    };

    // 장애물 배열
    const obstacles = [];

    // 장애물 클래스
    class Obstacle {
        constructor(type) {
            this.x = canvas.width;
            this.type = type; 

            if (this.type === 'high') {
                this.width = 40;   
                this.height = 115; 
                this.y = 165;      
                this.color = "#d9534f"; 
            } else {
                this.width = 25;
                this.height = 40;
                this.y = 240;
                this.color = "#8a2be2";
            }
        }
        draw() {
            let currentObsImg = this.type === 'high' ? obsHighImg : obsNormalImg;

            if (currentObsImg.complete) {
                ctx.drawImage(currentObsImg, this.x, this.y, this.width, this.height);
            } else {
                ctx.fillStyle = "#8a2be2"; 
                ctx.fillRect(this.x, this.y, this.width, this.height);
            }

            if (this.type === 'high') {
                ctx.fillStyle = "#ffffff";
                ctx.font = "bold 14px Arial";
                ctx.fillText("!!", this.x + (this.width / 2) - 5, this.y - 8);
            }
        }
        update() {
            this.x -= gameSpeed;
            this.draw();
        }
    }

    function playJumpSound() {
        jumpSfx.currentTime = 0; 
        jumpSfx.play().catch(error => console.log("효과음 재생 차단 우회 중...", error));
    }

    function playDashSound() {
        dashSfx.currentTime = 0; 
        dashSfx.play().catch(error => console.log("대시 효과음 재생 차단 우회 중...", error));
    }

    // 🛠️ 중복 조작을 방지하고 점프 로직을 통일하기 위한 핵심 함수
    function handleJumpAction() {
        if (!dino.isDashing && dino.jumpCount < dino.maxJumps) {
            dino.jumpCount++;
            if (dino.jumpCount === 1) {
                dino.vy = dino.jumpPower;
                playJumpSound(); 
            } else if (dino.jumpCount === 2) {
                dino.vy = dino.doubleJumpPower;
                playJumpSound(); 
            }
        }
    }

    // 키보드 조작 (기존 로직에서 함수 호출 형태로 깔끔하게 변경)
    window.addEventListener("keydown", (e) => {
        if (isGameStarted && !isGameOver) {
            if (e.code === "Space" || e.code === "ArrowUp") {
                handleJumpAction();
            }
            if ((e.code === "ShiftLeft" || e.code === "ShiftRight" || e.code === "ArrowRight") && dino.canDash) {
                triggerDash();
            }
        }
        
        if (isGameOver) {
            returnToMenu();
        }
    });

    // 🛠️ [모바일 추가] 화면 터치 조작 이벤트 리스너
    window.addEventListener("touchstart", (e) => {
        // 게임오버일 땐 어디를 터치하든 메뉴로 복귀
        if (isGameOver) {
            returnToMenu();
            return;
        }

        // 게임 도중일 때 터치 조작 처리
        if (isGameStarted && !isGameOver) {
            // 클릭(터치)된 손가락의 X 좌표
            const touchX = e.touches[0].clientX;
            // 현재 스마트폰 전체 화면의 가로 너비
            const screenWidth = window.innerWidth;

            if (touchX < screenWidth / 2) {
                // 1. 화면 기준 [왼쪽 절반] 터치 시 -> 대시 발동!
                if (dino.canDash) {
                    triggerDash();
                }
            } else {
                // 2. 화면 기준 [오른쪽 절반] 터치 시 -> 점프 발동!
                handleJumpAction();
            }
        }
    }, { passive: true }); // 모바일 터치 스크롤 성능 저하 방지


    function triggerDash() {
        dino.isDashing = true;
        dino.canDash = false;
        dino.dashTimer = dino.dashDuration;
        dino.cooldownTimer = dino.dashCooldown;
        dino.vy = 0; 
        playDashSound(); 
    }

    function updateSkillLogic() {
        if (dino.isDashing) {
            dino.dashTimer--;
            dino.x = dino.baseX + 60; 
            
            if (dino.dashTimer <= 0) {
                dino.isDashing = false;
            }
        } else {
            if (dino.x > dino.baseX) {
                dino.x -= 3; 
                if(dino.x < dino.baseX) dino.x = dino.baseX;
            }
        }

        if (!dino.canDash) {
            dino.cooldownTimer--;
            let remainingSec = (dino.cooldownTimer / 60).toFixed(1);
            skillHud.innerText = `⏳ COOLDOWN: ${remainingSec}s`;
            skillHud.style.color = "#ff4444";

            if (dino.cooldownTimer <= 0) {
                dino.canDash = true;
                skillHud.innerText = "⚡ SKILL: READY (Shift)";
                skillHud.style.color = "#00ffcc";
            }
        }
    }

    function checkCollision(rect1, rect2) {
        return (
            rect1.x < rect2.x + rect2.width &&
            rect1.x + rect1.width > rect2.x &&
            rect1.y < rect2.y + rect2.height &&
            rect1.y + rect1.height > rect2.y
        );
    }

    let spawnTimer = 0;

    function startGame() {
        score = 0;
        scoreElement.innerText = "Score: 0";
        isGameOver = false;
        gameSpeed = 4;
        bgX = 0; 
        obstacles.length = 0;
        
        dino.x = dino.baseX;
        dino.y = 230;
        dino.vy = 0;
        dino.jumpCount = 0; 
        dino.isDashing = false;
        dino.canDash = true;
        dino.cooldownTimer = 0;

        isGameStarted = true;
        menuUi.style.display = "none"; 
        skillHud.style.display = "block";
        skillHud.innerText = "⚡ SKILL: READY (Shift)";
        skillHud.style.color = "#00ffcc";
        
        bgm.currentTime = 0;
        bgm.play().catch(error => console.log("오디오 재생 차단 우회 중...", error));
        animate();
    }

    function drawBackground() {
        if (bgImage.complete) {
            ctx.drawImage(bgImage, bgX, 0, canvas.width, canvas.height);
            ctx.drawImage(bgImage, bgX + canvas.width, 0, canvas.width, canvas.height);
            
            if (isGameStarted && !isGameOver) {
                let currentScroll = dino.isDashing ? bgScrollSpeed * 3 : bgScrollSpeed;
                bgX -= currentScroll;
                if (bgX <= -canvas.width) {
                    bgX = 0;
                }
            }
        }
    }

    function animate() {
        if (isGameOver || !isGameStarted) return;

        requestAnimationFrame(animate);
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        updateSkillLogic();
        drawBackground();

        ctx.strokeStyle = "#535353";
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.moveTo(0, 280);
        ctx.lineTo(canvas.width, 280);
        ctx.stroke();

        score++;
        scoreElement.innerText = "Score: " + score;
        if (score % 600 === 0) {
            gameSpeed += 0.5;
            bgScrollSpeed += 0.1; 
        }

        if (!dino.isDashing) {
            dino.vy += dino.gravity;
            dino.y += dino.vy;
        }

        if (dino.y > 230) {
            dino.y = 230;
            dino.vy = 0;
            dino.jumpCount = 0; 
        }
        dino.draw();

        spawnTimer++;
        if (spawnTimer > 50) { 
            if (Math.random() > 0.5) {
                let obstacleType = 'normal';
                if (Math.random() < 0.2) { 
                    obstacleType = 'high';
                }
                obstacles.push(new Obstacle(obstacleType));
            }
            spawnTimer = 0;
        }

        obstacles.forEach((obs, index) => {
            let currentObsSpeed = dino.isDashing ? gameSpeed + 6 : gameSpeed;
            obs.x -= currentObsSpeed;
            obs.draw();

            if (obs.x + obs.width < 0) {
                obstacles.splice(index, 1);
            }

            if (checkCollision(dino, obs)) {
                if (dino.isDashing) {
                    obstacles.splice(index, 1); 
                    score += 100; 
                    return;
                }

                isGameOver = true;
                bgm.pause(); 

                if (gameOverImg.complete) {
                    ctx.drawImage(gameOverImg, 0, 0, canvas.width, canvas.height);
                } else {
                    ctx.fillStyle = "#d9534f"; 
                    ctx.font = "bold 30px Arial";
                    ctx.fillText("GAME OVER", canvas.width / 2 - 90, canvas.height / 2 - 10);
                }

                ctx.fillStyle = "#ffffff";
                ctx.font = "bold 20px Arial";
                ctx.shadowColor = "black";
                ctx.shadowBlur = 4;
                ctx.fillText("Press Any Key or Touch Screen to Return Menu", canvas.width / 2 - 200, canvas.height / 2 + 50);
                ctx.shadowBlur = 0;
            }
        });
    }

    function returnToMenu() {
        isGameStarted = false;
        isGameOver = false;
        skillHud.style.display = "none"; 
        
        menuSubtitle.innerHTML = "PC: Space (Jump) / Shift (Dash)<br>Mobile: Touch Left (Dash) / Touch Right (Jump)";
        startButton.innerText = "RESTART";
        menuUi.style.display = "flex";
        resizeGame(); 

        ctx.clearRect(0, 0, canvas.width, canvas.height);
        drawBackground(); 
        ctx.strokeStyle = "#535353";
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.moveTo(0, 280);
        ctx.lineTo(canvas.width, 280);
        ctx.stroke();
        dino.draw();
    }

    window.onload = function() {
        resizeGame(); 
        if(!isGameStarted) {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            drawBackground();
            ctx.strokeStyle = "#535353";
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo(0, 280);
            ctx.lineTo(canvas.width, 280);
            ctx.stroke();
            dino.draw();
        }
    };

</script>
</body>
</html>
