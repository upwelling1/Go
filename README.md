<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8" />
  <title>題海 Go</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <style>
    body {
      margin: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI",
        "Noto Sans TC", sans-serif;
      background: radial-gradient(circle at top, #1c1f2b, #0b0d13);
      color: #f5f5f5;
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .card {
      background: rgba(255, 255, 255, 0.06);
      backdrop-filter: blur(12px);
      border-radius: 20px;
      padding: 32px;
      width: 90%;
      max-width: 520px;
      box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
    }

    h1 {
      margin-top: 0;
      text-align: center;
      letter-spacing: 1px;
    }

    .lang {
      text-align: center;
      margin-bottom: 20px;
    }

    .lang button {
      background: transparent;
      color: #aaa;
      border: 1px solid #444;
      border-radius: 999px;
      padding: 6px 14px;
      margin: 0 6px;
      cursor: pointer;
    }

    .lang button.active {
      background: #d4af37;
      color: #000;
      border-color: #d4af37;
    }

    .question {
      font-size: 18px;
      margin-bottom: 20px;
    }

    .option {
      display: block;
      width: 100%;
      background: rgba(255, 255, 255, 0.08);
      border: 1px solid transparent;
      color: #f5f5f5;
      padding: 12px 16px;
      margin-bottom: 12px;
      border-radius: 12px;
      cursor: pointer;
      text-align: left;
    }

    .option:hover {
      border-color: #d4af37;
    }

    .result {
      margin-top: 16px;
      padding-top: 12px;
      border-top: 1px solid #333;
      font-size: 14px;
      color: #ccc;
    }

    .correct {
      color: #7CFF8A;
    }

    .wrong {
      color: #FF7C7C;
    }

    .source {
      margin-top: 10px;
      font-size: 12px;
      color: #888;
    }
  </style>
</head>

<body>
  <div class="card">
    <h1>題海 Go</h1>

    <div class="lang">
      <button id="btnZh" class="active">中文</button>
      <button id="btnEn">English</button>
    </div>

    <div class="question" id="question"></div>

    <button class="option" onclick="answer('A')" id="optA"></button>
    <button class="option" onclick="answer('B')" id="optB"></button>
    <button class="option" onclick="answer('C')" id="optC"></button>
    <button class="option" onclick="answer('D')" id="optD"></button>

    <div class="result" id="result"></div>
    <div class="source" id="source"></div>
  </div>

  <script>
    const data = {
      source: "Wikipedia – Newton's Laws of Motion (CC BY-SA 4.0)",
      zh: {
        question: "根據牛頓第二運動定律，加速度與哪一項成正比？",
        options: {
          A: "質量",
          B: "速度",
          C: "作用力",
          D: "位移"
        },
        answer: "C",
        explanation: "牛頓第二定律指出，加速度與作用在物體上的淨力成正比。"
      },
      en: {
        question: "According to Newton’s Second Law, acceleration is proportional to:",
        options: {
          A: "Mass",
          B: "Velocity",
          C: "Applied force",
          D: "Displacement"
        },
        answer: "C",
        explanation: "Acceleration is directly proportional to the net force applied."
      }
    };

    let lang = "zh";

    const qEl = document.getElementById("question");
    const optA = document.getElementById("optA");
    const optB = document.getElementById("optB");
    const optC = document.getElementById("optC");
    const optD = document.getElementById("optD");
    const resultEl = document.getElementById("result");
    const sourceEl = document.getElementById("source");

    function render() {
      const d = data[lang];
      qEl.textContent = d.question;
      optA.textContent = "A. " + d.options.A;
      optB.textContent = "B. " + d.options.B;
      optC.textContent = "C. " + d.options.C;
      optD.textContent = "D. " + d.options.D;
      resultEl.textContent = "";
      sourceEl.textContent = "Source: " + data.source;
    }

    function answer(opt) {
      const correct = data[lang].answer;
      if (opt === correct) {
        resultEl.innerHTML =
          "<span class='correct'>✔ 正確</span><br>" +
          data[lang].explanation;
      } else {
        resultEl.innerHTML =
          "<span class='wrong'>✘ 錯誤</span><br>" +
          data[lang].explanation;
      }
    }

    document.getElementById("btnZh").onclick = () => {
      lang = "zh";
      document.getElementById("btnZh").classList.add("active");
      document.getElementById("btnEn").classList.remove("active");
      render();
    };

    document.getElementById("btnEn").onclick = () => {
      lang = "en";
      document.getElementById("btnEn").classList.add("active");
      document.getElementById("btnZh").classList.remove("active");
      render();
    };

    render();
  </script>
</body>
</html>
