
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
            gap: 12px;
            width: 200px;
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
        #grid { display: grid; gap: 4px; }
        .cell {
            cursor: pointer;
            font-size: 24px;
            user-select: none;
            transition: transform 0.1s;
            width: 32px;
            height: 32px;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        .cell:hover:not(.locked) { transform: scale(1.1); }
        .cell.locked { cursor: default; }

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
        button {
            padding: 10px;
            cursor: pointer;
            border: none;
            border-radius: 6px;
            font-weight: bold;
            color: white;
            transition: all 0.1s ease;
        }
        #btn-pip { background: #6f42c1; margin-bottom: 10px; width: 200px; }
        #btn-dollar { background: #007bff; box-shadow: 0 4px #0056b3; }
        #btn-dollar.active { background: #28a745; box-shadow: 0 1px #1e7e34; transform: translateY(3px); }
        
        .action-row { display: flex; gap: 8px; }
        #btn-play { background: #dc3545; flex: 1; }
        #btn-play:disabled { background: #555; cursor: not-allowed; opacity: 0.5; }
        #btn-reset { background: #0d6efd; flex: 1; }
        
        #btn-copy { background: #ff8c00; margin-top: 5px; }
        
        #coord-display {
            margin-top: 20px;
            width: 100%;
            max-width: 600px;
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

    <button id="btn-pip">🗔 Pop Out Tool</button>

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

            <div class="action-row">
                <button id="btn-play">PLAY</button>
                <button id="btn-reset">RESET</button>
            </div>

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
    const playBtn = document.getElementById('btn-play');
    const resetBtn = document.getElementById('btn-reset');
    const grid = document.getElementById('grid');
    const coordDisplay = document.getElementById('coord-display');
    const copyBtn = document.getElementById('btn-copy');
    const pipBtn = document.getElementById('btn-pip');
    const mainContent = document.getElementById('main-content');

    let clickedCoordinates = [];
    let isCashoutActive = false;
    let hasBeenPlayed = false;
    let currentContext = document; // Tracks if we are in main page or PiP

    dollarBtn.onclick = () => {
        isCashoutActive = !isCashoutActive;
        dollarBtn.classList.toggle('active');
        playBtn.disabled = isCashoutActive;
        updateDisplay();
    };

    playBtn.onclick = () => {
        if (isCashoutActive) return;
        hasBeenPlayed = true;
        
        // Use currentContext to find cells even if popped out
        const cells = currentContext.querySelectorAll('.cell');
        cells.forEach(cell => {
            if (cell.textContent === '🟧') {
                cell.textContent = '💎';
                cell.classList.add('locked');
            }
        });

        clickedCoordinates = [];
        updateDisplay();
    };

    resetBtn.onclick = () => {
        sizeSlider.value = 5;
        mineText.value = 3;
        mineSlider.value = 3;
        betText.value = 0;
        isCashoutActive = false;
        hasBeenPlayed = false;
        dollarBtn.classList.remove('active');
        playBtn.disabled = false;
        generateGrid();
    };

    function updateDisplay() {
        const size = sizeSlider.value;
        const mines = mineText.value;
        const bet = betText.value || 0;
        const coords = clickedCoordinates.join(' ');
        const cashout = isCashoutActive ? "cashout" : "";
        
        let output = hasBeenPlayed ? `!mines ${coords} ${cashout}` : `!mines ${bet} ${size} ${mines} ${coords} ${cashout}`;
        coordDisplay.value = output.trim().replace(/\s\s+/g, ' ');
    }

    function generateGrid() {
        const size = parseInt(sizeSlider.value);
        sizeValDisplay.textContent = size;
        const maxMines = Math.pow(size, 2) - 1;
        mineText.max = maxMines; mineSlider.max = maxMines;
        if (parseInt(mineText.value) > maxMines) { mineText.value = maxMines; mineSlider.value = maxMines; }

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
                    if (cell.classList.contains('locked')) return;
                    if (cell.textContent === '⬜') {
                        cell.textContent = '🟧';
                        clickedCoordinates.push(coordStr);
                    } else if (cell.textContent === '🟧') {
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

    // Input Listeners
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

    // Pop Out Logic
    pipBtn.onclick = async () => {
        if (!('documentPictureInPicture' in window)) { alert("Browser not supported."); return; }
        
        const pipWindow = await window.documentPictureInPicture.requestWindow({ width: 450, height: 650 });
        currentContext = pipWindow.document; // Update context to the pop-up

        const style = document.createElement('style');
        style.textContent = document.getElementById('main-styles').textContent;
        pipWindow.document.head.appendChild(style);
        pipWindow.document.body.style.backgroundColor = "#121212";
        pipWindow.document.body.append(mainContent);
        pipWindow.document.body.append(coordDisplay);

        pipWindow.addEventListener("pagehide", () => {
            currentContext = document; // Reset context back to main page
            document.body.append(mainContent);
            document.body.append(coordDisplay);
        });
    };

    generateGrid();
</script>
</body>
</html>
