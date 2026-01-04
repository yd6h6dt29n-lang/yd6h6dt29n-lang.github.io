<button id="b" style="width:100%;padding:50px;font-size:30px;background:#007aff;color:#fff;border:none;border-radius:20px;">A10接続＆ランダム開始</button>
<p id="s" style="text-align:center;font-size:20px;margin-top:20px;">停止中</p>
<script>
const S='40ee1111-63ec-4b7f-8ce7-712efd55b90e',C='40ee2222-63ec-4b7f-8ce7-712efd55b90e';
document.getElementById('b').onclick=async()=>{
try{
const d=await navigator.bluetooth.requestDevice({filters:[{services:[S]}]});
const g=await d.gatt.connect();
const ch=await(await g.getPrimaryService(S)).getCharacteristic(C);
document.getElementById('s').innerText="ランダム実行中...";
setInterval(async()=>{
const v=Math.floor(Math.random()*60)+10;
const f=Math.random()>0.5?0:128;
await ch.writeValue(new Uint8Array([1,1,f+v]));
},3000);
}catch(e){alert("エラー:"+e);}
};
</script>
