<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Quiz with DOCX Upload</title>
    <style>
      /* ... same styles from previous code (login, quiz layout) ... */
      body {
        font-family: Arial, sans-serif;
        margin: 0;
        padding: 0;
        background: #f0f4f8;
      }
      #loginSection,
      #uploadSection,
      #quizSection,
      #resultSection {
        max-width: 900px;
        margin: 20px auto;
        background: white;
        border-radius: 8px;
        padding: 20px;
        box-shadow: 0 2px 8px rgb(0 0 0 / 0.1);
      }
      #loginSection {
        text-align: center;
      }
      #loginSection input[type="text"],
      #loginSection input[type="file"],
      #uploadSection input[type="file"] {
        margin: 10px 0;
        padding: 10px;
        font-size: 16px;
      }
      #loginSection button,
      #uploadSection button {
        padding: 12px 25px;
        font-size: 18px;
        background: #007bff;
        border: none;
        color: white;
        cursor: pointer;
        border-radius: 5px;
      }
      #loginSection button:hover,
      #uploadSection button:hover {
        background: #0056b3;
      }
      #quizContainer {
        display: flex;
        gap: 10px;
      }
      #questionNumbers {
        flex: 1;
        max-width: 70px;
        border-right: 1px solid #ddd;
        overflow-y: auto;
        height: 350px;
      }
      #questionNumbers button {
        display: block;
        width: 100%;
        border: none;
        background: #eee;
        padding: 10px 0;
        margin-bottom: 5px;
        cursor: pointer;
        font-weight: bold;
        border-radius: 0 5px 5px 0;
      }
      #questionNumbers button.active {
        background: #007bff;
        color: white;
      }
      #questionNumbers button.answered {
        background: #28a745;
        color: white;
      }
      #questionContent {
        flex: 4;
        padding: 0 20px;
      }
      #questionContent h2 {
        margin-top: 0;
      }
      .options {
        margin-top: 15px;
      }
      .options label {
        display: block;
        margin: 10px 0;
        cursor: pointer;
        font-size: 18px;
      }
      #timer {
        font-size: 18px;
        font-weight: bold;
        color: #dc3545;
      }
      #studentDetails {
        flex: 2;
        border-left: 1px solid #ddd;
        padding-left: 20px;
        text-align: center;
      }
      #studentDetails img {
        width: 120px;
        height: 120px;
        border-radius: 50%;
        object-fit: cover;
        margin-bottom: 15px;
        border: 3px solid #007bff;
      }
      #submitQuizBtn {
        margin-top: 20px;
        background: #28a745;
        border: none;
        color: white;
        font-size: 18px;
        padding: 12px 25px;
        cursor: pointer;
        border-radius: 6px;
        width: 100%;
      }
      #submitQuizBtn:hover {
        background: #1e7e34;
      }
      #resultSection {
        display: none;
        text-align: center;
      }
      #resultSection h2 {
        margin-bottom: 10px;
      }
    </style>
  </head>
  <body>
    <!-- LOGIN -->
    <div id="loginSection">
      <h1>Student Login</h1>
      <input type="text" id="studentName" placeholder="Enter Name" required />
      <br />
      <input type="text" id="rollNo" placeholder="Enter Roll Number" required />
      <br />
      <input type="file" id="studentImage" accept="image/*" required />
      <br />
      <button id="loginBtn">Login & Proceed</button>
    </div>

    <!-- UPLOAD QUESTIONS -->
    <div id="uploadSection" style="display: none; text-align: center">
      <h2>Upload Question Paper (.docx)</h2>
      <input type="file" id="docxFile" accept=".docx" />
      <br />
      <button id="uploadQuestionsBtn">Load Questions</button>
      <p id="uploadStatus" style="color: green"></p>
    </div>

    <!-- QUIZ -->
    <div id="quizSection" style="display: none">
      <div id="quizContainer">
        <div id="questionNumbers"></div>

        <div id="questionContent">
          <h2 id="questionText">Question Text</h2>
          <div class="options" id="optionsContainer"></div>
          <div id="timer">Time left: 30s</div>
          <button id="submitQuizBtn">Submit Quiz</button>
        </div>

        <div id="studentDetails">
          <img id="profileImage" src="" alt="Profile Image" />
          <h3 id="displayName">Name:</h3>
          <h4 id="displayRoll">Roll No:</h4>
          <h3>Score: <span id="scoreDisplay">0</span></h3>
        </div>
      </div>
    </div>

    <!-- RESULT -->
    <div id="resultSection">
      <h2>Quiz Completed!</h2>
      <img id="resultImage" src="" alt="Profile" />
      <h3 id="resultName"></h3>
      <h4 id="resultRoll"></h4>
      <h3>
        Your Score: <span id="finalScore"></span> /
        <span id="totalQuestions"></span>
      </h3>
    </div>

    <!-- Mammoth.js to parse DOCX -->
    <script src="https://unpkg.com/mammoth/mammoth.browser.min.js"></script>

    <script>
      let questions = [];

      let currentQuestionIndex = 0;
      let userAnswers = [];
      let timer;
      let timeLeft = 30;
      let score = 0;

      const loginSection = document.getElementById("loginSection");
      const uploadSection = document.getElementById("uploadSection");
      const quizSection = document.getElementById("quizSection");
      const resultSection = document.getElementById("resultSection");

      const questionNumbersDiv = document.getElementById("questionNumbers");
      const questionText = document.getElementById("questionText");
      const optionsContainer = document.getElementById("optionsContainer");
      const timerDisplay = document.getElementById("timer");
      const scoreDisplay = document.getElementById("scoreDisplay");

      const profileImage = document.getElementById("profileImage");
      const displayName = document.getElementById("displayName");
      const displayRoll = document.getElementById("displayRoll");

      const submitQuizBtn = document.getElementById("submitQuizBtn");

      const resultImage = document.getElementById("resultImage");
      const resultName = document.getElementById("resultName");
      const resultRoll = document.getElementById("resultRoll");
      const finalScore = document.getElementById("finalScore");
      const totalQuestions = document.getElementById("totalQuestions");

      const studentNameInput = document.getElementById("studentName");
      const rollNoInput = document.getElementById("rollNo");
      const studentImageInput = document.getElementById("studentImage");
      const loginBtn = document.getElementById("loginBtn");

      const docxFileInput = document.getElementById("docxFile");
      const uploadQuestionsBtn = document.getElementById("uploadQuestionsBtn");
      const uploadStatus = document.getElementById("uploadStatus");

      let studentData = {
        name: "",
        roll: "",
        imageSrc: "",
      };

      // -- LOGIN --

      loginBtn.addEventListener("click", () => {
        const name = studentNameInput.value.trim();
        const roll = rollNoInput.value.trim();
        const file = studentImageInput.files[0];

        if (!name || !roll || !file) {
          alert("Please fill all fields and select an image.");
          return;
        }

        const reader = new FileReader();
        reader.onload = function (e) {
          studentData = {
            name: name,
            roll: roll,
            imageSrc: e.target.result,
          };
          displayName.textContent = `Name: ${studentData.name}`;
          displayRoll.textContent = `Roll No: ${studentData.roll}`;
          profileImage.src = studentData.imageSrc;

          loginSection.style.display = "none";
          uploadSection.style.display = "block";
        };
        reader.readAsDataURL(file);
      });

      // -- UPLOAD AND PARSE DOCX --

      uploadQuestionsBtn.addEventListener("click", () => {
        const file = docxFileInput.files[0];
        if (!file) {
          alert("Please select a .docx file.");
          return;
        }
        uploadStatus.textContent = "Parsing document... Please wait.";

        const reader = new FileReader();
        reader.onload = function (event) {
          const arrayBuffer = event.target.result;
          mammoth
            .extractRawText({ arrayBuffer: arrayBuffer })
            .then(displayResult)
            .catch((err) => {
              uploadStatus.style.color = "red";
              uploadStatus.textContent =
                "Error parsing document: " + err.message;
            });
        };
        reader.readAsArrayBuffer(file);
      });

      function displayResult(result) {
        const text = result.value;
        // Parse questions from text
        questions = parseQuestionsFromText(text);
        if (questions.length === 0) {
          uploadStatus.style.color = "red";
          uploadStatus.textContent =
            "No questions detected in document. Please check format.";
          return;
        }
        uploadStatus.style.color = "green";
        uploadStatus.textContent = `Loaded ${questions.length} questions successfully! Starting quiz...`;

        userAnswers = new Array(questions.length).fill(null);
        uploadSection.style.display = "none";
        quizSection.style.display = "block";
        initQuestionNumbers();
        renderQuestion(0);
      }

      // Example parsing function - adapt based on your Word doc format
      function parseQuestionsFromText(text) {
        /* Expected format in docx text (simple example):
    Q1. What is the capital of India?
    A. Delhi
    B. Mumbai
    C. Chennai
    D. Kolkata
    Answer: A

    Q2. ...
  */

        const lines = text
          .split("\n")
          .map((l) => l.trim())
          .filter((l) => l.length > 0);
        const parsedQuestions = [];

        let currentQuestion = null;

        for (let line of lines) {
          // Match question line like "Q1. ..."
          if (/^Q\d+[\.\)]\s*/i.test(line)) {
            if (currentQuestion) {
              parsedQuestions.push(currentQuestion);
            }
            currentQuestion = { question: "", options: [], answer: null };
            currentQuestion.question = line
              .replace(/^Q\d+[\.\)]\s*/i, "")
              .trim();
            continue;
          }

          // Match options like "A. option text"
          let optionMatch = line.match(/^([A-D])[\.\)]\s*(.*)/i);
          if (optionMatch && currentQuestion) {
            currentQuestion.options.push(optionMatch[2]);
            continue;
          }

          // Match answer line: "Answer: A"
          let answerMatch = line.match(/^Answer\s*:\s*([A-D])/i);
          if (answerMatch && currentQuestion) {
            let ansLetter = answerMatch[1].toUpperCase();
            currentQuestion.answer = ansLetter.charCodeAt(0) - 65; // 'A'->0, 'B'->1 ...
            continue;
          }
        }

        // Push last question if exists
        if (currentQuestion) {
          parsedQuestions.push(currentQuestion);
        }

        // Filter incomplete questions
        return parsedQuestions.filter(
          (q) => q.question && q.options.length >= 2 && q.answer !== null
        );
      }

      // -- QUIZ FUNCTIONS --

      function startTimer() {
        timeLeft = 30;
        timerDisplay.textContent = `Time left: ${timeLeft}s`;
        clearInterval(timer);
        timer = setInterval(() => {
          timeLeft--;
          timerDisplay.textContent = `Time left: ${timeLeft}s`;
          if (timeLeft <= 0) {
            clearInterval(timer);
            saveAnswer(null); // no answer selected for this question
            nextQuestion();
          }
        }, 1000);
      }

      function renderQuestion(index) {
        currentQuestionIndex = index;
        let q = questions[index];
        questionText.textContent = `${index + 1}. ${q.question}`;

        optionsContainer.innerHTML = "";
        q.options.forEach((opt, i) => {
          const optionId = `option_${i}`;
          const isChecked = userAnswers[index] === i ? "checked" : "";
          optionsContainer.innerHTML += `
      <label for="${optionId}">
        <input type="radio" name="option" id="${optionId}" value="${i}" ${isChecked}>
        ${opt}
      </label>
    `;
        });

        highlightQuestionNumber();
        startTimer();
      }

      function highlightQuestionNumber() {
        const buttons = document.querySelectorAll("#questionNumbers button");
        buttons.forEach((btn, idx) => {
          btn.classList.remove("active");
          if (userAnswers[idx] !== null) btn.classList.add("answered");
        });
        buttons[currentQuestionIndex].classList.add("active");
      }

      function saveAnswer(selected) {
        userAnswers[currentQuestionIndex] = selected;
        updateScore();
        highlightQuestionNumber();
      }

      function updateScore() {
        let s = 0;
        userAnswers.forEach((ans, i) => {
          if (ans !== null && ans === questions[i].answer) s++;
        });
        score = s;
        scoreDisplay.textContent = score;
      }

      function nextQuestion() {
        if (currentQuestionIndex < questions.length - 1) {
          renderQuestion(currentQuestionIndex + 1);
        } else {
          finishQuiz();
        }
      }

      function finishQuiz() {
        clearInterval(timer);
        quizSection.style.display = "none";
        resultSection.style.display = "block";

        finalScore.textContent = score;
        totalQuestions.textContent = questions.length;
        resultName.textContent = `Name: ${studentData.name}`;
        resultRoll.textContent = `Roll No: ${studentData.roll}`;
        resultImage.src = studentData.imageSrc;
      }

      function initQuestionNumbers() {
        questionNumbersDiv.innerHTML = "";
        questions.forEach((_, i) => {
          const btn = document.createElement("button");
          btn.textContent = i + 1;
          btn.addEventListener("click", () => {
            clearInterval(timer);
            renderQuestion(i);
          });
          questionNumbersDiv.appendChild(btn);
        });
      }

      optionsContainer.addEventListener("change", (e) => {
        if (e.target.name === "option") {
          const selected = parseInt(e.target.value);
          saveAnswer(selected);
          clearInterval(timer);
          nextQuestion();
        }
      });

      submitQuizBtn.addEventListener("click", () => {
        if (confirm("Are you sure you want to submit the quiz?")) {
          finishQuiz();
        }
      });
    </script>
  </body>
</html>
