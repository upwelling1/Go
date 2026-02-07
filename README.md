<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<title>題海 Go</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
  font-family:sans-serif;
  background:#f4f4f4;
  margin:0;
}
.screen{
  display:none;
  padding:20px;
}
.screen.active{
  display:block;
}
h1,h2{
  text-align:center;
}
button{
  padding:10px 16px;
  margin:6px;
  font-size:16px;
}
.option{
  display:block;
  width:100%;
  margin:8px 0;
}
#timer{
  font-size:20px;
  text-align:center;
  margin:10px;
}

/* 排行榜樣式 */
.rank-num{
  color:#ff5722;
  font-weight:bold;
  margin-right:6px;
}
.rank-name{
  color:#222;
}
.rank-score{
  color:#666;
  margin-left:4px;
}
</style>
</head>

<body>

<!-- ===== 首頁 ===== -->
<div id="home" class="screen active">
  <h1>題海 Go</h1>
  <p style="text-align:center">選擇語言開始</p>
  <div style="text-align:center">
    <button onclick="startGame('zh')">中文</button>
    <button onclick="startGame('en')">English</button>
  </div>
</div>

<!-- ===== 遊戲畫面 ===== -->
<div id="game" class="screen">
  <div id="timer">30</div>
  <h2 id="question"></h2>
  <div id="options"></div>
</div>

<!-- ===== 結果畫面 ===== -->
<div id="result" class="screen">
  <h2>時間到！</h2>
  <p id="scoreText"></p>
  <input id="nameInput" placeholder="你的名字">
  <div style="text-align:center">
    <button onclick="saveRank()">送出成績</button>
    <button onclick="backHome()">回首頁</button>
  </div>

  <h3>排行榜</h3>
  <ol id="rankList"></ol>
</div>

<!-- ===== 題庫 ===== -->
<script>
/* ========= 題庫（100 題） ========= */
window.QUESTION_BANK = {
  zh: [
    {q:"光速約為每秒多少公里？",o:["300","3,000","30,000","300,000"],a:3},
    {q:"世界上最大的海洋是？",o:["太平洋","大西洋","印度洋","北冰洋"],a:0},
    {q:"《紅樓夢》的作者是？",o:["施耐庵","羅貫中","曹雪芹","吳承恩"],a:2},
    {q:"2 的 6 次方是多少？",o:["32","48","64","128"],a:2},
    {q:"水的化學式是？",o:["CO₂","H₂O","O₂","NaCl"],a:1},
    {q:"人體最大的器官是？",o:["心臟","肺","皮膚","肝臟"],a:2},
    {q:"植物行光合作用需要哪種氣體？",o:["氧氣","氮氣","二氧化碳","氫氣"],a:2},
    {q:"台灣最高的山是？",o:["雪山","玉山","合歡山","阿里山"],a:1},
    {q:"下列哪一個是質數？",o:["4","6","9","11"],a:3},
    {q:"地球繞太陽一圈約多久？",o:["24小時","30天","180天","365天"],a:3},

    {q:"人體主要呼吸器官是？",o:["心臟","肺","腎","胃"],a:1},
    {q:"1 公斤等於幾公克？",o:["10","100","1,000","10,000"],a:2},
    {q:"牛頓提出哪一項定律？",o:["相對論","萬有引力","量子力學","演化論"],a:1},
    {q:"氧氣的化學符號是？",o:["O","O₂","CO₂","H₂"],a:1},
    {q:"哪一個不是再生能源？",o:["太陽能","風能","煤","水力"],a:2},
    {q:"《論語》是誰的著作？",o:["孟子","孔子","老子","莊子"],a:1},
    {q:"正方形有幾條邊？",o:["3","4","5","6"],a:1},
    {q:"地球的天然衛星是？",o:["火星","月球","金星","太陽"],a:1},
    {q:"人體血液呈什麼顏色？",o:["藍色","黑色","紅色","綠色"],a:2},
    {q:"下列何者屬於哺乳類？",o:["青蛙","鯊魚","蝙蝠","企鵝"],a:2},

    {q:"一打等於多少？",o:["10","11","12","13"],a:2},
    {q:"地球表面最多的是？",o:["陸地","沙漠","森林","海洋"],a:3},
    {q:"化學中 pH 值小於 7 代表？",o:["中性","鹼性","酸性","不確定"],a:2},
    {q:"光合作用主要在植物哪個部位？",o:["根","莖","葉","花"],a:2},
    {q:"下列哪一個是行星？",o:["月球","太陽","地球","北極星"],a:2},
    {q:"1 公里等於幾公尺？",o:["100","500","1,000","10,000"],a:2},
    {q:"DNA 的全名是？",o:["核糖核酸","去氧核糖核酸","胺基酸","蛋白質"],a:1},
    {q:"人體哪個器官負責思考？",o:["心臟","肺","大腦","胃"],a:2},
    {q:"哪一個不是五感？",o:["視覺","聽覺","嗅覺","平衡感"],a:3},
    {q:"下列哪個是金屬？",o:["氧","鐵","氫","碳"],a:1},

    {q:"台灣使用的貨幣是？",o:["人民幣","日圓","新台幣","美元"],a:2},
    {q:"下列哪個是偶數？",o:["3","5","7","8"],a:3},
    {q:"心臟的功能是？",o:["消化","呼吸","輸送血液","排尿"],a:2},
    {q:"下列何者為液體？",o:["冰","水","水蒸氣","雲"],a:1},
    {q:"人類正常體溫約為？",o:["35°C","36.5°C","38°C","40°C"],a:1},
    {q:"聲音需要什麼傳播？",o:["真空","介質","光","磁場"],a:1},
    {q:"植物的根主要功能是？",o:["製造養分","吸收水分","光合作用","開花"],a:1},
    {q:"下列哪一個是星系？",o:["太陽系","地球","月球","火星"],a:0},
    {q:"電壓的單位是？",o:["安培","瓦特","伏特","歐姆"],a:2},
    {q:"哪一種不是化石燃料？",o:["煤","石油","天然氣","太陽能"],a:3}
  ],

  en: [
    {q:"What is the speed of light (km/s)?",o:["300","3,000","30,000","300,000"],a:3},
    {q:"Which is the largest ocean?",o:["Atlantic","Indian","Pacific","Arctic"],a:2},
    {q:"Who wrote Romeo and Juliet?",o:["Dickens","Shakespeare","Hemingway","Twain"],a:1},
    {q:"What is 9 × 9?",o:["72","80","81","90"],a:2},
    {q:"Chemical formula of water?",o:["CO₂","H₂O","O₂","NaCl"],a:1},

    {q:"Which planet is closest to the Sun?",o:["Earth","Venus","Mercury","Mars"],a:2},
    {q:"How many continents are there?",o:["5","6","7","8"],a:2},
    {q:"What gas do humans need to breathe?",o:["Carbon dioxide","Oxygen","Nitrogen","Hydrogen"],a:1},
    {q:"Which organ pumps blood?",o:["Brain","Lungs","Heart","Kidney"],a:2},
    {q:"What is the largest mammal?",o:["Elephant","Blue whale","Giraffe","Hippo"],a:1},

    {q:"What is HCl?",o:["Salt","Water","Acid","Base"],a:2},
    {q:"Which is a prime number?",o:["4","6","9","11"],a:3},
    {q:"Earth is a ___?",o:["Star","Planet","Moon","Comet"],a:1},
    {q:"How many days in a year?",o:["360","365","370","380"],a:1},
    {q:"Which is renewable energy?",o:["Coal","Oil","Wind","Gas"],a:2},

    {q:"What do plants need for photosynthesis?",o:["Oxygen","Carbon dioxide","Nitrogen","Hydrogen"],a:1},
    {q:"Which is a solid?",o:["Water","Ice","Steam","Air"],a:1},
    {q:"Human body temperature is about?",o:["35°C","36.5°C","38°C","40°C"],a:1},
    {q:"Which sense uses the eyes?",o:["Hearing","Smell","Sight","Taste"],a:2},
    {q:"Which is not a metal?",o:["Iron","Gold","Oxygen","Silver"],a:2},

    {q:"What does DNA stand for?",o:["Deoxyribonucleic Acid","Ribonucleic Acid","Protein","Enzyme"],a:0},
    {q:"How many sides does a square have?",o:["3","4","5","6"],a:1},
    {q:"Which animal can fly?",o:["Dog","Cat","Bat","Cow"],a:2},
    {q:"What is 1000 grams?",o:["1 kg","10 kg","100 kg","0.1 kg"],a:0},
    {q:"Which is a galaxy?",o:["Earth","Moon","Milky Way","Sun"],a:2}
  ]
};
</script>


