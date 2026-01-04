<body style="background:#121212; color:#fff; font-family:sans-serif; text-align:center;">
    <div style="display:flex; flex-direction:column; gap:15px; padding:20px;">
        <button id="b_mild" style="padding:25px; font-size:20px; background:#34c759; color:#fff; border:none; border-radius:15px; font-weight:bold;">🍀 優しいモード (低速)</button>
        <button id="b_hard" style="padding:25px; font-size:20px; background:#ff9500; color:#fff; border:none; border-radius:15px; font-weight:bold;">🔥 激しいモード (高速)</button>
        <button id="b_stop" style="padding:20px; font-size:18px; background:#ff3b30; color:#fff; border:none; border-radius:15px; font-weight:bold;">🛑 停止</button>
    </div>
    <p id="t" style="font-size:18px; margin-top:10px;">ステータス: 待機中</p>
    <p id="m" style="font-size:14px; color:#aaa;">現在: 未接続</p>

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
        if (timer) clearInterval(timer);
        
        document.getElementById('t').innerText = "ステータス: 実行中";
        document.getElementById('m').innerText = mode === 'mild' ? "モード: 🍀優しい" : "モード: 🔥激しい";

        timer = setInterval(async () => {
            let speed, wait;
            if (mode === 'mild') {
                speed = Math.floor(Math.random() * 20) + 5; // 5%〜25%のゆったり
                wait = 4000; // 4秒おき
            } else {
                speed = Math.floor(Math.random() * 50) + 40; // 40%〜90%の激しさ
                wait = 2000; // 2秒おきに変化
            }
            const f = Math.random() > 0.5 ? 0 : 128; // 回転方向もランダム
            await char.writeValue(new Uint8Array([1, 1, f + speed]));
        }, mode === 'mild' ? 4000 : 2000);
    } catch (e) { alert("エラー: " + e); }
}

document.getElementById('b_mild').onclick = () => startMode('mild');
document.getElementById('b_hard').onclick = () => startMode('hard');
document.getElementById('b_stop').onclick = async () => {
    if (timer) clearInterval(timer);
    if (char) {
        await char.writeValue(new Uint8Array([1, 1, 0]));
        document.getElementById('t').innerText = "ステータス: 停止中";
        document.getElementById('m').innerText = "モード: オフ";
    }
};
</script>
</body>
