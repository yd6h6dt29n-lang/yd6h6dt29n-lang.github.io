<body style="background:#121212; color:#fff; text-align:center;">
    <audio id="dummy" loop><source src="data:audio/wav;base64,UklGRiQAAABXQVZFZm10IBAAAAABAAEAAAIARKAAABCxAgAEABAAZGF0YQAAAAA="></audio>
    <div style="padding:20px; display:flex; flex-direction:column; gap:10px;">
        <button id="b_mild" style="padding:20px; background:#34c759; color:#fff; border-radius:10px;">🍀 優しい</button>
        <button id="b_hard" style="padding:20px; background:#ff9500; color:#fff; border-radius:10px;">🔥 激しい</button>
        <button id="b_stop" style="padding:15px; background:#ff3b30; color:#fff; border-radius:10px;">🛑 停止</button>
    </div>
    <p id="t">ステータス: 待機中</p>

<script>
// このコードには「バックグラウンド維持」のための無音再生処理が含まれています
let timer = null, char = null;
const S='40ee1111-63ec-4b7f-8ce7-712efd55b90e', C='40ee2222-63ec-4b7f-8ce7-712efd55b90e';

async function startMode(mode) {
    try {
        // 無音を再生してバックグラウンド落ちを防ぐ
        document.getElementById('dummy').play();
        if (!char) {
            const d = await navigator.bluetooth.requestDevice({filters:[{services:[S]}]});
            const g = await d.gatt.connect();
            char = await (await g.getPrimaryService(S)).getCharacteristic(C);
        }
        if (timer) clearInterval(timer);
        document.getElementById('t').innerText = mode === 'mild' ? "🍀実行中" : "🔥実行中";
        timer = setInterval(async () => {
            let speed = mode === 'mild' ? Math.floor(Math.random()*20)+5 : Math.floor(Math.random()*50)+40;
            const f = Math.random() > 0.5 ? 0 : 128;
            await char.writeValue(new Uint8Array([1, 1, f + speed]));
        }, mode === 'mild' ? 4000 : 2000);
    } catch (e) { alert(e); }
}
document.getElementById('b_mild').onclick = () => startMode('mild');
document.getElementById('b_hard').onclick = () => startMode('hard');
document.getElementById('b_stop').onclick = () => {
    clearInterval(timer);
    document.getElementById('dummy').pause();
    if(char) char.writeValue(new Uint8Array([1,1,0]));
    document.getElementById('t').innerText = "停止中";
};
</script>
</body>
