<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Reaction Timer</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            padding: 30px;
        }
        #introModal {
            position: fixed;
            top: 0; left: 0;
            width: 100vw; height: 100vh;
            background: rgba(0, 0, 0, 0.7);
            color: white;
            display: none;
            align-items: center;
            justify-content: center;
            flex-direction: column;
            z-index: 999;
            padding: 20px;
            box-sizing: border-box;
        }
        #introModal p {
            font-size: 20px;
            max-width: 600px;
            line-height: 1.5;
        }
        input, button {
            padding: 10px;
            margin: 10px;
            font-size: 16px;
        }
        #message {
            margin-top: 20px;
            font-size: 20px;
        }
    </style>
</head>
<body>

    <!-- Instruction Modal -->
    <div id="introModal">
        <p>
            Welcome to the Reaction Timer Test!<br><br>
            You will begin with a practice round.<br><br>
            When you're ready, press and hold the <strong>spacebar</strong>.<br>
            Wait for the "Now!" prompt to appear, then <strong>release and press space again</strong> as quickly as you can.<br><br>
            After the practice, you'll complete 10 official attempts.<br>
            At the end, your <strong>average reaction time</strong> will be shown.<br><br>
            Click OK to begin.
        </p>
        <button id="okButton">OK</button>
    </div>

    <!-- Main UI -->
    <div>
        <input id="nameInput" placeholder="Enter your name" />
        <button id="startButton">Start</button>
        <button id="restartButton" style="display: none;">Restart</button>
        <p id="message"></p>
    </div>

    <script>
        const message = document.getElementById('message');
        const startButton = document.getElementById('startButton');
        const restartButton = document.getElementById('restartButton');
        const nameInput = document.getElementById('nameInput');
        const introModal = document.getElementById('introModal');
        const okButton = document.getElementById('okButton');

        let state = 'idle';
        let startTime, endTime, reactionTime;
        let waitTimeout;
        let isPracticeRound = true;
        let attemptCount = 0;
        let results = [];
        let spaceDown = false;
        let spaceReleasedAfterPrompt = false;
        let attemptInProgress = false;

        // Show instructions modal on page load
        window.onload = () => {
            introModal.style.display = 'flex';
        };

        // Hide modal and show message
        okButton.addEventListener('click', () => {
            introModal.style.display = 'none';
            message.textContent = "Press Start for your practice try";
        });

        startButton.addEventListener('click', () => {
            if (!nameInput.value) {
                message.textContent = "Please enter your name.";
                return;
            }

            if (state !== 'idle') return;

            startButton.style.display = 'none';
            message.textContent = "Hold down the spacebar to begin.";
            state = 'waitingToStart';
            attemptInProgress = false;
        });

        restartButton.addEventListener('click', () => {
            resetTest();
        });

        document.addEventListener('keydown', (e) => {
            if (e.code !== 'Space') return;
            if (spaceDown) return;
            if (!['waitingToStart', 'readyToReact'].includes(state)) return;

            spaceDown = true;

            if (state === 'readyToReact' && spaceReleasedAfterPrompt && !attemptInProgress) {
                endTime = Date.now();
                reactionTime = endTime - startTime;
                attemptInProgress = true;
                finishAttempt();
                state = 'completed';
            }
        });

        document.addEventListener('keyup', (e) => {
            if (e.code !== 'Space') return;
            spaceDown = false;

            if (state === 'waitingToStart') {
                startReactionSequence();
            }

            if (state === 'readyToReact') {
                spaceReleasedAfterPrompt = true;
                message.textContent = "Now! Press space again!";
            }
        });

        function startReactionSequence() {
            message.textContent = "Hold space... wait for the prompt...";
            state = 'waitingForPrompt';
            attemptInProgress = false;
            spaceReleasedAfterPrompt = false;

            const delay = Math.floor(Math.random() * 5000) + 5000;

            waitTimeout = setTimeout(() => {
                startTime = Date.now();
                message.textContent = "Now! Press space!";
                state = 'readyToReact';
            }, delay);
        }

        function finishAttempt() {
            clearTimeout(waitTimeout);

            if (isPracticeRound) {
                message.innerHTML = `${nameInput.value}, your practice reaction time: ${reactionTime} ms<br><br>Get ready for your 10 official attempts.`;
                isPracticeRound = false;
                setTimeout(() => {
                    startReactionSequence();
                }, 2000);
            } else {
                attemptCount++;
                results.push(reactionTime);

                if (attemptCount < 10) {
                    message.innerHTML = `${nameInput.value}, attempt ${attemptCount} of 10: ${reactionTime} ms<br><br>Next round starting...`;
                    setTimeout(() => {
                        startReactionSequence();
                    }, 2000);
                } else {
                    const average = Math.round(results.reduce((a, b) => a + b, 0) / results.length);
                    message.innerHTML = `
                        ${nameInput.value}, attempt 10 of 10: ${reactionTime} ms<br><br>
                        <strong>Your average reaction time: ${average} ms</strong><br><br>
                        Test complete! You can restart if you'd like.
                    `;
                    restartButton.style.display = 'inline-block';
                }
            }

            state = 'idle';
            attemptInProgress = false;
            spaceReleasedAfterPrompt = false;
            startButton.style.display = 'none';
        }

        function resetTest() {
            state = 'idle';
            isPracticeRound = true;
            attemptCount = 0;
            results = [];
            spaceDown = false;
            spaceReleasedAfterPrompt = false;
            attemptInProgress = false;
            message.textContent = "Press Start for your practice try";
            startButton.style.display = 'inline-block';
            restartButton.style.display = 'none';
        }
    </script>

</body>
</html>
