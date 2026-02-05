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
  font-size:16px;
}

button{
  width:100%;
  padding:12px;
  border-radius:12px;
  border:none;
  background:#d4af37;
  color:#000;
  font-weight:bold;
  cursor:pointer;
  margin-bottom:12px;
}

.secondary{
  background:transparent;
  color:#ccc;
  border:1px solid #444;
}

/* 語言按鍵 */
.lang{text-align:center;margin-bottom:16px}
.lang button{
  width:auto;
  background:transparent;
  color:#ccc;
  border:1px solid #555;
  border-radius:999px;
  padding:6px 16px;
  margin:0 6px;
  cursor:pointer;
  transition:all .2s ease;
}
.lang button.active{
  background:#d4af37;
  color:#000;
  border-color:#d4af37;
}

/* 遊戲 */
.timer{
  text-align:center;
  font-size:20px;
  margin-bottom:12px;
  color:#d4af37;
}
.option{
  background:rgba(255,255,255,.08);
  color:#fff;
}
.correct{background:#2ecc71!important;color:#000}
.wrong{background:#e74c3c!important}

.hidden{display:none}
.leaderboard li{font-size:14px;color:#ccc;margin-bottom:4px}
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

<!-- 遊戲畫面 -->
<div class="card hidden" id="game">
  <div class="timer">⏱ <span id="time">30</span> 秒</div>
  <h2 id="question"></h2>

  <button class="option" id="A"></button>
  <button class="option" id="B"></button>
  <button class="option" id="C"></button>
  <button class="option" id="D"></button>

  <p>分數：<span id="score">0</span></p>
</div>

<!-- 結算 -->
<div class="card hidden" id="result">
  <h2>⏱ 時間到！</h2>
  <p id="finalText" style="text-align:center"></p>
  <button onclick="confirmSave(true)">加入排行榜</button>
  <button class="secondary" onclick="confirmSave(false)">回首頁</button>
</div>

<!-- 排行榜 -->
<div class="card hidden" id="rank">
  <h2>🏆 排行榜</h2>
  <ol id="rankList" class="leaderboard"></ol>
  <button class="secondary" onclick="backHome()">回首頁</button>
</div>

<script>
let lang="zh",player="",score=0,time=30,timer,inGame=false,locked=false,current;

const qs={
  zh:[
    {q:"世界上最大的海洋是？",o:["太平洋","大西洋","印度洋","北冰洋"],a:0},
    {q:"光速約為每秒多少公里？",o:["300","3,000","30,000","300,000"],a:3}
    zh: [
  // 🌍 世界地理
  {q:"世界上最大的海洋是？",o:["太平洋","大西洋","印度洋","北冰洋"],a:0},
  {q:"世界面積最大的國家是？",o:["中國","美國","俄羅斯","加拿大"],a:2},
  {q:"撒哈拉沙漠位於哪個洲？",o:["亞洲","非洲","南美洲","澳洲"],a:1},
  {q:"赤道通過下列哪一個國家？",o:["日本","印度","肯亞","智利"],a:2},
  {q:"世界最長的河流是哪一條？",o:["亞馬遜河","尼羅河","長江","密西西比河"],a:1},

  // ⚛️ 物理
  {q:"光速約為每秒多少公里？",o:["300","3,000","30,000","300,000"],a:3},
  {q:"國際單位制中，力的單位是？",o:["瓦特","焦耳","牛頓","赫茲"],a:2},
  {q:"聲音無法在下列哪種介質中傳播？",o:["空氣","水","真空","金屬"],a:2},
  {q:"電流的國際單位是？",o:["伏特","安培","歐姆","瓦特"],a:1},
  {q:"地球的重力加速度約為？",o:["3.8","6.7","9.8","12.5"],a:2},

  // 📖 國文
  {q:"《論語》的作者是？",o:["孟子","孔子","老子","荀子"],a:1},
  {q:"「學而時習之，不亦說乎」出自哪部作品？",o:["大學","中庸","論語","孟子"],a:2},
  {q:"下列哪一個成語形容讀書非常勤奮？",o:["畫蛇添足","懸梁刺股","刻舟求劍","對牛彈琴"],a:1},
  {q:"「三人行，必有我師焉」的意思是？",o:["三人一起教書","任何人都有可學之處","老師一定有三個","學習要結伴"],a:1},
  {q:"下列哪一個不是唐代詩人？",o:["李白","杜甫","白居易","蘇軾"],a:3},

  // ➗ 數學
  {q:"下列哪一個是質數？",o:["4","6","9","11"],a:3},
  {q:"2 的 5 次方是多少？",o:["16","32","64","128"],a:1},
  {q:"圓的周長公式為？",o:["πr²","2πr","πd²","r²"],a:1},
  {q:"一個三角形內角和為？",o:["90°","180°","270°","360°"],a:1},
  {q:"下列哪一個不是偶數？",o:["2","4","7","8"],a:2},

  // 🧪 化學
  {q:"水的化學式是？",o:["CO₂","H₂O","O₂","NaCl"],a:1},
  {q:"下列哪一種是酸？",o:["氫氧化鈉","鹽酸","氨水","石灰水"],a:1},
  {q:"元素週期表的第一號元素是？",o:["氦","氫","氧","碳"],a:1},
  {q:"食鹽的主要成分是？",o:["氯化鈉","碳酸鈣","硫酸","葡萄糖"],a:0},
  {q:"下列哪一種屬於化學變化？",o:["冰融化","水蒸發","鐵生鏽","玻璃破裂"],a:2},

  // 🧬 生物
  {q:"人體進行呼吸作用的主要器官是？",o:["心臟","肺","肝臟","腎臟"],a:1},
  {q:"植物進行光合作用主要在細胞的哪個構造？",o:["粒線體","葉綠體","細胞核","液泡"],a:1},
  {q:"人類的遺傳物質主要是？",o:["蛋白質","脂肪","DNA","醣類"],a:2},
  {q:"下列哪一項不是五大類營養素？",o:["蛋白質","脂肪","維生素","氧氣"],a:3},
  {q:"生物分類中，最大的單位是？",o:["種","科","綱","界"],a:3}
]  ],
  en:[
    {q:"Largest ocean on Earth?",o:["Pacific","Atlantic","Indian","Arctic"],a:0},
    {q:"Speed of light (km/s)?",o:["300","3,000","30,000","300,000"],a:3}
    en: [
  // 🌍 World Geography
  {q:"What is the largest ocean on Earth?",o:["Pacific Ocean","Atlantic Ocean","Indian Ocean","Arctic Ocean"],a:0},
  {q:"Which country has the largest land area in the world?",o:["China","United States","Russia","Canada"],a:2},
  {q:"The Sahara Desert is located on which continent?",o:["Asia","Africa","South America","Australia"],a:1},
  {q:"The Equator passes through which country?",o:["Japan","India","Kenya","Chile"],a:2},
  {q:"Which is the longest river in the world?",o:["Amazon River","Nile River","Yangtze River","Mississippi River"],a:1},

  // ⚛️ Physics
  {q:"What is the approximate speed of light (km/s)?",o:["300","3,000","30,000","300,000"],a:3},
  {q:"What is the SI unit of force?",o:["Watt","Joule","Newton","Hertz"],a:2},
  {q:"Which of the following is NOT a fundamental force?",o:["Gravity","Electromagnetic force","Friction","Strong nuclear force"],a:2},
  {q:"Sound cannot travel through which medium?",o:["Air","Water","Vacuum","Metal"],a:2},
  {q:"What is the SI unit of electric current?",o:["Volt","Ampere","Ohm","Watt"],a:1},

  // 📖 Chinese Literature (General Knowledge)
  {q:"Who is the author of *The Analects*?",o:["Mencius","Confucius","Laozi","Xunzi"],a:1},
  {q:"The quote 'To learn and practice constantly, is this not a pleasure?' comes from which book?",o:["The Great Learning","Doctrine of the Mean","The Analects","Mencius"],a:2},
  {q:"Which idiom describes studying very diligently?",o:["Draw a snake and add feet","Hang one's head from a beam and stab one's thigh","Carve a mark on a boat","Play the lute to a cow"],a:1},
  {q:"What does the saying 'Among three people, there must be one who can be my teacher' mean?",o:["Teaching requires three people","Everyone has something worth learning","Teachers always come in threes","Learning must be done in groups"],a:1},
  {q:"Which of the following is NOT a poet from the Tang Dynasty?",o:["Li Bai","Du Fu","Bai Juyi","Su Shi"],a:3},

  // ➗ Mathematics
  {q:"Which of the following is a prime number?",o:["4","6","9","11"],a:3},
  {q:"What is 2 to the power of 5?",o:["16","32","64","128"],a:1},
  {q:"What is the formula for the circumference of a circle?",o:["πr²","2πr","πd²","r²"],a:1},
  {q:"What is the sum of the interior angles of a triangle?",o:["90°","180°","270°","360°"],a:1},
  {q:"Which of the following is NOT an even number?",o:["2","4","7","8"],a:2},

  // 🧪 Chemistry
  {q:"What is the chemical formula of water?",o:["CO₂","H₂O","O₂","NaCl"],a:1},
  {q:"Which of the following is an acid?",o:["Sodium hydroxide","Hydrochloric acid","Ammonia solution","Limewater"],a:1},
  {q:"What is the first element in the periodic table?",o:["Helium","Hydrogen","Oxygen","Carbon"],a:1},
  {q:"What is the main component of table salt?",o:["Sodium chloride","Calcium carbonate","Sulfuric acid","Glucose"],a:0},
  {q:"Which of the following is a chemical change?",o:["Melting ice","Water evaporation","Rusting iron","Breaking glass"],a:2},

  // 🧬 Biology
  {q:"Which organ is mainly responsible for respiration in humans?",o:["Heart","Lungs","Liver","Kidneys"],a:1},
  {q:"Photosynthesis mainly occurs in which cell organelle?",o:["Mitochondria","Chloroplast","Nucleus","Vacuole"],a:1},
  {q:"What is the primary genetic material in humans?",o:["Protein","Fat","DNA","Carbohydrate"],a:2},
  {q:"Which of the following is NOT one of the five major nutrients?",o:["Protein","Fat","Vitamin","Oxygen"],a:3},
  {q:"In biological classification, which is the largest unit?",o:["Species","Family","Class","Kingdom"],a:3}
]  ]
};

const $=id=>document.getElementById(id);

zhBtn.onclick=()=>{
  if(inGame)return;
  lang="zh";
  zhBtn.classList.add("active");
  enBtn.classList.remove("active");
};
enBtn.onclick=()=>{
  if(inGame)return;
  lang="en";
  enBtn.classList.add("active");
  zhBtn.classList.remove("active");
};

function startGame(){
  player=nickname.value||"玩家";
  score=0;time=30;inGame=true;
  $("score").textContent=score;
  $("time").textContent=time;
  show("game");
  nextQ();
  timer=setInterval(()=>{
    time--; $("time").textContent=time;
    if(time<=0)endGame();
  },1000);
}

function nextQ(){
  locked=false;
  ["A","B","C","D"].forEach(i=>$(i).className="option");
  current=qs[lang][Math.floor(Math.random()*qs[lang].length)];
  question.textContent=current.q;
  ["A","B","C","D"].forEach((id,i)=>{
    $(id).textContent=current.o[i];
    $(id).onclick=()=>answer(i,id);
  });
}

function answer(i,id){
  if(locked)return;
  locked=true;
  ["A","B","C","D"][current.a] && $(["A","B","C","D"][current.a]).classList.add("correct");
  if(i===current.a) score+=10;
  else $(id).classList.add("wrong");
  $("score").textContent=score;
  setTimeout(nextQ,400);
}

function endGame(){
  clearInterval(timer);
  inGame=false;
  $("finalText").textContent=`${player}，你獲得 ${score} 分`;
  show("result");
}

function confirmSave(save){
  if(save){
    const list=JSON.parse(localStorage.getItem("tihai")||"[]");
    list.push({player,score});
    list.sort((a,b)=>b.score-a.score);
    localStorage.setItem("tihai",JSON.stringify(list.slice(0,5)));
  }
  save?showRank():backHome();
}

function showRank(){
  rankList.innerHTML="";
  JSON.parse(localStorage.getItem("tihai")||"[]").forEach(i=>{
    const li=document.createElement("li");
    li.textContent=`${i.player} - ${i.score} 分`;
    rankList.appendChild(li);
  });
  show("rank");
}

function show(id){
  ["home","game","result","rank"].forEach(i=>$(i).classList.add("hidden"));
  $(id).classList.remove("hidden");
}
function backHome(){show("home")}
</script>

</body>
</html>
