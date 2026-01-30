[index.html](https://github.com/user-attachments/files/24968032/index.html)
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
        .stats-bar { display: flex; justify-content: space-between; background: #222; color: white; padding: 15px; border-radius: 8px; margin: 20px 0; sticky: top; }
        .question-text { font-size: 1.25rem; font-weight: bold; margin-bottom: 20px; color: #111; }
        .options-grid { display: grid; gap: 12px; }
        .opt-btn { padding: 15px; border: 2px solid #ddd; border-radius: 8px; cursor: pointer; transition: 0.2s; background: white; text-align: left; font-size: 1rem; }
        .opt-btn:hover { border-color: var(--primary); background: #f9fdf4; }
        .selected { background: var(--primary) !important; color: white; border-color: var(--primary); }
        .nav-tools { display: flex; justify-content: space-between; margin-top: 40px; }
        button { padding: 12px 30px; border-radius: 6px; border: none; font-weight: bold; cursor: pointer; }
        .btn-main { background: var(--primary); color: white; }
        .btn-sec { background: #6c757d; color: white; }
        #results-screen { display: none; text-align: center; }
        .score-big { font-size: 4rem; font-weight: 900; color: var(--primary); }
        .category-tag { background: var(--accent); color: white; padding: 4px 12px; border-radius: 20px; font-size: 0.8rem; margin-bottom: 10px; display: inline-block; }
    </style>
</head>
<body>

<div class="container">
    <header>
        <img src="https://via.placeholder.com/80/556b2f/ffffff?text=GIS" alt="GIS Logo" style="border-radius: 50%;">
        <h1>Ghana Immigration Service</h1>
        <p>Official Aptitude Test Simulation (100 Questions)</p>
    </header>

    <div id="quiz-ui">
        <div class="stats-bar">
            <span>Question: <span id="q-num">1</span> / 100</span>
            <span id="timer-display">Time: 60:00</span>
        </div>

        <div id="q-container">
            <span id="q-cat" class="category-tag">General Knowledge</span>
            <div id="q-text" class="question-text">Loading question...</div>
            <div id="q-options" class="options-grid"></div>
        </div>

        <div class="nav-tools">
            <button class="btn-sec" onclick="move(-1)">Previous</button>
            <button class="btn-main" id="next-btn" onclick="move(1)">Next Question</button>
        </div>
    </div>

    <div id="results-screen">
        <h2>Examination Complete</h2>
        <div class="score-big" id="final-score">0%</div>
        <p id="summary-text"></p>
        <button class="btn-main" onclick="location.reload()">Re-take Examination</button>
    </div>
</div>

<script>
    // DATA SOURCE: Immigration.pdf [cite: 1-99]
    const quizData = [
        // SECTION A: GENERAL KNOWLEDGE [cite: 1]
        { cat: "GENERAL KNOWLEDGE", q: "In which year was the Ghana Immigration Service established as a paramilitary organization under PNDC Law 226?", a: ["1957", "1963", "1989", "1992"], c: 2 }, // [cite: 2, 3]
        { cat: "GENERAL KNOWLEDGE", q: "What is the motto of the Ghana Immigration Service?", a: ["Service with Integrity", "Friendship with Vigilance", "Protection and Service", "Defending the Borders"], c: 1 }, // [cite: 4]
        { cat: "GENERAL KNOWLEDGE", q: "Who is the current Comptroller-General of Immigration (as of Nov 2025)?", a: ["Mr. Kwame Asuah Takyi", "Mr. Samuel Basintale Amadu", "COP Dr. Peter Wiredu", "Mr. Felix Yaw Sarpong"], c: 1 }, // [cite: 5]
        { cat: "GENERAL KNOWLEDGE", q: "Which Ministry supervises the Ghana Immigration Service?", a: ["Ministry of Foreign Affairs", "Ministry of National Security", "Ministry of the Interior", "Ministry of Defence"], c: 2 }, // [cite: 6]
        { cat: "GENERAL KNOWLEDGE", q: "Who was the Head of the Immigration and Passport Unit under the colonial police force?", a: ["Mr. Neville C. Hill", "Mr. E.R.T. Madjitey", "Kwame Nkrumah", "Sir Charles Arden-Clarke"], c: 0 }, // [cite: 8, 9]
        { cat: "GENERAL KNOWLEDGE", q: "Which Act currently governs the operations of the Ghana Immigration Service (repealing PNDC Law 226)?", a: ["Immigration Service Act, 2016 (Act 908)", "Police Service Act, 1970", "Aliens Act, 1963", "Interior Ministry Act, 2000"], c: 0 }, // [cite: 10]
        { cat: "GENERAL KNOWLEDGE", q: "What is the highest rank in the Ghana Immigration Service?", a: ["Commissioner of Immigration", "Comptroller-General of Immigration", "Inspector General", "Deputy Comptroller-General"], c: 1 }, // [cite: 11]
        { cat: "GENERAL KNOWLEDGE", q: "The Immigration Service Academy and Training School (ISATS) is located at:", a: ["Tesano, Accra", "Winneba", "Assin Fosu", "Koforidua"], c: 2 }, // [cite: 11]
        { cat: "GENERAL KNOWLEDGE", q: "Which of these is NOT a core function of the GIS?", a: ["Issuing Ghanaian Passports", "Regulating entry/exit", "Issuing Residence Permits", "Border Patrol Management"], c: 0 }, // [cite: 12]
        { cat: "GENERAL KNOWLEDGE", q: "What is the minimum educational requirement for a Graduate Entry (Cadet) applicant?", a: ["WASSCE", "First Degree", "BECE", "Masters"], c: 1 }, // [cite: 14, 15]
        { cat: "GENERAL KNOWLEDGE", q: "Which document is primarily required for a foreigner to work legally in Ghana?", a: ["Birth Certificate", "Work and Residence Permit", "Driver's License", "Voter ID"], c: 1 }, // [cite: 16]
        { cat: "GENERAL KNOWLEDGE", q: "Who is the current Minister for the Interior (as of Nov 2025)?", a: ["Hon. Ambrose Dery", "Hon. Henry Quartey", "Hon. Muntaka Mohammad-Mubarak", "Hon. Dominic Nitiwul"], c: 2 }, // [cite: 17, 18]
        { cat: "GENERAL KNOWLEDGE", q: "Which unit in GIS is responsible for patrolling borders to prevent smuggling?", a: ["Border Patrol Unit (BPU)", "Rapid Response Unit", "CID", "MTTD"], c: 0 }, // [cite: 19]
        { cat: "GENERAL KNOWLEDGE", q: "The 'Visa on Arrival' facility is typically valid for how many days initially?", a: ["60 days", "90 days", "30 days", "7 days"], c: 2 }, // [cite: 20]
        { cat: "GENERAL KNOWLEDGE", q: "What is the lowest rank for a Junior Officer in the GIS?", a: ["Assistant Immigration Control Officer II (AICO II)", "Immigration Control Officer (ICO)", "Inspector", "Constable"], c: 0 }, // [cite: 22]
        { cat: "GENERAL KNOWLEDGE", q: "The color of the Ghana Immigration Service uniform for general duty is:", a: ["Black and White", "Camouflage", "Olive Green", "Blue"], c: 2 }, // [cite: 22]
        { cat: "GENERAL KNOWLEDGE", q: "What does ECOWAS stand for?", a: ["Economic Community of West African States", "Eastern Community...", "Economic Council...", "Economic Center..."], c: 0 }, // [cite: 23]
        { cat: "GENERAL KNOWLEDGE", q: "Citizens of ECOWAS member states can enter Ghana without a visa for how long?", a: ["30 Days", "60 Days", "90 Days", "Forever"], c: 2 }, // [cite: 24]
        { cat: "GENERAL KNOWLEDGE", q: "Which specialized unit handles document fraud and expertise?", a: ["Migration Management Bureau", "Document Fraud Expertise Centre (DFEC)", "Intelligence Unit", "Operations Unit"], c: 1 }, // [cite: 25]
        { cat: "GENERAL KNOWLEDGE", q: "When does the 2025/2026 GIS Recruitment application officially close?", a: ["Nov 30, 2025", "Dec 19, 2025", "Jan 1, 2026", "Dec 15, 2025"], c: 3 }, // [cite: 27]
        { cat: "GENERAL KNOWLEDGE", q: "Who appoints the Comptroller-General of Immigration?", a: ["Police Council", "Public Services Commission", "The President of Ghana", "Chief Justice"], c: 2 }, // [cite: 28]
        { cat: "GENERAL KNOWLEDGE", q: "Which of the following is considered a 'travel document'?", a: ["Ghana Card", "Passport", "Birth Certificate", "NHIS Card"], c: 1 }, // [cite: 29]
        { cat: "GENERAL KNOWLEDGE", q: "The 'Aliens Compliance Order' of 1969 was enforced by which government?", a: ["CPP (Nkrumah)", "PP (Busia)", "NDC (Rawlings)", "NPP (Kufuor)"], c: 1 }, // [cite: 30]
        { cat: "GENERAL KNOWLEDGE", q: "The Ghana Immigration Service Tactical Training School is located at:", a: ["Kyebi", "Tepa", "Accra", "Ho"], c: 0 }, // [cite: 31]
        { cat: "GENERAL KNOWLEDGE", q: "Core values of GIS include Professionalism, Integrity, and _____?", a: ["Brutality", "Human Rights", "Secrecy", "Speed"], c: 1 }, // [cite: 32]

        // SECTION B: MATH [cite: 33]
        { cat: "MATHEMATICS", q: "Solve for x: 4x - 8 = 20", a: ["5", "6", "7", "8"], c: 2 }, // [cite: 33]
        { cat: "MATHEMATICS", q: "A car travels 120 km in 2 hours. What is its average speed?", a: ["50 km/h", "60 km/h", "100 km/h", "240 km/h"], c: 1 }, // [cite: 34]
        { cat: "MATHEMATICS", q: "What is 15% of 200?", a: ["20", "25", "30", "35"], c: 2 }, // [cite: 35]
        { cat: "MATHEMATICS", q: "If you buy a shirt for GH 80 and sell it for GH 100, what is your percentage profit?", a: ["20%", "25%", "10%", "50%"], c: 1 }, // [cite: 36]
        { cat: "MATHEMATICS", q: "Simplify: 1/2 + 1/3", a: ["2/5", "1/6", "5/6", "2/3"], c: 2 }, // [cite: 37]
        { cat: "MATHEMATICS", q: "Complete the series: 5, 10, 20, 40, ...", a: ["60", "80", "50", "100"], c: 1 }, // [cite: 37]
        { cat: "MATHEMATICS", q: "What is the square root of 225?", a: ["12", "13", "15", "25"], c: 2 }, // [cite: 38]
        { cat: "MATHEMATICS", q: "Convert 0.75 to a fraction.", a: ["1/4", "3/5", "3/4", "2/3"], c: 2 }, // [cite: 39]
        { cat: "MATHEMATICS", q: "In a class of 50 students, 30 are boys. What is the ratio of boys to girls?", a: ["3:2", "2:3", "3:5", "1:1"], c: 0 }, // [cite: 40]
        { cat: "MATHEMATICS", q: "If P=4 and Q=5, find the value of 2P+3Q", a: ["18", "20", "23", "25"], c: 2 }, // [cite: 42]
        { cat: "MATHEMATICS", q: "A rectangle has a length of 10cm and width of 4cm. What is the area?", a: ["14cm²", "20cm²", "40cm²", "80cm²"], c: 2 }, // [cite: 43]
        { cat: "MATHEMATICS", q: "How many seconds are there in 1 hour?", a: ["60", "360", "3600", "6000"], c: 2 }, // [cite: 44]
        { cat: "MATHEMATICS", q: "Which of these numbers is an odd number?", a: ["452", "331", "220", "108"], c: 1 }, // [cite: 45]
        { cat: "MATHEMATICS", q: "Divide 450 by 9.", a: ["40", "45", "50", "60"], c: 2 }, // [cite: 45]
        { cat: "MATHEMATICS", q: "If today is Monday, what day will it be in 10 days?", a: ["Wednesday", "Thursday", "Friday", "Saturday"], c: 1 }, // [cite: 47]
        { cat: "MATHEMATICS", q: "The price of fuel increased from GH 10 to GH 12. What is the percentage increase?", a: ["10%", "20%", "25%", "50%"], c: 1 }, // [cite: 48]
        { cat: "MATHEMATICS", q: "Evaluate: (5+5) x (4-2)", a: ["10", "20", "0", "15"], c: 1 }, // [cite: 48]
        { cat: "MATHEMATICS", q: "A man is 4 times as old as his son. If the son is 10, how old is the man?", a: ["30", "40", "50", "20"], c: 1 }, // [cite: 50]
        { cat: "MATHEMATICS", q: "What is the value of Pi (π) to two decimal places?", a: ["3.12", "3.14", "3.41", "2.14"], c: 1 }, // [cite: 51]
        { cat: "MATHEMATICS", q: "Which is greater? 0.5 or 0.05?", a: ["0.5", "0.05", "Equal", "N/A"], c: 0 }, // [cite: 52]

        // SECTION C: ENGLISH [cite: 53]
        { cat: "ENGLISH", q: "Choose the correct spelling:", a: ["Accomodation", "Accommodation", "Acommodation", "Acomodation"], c: 1 }, // [cite: 54]
        { cat: "ENGLISH", q: "The opposite of 'Illegal' is:", a: ["Legal", "Lawless", "Criminal", "Banned"], c: 0 }, // [cite: 54]
        { cat: "ENGLISH", q: "Choose the synonym for 'Vigilance':", a: ["Carelessness", "Sleep", "Alertness", "Ignorance"], c: 2 }, // [cite: 54]
        { cat: "ENGLISH", q: "Fill in the blank: The officer ____ the suspect at the border.", a: ["arrest", "arrested", "arresting", "arrests"], c: 1 }, // [cite: 54]
        { cat: "ENGLISH", q: "'To sit on the fence' means:", a: ["To sit on a wall", "To be lazy", "To be undecided", "To guard a house"], c: 2 }, // [cite: 56]
        { cat: "ENGLISH", q: "Which sentence is correct?", a: ["He have a passport.", "He has a passport.", "He having a passport.", "He had have a passport."], c: 1 }, // [cite: 58]
        { cat: "ENGLISH", q: "Identify the noun: 'The bus arrived late.'", a: ["Arrived", "Late", "Bus", "The"], c: 2 }, // [cite: 59]
        { cat: "ENGLISH", q: "Choose the correct preposition: He is accused ____ theft.", a: ["with", "for", "of", "by"], c: 2 }, // [cite: 61]
        { cat: "ENGLISH", q: "What is the plural of 'Passer-by'?", a: ["Passer-bys", "Passers-by", "Passers-bys", "Passer-byes"], c: 1 }, // [cite: 60]
        { cat: "ENGLISH", q: "'The team was ____ to the border.'", a: ["send", "sent", "sending", "sended"], c: 1 }, // [cite: 61]
        { cat: "ENGLISH", q: "Choose the correct word: The officer checked my ____.", a: ["dairy", "diary", "diery", "daury"], c: 1 }, // [cite: 63]
        { cat: "ENGLISH", q: "Antonym of 'Entrance':", a: ["Entry", "Exit", "Access", "Door"], c: 1 }, // [cite: 63]
        { cat: "ENGLISH", q: "Synonym of 'Prohibit':", a: ["Allow", "Permit", "Forbid", "Force"], c: 2 }, // [cite: 63]
        { cat: "ENGLISH", q: "'A person who enters a country illegally' is called:", a: ["Expatriate", "Irregular Migrant", "Tourist", "Citizen"], c: 1 }, // [cite: 64]
        { cat: "ENGLISH", q: "Choose the correctly spelt word:", a: ["Maintenance", "Maintainance", "Mentenance", "Maintenence"], c: 0 }, // [cite: 64]
        { cat: "ENGLISH", q: "Which of these is a verb?", a: ["Run", "Fast", "Runner", "Quickly"], c: 0 }, // [cite: 65]
        { cat: "ENGLISH", q: "'He didn't know ____ to go.'", a: ["wear", "where", "were", "we're"], c: 1 }, // [cite: 65]
        { cat: "ENGLISH", q: "Proverb: 'Birds of a feather ____'", a: ["fly away", "flock together", "sing together", "eat together"], c: 1 }, // [cite: 66]
        { cat: "ENGLISH", q: "The past tense of 'Go' is:", a: ["Gone", "Went", "Going", "Goes"], c: 1 }, // [cite: 66]
        { cat: "ENGLISH", q: "'Patriotism' refers to:", a: ["Love for family", "Love for money", "Love for country", "Dislike of foreigners"], c: 2 }, // [cite: 66]

        // SECTION D: LOGIC & GK [cite: 67]
        { cat: "LOGIC / GK", q: "Ghana shares a border to the North with:", a: ["Togo", "Cote d'Ivoire", "Burkina Faso", "Nigeria"], c: 2 }, // [cite: 68]
        { cat: "LOGIC / GK", q: "Which of these is NOT an arm of Government?", a: ["Executive", "Legislature", "Media", "Judiciary"], c: 2 }, // [cite: 69]
        { cat: "LOGIC / GK", q: "Dictionary is to Words, as Atlas is to ____?", a: ["Books", "Maps", "Earth", "Schools"], c: 1 }, // [cite: 69]
        { cat: "LOGIC / GK", q: "Find the odd one out:", a: ["Cedi", "Dollar", "Euro", "Visa"], c: 3 }, // [cite: 69]
        { cat: "LOGIC / GK", q: "Who is the current President of Ghana (as of Nov 2025)?", a: ["Nana Akufo-Addo", "John Dramani Mahama", "Dr. Mahamudu Bawumia", "Alan Kyerematen"], c: 1 }, // [cite: 72]
        { cat: "LOGIC / GK", q: "Which region is the GIS Headquarters located in?", a: ["Ashanti", "Greater Accra", "Central", "Eastern"], c: 2 }, // [cite: 73]
        { cat: "LOGIC / GK", q: "What does 'Green' in the Ghana flag represent?", a: ["Gold", "Blood", "Forests/Vegetation", "Black Star"], c: 2 }, // [cite: 73]
        { cat: "LOGIC / GK", q: "A person who LEAVES their country to live elsewhere is an:", a: ["Emigrant", "Immigrant", "Tourist", "Expat"], c: 0 }, // [cite: 73]
        { cat: "LOGIC / GK", q: "Complete the series: January, March, May, ____", a: ["June", "July", "August", "September"], c: 1 }, // [cite: 74]
        { cat: "LOGIC / GK", q: "What is the main duty of the 'judiciary'?", a: ["Making laws", "Enforcing laws", "Interpreting laws", "Arresting criminals"], c: 2 }, // [cite: 75]
        { cat: "LOGIC / GK", q: "Which of these is a major border town in the Volta Region?", a: ["Elubo", "Aflao", "Paga", "Hamile"], c: 1 }, // [cite: 77]
        { cat: "LOGIC / GK", q: "Elubo is a border town connecting Ghana to:", a: ["Togo", "Burkina Faso", "Cote d'Ivoire", "Benin"], c: 2 }, // [cite: 77]
        { cat: "LOGIC / GK", q: "Paga is a border town located in which region?", a: ["Upper East", "Upper West", "Northern", "Savannah"], c: 0 }, // [cite: 78]
        { cat: "LOGIC / GK", q: "If A=1, B=2, C=3, what is the value of BAG?", a: ["6", "10", "8", "12"], c: 1 }, // [cite: 79]
        { cat: "LOGIC / GK", q: "Which item is prohibited across borders without declaration?", a: ["Clothing", "Illegal Drugs", "Books", "Food"], c: 1 }, // [cite: 80]
        { cat: "LOGIC / GK", q: "'Biometric details' usually refers to:", a: ["Name/Address", "Fingerprints/Facial Recognition", "Height/Weight", "Signature"], c: 2 }, // [cite: 82]
        { cat: "LOGIC / GK", q: "Why are borders patrolled?", a: ["To stop travel", "National Security/Prevention of smuggling", "Taxes only", "To welcome tourists"], c: 1 }, // [cite: 83]
        { cat: "LOGIC / GK", q: "Who is the Head of the Ghana Police Service?", a: ["IGP", "CDS", "CGI", "CFO"], c: 0 }, // [cite: 84]
        { cat: "LOGIC / GK", q: "'Deportation' means:", a: ["Allowing entry", "Forcing a foreigner to leave", "Giving a visa", "Arresting a citizen"], c: 1 }, // [cite: 84]
        { cat: "LOGIC / GK", q: "Doctor is to Hospital as Teacher is to ____?", a: ["Market", "School", "Farm", "Court"], c: 1 }, // [cite: 85]
        { cat: "LOGIC / GK", q: "Next number: 100, 90, 80, 70, ____", a: ["50", "60", "65", "55"], c: 1 }, // [cite: 85]
        { cat: "LOGIC / GK", q: "How many regions does Ghana have currently?", a: ["10", "14", "16", "12"], c: 2 }, // [cite: 87]
        { cat: "LOGIC / GK", q: "The capital city of the Ashanti Region is:", a: ["Sunyani", "Kumasi", "Obuasi", "Koforidua"], c: 1 }, // [cite: 87]
        { cat: "LOGIC / GK", q: "If a plane crashes on the border of Ghana and Togo, where do you bury survivors?", a: ["Ghana", "Togo", "No Man's Land", "You don't bury survivors"], c: 3 }, // [cite: 88]
        { cat: "LOGIC / GK", q: "Valid form of identification in Ghana:", a: ["Library Card", "Ghana Card", "Gym Card", "Business Card"], c: 1 }, // [cite: 89]
        { cat: "LOGIC / GK", q: "Primary purpose of a Visa:", a: ["ID", "Permission to enter", "Health status", "Birth date"], c: 1 }, // [cite: 90]
        { cat: "LOGIC / GK", q: "Who is the Vice President of Ghana (as of Nov 2025)?", a: ["Dr. Mahamudu Bawumia", "Prof. Naana Jane Opoku-Agyemang", "Amissah Arthur", "Aliu Mahama"], c: 1 }, // [cite: 93]
        { cat: "LOGIC / GK", q: "Which continent is Ghana located in?", a: ["Asia", "South America", "Africa", "Europe"], c: 2 }, // [cite: 94]
        { cat: "LOGIC / GK", q: "Official language of Ghana:", a: ["Twi", "Ga", "English", "French"], c: 2 }, // [cite: 95]
        { cat: "LOGIC / GK", q: "A 'Refugee' is someone who:", a: ["Travels for fun", "Moves for work", "Flees war/persecution", "Studies abroad"], c: 2 }, // [cite: 95]
        { cat: "LOGIC / GK", q: "Which body organizes national elections in Ghana?", a: ["GIS", "GPS", "Electoral Commission (EC)", "NCCE"], c: 2 }, // [cite: 96]
        { cat: "LOGIC / GK", q: "'Smuggling' involves:", a: ["Legal imports", "Paying taxes", "Moving goods illegally", "Buying in market"], c: 2 }, // [cite: 97]
        { cat: "LOGIC / GK", q: "Odd one out:", a: ["Lorry", "Bus", "Car", "Ship"], c: 3 }, // [cite: 97]
        { cat: "LOGIC / GK", q: "The GIS was formerly known as:", a: ["Border Guards", "Immigration and Passport Unit", "CEPS", "Prisons Service"], c: 1 }, // [cite: 98]
        { cat: "LOGIC / GK", q: "Why do you want to join the GIS?", a: ["Money", "Travel", "To serve the nation and protect borders", "Jobless"], c: 2 } // [cite: 98, 99]
    ];

    let current = 0;
    let scores = new Array(quizData.length).fill(null);
    let timeLeft = 3600; // 60 minutes

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
        const q = quizData[current];
        document.getElementById('q-num').innerText = current + 1;
        document.getElementById('q-cat').innerText = q.cat;
        document.getElementById('q-text').innerText = q.q;
        
        const optionsDiv = document.getElementById('q-options');
        optionsDiv.innerHTML = '';
        q.a.forEach((opt, i) => {
            const btn = document.createElement('button');
            btn.className = 'opt-btn' + (scores[current] === i ? ' selected' : '');
            btn.innerText = opt;
            btn.onclick = () => { scores[current] = i; render(); };
            optionsDiv.appendChild(btn);
        });

        document.getElementById('next-btn').innerText = (current === quizData.length - 1) ? "Final Submit" : "Next Question";
    }

    function move(step) {
        if (step === 1 && current === quizData.length - 1) { showResults(); return; }
        current = Math.max(0, Math.min(quizData.length - 1, current + step));
        render();
    }

    function showResults() {
        function showResults() {
    // Hide the quiz interface and show the results screen
    document.getElementById('quiz-ui').style.display = 'none';
    const resultsScreen = document.getElementById('results-screen');
    resultsScreen.style.display = 'block';
    
    // Calculate the total number of correct answers
    let correctCount = scores.filter((ans, i) => ans === quizData[i].c).length;
    let totalQuestions = quizData.length;
    let percent = Math.round((correctCount / totalQuestions) * 100);
    
    // Update the score display
    document.getElementById('final-score').innerText = percent + "%";
    
    // Build the Review List
    let reviewHTML = `
        <div style="margin-top:30px; text-align:left;">
            <h3 style="color:var(--primary); border-bottom:2px solid var(--primary); padding-bottom:10px;">
                Detailed Review
            </h3>
            <div style="max-height:500px; overflow-y:auto; padding:15px; background:#fff; border:1px solid #ddd; border-radius:8px;">
    `;
    
    quizData.forEach((q, i) => {
        const userSelection = scores[i];
        const correctIndex = q.c;
        const isCorrect = userSelection === correctIndex;
        
        reviewHTML += `
            <div style="margin-bottom:20px; padding-bottom:15px; border-bottom:1px dashed #ccc;">
                <p style="font-weight:bold; margin-bottom:5px;">${i + 1}. ${q.q}</p>
                <p style="margin:2px 0;">
                    <span style="color:${isCorrect ? '#2e7d32' : '#d32f2f'}; font-weight:600;">
                        Your Answer: ${userSelection !== null ? q.a[userSelection] : 'Skipped'}
                    </span>
                </p>
                ${!isCorrect ? `
                    <p style="margin:2px 0; color:#2e7d32; font-weight:600;">
                        Correct Answer: ${q.a[correctIndex]}
                    </p>` : ''}
            </div>
        `;
    });
    
    reviewHTML += `</div></div>`;
    
    // Inject the review content into the summary text area
    document.getElementById('summary-text').innerHTML = `
        <p style="font-size:1.2rem;">You answered <strong>${correctCount}</strong> out of <strong>${totalQuestions}</strong> correctly.</p>
    ` + reviewHTML;
}
    }

    startTimer();
    render();
</script>

</body>
</html>
