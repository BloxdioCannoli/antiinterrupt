# Usage
> [!Note]
> This is very simple as of right now and does not support everything, eg other callbacks or certain syntax.

## Add loaded safe cde
```js
function tick() {
    runSafeCodeFromCodeBlock("tick", [x, y, z])
}
```

# The tool
> [!Important]
> This is in BETA and I've copied it directly from my world code.

```js
function initFuncIfNeeded(fun = "tick") {
    if (!globalThis.counts) { globalThis.counts = {}; }
    if (globalThis.counts[fun]?.count === undefined) { globalThis.counts = {}; globalThis.counts[`${fun}`] = {}; globalThis.counts[`${fun}`].count = 0; }
}

function resetCount(fun = "tick") {
    initFuncIfNeeded(fun);
    globalThis.counts[fun].count = 0;
}

function addCount(fun = "tick") {
    initFuncIfNeeded(fun);
    globalThis.counts[fun].count++;
}

function getCount(fun = "tick") {
    initFuncIfNeeded(fun);
    return globalThis.counts[fun].count;
}

function benchmark(fun, code = () => { }, countNum) {
    return eval(`if (getCount("${fun}") === ${countNum}) { globalThis.run = () => { if (api.isNearInterrupt()) { return; } else { code(); addCount("${fun}"); } }; globalThis.run(); }`);
}

function breakUp(text) {
    let splitText = text.split("\n");
    let newText = [];

    for (let s in splitText) {
        newText.push(splitText[s].split(";"));
    }
    return splitText;
}

function rejoinText(text) {
    let splitText = [...text];
    splitText = splitText.join(";");
    return splitText;
}

function wrap(text) {
    let splitText = breakUp(text);
    let newText = [];

    let fun = "tick";

    for (let lineNum in splitText) {
        let original = splitText[lineNum];

        newText.push(`benchmark("${fun}", () => { ${original} }, ${lineNum})`);
    }; newText.push(`; resetCount("${fun}")`);

    newText = rejoinText(newText);

    return newText;
}

function initStoredCodeIfNeeded(func = "tick") {
    if (!globalThis.storedCode) { globalThis.storedCode = {}; }
    if (!globalThis.storedCode[func]) { globalThis.storedCode[func] = null; }
}

function addBenchmarksToCodeBlock(pos) {
    let block = api.getBlock(pos);
    if (block == "Unloaded") { return null; }
    let text = api.getBlockData(...pos).persisted.shared.text;
    return wrap(text);
}

function getStoredSafeCode(func = "tick") {
    initStoredCodeIfNeeded(func);
    let stored = globalThis.storedCode[func];
    return stored;
}

function storeSafeCode(func = "tick", code = "") {
    initStoredCodeIfNeeded(func);
    globalThis.storedCode[func] = code;
    return globalThis.storedCode[func];
}

function runSafeCodeFromCodeBlock(func = "tick", pos = [0, 0, 0]) {
    let stored = getStoredSafeCode(func);
    if (!stored) {
        let code = addBenchmarksToCodeBlock(pos);
        stored = storeSafeCode(func, code);
    }
    eval(stored);
}
```
