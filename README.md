
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
  color:#fff;
  min-height:100vh;
  display:flex;
  justify-content:center;
  align-items:center;
}
.card{
  background:rgba(255,255,255,.06);
  backdrop-filter:blur(12px);
  border-radius:20px;
  padding:28px;
  width:90%;
  max-width:520px;
}
button,input{
  width:100%;
  padding:12px;
  border-radius:12px;
  border:none;
  margin-bottom:12px;
}
button{background:#d4af37;font-weight:bold}
.secondary{background:transparent;border:1px solid #444;color:#ccc}
.option{background:rgba(255,255,255,.08);color:#fff}
.correct{background:#2ecc71!important;color:#000}
.wrong{background:#e74c3c!important}
.hidden{display:none}
.lang button{width:auto;margin:0 4px}
.lang .active{background:#d4af37;color:#000}
.timer{text-align:center;color:#d4af37;font-size:20px}
</style>
</head>

<body>

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

<div class="card hidden" id="game">
  <div class="timer">⏱ <span id="time">30</span></div>
  <h2 id="question"></h2>
  <button class="option" id="A"></button>
  <button class="option" id="B"></button>
  <button class="option" id="C"></button>
  <button class="option" id="D"></button>
  <p>分數：<span id="score">0</span></p>
</div>

<div class="card hidden" id="result">
  <h2>時間到</h2>
  <p id="finalText"></p>
  <button onclick="saveScore()">加入排行榜</button>
  <button class="secondary" onclick="backHome()">回首頁</button>
</div>

<div class="card hidden" id="rank">
  <h2>🏆 排行榜</h2>
  <ol id="rankList"></ol>
  <button class="secondary" onclick="backHome()">回首頁</button>
</div>

<script type="module">
import { db } from "./firebase.js";
import {
  collection,addDoc,getDocs,query,orderBy,limit
} from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

const $=id=>document.getElementById(id);
let lang="zh",player="",score=0,time=30,timer;
let index=0,current;
let pool={
  zh:[
    {q:"世界上最大的海洋是？",o:["太平洋","大西洋","印度洋","北冰洋"],a:0},
    {q:"光速約為每秒多少公里？",o:["300","3,000","30,000","300,000"],a:3},
    {q:"《論語》的作者是？",o:["孟子","孔子","老子","荀子"],a:1},
    {q:"2 的 5 次方是多少？",o:["16","32","64","128"],a:1}
  ],
  en:[
    {q:"Largest ocean?",o:["Pacific","Atlantic","Indian","Arctic"],a:0},
    {q:"Speed of light?",o:["300","3k","30k","300k"],a:3},
    {q:"Who wrote The Analects?",o:["Mencius","Confucius","Laozi","Xunzi"],a:1},
    {q:"2^5 equals?",o:["16","32","64","128"],a:1}
  ]
};

zhBtn.onclick=()=>{lang="zh";zhBtn.classList.add("active");enBtn.classList.remove("active")}
enBtn.onclick=()=>{lang="en";enBtn.classList.add("active");zhBtn.classList.remove("active")}

function show(id){
  ["home","game","result","rank"].forEach(i=>$(i).classList.add("hidden"));
  $(id).classList.remove("hidden");
}

window.startGame=()=>{
  player=nickname.value||"玩家";
  score=0;time=30;index=0;
  score.textContent=0;time.textContent=30;
  show("game");
  nextQ();
  timer=setInterval(()=>{
    time--; $("time").textContent=time;
    if(time<=0) endGame();
  },1000);
}

function nextQ(){
  if(index>=pool[lang].length)return;
  current=pool[lang][index++];
  question.textContent=current.q;
  ["A","B","C","D"].forEach((id,i)=>{
    $(id).textContent=current.o[i];
    $(id).className="option";
    $(id).onclick=()=>answer(i,id);
  });
}

function answer(i,id){
  if(i===current.a){
    score+=10;
    $(id).classList.add("correct");
  }else{
    $(id).classList.add("wrong");
    $(["A","B","C","D"][current.a]).classList.add("correct");
  }
  $("score").textContent=score;
  setTimeout(nextQ,400);
}

function endGame(){
  clearInterval(timer);
  finalText.textContent=`${player} 得到 ${score} 分`;
  show("result");
}

window.saveScore=async()=>{
  await addDoc(collection(db,"scores"),{
    player,score,ts:Date.now()
  });
  showRank();
}

window.showRank=async()=>{
  rankList.innerHTML="";
  const q=query(collection(db,"scores"),orderBy("score","desc"),limit(5));
  const snap=await getDocs(q);
  snap.forEach(d=>{
    const li=document.createElement("li");
    li.textContent=`${d.data().player} - ${d.data().score}`;
    rankList.appendChild(li);
  });
  show("rank");
}

window.backHome=()=>show("home");
</script>
</body>
</html>
