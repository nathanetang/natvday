# natvday
rah

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Word Hunt & Hangman</title>
    <style>
        body {
            background-color: rgb(76, 163, 114);
            font-family: Arial, sans-serif;
            text-align: center;
            touch-action: none;
            position: relative;
            transition: background 0.5s ease-in-out;
        }
        #grid {
            display: grid;
            grid-template-columns: repeat(4, 50px);
            gap: 5px;
            justify-content: center;
            margin-top: 10px;
            position: relative;
        }
        .cell {
            width: 50px;
            height: 50px;
            display: flex;
            align-items: center;
            justify-content: center;
            border: 2px solid #2c3e20;
            font-size: 20px;
            font-weight: bold;
            user-select: none;
            background-color: sandybrown;
            border-radius: 5px;
            box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
            position: relative;
        }
        #selectedWord {
            font-size: 18px;
            font-weight: bold;
            margin-bottom: 10px;
        }
        #hangmanContainer {
            margin-top: 20px;
            font-size: 24px;
            font-weight: bold;
        }
        .hangman-word {
            display: inline-block;
            text-align: center;
            margin: 10px;
        }
        .revealed {
            color: black;
            font-weight: bold;
        }
        canvas {
            position: absolute;
            top: 0;
            left: 0;
            pointer-events: none;
            z-index: 2;
        }
        /* Heart background when completed */
        .completed {
    background-color: lightpink !important; /* Solid light pink background */
    overflow: hidden; /* Prevent hearts from scrolling */
}

/* Floating heart styling */
.heart {
    position: absolute;
    color: red;
    animation: floatUp 8s linear infinite; /* Lasts longer */
    opacity: 0.8;
}

/* Floating animation */
@keyframes floatUp {
    0% {
        transform: translateY(0);
        opacity: 1;
    }
    100% {
        transform: translateY(-120vh);
        opacity: 0;
    }
}


    </style>
