<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Math Snake Game</title>
<style>
  body, html {
    margin: 0;
    padding: 0;
    background: #111;
    color: #fff;
    font-family: Arial, sans-serif;
    overflow: hidden;
    touch-action: none;
  }
  canvas {
    display: block;
    background: #222;
    margin: auto;
    position: absolute;
    top: 0; bottom: 0; left: 0; right: 0;
  }
  #questionContainer, #restartContainer {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background: #001f4d;
    padding: 25px 40px;
    border-radius: 15px;
    text-align: center;
    font-size: 24px;
    box-shadow: 0 0 20px #000;
  }
  #questionContainer { display: none; }
  #restartContainer { display: none; }
  .option, #restartButton {
    display: block;
    margin: 15px 0;
    padding: 12px 20px;
    background: #003366;
    border: none;
    border-radius: 8px;
    color: white;
    cursor: pointer;
    font-size: 20px;
    transition: 0.2s;
  }
  .option:hover, #restartButton:hover {
    background: #0055aa;
  }
  #mobileControls {
    position: absolute;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 20px;
  }
  .controlBtn {
    width: 60px;
    height: 60px;
    border-radius: 50%;
    border: none;
    background: #003366;
    color: white;
    font-size: 24px;
    touch-action: none;
  }
  .controlBtn:active {
    background: #0055aa;
  }
</style>
</head>
<body>

<canvas id="gameCanvas"></canvas>

<div id="questionContainer">
  <div id="questionText"></div>
  <button class="option" id="opt1"></button>
  <button class="option" id="opt2"></button>
  <button class="option" id="opt3"></button>
  <button class="option" id="opt4"></button>
</div>

<div id="restartContainer">
  <div id="correctAnswerText"></div>
  <button id="restartButton">Restart Game</button>
</div>

<div id="mobileControls">
  <button class="controlBtn" id="upBtn">↑</button>
  <button class="controlBtn" id="leftBtn">←</button>
  <button class="controlBtn" id="downBtn">↓</button>
  <button class="controlBtn" id="rightBtn">→</button>
</div>

<audio id="bgMusic" loop>
  <source src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_20db1c8f76.mp3?filename=chill-gaming-112186.mp3" type="audio/mpeg">
</audio>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
function resizeCanvas() {
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
}
resizeCanvas();

const cellSize = 30;
let snake = [{x: 5, y: 5}];
let direction = 'RIGHT';
let food = randomPosition();
let gameInterval;
let waitingForAnswer = false;
let score = 0;
let currentQuestion;

const bgMusic = document.getElementById('bgMusic');
bgMusic.volume = 0.5;
bgMusic.play().catch(() => {
  document.addEventListener('click', () => bgMusic.play(), {once:true});
});

const questionContainer = document.getElementById('questionContainer');
const questionText = document.getElementById('questionText');
const optButtons = [document.getElementById('opt1'), document.getElementById('opt2'), document.getElementById('opt3'), document.getElementById('opt4')];

const restartContainer = document.getElementById('restartContainer');
const correctAnswerText = document.getElementById('correctAnswerText');
const restartButton = document.getElementById('restartButton');

// Touch controls
document.getElementById('upBtn').addEventListener('touchstart', () => { if(direction!=='DOWN') direction='UP'; });
document.getElementById('downBtn').addEventListener('touchstart', () => { if(direction!=='UP') direction='DOWN'; });
document.getElementById('leftBtn').addEventListener('touchstart', () => { if(direction!=='RIGHT') direction='LEFT'; });
document.getElementById('rightBtn').addEventListener('touchstart', () => { if(direction!=='LEFT') direction='RIGHT'; });

// Keyboard controls
window.addEventListener('keydown', e => {
  if(e.key === 'w' && direction !== 'DOWN') direction = 'UP';
  if(e.key === 's' && direction !== 'UP') direction = 'DOWN';
  if(e.key === 'a' && direction !== 'RIGHT') direction = 'LEFT';
  if(e.key === 'd' && direction !== 'LEFT') direction = 'RIGHT';
});

function randomPosition() {
  return {
    x: Math.floor(Math.random() * Math.floor(canvas.width / cellSize)),
    y: Math.floor(Math.random() * Math.floor(canvas.height / cellSize))
  };
}

function draw() {
  ctx.fillStyle = '#222';
  ctx.fillRect(0, 0, canvas.width, canvas.height);
  // Snake
  ctx.fillStyle = '#0f0';
  snake.forEach(seg => ctx.fillRect(seg.x*cellSize, seg.y*cellSize, cellSize-1, cellSize-1));
  // Food
  ctx.fillStyle = '#f00';
  ctx.fillRect(food.x*cellSize, food.y*cellSize, cellSize-1, cellSize-1);
  // Score
  ctx.fillStyle = '#fff';
  ctx.font = '20px Arial';
  ctx.fillText("Score: " + score, 20, 30);
}

function moveSnake() {
  if(waitingForAnswer) return;
  const head = {...snake[0]};
  if(direction==='UP') head.y--;
  else if(direction==='DOWN') head.y++;
  else if(direction==='LEFT') head.x--;
  else if(direction==='RIGHT') head.x++;

  if(head.x < 0 || head.x >= canvas.width/cellSize || head.y < 0 || head.y >= canvas.height/cellSize || snake.some(s=>s.x===head.x && s.y===head.y)) {
    clearInterval(gameInterval);
    showRestart("Game Over!");
    return;
  }

  snake.unshift(head);
  if(head.x===food.x && head.y===food.y) {
    waitingForAnswer=true;
    showQuestion();
  } else {
    snake.pop();
  }
}

function generateQuestion() {
  const a=Math.floor(Math.random()*10)+1;
  const b=Math.floor(Math.random()*10)+1;
  const correct=a+b;
  let options=[correct];
  while(options.length<4){
    const rand=Math.floor(Math.random()*20)+1;
    if(!options.includes(rand)) options.push(rand);
  }
  options.sort(()=>Math.random()-0.5);
  return {text:`${a} + ${b} = ?`, correct, options};
}

function showQuestion(){
  currentQuestion=generateQuestion();
  questionText.textContent=currentQuestion.text;
  optButtons.forEach((btn,i)=>{
    btn.textContent=currentQuestion.options[i];
    btn.onclick=()=>{
      if(parseInt(btn.textContent)===currentQuestion.correct){
        score++;
        snake.push({...snake[snake.length-1]});
        food=randomPosition();
        waitingForAnswer=false;
        questionContainer.style.display='none';
      } else {
        waitingForAnswer=false;
        questionContainer.style.display='none';
        showRestart("Wrong Answer! Correct was: "+currentQuestion.correct);
      }
    }
  });
  questionContainer.style.display='block';
}

function showRestart(message){
  clearInterval(gameInterval);
  correctAnswerText.textContent=message;
  restartContainer.style.display='block';
}

restartButton.onclick=()=>{
  snake=[{x:5,y:5}];
  direction='RIGHT';
  food=randomPosition();
  score=0;
  restartContainer.style.display='none';
  gameInterval=setInterval(gameLoop,150);
}

function gameLoop(){ moveSnake(); draw(); }

food=randomPosition();
gameInterval=setInterval(gameLoop,150);

window.addEventListener('resize',resizeCanvas);
</script>

</body>
</html>

</script>

</body>
</html>