<!-- ===== 遊戲邏輯 ===== -->
<script>
let pool=[], idx=0, score=0, time=30, timer;

function show(id){
  document.querySelectorAll(".screen")
    .forEach(s=>s.classList.remove("active"));
  document.getElementById(id).classList.add("active");
}

function startGame(lang){
  pool=[...window.QUESTION_BANK[lang]]
    .sort(()=>Math.random()-0.5);
  idx=0;
  score=0;
  time=30;
  show("game");
  nextQ();
  timer=setInterval(()=>{
    time--;
    document.getElementById("timer").innerText=time;
    if(time<=0) endGame();
  },1000);
}

function nextQ(){
  if(idx>=pool.length){
    endGame();
    return;
  }
  const q=pool[idx++];
  document.getElementById("question").innerText=q.q;
  const box=document.getElementById("options");
  box.innerHTML="";
  q.o.forEach((text,i)=>{
    const b=document.createElement("button");
    b.className="option";
    b.innerText=text;
    b.onclick=()=>{
      if(i===q.a) score++;
      nextQ();
    };
    box.appendChild(b);
  });
}

function endGame(){
  clearInterval(timer);
  show("result");
  document.getElementById("scoreText").innerText=`得分：${score}`;
  showRank();
}

function saveRank(){
  const name=document.getElementById("nameInput").value || "無名";
  const list=JSON.parse(localStorage.getItem("tihai")||"[]");
  list.push({name,score});
  list.sort((a,b)=>b.score-a.score);
  localStorage.setItem("tihai",JSON.stringify(list.slice(0,10)));
  showRank();
}

function showRank(){
  const list=JSON.parse(localStorage.getItem("tihai")||"[]");
  const ol=document.getElementById("rankList");
  ol.innerHTML="";
  list.forEach((r,i)=>{
    const li=document.createElement("li");
    li.innerHTML=`
      <span class="rank-num">${i+1}</span>
      <span class="rank-name">${r.name}</span>
      <span class="rank-score">- ${r.score} 分</span>
    `;
    ol.appendChild(li);
  });
}

function backHome(){
  show("home");
}
</script>

</body>
</html>