</head>
<body id="gameBody">
    <h1>Word Hunt</h1>
    <p>Fill in the blanks!</p>
    
    <p id="selectedWord">Selected Word: </p>

    <div id="grid"></div>
    <canvas id="lineCanvas"></canvas>

    <p id="result"></p>

    <div id="hangmanContainer"></div>

    <script>
        const words = ["WILL", "NAT", "BE", "MY", "VALENTINE"];
        const foundWords = new Set(); 
        const gridSize = 4;
        const gridLetters = [
            "W", "L", "E", "B",
            "I", "L", "N", "A",
            "A", "N", "T", "Y",
            "V", "I", "E", "M"
        ];
        const grid = document.getElementById("grid");
        const canvas = document.getElementById("lineCanvas");
        const ctx = canvas.getContext("2d");
        const body = document.getElementById("gameBody");
        let selectedCells = [];
        let selectedWord = "";
        let selecting = false;

        function resizeCanvas() {
            canvas.width = grid.offsetWidth;
            canvas.height = grid.offsetHeight;
            canvas.style.top = grid.offsetTop + "px";
            canvas.style.left = grid.offsetLeft + "px";
        }

        function createGrid() {
            grid.innerHTML = "";
            for (let i = 0; i < gridSize * gridSize; i++) {
                let cell = document.createElement("div");
                cell.classList.add("cell");
                cell.textContent = gridLetters[i];
                cell.dataset.index = i;
                
                cell.addEventListener("mousedown", startSelection);
                cell.addEventListener("mouseenter", selectCell);
                cell.addEventListener("mouseup", endSelection);

                cell.addEventListener("touchstart", startSelection);
                cell.addEventListener("touchmove", selectCellTouch);
                cell.addEventListener("touchend", endSelection);

                grid.appendChild(cell);
            }
            resizeCanvas();
        }

        function startSelection(event) {
            selecting = true;
            selectedCells = [];
            selectedWord = "";
            document.querySelectorAll(".cell").forEach(cell => cell.classList.remove("selected"));
            selectCell(event);
            document.addEventListener("mouseup", endSelection);
        }

        function selectCell(event) {
            if (!selecting) return;
            let cell = event.target;
            if (!selectedCells.includes(cell)) {
                selectedCells.push(cell);
                selectedWord += cell.textContent;
                document.getElementById("selectedWord").textContent = "Selected Word: " + selectedWord;
            }
            drawLines();
        }

        function selectCellTouch(event) {
            let touch = event.touches[0];
            let element = document.elementFromPoint(touch.clientX, touch.clientY);
            if (element && element.classList.contains("cell")) {
                selectCell({ target: element });
            }
        }

        function endSelection() {
            selecting = false;
            document.removeEventListener("mouseup", endSelection);
            checkWord();
            clearLines(); 
        }

        function checkWord() {
            if (words.includes(selectedWord)) {
                foundWords.add(selectedWord);
                document.getElementById("result").textContent = "Correct!";
            } else {
                document.getElementById("result").textContent = "Try again!";
            }
            updateHangmanDisplay();
            checkCompletion();
        }

        function updateHangmanDisplay() {
            const phraseWords = ["WILL", "NAT", "BE", "MY", "VALENTINE"];
            const hangmanContainer = document.getElementById("hangmanContainer");
            hangmanContainer.innerHTML = "";

            phraseWords.forEach(word => {
                const wordContainer = document.createElement("div");
                wordContainer.classList.add("hangman-word");

                if (foundWords.has(word)) {
                    wordContainer.innerHTML = `<span class="revealed">${word}</span><br>`;
                } else {
                    wordContainer.innerHTML = `<span>${"_ ".repeat(word.length).trim()}</span><br>`;
                }

                hangmanContainer.appendChild(wordContainer);
            });
        }

        function drawLines() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            if (selectedCells.length < 2) return;

            ctx.strokeStyle = "rgba(255, 255, 255, 0.7)";
            ctx.lineWidth = 6;
            ctx.lineCap = "round"; 
            ctx.beginPath();

            selectedCells.forEach((cell, index) => {
                let rect = cell.getBoundingClientRect();
                let gridRect = grid.getBoundingClientRect();
                let x = rect.left - gridRect.left + rect.width / 2;
                let y = rect.top - gridRect.top + rect.height / 2;
                if (index === 0) {
                    ctx.moveTo(x, y);
                } else {
                    ctx.lineTo(x, y);
                }
            });

            ctx.stroke();
        }

        function clearLines() {
            setTimeout(() => {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
            }, 150); 
        }

        function checkCompletion() {
    if (foundWords.size === words.length) {
        body.classList.add("completed");

        // Make grid transparent
        grid.style.opacity = "0.5";

        // Update hangman display with bigger font and a question mark
        const hangmanContainer = document.getElementById("hangmanContainer");
        hangmanContainer.style.fontSize = "40px";
        hangmanContainer.innerHTML = `<span class="revealed">WILL NAT BE MY VALENTINE?</span>`;

        // Start continuous heart animation
        setInterval(createHeart, 500); // Spawns a new heart every 0.5 seconds
    }
}

function createHeart() {
    let heart = document.createElement("div");
    heart.classList.add("heart");
    heart.innerHTML = "🤍"; // You can also use "💖" or "💜"

    // Random position and size
    heart.style.left = Math.random() * 100 + "vw";
    heart.style.fontSize = Math.random() * 30 + 10 + "px"; // Random size
    heart.style.animationDuration = Math.random() * 5 + 3 + "s"; // Random speed

    document.body.appendChild(heart);

    // Remove heart after animation completes
    setTimeout(() => {
        heart.remove();
    }, 8000); // Increased duration to keep hearts visible longer
}


        window.addEventListener("resize", resizeCanvas);
        createGrid();
        updateHangmanDisplay();
    </script>
</body>
</html>
