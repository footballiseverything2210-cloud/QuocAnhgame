# game

<!DOCTYPE html>
<html lang="vi">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kiên Săn Cục Cứt - Snake Game</title>

    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: url('https://media.discordapp.net/attachments/6e46f401-fa81-48d9-b946-f742e1be7b31.png') center/cover fixed;
            background-attachment: fixed;
            font-family: Arial, sans-serif;
        }

        body::before {
            content: '';
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.35);
            z-index: -1;
            pointer-events: none;
        }

        .container {
            text-align: center;
        }

        h1 {
            color: white;
            margin-bottom: 20px;
            font-size: 2.5em;
        }

        #gameCanvas {
            border: 5px solid white;
            background: #1a1a2e;
            display: block;
            border-radius: 10px;
        }

        .scoreboard {
            display: flex;
            justify-content: space-around;
            width: 600px;
            margin-top: 20px;
            padding: 20px;
            border-radius: 15px;
            background: rgba(255,255,255,0.15);
        }

        .score {
            font-size: 2em;
            color: gold;
        }

        .controls {
            margin-top: 20px;
            color: white;
        }

        .button-group {
            margin-top: 20px;
        }

        button {
            padding: 12px 25px;
            margin: 5px;
            cursor: pointer;
        }
    </style>
</head>

<body>

<div class="container">

    <h1>🐍 KIÊN SĂN CỤC CỨT 💩</h1>

    <canvas id="gameCanvas" width="600" height="600"></canvas>

    <div class="scoreboard">

        <div>
            <h2>Score</h2>
            <div class="score" id="score">0</div>
        </div>

        <div>
            <h2>Độ Dài</h2>
            <div class="score" id="length">1</div>
        </div>

        <div>
            <h2>Tốc Độ</h2>
            <div class="score" id="speed">Chậm</div>
        </div>

    </div>

    <div class="controls">
        <p>⬆️⬇️⬅️➡️ hoặc W/A/S/D</p>
        <p>💩 Ăn để tăng điểm</p>
        <p>🌀 Xuyên tường</p>
        <p>❌ Chỉ chết khi cắn mình</p>
    </div>

    <div class="button-group">
        <button onclick="startGame()">▶️ Start</button>
        <button onclick="resetGame()">🔄 Reset</button>
    </div>

</div>

<script>

const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

const gridSize = 20;
const tileCount = canvas.width / gridSize;

let snake = [
    {
        x: Math.floor(tileCount/2),
        y: Math.floor(tileCount/2)
    }
];

let food = {
    x: Math.floor(Math.random()*tileCount),
    y: Math.floor(Math.random()*tileCount)
};

let direction = {x:1,y:0};
let nextDirection = {x:1,y:0};

let score = 0;
let gameRunning = false;
let gameOver = false;

const baseGameSpeed = 120;

window.addEventListener("keydown",(e)=>{

    if(e.key==="ArrowUp" || e.key==="w" || e.key==="W"){
        if(direction.y===0){
            nextDirection={x:0,y:-1};
        }
    }

    else if(e.key==="ArrowDown" || e.key==="s" || e.key==="S"){
        if(direction.y===0){
            nextDirection={x:0,y:1};
        }
    }

    else if(e.key==="ArrowLeft" || e.key==="a" || e.key==="A"){
        if(direction.x===0){
            nextDirection={x:-1,y:0};
        }
    }

    else if(e.key==="ArrowRight" || e.key==="d" || e.key==="D"){
        if(direction.x===0){
            nextDirection={x:1,y:0};
        }
    }

});

function getGameSpeed(){

    const speedReduction =
        Math.min((snake.length-1)*4,80);

    return baseGameSpeed-speedReduction;
}

function updateSpeedDisplay(){

    const speed=getGameSpeed();

    let text="Chậm";

    if(speed<80){
        text="🔥 Cực nhanh";
    }

    else if(speed<100){
        text="⚡ Rất nhanh";
    }

    document.getElementById("speed").textContent=text;
}

function drawGame(){

    ctx.fillStyle="#1a1a2e";
    ctx.fillRect(
        0,
        0,
        canvas.width,
        canvas.height
    );

    snake.forEach((segment,index)=>{

        ctx.fillStyle=
            index===0
            ? "#00ff88"
            : "#00cc66";

        ctx.fillRect(
            segment.x*gridSize+1,
            segment.y*gridSize+1,
            gridSize-2,
            gridSize-2
        );

    });

    ctx.font="18px Arial";
    ctx.fillText(
        "💩",
        food.x*gridSize+10,
        food.y*gridSize+10
    );

}

function updateGame(){

    if(!gameRunning || gameOver){
        return;
    }

    direction=nextDirection;

    let newX=
        snake[0].x+
        direction.x;

    let newY=
        snake[0].y+
        direction.y;

    if(newX<0) newX=tileCount-1;
    if(newX>=tileCount) newX=0;

    if(newY<0) newY=tileCount-1;
    if(newY>=tileCount) newY=0;

    const head={
        x:newX,
        y:newY
    };

    for(let segment of snake){

        if(
            head.x===segment.x &&
            head.y===segment.y
        ){
            gameOver=true;
            return;
        }

    }

    snake.unshift(head);

    if(
        head.x===food.x &&
        head.y===food.y
    ){

        score+=10;

        document.getElementById("score")
        .textContent=score;

        document.getElementById("length")
        .textContent=snake.length;

        updateSpeedDisplay();

    }else{
        snake.pop();
    }

}

function startGame(){
    gameRunning=true;
}

function resetGame(){

    snake=[
        {
            x:Math.floor(tileCount/2),
            y:Math.floor(tileCount/2)
        }
    ];

    score=0;

    gameRunning=false;

    gameOver=false;
}

let lastUpdate=0;

function loop(timestamp){

    if(
        timestamp-lastUpdate >
        getGameSpeed()
    ){

        updateGame();

        lastUpdate=timestamp;
    }

    drawGame();

    requestAnimationFrame(loop);

}

drawGame();

requestAnimationFrame(loop);

</script>

</body>
</html>
