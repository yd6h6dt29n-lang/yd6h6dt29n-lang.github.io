<body style="background:#121212; color:#fff; font-family:sans-serif; margin:0; text-align:center; padding:20px;">

    <h2 style="color:#007aff; margin-bottom:5px;">A10 Random Controller</h2>
    <p id="t" style="font-size:18px; font-weight:bold; margin:0 0 10px 0;">ステータス: 待機中</p>
    <p id="m" style="font-size:14px; color:#aaa; margin-bottom:20px;">現在: 未接続</p>

    <div style="display:grid; grid-template-columns: 1fr 1fr; gap:10px; margin-bottom:25px;">
        <button id="b_low" style="padding:20px; font-size:16px; background:#34c759; color:#fff; border:none; border-radius:12px; font-weight:bold;">🍀 低速<br><small>(2s間隔)</small></button>
        <button id="b_mid" style="padding:20px; font-size:16px; background:#007aff; color:#fff; border:none; border-radius:12px; font-weight:bold;">💎 中速<br><small>(2s間隔)</small></button>
        <button id="b_high" style="padding:20px; font-size:16px; background:#ff9500; color:#fff; border:none; border-radius:12px; font-weight:bold;">🔥 高速<br><small>(2s間隔)</small></button>
        <button id="b_rand" style="padding:20px; font-size:16px; background:#af52de; color:#fff; border:none; border-radius:12px; font-weight:bold;">🎲 乱数<br><small>(1-4s間隔)</small></button>
    </div>

    <div style="background:#1e1e1e; padding:15px; border-radius:15px; margin-bottom:25px;">
        <p style="margin:0 0 15px 0; font-weight:bold; color:#00ffff;">↔️ 左右単調モード (0.5s切替)</p>
        <div style="display:grid; grid-template-columns: repeat(4, 1fr); gap:8px;">
            <button onclick="startSwing(15, 1)" style="padding:15px 5px; background:#444; color:#fff; border:none; border-radius:8px; font-weight:bold;">強さ1</button>
            <button onclick="startSwing(35, 3)" style="padding:15px 5px; background:#666; color:#fff; border:none; border-radius:8px; font-weight:bold;">強さ3</button>
            <button onclick="startSwing(55, 5)" style="padding:15px 5px; background:#888; color:#fff; border:none; border-radius:8px; font-weight:bold;">強さ5</button>
            <button onclick="startSwing(75, 7)" style="padding:15px 5px; background:#aaa; color:#000; border:none; border-radius:8px; font-weight:bold;">強さ7</button>
        </div>
    </div>

    <button id="b_stop" style="width:100%; padding:25px; font-size:20px; background:#ff3b30; color:#fff; border:none; border-radius:15px; font-weight:bold; box-shadow:0 4px 10px rgba(0,0,0,0.3);">🛑 停止</button>

<script>
let timer = null;
let char = null;
let currentDir = 0;
const S='40ee1111-63ec-4b7f-8ce7-712efd55b90e', C='40ee2222-63ec-4b7f-8ce7-712efd55b90e';

async function connect() {
    if (!char) {
        const d = await navigator.bluetooth.requestDevice({filters:[{services:[S]}]});
        const g = await d.gatt.connect();
        char = await (await g.getPrimaryService(S)).getCharacteristic(C);
    }
}

// 通常モードの開始
async function startMode(mode) {
    try {
        await connect();
        stopTimer();
        document.getElementById('t').innerText = "ステータス: 実行中";
        const labels = {low:"🍀 低速", mid:"💎 中速", high:"🔥 高速", rand:"🎲 ランダム"};
        document.getElementById('m').innerText = "モード: " + labels[mode];

        async function loop() {
            let speed, wait;
            const f = Math.random() > 0.5 ? 0 : 128;
            if (mode === 'low') { speed = Math.floor(Math.random() * 20) + 5; wait = 2000; }
            else if (mode === 'mid') { speed = Math.floor(Math.random() * 25) + 25; wait = 2000; }
            else if (mode === 'high') { speed = Math.floor(Math.random() * 40) + 50; wait = 2000; }
            else if (mode === 'rand') { speed = Math.floor(Math.random() * 85) + 5; wait = Math.floor(Math.random() * 3000) + 1000; }

            await char.writeValue(new Uint8Array([1, 1, f + speed]));
            timer = setTimeout(loop, wait);
        }
        loop();
    } catch (e) { alert(e); }
}

// 左右単調モードの開始
async function startSwing(speed, level) {
    try {
        await connect();
        stopTimer();
        document.getElementById('t').innerText = "ステータス: 実行中";
        document.getElementById('m').innerText = "モード: ↔️ 左右単調 (強さ" + level + ")";

        timer = setInterval(async () => {
            currentDir = (currentDir === 0) ? 128 : 0; // 0.5sごとに反転
            await char.writeValue(new Uint8Array([1, 1, currentDir + speed]));
        }, 500);
    } catch (e) { alert(e); }
}

function stopTimer() {
    if (timer) {
        clearTimeout(timer);
        clearInterval(timer);
        timer = null;
    }
}

document.getElementById('b_low').onclick = () => startMode('low');
document.getElementById('b_mid').onclick = () => startMode('mid');
document.getElementById('b_high').onclick = () => startMode('high');
document.getElementById('b_rand').onclick = () => startMode('rand');
document.getElementById('b_stop').onclick = async () => {
    stopTimer();
    if (char) {
        await char.writeValue(new Uint8Array([1, 1, 0]));
        document.getElementById('t').innerText = "ステータス: 停止中";
        document.getElementById('m').innerText = "モード: オフ";
    }
};
</script>
</body>
