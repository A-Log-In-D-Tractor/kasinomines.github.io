<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kasino Mines Interface</title>
    <style id="main-styles">
        body {
            font-family: 'Segoe UI', Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 20px;
            background-color: #121212;
            color: #e0e0e0;
            margin: 0;
        }
        #pip-instructions {
            margin-bottom: 15px;
            color: #ff8c00;
            font-size: 0.9em;
        }
        .container {
            display: flex;
            flex-direction: row;
            gap: 20px;
            background: #1e1e1e;
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
            border: 1px solid #333;
        }
        .left-panel {
            display: flex;
            flex-direction: column;
            gap: 15px;
            width: 180px;
        }
        .right-panel {
            min-width: 250px;
            display: flex;
            justify-content: center;
            align-items: center;
            background: #252525;
            border-radius: 10px;
            padding: 10px;
        }
        #grid {
            display: grid;
            gap: 4px;
        }
        .cell {
            cursor: pointer;
            font-size: 24px;
            user-select: none;
            transition: transform 0.1s;
        }
        .cell:hover { transform: scale(1.1); }
        
        .controls label {
            display: block;
            margin-bottom: 5px;
            font-weight: 600;
            color: #bbb;
            font-size: 0.85em;
        }
        input {
            background: #333;
            border: 1px solid #444;
            color: white;
            padding: 6px;
            border-radius: 4px;
            width: 100%;
            box-sizing: border-box;
        }
        input[type="range"] { padding: 0; }

        button {
            padding: 10px;
            cursor: pointer;
            border: none;
            border-radius: 6px;
            font-weight: bold;
            transition: all 0.1s ease;
        }

        #btn-pip {
            background: #6f42c1;
            color: white;
            margin-bottom: 10px;
            width: 200px;
        }

        #btn-dollar {
            background: #007bff;
            color: white;
            box-shadow: 0 4px #0056b3;
            transform: translateY(0);
        }
        #btn-dollar.active {
            background: #28a745;
            box-shadow: 0 1px #1e7e34;
            transform: translateY(3px);
        }

        #btn-copy {
            background: #ff8c00;
            color: white;
            margin-top: 5px;
        }
        
        #coord-display {
            margin-top: 20px;
            width: 100%;
            max-width: 500px;
            padding: 12px;
            font-family: 'Courier New', monospace;
            font-size: 0.9em;
            background: #000;
            color: #00ff00;
            border: 1px solid #333;
            border-radius: 8px;
            text-align: center;
        }
    </style>
</head>
<body>

    <div id="pip-instructions">Pop out to get an overlay</div>
    <button id="btn-pip">🗔 Pop Out</button>

    <div class="container" id="main-content">
        <div class="left-panel">
            <div class="controls">
                <label>Size: <span id="size-val">5</span></label>
                <input type="range" id="size-slider" min="2" max="10" value="5">
            </div>

            <div class="controls">
                <label>Mines:</label>
                <input type="number" id="mine-text" value="3" min="1">
                <input type="range" id="mine-slider" min="1" value="3">
            </div>

            <div class="controls">
                <label>Bet $:</label>
                <input type="number" id="bet-text" value="0" min="0">
            </div>

            <button id="btn-dollar">$</button>
            <button id="btn-copy">Copy Command</button>
        </div>

        <div class="right-panel">
            <div id="grid"></div>
        </div>
    </div>

    <input type="text" id="coord-display" readonly>

<script>
    const sizeSlider = document.getElementById('size-slider');
    const sizeValDisplay = document.getElementById('size-val');
    const mineText = document.getElementById('mine-text');
    const mineSlider = document.getElementById('mine-slider');
    const betText = document.getElementById('bet-text');
    const dollarBtn = document.getElementById('btn-dollar');
    const grid = document.getElementById('grid');
    const coordDisplay = document.getElementById('coord-display');
    const copyBtn = document.getElementById('btn-copy');
    const pipBtn = document.getElementById('btn-pip');
    const mainContent = document.getElementById('main-content');

    let clickedCoordinates = [];
    let isCashoutActive = false;

    // Toggle $ Button
    dollarBtn.onclick = () => {
        isCashoutActive = !isCashoutActive;
        dollarBtn.classList.toggle('active');
        updateDisplay();
    };

    function updateDisplay() {
        const size = sizeSlider.value;
        const mines = mineText.value;
        const bet = betText.value || 0;
        const coords = clickedCoordinates.join(' ');
        const cashoutSuffix = isCashoutActive ? " cashout" : "";
        let output = `!mines ${bet} ${size} ${mines} ${coords} ${cashoutSuffix}`;
        coordDisplay.value = output.trim().replace(/\s\s+/g, ' ');
    }

    function generateGrid() {
        const size = parseInt(sizeSlider.value);
        sizeValDisplay.textContent = size;
        const maxMines = Math.pow(size, 2) - 1;
        mineText.max = maxMines;
        mineSlider.max = maxMines;
        if (parseInt(mineText.value) > maxMines) {
            mineText.value = maxMines;
            mineSlider.value = maxMines;
        }

        grid.innerHTML = '';
        grid.style.gridTemplateColumns = `repeat(${size}, 1fr)`;
        clickedCoordinates = [];
        
        for (let r = 1; r <= size; r++) {
            for (let c = 1; c <= size; c++) {
                const cell = document.createElement('div');
                cell.className = 'cell';
                cell.textContent = '⬜';
                const coordStr = `${r},${c}`;
                
                cell.onclick = () => {
                    if (cell.textContent === '⬜') {
                        cell.textContent = '🟧';
                        clickedCoordinates.push(coordStr);
                    } else {
                        cell.textContent = '⬜';
                        clickedCoordinates = clickedCoordinates.filter(item => item !== coordStr);
                    }
                    updateDisplay();
                };
                grid.appendChild(cell);
            }
        }
        updateDisplay();
    }

    // Input Events
    mineText.oninput = () => { mineSlider.value = mineText.value; updateDisplay(); };
    mineSlider.oninput = () => { mineText.value = mineSlider.value; updateDisplay(); };
    betText.oninput = updateDisplay;
    sizeSlider.oninput = generateGrid;

    copyBtn.onclick = () => {
        coordDisplay.select();
        document.execCommand('copy');
        const originalText = copyBtn.textContent;
        copyBtn.textContent = 'Copied!';
        setTimeout(() => copyBtn.textContent = originalText, 1000);
    };

    // --- POP OUT LOGIC ---
    pipBtn.onclick = async () => {
        if (!('documentPictureInPicture' in window)) {
            alert("Your browser doesn't support the Pop Out API yet. Try Chrome or Edge.");
            return;
        }

        // Open the PiP window
        const pipWindow = await window.documentPictureInPicture.requestWindow({
            width: mainContent.clientWidth + 40,
            height: mainContent.clientHeight + 150,
        });

        // Copy styles to the new window so it looks right
        const style = document.createElement('style');
        style.textContent = document.getElementById('main-styles').textContent;
        pipWindow.document.head.appendChild(style);

        // Move the content and the coordinate display into the PiP window
        pipWindow.document.body.style.backgroundColor = "#121212";
        pipWindow.document.body.append(mainContent);
        pipWindow.document.body.append(coordDisplay);

        // When the PiP window closes, move everything back to the main page
        pipWindow.addEventListener("pagehide", (event) => {
            document.body.append(mainContent);
            document.body.append(coordDisplay);
        });
    };

    generateGrid();
</script>
</body>
</html>
