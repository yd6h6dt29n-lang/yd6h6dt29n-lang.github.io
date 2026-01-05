<body style="background:#121212; color:#fff; font-family:sans-serif; margin:0; height:100vh; display:flex; flex-direction:column;">

    <div style="flex: 1; display:flex; flex-direction:column; justify-content:center; align-items:center; padding:20px;">
        <p id="t" style="font-size:24px; font-weight:bold; margin:0; color:#007aff;">ステータス: 待機中</p>
        <p id="m" style="font-size:18px; color:#aaa; margin-top:10px;">現在: 未接続</p>
        <div style="margin-top:30px; border:1px dashed #444; padding:20px; border-radius:10px; color:#555; font-size:12px;">
            ここに動画の小窓（PiP）を置いてください
        </div>
    </div>

    <div style="background:#1e1e1e; padding:20px; border-radius:30px 30px 0 0; display:flex; flex-direction:column; gap:12px; padding-bottom:40px; box-shadow: 0 -5px 15px rgba(0,0,0,0.5);">
        
        <div style="display:grid; grid-template-columns: 1fr 1fr; gap:12px;">
            <button id="b_low" style="padding:25px 10px; font-size:18px; background:#34c759; color:#fff; border:none; border-radius:15px; font-weight:bold;">🍀 低速</button>
            <button id="b_mid" style="padding:25px 10px; font-size:18px; background:#007aff; color:#fff; border:none; border-radius:15px; font-weight:bold;">💎 中速</button>
        </div>

        <div style="display:grid; grid-template-columns: 1fr 1fr; gap:12px;">
            <button id="b_high" style="padding:25px 10px; font-size:18px; background:#ff9500; color:#fff; border:none; border-radius:15px; font-weight:bold;">🔥 高速</button>
            <button id="b_rand" style="padding:25px 10px; font-size:18px; background:#af52de; color:#fff; border:none; border-radius:15px; font-weight:bold;">🎲 乱数</button>
        </div>

        <button id="b_stop" style="padding:20px; font-size:20px; background:#ff3b30; color:#fff; border:none; border-radius:15px; font-weight:bold; margin-top:5px;">🛑 停止</button>
    </div>

<script>
let timer = null;
let char = null;
const S='40ee1111-63ec-4b7f-8ce7-712efd55b90e', C='40ee2222-63ec-4b7f-8ce7-712efd55b90e';

async function connect() {
    if (!char) {
        const d = await navigator.bluetooth.requestDevice({filters:[{services:[S]}]});
        const g = await d.gatt.connect();
        char = await (await g.getPrimaryService(S)).getCharacteristic(C);
    }
}

async function startMode(mode) {
    try {
        await connect();
        if (timer) clearTimeout(timer);
        
        document.getElementById('t').innerText = "ステータス: 実行中";
        const labels = {low:"🍀 低速", mid:"💎 中速", high:"🔥 高速", rand:"🎲 ランダム"};
        document.getElementById('m').innerText = "モード: " + labels[mode];

        async function loop() {
            let speed, wait;
            const f = Math.random() > 0.5 ? 0 : 128; // 回転方向をランダムに決定

            if (mode === 'low') {
                speed = Math.floor(Math.random() * 20) + 5; // 5-25%
                wait = 2000; // 2秒
            } else if (mode === 'mid') {
                speed = Math.floor(Math.random() * 25) + 25; // 25-50%
                wait = 2000; // 2秒
            } else if (mode === 'high') {
                speed = Math.floor(Math.random() * 40) + 50; // 50-90%
                wait = 2000; // 2秒
            } else if (mode === 'rand') {
                speed = Math.floor(Math.random() * 85) + 5; // 全域(5-90%)
                wait = Math.floor(Math.random() * 3000) + 1000; // 変化も1-4秒でランダム
            }

            await char.writeValue(new Uint8Array([1, 1, f + speed]));
            timer = setTimeout(loop, wait);
        }
        loop();
    } catch (e) { alert("エラー: " + e); }
}

document.getElementById('b_low').onclick = () => startMode('low');
document.getElementById('b_mid').onclick = () => startMode('mid');
document.getElementById('b_high').onclick = () => startMode('high');
document.getElementById('b_rand').onclick = () => startMode('rand');
document.getElementById('b_stop').onclick = async () => {
    if (timer) clearTimeout(timer);
    timer = null;
    if (char) {
        await char.writeValue(new Uint8Array([1, 1, 0]));
        document.getElementById('t').innerText = "ステータス: 停止中";
        document.getElementById('m').innerText = "モード: オフ";
    }
};
</script>
</body>
