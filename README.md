# Math-quiz

<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>मैथ क्विज - मल्टीप्लेयर</title>
    <style>
        body {
            font-family: 'Arial Rounded MT Bold', Arial, sans-serif;
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            color: white;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
            transition: background 0.5s;
        }

        .game-container {
            background: rgba(255, 255, 255, 0.1);
            padding: 2rem;
            border-radius: 15px;
            box-shadow: 0 0 20px rgba(0,0,0,0.3);
            text-align: center;
            max-width: 600px;
            width: 90%;
            transition: background 0.5s;
        }

        .stats {
            display: flex;
            justify-content: space-around;
            margin-bottom: 2rem;
        }

        .question-box {
            font-size: 2.5rem;
            margin: 2rem 0;
            padding: 1.5rem;
            background: rgba(255,255,255,0.1);
            border-radius: 10px;
            animation: float 3s ease-in-out infinite;
        }

        input {
            font-size: 1.5rem;
            padding: 0.8rem;
            width: 150px;
            text-align: center;
            border: none;
            border-radius: 8px;
            margin: 1rem;
        }

        button {
            font-size: 1.2rem;
            padding: 1rem 2rem;
            border: none;
            border-radius: 8px;
            background: #4CAF50;
            color: white;
            cursor: pointer;
            transition: transform 0.2s;
            margin: 0.5rem;
        }

        button:hover {
            transform: scale(1.1);
            background: #45a049;
        }

        .hearts {
            display: flex;
            gap: 0.5rem;
        }

        .heart {
            width: 30px;
            height: 30px;
            background: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="%23ff0000"><path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/></svg>');
        }

        .feedback {
            font-size: 1.2rem;
            margin: 1rem 0;
            min-height: 2rem;
        }

        @keyframes float {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-10px); }
        }

        .correct { animation: correct 0.5s; }
        @keyframes correct {
            50% { transform: scale(1.2); color: #4CAF50; }
        }

        .incorrect { animation: incorrect 0.5s; }
        @keyframes incorrect {
            50% { transform: translateX(10px); color: #ff4444; }
        }

        .progress-bar {
            width: 100%;
            height: 10px;
            background: rgba(255,255,255,0.2);
            border-radius: 5px;
            margin: 1rem 0;
            overflow: hidden;
        }

        .progress {
            height: 100%;
            background: #4CAF50;
            width: 0;
            transition: width 0.5s;
        }

        .power-up {
            font-size: 1rem;
            padding: 0.5rem;
            background: #FFD700;
            color: black;
            border-radius: 5px;
            margin: 0.5rem;
            display: inline-block;
            animation: pop 0.5s;
        }

        @keyframes pop {
            50% { transform: scale(1.2); }
        }

        .multiplayer-options {
            margin: 1rem 0;
        }

        .multiplayer-options input {
            width: 200px;
            margin: 0.5rem;
        }
    </style>
</head>
<body>
    <div class="game-container">
        <h1>🎮 मैथ क्विज - मल्टीप्लेयर</h1>
        <div class="multiplayer-options" id="multiplayer-options">
            <button onclick="hostGame()">होस्ट गेम</button>
            <button onclick="joinGame()">जॉइन गेम</button>
            <input type="text" id="game-code" placeholder="गेम कोड दर्ज करें" style="display: none;">
        </div>
        <div class="stats">
            <div>⭐ स्कोर: <span id="score">0</span></div>
            <div>❤️ लाइव्स: <span class="hearts" id="lives"></span></div>
            <div>⏱️ समय: <span id="timer">0:00</span></div>
        </div>
        <div class="progress-bar">
            <div class="progress" id="progress"></div>
        </div>
        <div class="question-box" id="question"></div>
        <input type="number" id="answer" placeholder="जवाब दें" autofocus style="display: none;">
        <div class="feedback" id="feedback"></div>
        <button onclick="checkAnswer()" style="display: none;">जमा करें</button>
        <div id="power-ups"></div>
    </div>

    <audio id="correct-sound" src="https://assets.mixkit.co/active_storage/sfx/2211/2211-preview.mp3"></audio>
    <audio id="incorrect-sound" src="https://assets.mixkit.co/active_storage/sfx/2747/2747-preview.mp3"></audio>

    <script>
        let score = 0;
        let lives = 3;
        let level = 1;
        let time = 0;
        let timerInterval;
        let currentAnswer;
        let powerUps = [];
        let socket;
        let isHost = false;
        let gameCode;

        const operations = [
            { symbol: '+', fn: (a, b) => a + b },
            { symbol: '-', fn: (a, b) => a - b },
            { symbol: '×', fn: (a, b) => a * b },
            { symbol: '÷', fn: (a, b) => a / b }
        ];

        const positiveFeedback = [
            "🎉 वाह! शानदार!", "🚀 बहुत खूब!", "💡 जीनियस!", 
            "🌟 बिल्कुल सही!", "🔥 आप तो धुआंधार हैं!", "✅ एकदम सटीक!"
        ];

        function hostGame() {
            isHost = true;
            document.getElementById('multiplayer-options').style.display = 'none';
            document.getElementById('answer').style.display = 'block';
            document.querySelector('button').style.display = 'block';
            gameCode = Math.random().toString(36).substring(2, 6).toUpperCase();
            alert(`आपका गेम कोड: ${gameCode}\nइसे दोस्तों के साथ शेयर करें!`);
            connectWebSocket();
        }

        function joinGame() {
            document.getElementById('game-code').style.display = 'block';
            document.getElementById('game-code').addEventListener('keypress', (e) => {
                if (e.key === 'Enter') {
                    gameCode = document.getElementById('game-code').value.toUpperCase();
                    connectWebSocket();
                }
            });
        }

        function connectWebSocket() {
            socket = new WebSocket('wss://your-websocket-server-url'); // Replace with your WebSocket server URL
            socket.onopen = () => {
                if (isHost) {
                    socket.send(JSON.stringify({ type: 'host', code: gameCode }));
                } else {
                    socket.send(JSON.stringify({ type: 'join', code: gameCode }));
                }
            };
            socket.onmessage = (event) => {
                const data = JSON.parse(event.data);
                if (data.type === 'question') {
                    document.getElementById('question').textContent = data.question;
                    currentAnswer = data.answer;
                } else if (data.type === 'feedback') {
                    document.getElementById('feedback').textContent = data.message;
                    document.getElementById('feedback').className = `feedback ${data.status}`;
                }
            };
        }

        function generateQuestion() {
            const maxNumber = 5 + (level * 3);
            const num1 = Math.floor(Math.random() * maxNumber) + 1;
            const num2 = Math.floor(Math.random() * maxNumber) + 1;
            const op = operations[Math.floor(Math.random() * operations.length)];
            
            let question, answer;
            if(op.symbol === '÷') {
                answer = num1;
                question = `${num1 * num2} ${op.symbol} ${num2}`;
            } else {
                answer = op.fn(num1, num2);
                question = `${num1} ${op.symbol} ${num2}`;
            }
            
            currentAnswer = answer;
            if (isHost) {
                socket.send(JSON.stringify({ type: 'question', question, answer }));
            }
            document.getElementById('question').textContent = question;
        }

        function checkAnswer() {
            const userAnswer = parseFloat(document.getElementById('answer').value);
            const feedbackEl = document.getElementById('feedback');
            
            if(userAnswer === currentAnswer) {
                score++;
                level = 1 + Math.floor(score / 3);
                feedbackEl.textContent = positiveFeedback[Math.floor(Math.random() * positiveFeedback.length)];
                feedbackEl.className = 'feedback correct';
                document.getElementById('correct-sound').play();
                if(score % 3 === 0) addPowerUp();
                generateQuestion();
            } else {
                lives--;
                feedbackEl.textContent = `❌ गलत जवाब! सही जवाब: ${currentAnswer}`;
                feedbackEl.className = 'feedback incorrect';
                document.getElementById('incorrect-sound').play();
                if(lives <= 0) endGame();
            }

            document.getElementById('score').textContent = score;
            updateLives();
            updateProgress();
            document.getElementById('answer').value = '';
        }

        function updateLives() {
            document.getElementById('lives').innerHTML = 
                Array(lives).fill('<div class="heart"></div>').join('');
        }

        function updateProgress() {
            const progress = (score % 3) * 33.33;
            document.getElementById('progress').style.width = `${progress}%`;
        }

        function updateTimer() {
            const minutes = Math.floor(time / 60);
            const seconds = time % 60;
            document.getElementById('timer').textContent = 
                `${minutes}:${seconds.toString().padStart(2, '0')}`;
            time++;
        }

        function endGame() {
            clearInterval(timerInterval);
            document.getElementById('answer').disabled = true;
            document.querySelector('button').disabled = true;
            document.getElementById('feedback').textContent = 
                `खेल समाप्त! अंतिम स्कोर: ${score} | समय: ${document.getElementById('timer').textContent}`;
        }

        // Start game
        document.getElementById('answer').addEventListener('keypress', (e) => {
            if(e.key === 'Enter') checkAnswer();
        });

        if (isHost) {
            generateQuestion();
            updateLives();
            timerInterval = setInterval(updateTimer, 1000);
        }
    </script>
</body>
</html>
