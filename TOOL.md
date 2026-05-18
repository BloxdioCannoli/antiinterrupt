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
toLoad = [71, 37, -20];
oldGlobalThis = null;
let tickNum = 0;
let maxTick = 1;

function removeIds(ids = []) {
    /*if (!Array.isArray(ids)) ids = [ids];
    globalThis.ranIds = (globalThis.ranIds || []).filter(id => !ids.includes(id));*/
}

function initIdsIfNeeded() {
    if (!globalThis.ranIds) globalThis.ranIds = [];
}

function benchmark(fun, code = () => { }) {
    initIdsIfNeeded();
    return eval(`
    if (!globalThis.ranIds.includes("${fun}")) {
        globalThis.run = () => {
            if (api.isNearInterrupt()) {
                return;
            } else {
                code();
                globalThis.ranIds.push("${fun}");
            }
        };
        globalThis.run();
    }`);
}

function breakUp(text) {
    let splitText = text.split("\n");
    let newText = [];
    for (let s in splitText) {
        let splitAgain = splitText[s].split(";");
        for (let t in splitAgain) {
            let text = splitAgain[t];
            newText.push(text.trim());
        }
    }
    return newText;
}

function rejoinText(text) {
    return [...text].join(";");
}

let lastBenchmarkId = "";

function wrap(text, fun = "tick") {
    let splitText = breakUp(text);
    let newText = [];

    let nowStart = api.now();

    let scope = [fun];
    let curScopeNum = 0;

    let idsToRemove = {};

    for (let lineNum in splitText) {
        let original = splitText[lineNum];

        let curScope = scope[curScopeNum];
        if (!idsToRemove[curScope]) idsToRemove[curScope] = [];

        let benchmarkId = `${curScope}_${lineNum}_${nowStart}`;

        if (original.at(-1) == "{") {
            newText.push(`${original}`);
            scope.push(benchmarkId);
            curScopeNum++;
        } else if (original.at(-1) == "}" || original.at(-2) == "}") {
            newText.push(`; removeIds(["${idsToRemove[curScope].join('","')}"])`);
            newText.push(`}`);
            idsToRemove[curScope] = [];
            curScopeNum--;
        } else {
            newText.push(`benchmark("${curScope}", () => { ${original} })`);
            idsToRemove[curScope].push(benchmarkId);
        }

        lastBenchmarkId = benchmarkId;
    }

    //newText.push(`; removeIds("${fun}")`);
    newText.push(`; globalThis.ranIds=[]`);
    return rejoinText(newText);
}

function initStoredCodeIfNeeded(func = "tick") {
    if (!globalThis.storedCode) globalThis.storedCode = {};
    if (!globalThis.storedCode[func]) globalThis.storedCode[func] = null;
}

function addBenchmarksToCodeBlock(pos) {
    let block = api.getBlock(pos);
    if (block == "Unloaded") return null;
    let text = api.getBlockData(...pos).persisted.shared.text;
    return wrap(text);
}

function getStoredSafeCode(func = "tick") {
    initStoredCodeIfNeeded(func);
    return globalThis.storedCode[func];
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

function logToCodeBlock(text) {
    let pos = [60, 38, -15];

    api.setBlock(pos, "Code Block");

    api.setBlockData(...pos, {
        persisted: {
            shared: {
                text: `/*
${text}
*/`
            }
        }
    });
}
```
