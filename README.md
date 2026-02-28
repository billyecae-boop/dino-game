<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Dino Game Pro</title>

<style>
body{
    margin:0;
    overflow:hidden;
    background:#111;
    display:flex;
    justify-content:center;
    align-items:center;
    height:100vh;
}

canvas{
    background:white;
    max-width:100%;
    height:auto;
}
</style>
</head>

<body>

<canvas id="game"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

// ======================
// RESOLUÇÃO BASE
// ======================
const BASE_WIDTH = 900;
const BASE_HEIGHT = 300;

canvas.width = BASE_WIDTH;
canvas.height = BASE_HEIGHT;

// ======================
let gravity = 0.6;
let score = 0;
let highScore = localStorage.getItem("dinoHighScore") || 0;
let gameState = "menu";
let timeCycle = 0;

const dino = {
    x: 80,
    y: 200,
    width: 50,
    height: 50,
    vy: 0,
    jumping:false,
    frame:0
};

let cacti = [];
let spawnTimer = 0;
let groundOffset = 0;

let clouds = [
    {x:200,y:60},
    {x:500,y:80},
    {x:750,y:50}
];

let stars = [];
for(let i=0;i<40;i++){
    stars.push({
        x:Math.random()*900,
        y:Math.random()*150,
        size:Math.random()*2
    });
}

// ======================
// CONTROLES DESKTOP
// ======================
document.addEventListener("keydown", e=>{

    if(e.code==="Space"){
        handleAction();
    }

    if(gameState==="gameover" && e.code==="KeyR"){
        restart();
    }
});

// ======================
// CONTROLE MOBILE
// ======================
canvas.addEventListener("touchstart", e=>{
    e.preventDefault();
    handleAction();
});

// ======================
function handleAction(){

    if(gameState==="menu"){
        gameState="playing";
        return;
    }

    if(gameState==="playing" && !dino.jumping){
        dino.vy = -13;
        dino.jumping = true;
        return;
    }

    if(gameState==="gameover"){
        restart();
    }
}

// ======================
function spawnCactus(){
    let height = 30 + Math.random()*40;
    let width = 15 + Math.random()*20;

    cacti.push({
        x: 920,
        y: 250 - height,
        width: width,
        height: height
    });

    if(score > 10 && Math.random() < 0.3){
        cacti.push({
            x: 960,
            y: 250 - height,
            width: width,
            height: height
        });
    }
}

// ======================
function drawDino(){
    ctx.fillStyle="#2ecc71";
    ctx.fillRect(dino.x,dino.y,40,40);
    ctx.fillRect(dino.x+30,dino.y-10,25,25);
    ctx.fillStyle="black";
    ctx.fillRect(dino.x+45,dino.y-3,6,6);

    ctx.fillStyle="#27ae60";
    if(dino.frame%20 < 10){
        ctx.fillRect(dino.x+5,dino.y+40,8,15);
        ctx.fillRect(dino.x+20,dino.y+40,8,5);
    } else {
        ctx.fillRect(dino.x+5,dino.y+40,8,5);
        ctx.fillRect(dino.x+20,dino.y+40,8,15);
    }
}

function drawCactus(c){
    ctx.fillStyle="#145a32";
    ctx.fillRect(c.x,c.y,c.width,c.height);
}

function drawCloud(x,y){
    ctx.fillStyle="#ddd";
    ctx.beginPath();
    ctx.arc(x,y,15,0,Math.PI*2);
    ctx.arc(x+20,y+5,15,0,Math.PI*2);
    ctx.arc(x+40,y,15,0,Math.PI*2);
    ctx.fill();
}

// ======================
function update(){

    if(gameState!=="playing") return;

    dino.frame++;
    timeCycle += 0.002;

    let difficultySpeed = 6 + (score * 0.15);

    dino.y += dino.vy;
    dino.vy += gravity;

    if(dino.y>=200){
        dino.y=200;
        dino.jumping=false;
    }

    spawnTimer--;
    if(spawnTimer <= 0){
        spawnCactus();

        let minTime = 60 - score;
        if(minTime < 25) minTime = 25;

        spawnTimer = minTime + Math.random()*40;
    }

    for(let i=cacti.length-1;i>=0;i--){
        let c = cacti[i];
        c.x -= difficultySpeed;

        if(
            dino.x < c.x + c.width &&
            dino.x + dino.width > c.x &&
            dino.y < c.y + c.height &&
            dino.y + dino.height > c.y
        ){
            gameState="gameover";
        }

        if(c.x < -50){
            cacti.splice(i,1);
            score++;

            if(score > highScore){
                highScore = score;
                localStorage.setItem("dinoHighScore", highScore);
            }
        }
    }

    groundOffset -= difficultySpeed;
    if(groundOffset <= -40) groundOffset = 0;

    clouds.forEach(c=>{
        c.x -=2;
        if(c.x<-50) c.x=900;
    });
}

// ======================
function draw(){

    let skyValue = Math.sin(timeCycle) * 0.5 + 0.5;
    let skyColor = `rgb(${50 + skyValue*150},
                        ${100 + skyValue*120},
                        ${200})`;

    ctx.fillStyle = skyColor;
    ctx.fillRect(0,0,900,300);

    if(skyValue < 0.3){
        ctx.fillStyle="white";
        stars.forEach(s=>{
            ctx.fillRect(s.x,s.y,s.size,s.size);
        });
    }

    clouds.forEach(c=>drawCloud(c.x,c.y));

    ctx.strokeStyle="black";
    ctx.beginPath();
    ctx.moveTo(0,250);
    ctx.lineTo(900,250);
    ctx.stroke();

    for(let i=0;i<900;i+=40){
        ctx.fillRect(i+groundOffset,250,20,5);
    }

    if(gameState==="menu"){
        ctx.fillStyle="black";
        ctx.font="40px Arial";
        ctx.fillText("DINO RUN PRO",300,120);
        ctx.font="20px Arial";
        ctx.fillText("Toque ou Espaço para começar",250,170);
        ctx.fillText("Recorde: "+highScore,370,210);
        return;
    }

    drawDino();
    cacti.forEach(c=>drawCactus(c));

    ctx.fillStyle="black";
    ctx.font="20px Arial";
    ctx.fillText("Score: "+score,10,25);
    ctx.fillText("Recorde: "+highScore,750,25);

    if(gameState==="gameover"){
        ctx.font="30px Arial";
        ctx.fillText("GAME OVER",350,120);
        ctx.font="20px Arial";
        ctx.fillText("Toque ou pressione R",330,160);
    }
}

// ======================
function loop(){
    update();
    draw();
    requestAnimationFrame(loop);
}

function restart(){
    score=0;
    dino.y=200;
    dino.vy=0;
    cacti=[];
    spawnTimer=0;
    gameState="menu";
}

loop();
</script>

</body>
</html>
