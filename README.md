
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<title>題海 Go</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Noto Sans TC", sans-serif;
  background: radial-gradient(circle at top, #1c1f2b, #0b0d13);
  color: #f5f5f5;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

.card {
  background: rgba(255,255,255,0.06);
  backdrop-filter: blur(12px);
  border-radius: 20px;
  padding: 28px;
  width: 90%;
  max-width: 520px;
  box-shadow: 0 20px 60px rgba(0,0,0,.5);
}

h1, h2 { text-align: center; }

input {
  width: 100%;
  padding: 12px;
  border-radius: 12px;
  border: none;
  margin-bottom: 16px;
  font-size: 16px;
}

button {
  width: 100%;
  padding: 12px;
  border-radius: 12px;
  border: none;
  background: #d4af37;
  color: #000;
  font-weight: bold;
  cursor: pointer;
  margin-bottom: 12px;
}

.secondary {
  background: transparent;
  color: #ccc;
  border: 1px solid #444;
}

.lang button {
  width: auto;
  margin: 0 6px;
  padding: 6px 14px;
}

.lang .active {
  background: #d4af37;
  color: #000;
}

.option {
  background: rgba(255,255,255,0.08);
  color: #fff;
  margin-bottom: 10px;
  transition: all .2s;
}

.correct {
  background: #2ecc71 !important;
  color: #000;
}

.wrong {
  background: #e74c3c !important;
}

.hidden { display: none; }

.timer {
  text-align: center;
  font-size: 20px;
  margin-bottom: 12px;
  color: #d4af37;
}

.leaderboard li {
  font-size: 14px;
  color: #ccc;
  margin-bottom: 4px;
}
</style>
</head>

<body>

<!-- 首頁 -->
<div class="card" id="home">
  <h1>題海 Go</h1>
  <input id="nickname" placeholder="輸入暱稱" />

  <div class="lang" style="text-align:center;margin-bottom:16px;">
    <button id="zhBtn" class="active">中文</button>
    <button id="enBtn">English</button>
  </div>

  <button onclick="startGame()">開始遊戲（30 秒）</button>
  <button class="secondary" onclick="showRank()">排行榜</button>
</div>

<!-- 遊戲 -->
<div class="card hidden" id="game">
  <div class="timer">⏱ <span id="time">30</span> 秒</div>
  <h2 id="question"></h2>

  <button class="option" id="A"></button>
  <button class="option" id="B"></button>
  <button class="option" id="C"></button>
  <button class="option" id="D"></button>

  <p>分數：<span id="score">0</span></p>
</div>

<!-- 排行榜 -->
<div class="card hidden" id="rank">
  <h2>🏆 排行榜</h2>
  <ol id="rankList"></ol>
  <button class="secondary" onclick="backHome()">回首頁</button>
</div>

<script>
let lang = "zh";
let name = "";
let score = 0;
let time = 30;
let timerId;
let locked = false;
let current;

const questions = {
  zh: [
    {q:"世界上最大的海洋是？",o:["太平洋","大西洋","印度洋","北冰洋"],a:0},
    {q:"光速約為每秒多少公里？",o:["300","3,000","30,000","300,000"],a:3}
  ],
  en: [
    {q:"Largest ocean on Earth?",o:["Pacific","Atlantic","Indian","Arctic"],a:0},
    {q:"Speed of light (km/s)?",o:["300","3,000","30,000","300,000"],a:3}
  ]
};

function startGame(){
  name = nickname.value || "玩家";
  score = 0;
  time = 30;
  document.getElementById("score").textContent = score;
  document.getElementById("time").textContent = time;
  show("game");
  nextQ();

  timerId = setInterval(()=>{
    time--;
    document.getElementById("time").textContent = time;
    if(time<=0) endGame();
  },1000);
}

function nextQ(){
  locked = false;
  resetButtons();
  current = questions[lang][Math.floor(Math.random()*questions[lang].length)];
  question.textContent = current.q;
  ["A","B","C","D"].forEach((id,i)=>{
    const btn = document.getElementById(id);
    btn.textContent = current.o[i];
    btn.onclick = ()=>answer(i,btn);
  });
}

function answer(i,btn){
  if(locked) return;
  locked = true;

  const correctBtn = document.getElementById(["A","B","C","D"][current.a]);
  correctBtn.classList.add("correct");

  if(i === current.a){
    score += 10;
  } else {
    btn.classList.add("wrong");
  }

  document.getElementById("score").textContent = score;

  setTimeout(nextQ, 400);
}

function resetButtons(){
  ["A","B","C","D"].forEach(id=>{
    const btn=document.getElementById(id);
    btn.classList.remove("correct","wrong");
  });
}

function endGame(){
  clearInterval(timerId);
  saveScore();
  showRank();
}

function saveScore(){
  const list = JSON.parse(localStorage.getItem("tihai")||"[]");
  list.push({name,score});
  list.sort((a,b)=>b.score-a.score);
  localStorage.setItem("tihai",JSON.stringify(list.slice(0,5)));
}

function showRank(){
  const list = JSON.parse(localStorage.getItem("tihai")||"[]");
  rankList.innerHTML="";
  list.forEach(i=>{
    const li=document.createElement("li");
    li.textContent=`${i.name} - ${i.score} 分`;
    rankList.appendChild(li);
  });
  show("rank");
}

function show(id){
  ["home","game","rank"].forEach(i=>document.getElementById(i).classList.add("hidden"));
  document.getElementById(id).classList.remove("hidden");
}

function backHome(){ show("home"); }

zhBtn.onclick=()=>{lang="zh";zhBtn.classList.add("active");enBtn.classList.remove("active");}
enBtn.onclick=()=>{lang="en";enBtn.classList.add("active");zhBtn.classList.remove("active");}
</script>

</body>
</html>
