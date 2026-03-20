<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Boyer-Moore Visualizer (Animated)</title>
<style>
    body {
        font-family: monospace;
        background: #111;
        color: #eee;
        text-align: center;
    }

    h1 {
        margin-top: 20px;
    }

    .controls {
        margin: 20px;
    }

    input {
        padding: 8px;
        margin: 5px;
        font-family: monospace;
    }

    button {
        padding: 10px 20px;
        cursor: pointer;
    }

    .container {
        margin-top: 30px;
        font-size: 22px;
        letter-spacing: 5px;
    }

    .row {
        margin: 10px 0;
    }

    .char {
        display: inline-block;
        width: 25px;
        transition: all 0.5s ease;
    }

    .match {
        color: #4caf50;
        font-weight: bold;
    }

    .mismatch {
        color: #f44336;
        font-weight: bold;
    }

    .active {
        background: #333;
        border-radius: 4px;
    }

    .info {
        margin-top: 20px;
        font-size: 16px;
        min-height: 20px;
    }
</style>
</head>
<body>

<h1>Boyer–Moore Visualizer (Animated)</h1>

<div class="controls">
    <input id="text" value="ABAAABCD" />
    <input id="pattern" value="ABC" />
    <button onclick="start()">Start</button>
</div>

<div class="container">
    <div id="textRow" class="row"></div>
    <div id="patternRow" class="row"></div>
</div>

<div class="info" id="info"></div>

<script>
let text = "";
let pattern = "";
let badChar = {};
let shift = 0;
let j = 0;
let running = false;
let interval = null;

function buildBadChar(pattern) {
    const table = {};
    for (let i = 0; i < pattern.length; i++) {
        table[pattern[i]] = i;
    }
    return table;
}

function render(highlightIndex = -1, mismatch = false) {
    const textRow = document.getElementById("textRow");
    const patternRow = document.getElementById("patternRow");

    textRow.innerHTML = "";
    patternRow.innerHTML = "";

    // Text
    for (let i = 0; i < text.length; i++) {
        const span = document.createElement("span");
        span.className = "char";
        span.textContent = text[i];

        if (i === highlightIndex) {
            span.classList.add("active");
        }

        textRow.appendChild(span);
    }

    // Pattern with shift
    for (let i = 0; i < shift; i++) {
        const space = document.createElement("span");
        space.className = "char";
        space.textContent = " ";
        patternRow.appendChild(space);
    }

    for (let i = 0; i < pattern.length; i++) {
        const span = document.createElement("span");
        span.className = "char";
        span.textContent = pattern[i];

        if (shift + i === highlightIndex) {
            span.classList.add("active");
            span.classList.add(mismatch ? "mismatch" : "match");
        }

        patternRow.appendChild(span);
    }
}

function log(msg) {
    document.getElementById("info").textContent = msg;
}

function start() {
    text = document.getElementById("text").value;
    pattern = document.getElementById("pattern").value;

    badChar = buildBadChar(pattern);
    shift = 0;
    j = pattern.length - 1;
    running = true;

    log("Started animation...");
    render();

    if (interval) clearInterval(interval);
    interval = setInterval(step, 700); // 700ms per step
}

function step() {
    if (!running) return;

    if (shift > text.length - pattern.length) {
        log("Finished searching.");
        running = false;
        clearInterval(interval);
        return;
    }

    if (j < 0) {
        log(`🎉 Pattern found at index ${shift}`);
        shift += pattern.length;
        j = pattern.length - 1;
        render();
        return;
    }

    const textChar = text[shift + j];
    const patternChar = pattern[j];

    if (textChar === patternChar) {
        render(shift + j);
        log(`✔ Match '${patternChar}'`);
        j--;
    } else {
        render(shift + j, true);

        const badIndex = badChar[textChar] ?? -1;
        const move = Math.max(1, j - badIndex);

        log(`❌ Mismatch '${patternChar}' vs '${textChar}' → shift ${move}`);

        shift += move;
        j = pattern.length - 1;
    }
}
</script>

</body>
</html>
