<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<title>題海 Go</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
body{
  margin:0;
  font-family:-apple-system,BlinkMacSystemFont,"Noto Sans TC",sans-serif;
  background:radial-gradient(circle at top,#1c1f2b,#0b0d13);
  color:#f5f5f5;
  min-height:100vh;
  display:flex;
  align-items:center;
  justify-content:center;
}
.card{
  background:rgba(255,255,255,0.06);
  backdrop-filter:blur(12px);
  border-radius:20px;
  padding:28px;
  width:90%;
  max-width:560px;
  box-shadow:0 20px 60px rgba(0,0,0,.5);
}
h1,h2{text-align:center}
input{
  width:100%;
  padding:12px;
  border-radius:12px;
  border:none;
  margin-bottom:16px;
}
button{
  width:100%;
  padding:12px;
  border-radius:12px;
  border:none;
  background:#d4af37;
  font-weight:bold;
  cursor:pointer;
  margin-bottom:12px;
}
.secondary{
  background:transparent;
  color:#ccc;
  border:1px solid #444;
}
.lang{text-align:center;margin-bottom:16px}
.lang button{
  width:auto;
  background:transparent;
  color:#ccc;
  border:1px solid #555;
  border-radius:999px;
  padding:6px 16px;
  margin:0 6px;
}
.lang button.active{
  background:#d4af37;
  color:#000;
}
.timer{text-align:center;font-size:20px;color:#d4af37}
.option{background:rgba(255,255,255,.08);color:#fff}
.correct{background:#2ecc71!important;color:#000}
.wrong{background:#e74c3c!important}
.hidden{display:none}
.small{font-size:14px;color:#aaa}
</style>
</head>

<body>

<!-- 首頁 -->
<div class="card" id="home">
  <h1>題海 Go</h1>
  <input id="nickname" placeholder="輸入暱稱">
  <div class="lang">
    <button id="zhBtn" class="active">中文</button>
    <button id="enBtn">English</button>
  </div>
  <button onclick="startGame()">開始遊戲（30 秒）</button>
  <button class="secondary" onclick="showRank()">排行榜</button>
</div>

<!-- 遊戲 -->
<div class="card hidden" id="game">
  <div class="timer">⏱ <span id="time">30</span> 秒</div>
  <p class="small" id="progress"></p>
  <h2 id="question"></h2>
  <button class="option" id="A"></button>
  <button class="option" id="B"></button>
  <button class="option" id="C"></button>
  <button class="option" id="D"></button>
  <p>分數：<span id="score">0</span></p>
</div>

<!-- 結果 -->
<div class="card hidden" id="result">
  <h2>時間到！</h2>
  <p id="finalText" style="text-align:center"></p>
  <button onclick="confirmSave(true)">加入排行榜</button>
  <button class="secondary" onclick="confirmSave(false)">回首頁</button>
  <button class="secondary" onclick="showWrong()">查看錯題</button>
</div>

<!-- 錯題 -->
<div class="card hidden" id="wrong">
  <h2>❌ 錯題回顧</h2>
  <ol id="wrongList"></ol>
  <button class="secondary" onclick="backHome()">回首頁</button>
</div>

<!-- 排行榜 -->
<div class="card hidden" id="rank">
  <h2>🏆 排行榜</h2>
  <ol id="rankList"></ol>
  <button class="secondary" onclick="backHome()">回首頁</button>
</div>

<script>
/* ===== 基本狀態 ===== */
let lang="zh",player="",score=0,time=30,timer,inGame=false,locked=false;
let questionPool=[],questionIndex=0,current;
let wrongQuestions=[];

const $=id=>document.getElementById(id);

/* ===== 題庫（可自行擴充） ===== */
const qs={
zh:[
{q:"世界上最大的海洋是？",o:["太平洋","大西洋","印度洋","北冰洋"],a:0},
{q:"光速約為每秒多少公里？",o:["300","3,000","30,000","300,000"],a:3},
{q:"《論語》的作者是？",o:["孟子","孔子","老子","荀子"],a:1},
{q:"2 的 5 次方是多少？",o:["16","32","64","128"],a:1},
{q:"水的化學式是？",o:["CO₂","H₂O","O₂","NaCl"],a:1},
{q:"人體主要的呼吸器官是？",o:["心臟","肺","肝","腎"],a:1},
{q:"太陽系中最大的行星是？",o:["地球","木星","土星","火星"],a:1},
{q:"1 公斤等於幾公克？",o:["10","100","1000","10000"],a:2},
{q:"中國四書不包含哪一部？",o:["論語","孟子","大學","春秋"],a:3},
{q:"地球表面最多的是？",o:["陸地","森林","沙漠","海洋"],a:3},
{q:"氧氣的化學符號是？",o:["O","O₂","CO₂","H₂O"],a:1},
{q:"下列哪一種不是能源？",o:["太陽能","風能","核能","空氣"],a:3}
],
en:[
{q:"Largest ocean on Earth?",o:["Pacific","Atlantic","Indian","Arctic"],a:0},
{q:"Speed of light (km/s)?",o:["300","3,000","30,000","300,000"],a:3},
{q:"Who wrote The Analects?",o:["Mencius","Confucius","Laozi","Xunzi"],a:1},
{q:"2 to the power of 5 equals?",o:["16","32","64","128"],a:1},
{q:"Chemical formula of water?",o:["CO₂","H₂O","O₂","NaCl"],a:1},
{q:"Which organ is for breathing?",o:["Heart","Lungs","Liver","Kidney"],a:1},
{q:"Largest planet in the solar system?",o:["Earth","Jupiter","Saturn","Mars"],a:1},
{q:"How many grams in 1 kilogram?",o:["10","100","1000","10000"],a:2},
{q:"Which is NOT energy?",o:["Solar","Wind","Nuclear","Air"],a:3}
]
};

/* ===== 語言切換 ===== */
zhBtn.onclick=()=>{if(!inGame){lang="zh";zhBtn.classList.add("active");enBtn.classList.remove("active");}};
enBtn.onclick=()=>{if(!inGame){lang="en";enBtn.classList.add("active");zhBtn.classList.remove("active");}};

/* ===== 遊戲流程 ===== */
function startGame(){
  player=nickname.value||"玩家";
  score=0;time=30;inGame=true;locked=false;
  wrongQuestions=[];
  questionPool=[...qs[lang]];
  questionIndex=0;
  $("score").textContent=0;
  $("time").textContent=30;
  show("game");
  nextQ();
  timer=setInterval(()=>{
    time--; $("time").textContent=time;
    if(time<=0) endGame();
  },1000);
}

function nextQ(){
  if(questionIndex>=questionPool.length){endGame();return;}
  locked=false;
  ["A","B","C","D"].forEach(i=>$(i).className="option");
  current=questionPool[questionIndex];
  $("progress").textContent=`第 ${questionIndex+1} / ${questionPool.length} 題`;
  question.textContent=current.q;
  ["A","B","C","D"].forEach((id,i)=>{
    $(id).textContent=current.o[i];
    $(id).onclick=()=>answer(i,id);
  });
  questionIndex++;
}

function answer(i,id){
  if(locked)return;
  locked=true;
  if(i===current.a){
    score+=10;
  }else{
    wrongQuestions.push(current);
    $(id).classList.add("wrong");
  }
  $("score").textContent=score;
  $(["A","B","C","D"][current.a]).classList.add("correct");
  setTimeout(nextQ,500);
}

function endGame(){
  clearInterval(timer);
  inGame=false;
  finalText.textContent=`${player} 得到 ${score} 分`;
  show("result");
}

/* ===== 排行榜 ===== */
function confirmSave(save){
  if(save){
    let list=JSON.parse(localStorage.getItem("tihai")||"[]");
    list.push({player,score});
    list.sort((a,b)=>b.score-a.score);
    localStorage.setItem("tihai",JSON.stringify(list.slice(0,5)));
  }
  backHome();
}

function showRank(){
  rankList.innerHTML="";
  JSON.parse(localStorage.getItem("tihai")||"[]").forEach(i=>{
    let li=document.createElement("li");
    li.textContent=`${i.player} - ${i.score}`;
    rankList.appendChild(li);
  });
  show("rank");
}

/* ===== 錯題 ===== */
function showWrong(){
  wrongList.innerHTML="";
  wrongQuestions.forEach(q=>{
    let li=document.createElement("li");
    li.textContent=q.q;
    wrongList.appendChild(li);
  });
  show("wrong");
}

/* ===== 畫面控制 ===== */
function show(id){
  ["home","game","result","rank","wrong"].forEach(i=>$(i).classList.add("hidden"));
  $(id).classList.remove("hidden");
}
function backHome(){show("home")}
</script>
</body>
</html>
