
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
  max-width:520px;
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
</div>

<!-- 排行榜 -->
<div class="card hidden" id="rank">
  <h2>🏆 排行榜</h2>
  <ol id="rankList"></ol>
  <button class="secondary" onclick="backHome()">回首頁</button>
</div>

<script>
let lang="zh",player="",score=0,time=30,timer,inGame=false,locked=false;
let questionPool=[],questionIndex=0,current;

const qs={
  zh:[
    {q:"世界上最大的海洋是？",o:["太平洋","大西洋","印度洋","北冰洋"],a:0},
    {q:"光速約為每秒多少公里？",o:["300","3,000","30,000","300,000"],a:3},
    {q:"《論語》的作者是？",o:["孟子","孔子","老子","荀子"],a:1},
    {q:"2 的 5 次方是多少？",o:["16","32","64","128"],a:1},
    {q:"水的化學式是？",o:["CO₂","H₂O","O₂","NaCl"],a:1},
    {q:"人體主要的呼吸器官是？",o:["心臟","肺","肝","腎"],a:1}
  ],
  en:[
    {q:"Largest ocean on Earth?",o:["Pacific","Atlantic","Indian","Arctic"],a:0},
    {q:"Speed of light (km/s)?",o:["300","3,000","30,000","300,000"],a:3},
    {q:"Who wrote The Analects?",o:["Mencius","Confucius","Laozi","Xunzi"],a:1},
    {q:"What is 2 to the power of 5?",o:["16","32","64","128"],a:1},
    {q:"Chemical formula of water?",o:["CO₂","H₂O","O₂","NaCl"],a:1},
    {q:"Which organ is for breathing?",o:["Heart","Lungs","Liver","Kidney"],a:1}
  ]
};

const $=id=>document.getElementById(id);

zhBtn.onclick=()=>{ if(!inGame){lang="zh";zhBtn.classList.add("active");enBtn.classList.remove("active");}};
enBtn.onclick=()=>{ if(!inGame){lang="en";enBtn.classList.add("active");zhBtn.classList.remove("active");}};

function startGame(){
  player=nickname.value||"玩家";
  score=0;time=30;inGame=true;
  questionPool=[...qs[lang]];
  shuffle(questionPool);
  questionIndex=0;
  score.textContent=0;
  time.textContent=30;
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
  current=questionPool[questionIndex++];
  question.textContent=current.q;
  ["A","B","C","D"].forEach((id,i)=>{
    $(id).textContent=current.o[i];
    $(id).onclick=()=>answer(i,id);
  });
}

function answer(i,id){
  if(locked)return;
  locked=true;
  $("score").textContent=score+=(i===current.a?10:0);
  $(["A","B","C","D"][current.a]).classList.add("correct");
  if(i!==current.a) $(id).classList.add("wrong");
  setTimeout(nextQ,400);
}

function endGame(){
  clearInterval(timer);
  inGame=false;
  finalText.textContent=`${player} 得到 ${score} 分`;
  show("result");
}

function confirmSave(save){
  if(save){
    let list=JSON.parse(localStorage.getItem("tihai")||"[]");
    list.push({player,score});
    list.sort((a,b)=>b.score-a.score);
    localStorage.setItem("tihai",JSON.stringify(list.slice(0,5)));
  }
  save?showRank():backHome();
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

function show(id){
  ["home","game","result","rank"].forEach(i=>$(i).classList.add("hidden"));
  $(id).classList.remove("hidden");
}
function backHome(){show("home")}

function shuffle(arr){
  for(let i=arr.length-1;i>0;i--){
    const j=Math.floor(Math.random()*(i+1));
    [arr[i],arr[j]]=[arr[j],arr[i]];
  }
}
</script>
</body>
</html>
