<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GIS Recruitment Aptitude Portal</title>
    <style>
        :root { --primary: #556b2f; --secondary: #f4f4f4; --accent: #d4af37; --text: #333; }
        body { font-family: 'Segoe UI', sans-serif; background: var(--secondary); margin: 0; padding: 10px; }
        .container { max-width: 900px; margin: 20px auto; background: white; padding: 30px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); border-top: 10px solid var(--primary); }
        header { text-align: center; border-bottom: 2px solid #eee; padding-bottom: 20px; }
        .stats-bar { display: flex; justify-content: space-between; background: #222; color: white; padding: 15px; border-radius: 8px; margin: 20px 0; }
        .question-text { font-size: 1.25rem; font-weight: bold; margin-bottom: 20px; color: #111; }
        .options-grid { display: grid; gap: 12px; }
        .opt-btn { padding: 15px; border: 2px solid #ddd; border-radius: 8px; cursor: pointer; transition: 0.2s; background: white; text-align: left; font-size: 1rem; }
        
        /* Feedback Colors */
        .correct-flash { background: #2e7d32 !important; color: white !important; border-color: #1b5e20 !important; }
        .wrong-flash { background: #d32f2f !important; color: white !important; border-color: #b71c1c !important; }
        
        .nav-tools { display: flex; justify-content: space-between; margin-top: 40px; }
        button:disabled { opacity: 0.5; cursor: not-allowed; }
        .btn-main { background: var(--primary); color: white; padding: 12px 30px; border-radius: 6px; border: none; font-weight: bold; cursor: pointer;}
        #results-screen { display: none; text-align: center; }
        .category-tag { background: var(--accent); color: white; padding: 4px 12px; border-radius: 20px; font-size: 0.8rem; margin-bottom: 10px; display: inline-block; }
    </style>
</head>
<body>

<div class="container">
    <header>
        <h1>Ghana Immigration Service</h1>
        <p>Aptitude Practice: Real-time Feedback Mode</p>
    </header>

    <div id="quiz-ui">
        <div class="stats-bar">
            <span>Question: <span id="q-num">1</span> / 100</span>
            <span style="color: #d4af37;">Current Score: <span id="live-score">0</span></span>
            <span id="timer-display">Time: 30:00</span>
        </div>

        <div id="q-container">
            <span id="q-cat" class="category-tag">General Knowledge</span>
            <div id="q-text" class="question-text">Loading question...</div>
            <div id="q-options" class="options-grid"></div>
        </div>
    </div>

    <div id="results-screen">
        <h2>Examination Complete</h2>
        <h1 style="font-size: 4rem; color: var(--primary);" id="final-score">0%</h1>
        <div id="summary-text"></div>
        <button class="btn-main" onclick="location.reload()">Re-take Examination</button>
    </div>
</div>

<script>
    // DATA SOURCE: Immigration.pdf [cite: 1-640]
    const quizData = [
        { cat: "GENERAL KNOWLEDGE", q: "In which year was the Ghana Immigration Service established as a paramilitary organization under PNDC Law 226?", a: ["1957", "1963", "1989", "1992"], c: 2 }, [cite: 5, 10]
        { cat: "GENERAL KNOWLEDGE", q: "What is the motto of the Ghana Immigration Service?", a: ["Service with Integrity", "Friendship with Vigilance", "Protection and Service", "Defending the Borders"], c: 1 }, [cite: 11, 16]
        { cat: "GENERAL KNOWLEDGE", q: "Who is the current Comptroller-General of Immigration (as of Nov 2025)?", a: ["Mr. Kwame Asuah Takyi", "Mr. Samuel Basintale Amadu", "COP Dr. Peter Wiredu", "Mr. Felix Yaw Sarpong"], c: 1 }, [cite: 17, 22]
        { cat: "GENERAL KNOWLEDGE", q: "Which Ministry supervises the Ghana Immigration Service?", a: ["Ministry of Foreign Affairs", "Ministry of National Security", "Ministry of the Interior", "Ministry of Defence"], c: 2 }, [cite: 23, 28]
        { cat: "GENERAL KNOWLEDGE", q: "Which Act currently governs the operations of the Ghana Immigration Service (repealing PNDC Law 226)?", a: ["Immigration Service Act, 2016 (Act 908)", "Police Service Act, 1970", "Aliens Act, 1963", "Interior Ministry Act, 2000"], c: 0 }, [cite: 36, 41]
        { cat: "MATHEMATICS", q: "Solve for x: 4x - 8 = 20", a: ["5", "6", "7", "8"], c: 2 }, [cite: 162, 167]
        { cat: "MATHEMATICS", q: "A car travels 120 km in 2 hours. What is its average speed?", a: ["50 km/h", "60 km/h", "100 km/h", "240 km/h"], c: 1 }, [cite: 168, 174]
        { cat: "ENGLISH", q: "Choose the synonym for 'Vigilance':", a: ["Carelessness", "Sleep", "Alertness", "Ignorance"], c: 2 }, [cite: 302, 307]
        { cat: "LOGIC", q: "If Dictionary is to Words, then Atlas is to:", a: ["Books", "Maps", "Earth", "Schools"], c: 1 }, [cite: 433, 438]
        { cat: "LOGIC", q: "If a plane crashes on the border of Ghana and Togo, where do you bury the survivors?", a: ["Ghana", "Togo", "No Man's Land", "You don't bury survivors"], c: 3 } [cite: 567, 572]
        // Note: You can add all 100 questions here following this format
    ];

    let current = 0;
    let correctCount = 0;
    let timeLeft = 1800; // 30 Minutes
    let acceptingAnswers = true;

    function startTimer() {
        const timer = setInterval(() => {
            timeLeft--;
            let mins = Math.floor(timeLeft / 60);
            let secs = timeLeft % 60;
            document.getElementById('timer-display').innerText = `Time: ${mins}:${secs < 10 ? '0'+secs : secs}`;
            if (timeLeft <= 0) { clearInterval(timer); showResults(); }
        }, 1000);
    }

    function render() {
        if (current >= quizData.length) { showResults(); return; }
        acceptingAnswers = true;
        const q = quizData[current];
        document.getElementById('q-num').innerText = current + 1;
        document.getElementById('q-cat').innerText = q.cat;
        document.getElementById('q-text').innerText = q.q;
        
        const optionsDiv = document.getElementById('q-options');
        optionsDiv.innerHTML = '';
        q.a.forEach((opt, i) => {
            const btn = document.createElement('button');
            btn.className = 'opt-btn';
            btn.innerText = opt;
            btn.onclick = () => checkAnswer(i, btn);
            optionsDiv.appendChild(btn);
        });
    }

    function checkAnswer(selected, btn) {
        if (!acceptingAnswers) return;
        acceptingAnswers = false;
        
        const correctIndex = quizData[current].c;
        const allButtons = document.querySelectorAll('.opt-btn');

        if (selected === correctIndex) {
            btn.classList.add('correct-flash');
            correctCount++;
            document.getElementById('live-score').innerText = correctCount;
        } else {
            btn.classList.add('wrong-flash');
            allButtons[correctIndex].classList.add('correct-flash'); // Show the right one
        }

        // Wait 1.5 seconds then proceed
        setTimeout(() => {
            current++;
            render();
        }, 1500);
    }

    function showResults() {
        document.getElementById('quiz-ui').style.display = 'none';
        document.getElementById('results-screen').style.display = 'block';
        let percent = Math.round((correctCount / quizData.length) * 100);
        document.getElementById('final-score').innerText = percent + "%";
        document.getElementById('summary-text').innerText = `Final Score: ${correctCount} out of ${quizData.length}`;
    }

    startTimer();
    render();
</script>
</body>
</html>
