<!DOCTYPE html><html lang="fr"><head><meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Scanner Cross</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/peerjs/1.5.4/peerjs.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.js"></script>
<style>body{margin:0;background:#0d1117;color:#fff;font-family:sans-serif;text-align:center}
video{width:100%;max-height:75vh;object-fit:cover}#st{padding:16px;font-size:20px;font-weight:700}</style></head>
<body><video id="v" playsinline muted></video><div id="st">Connexion…</div>
<script>
const id=location.hash.slice(1), st=document.getElementById('st'), v=document.getElementById('v');
const cv=document.createElement('canvas'), cx=cv.getContext('2d',{willReadFrequently:true});
let conn=null,last='',lastT=0;
const peer=new Peer();
peer.on('open',()=>{
  conn=peer.connect(id,{reliable:true});
  conn.on('open',()=>st.textContent='✅ Connecté — scannez un dossard');
  conn.on('data',m=>{st.textContent=(m.ok?'✓ ':'⚠ ')+m.msg;
    st.style.color=m.ok?'#3dffa3':'#ff6b6b'; navigator.vibrate&&navigator.vibrate(m.ok?80:[80,60,80]);});
  conn.on('close',()=>st.textContent='Déconnecté');
});
navigator.mediaDevices.getUserMedia({video:{facingMode:'environment'}})
 .then(s=>{v.srcObject=s;v.play();requestAnimationFrame(tick);})
 .catch(e=>st.textContent='Caméra refusée : '+e.message);
function tick(){
  if(v.videoWidth){
    cv.width=v.videoWidth;cv.height=v.videoHeight;cx.drawImage(v,0,0);
    const im=cx.getImageData(0,0,cv.width,cv.height), c=jsQR(im.data,im.width,im.height);
    const now=Date.now();
    if(c&&conn&&conn.open&&(c.data!==last||now-lastT>4000)){last=c.data;lastT=now;conn.send(c.data);}
  }
  requestAnimationFrame(tick);
}
</script></body></html>
