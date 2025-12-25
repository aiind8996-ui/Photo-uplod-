<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Photo Vault</title>

<style>
*{
    margin:0;
    padding:0;
    box-sizing:border-box;
    font-family:monospace;
    user-select:none;
    -webkit-user-select:none;
}
body{
    background:#000;
    color:#00ff88;
    min-height:100vh;
    padding:20px;
    background-image:
        radial-gradient(circle,#0a0a0a 1px,transparent 1px),
        radial-gradient(circle,#050505 1px,transparent 1px);
    background-size:40px 40px;
    animation:bgMove 15s linear infinite;
}
@keyframes bgMove{
    0%{background-position:0 0;}
    100%{background-position:400px 400px;}
}
h1,h3{text-align:center;}

button{
    background:#000;
    color:#00ff88;
    border:2px solid #00ff88;
    padding:10px 18px;
    margin:6px;
    cursor:pointer;
    transition:.25s;
    font-weight:bold;
}
button:hover{
    background:#00ff88;
    color:#000;
    transform:scale(1.05);
}

#adminBtn{
    position:fixed;
    top:10px;
    left:10px;
    font-size:11px;
    padding:6px 10px;
}

input{display:none;}

.media-list{
    display:flex;
    flex-wrap:wrap;
    justify-content:center;
    gap:15px;
    margin-top:15px;
}
.media-box{
    width:190px;
    border:2px solid #00ff88;
    padding:8px;
    background:rgba(0,0,0,.7);
    animation:float 3s ease-in-out infinite alternate;
}
@keyframes float{
    to{transform:translateY(8px);}
}
.media-box img{
    width:100%;
    max-height:130px;
    object-fit:cover;
    border:1px solid #00ff88;
    pointer-events:none;
}
.user-label{
    font-size:11px;
    margin-bottom:5px;
    color:#00ffff;
}
.admin-controls button{
    font-size:11px;
    padding:5px 8px;
    margin:3px;
}
.hidden{display:none;}
hr{
    border:1px solid #00ff88;
    margin:20px 0;
}

/* FULL VIEW MODAL (ADMIN ONLY) */
#viewer{
    position:fixed;
    inset:0;
    background:rgba(0,0,0,.9);
    display:none;
    align-items:center;
    justify-content:center;
    z-index:999;
}
#viewer img{
    max-width:95%;
    max-height:95%;
    border:3px solid #00ff88;
}
</style>
</head>

<body>

<h1>Secure Photo Vault</h1>
<button id="adminBtn" onclick="adminLogin()">Settings</button>

<div style="text-align:center;margin-top:40px;">
    <button onclick="upload()">Upload Photo</button>
    <input type="file" id="fileInput" accept="image/*">
    <div id="msg" style="color:#00ffff;margin-top:10px;"></div>
</div>

<hr>
<h3>Uploaded Photos</h3>
<div class="media-list" id="list"></div>

<!-- FULL VIEW -->
<div id="viewer" onclick="closeView()">
    <img id="viewerImg">
</div>

<script>
/* BLOCK GOOGLE / SAVE */
["contextmenu","dragstart","selectstart"].forEach(e=>{
    document.addEventListener(e,x=>x.preventDefault());
});
document.addEventListener("touchstart",e=>{
    if(e.touches.length>1) e.preventDefault();
},{passive:false});

const ADMIN_PASSWORD="1221";
let isAdmin=false;

const fileInput=document.getElementById("fileInput");
const list=document.getElementById("list");
const msg=document.getElementById("msg");
const viewer=document.getElementById("viewer");
const viewerImg=document.getElementById("viewerImg");

let photos=JSON.parse(localStorage.getItem("vaultPhotos"))||[];

function render(){
    list.innerHTML="";
    photos.forEach((p,i)=>{
        const box=document.createElement("div");
        box.className="media-box";
        box.innerHTML=`
            <div class="user-label">You uploaded</div>
            <img src="${p}">
            <div class="admin-controls ${isAdmin?'':'hidden'}">
                <button onclick="view(${i})">View</button>
                <button onclick="download(${i})">Download</button>
                <button onclick="share(${i})">Send</button>
                <button onclick="del(${i})">Delete</button>
            </div>
        `;
        list.appendChild(box);
    });
}
render();

/* UPLOAD */
function upload(){
    fileInput.value="";
    fileInput.click();
}
fileInput.onchange=()=>{
    const f=fileInput.files[0];
    if(!f || !f.type.startsWith("image")) return;
    const r=new FileReader();
    r.onload=e=>{
        photos.push(e.target.result);
        localStorage.setItem("vaultPhotos",JSON.stringify(photos));
        msg.innerHTML="✔ Photo uploaded – visible below";
        render();
    };
    r.readAsDataURL(f);
};

/* ADMIN */
function adminLogin(){
    const a=prompt("Answer");
    if(a===ADMIN_PASSWORD){
        isAdmin=true;
        alert("Admin access granted");
        render();
    }else alert("Incorrect");
}

/* ADMIN ACTIONS */
function view(i){
    if(!isAdmin) return;
    viewerImg.src=photos[i];
    viewer.style.display="flex";
}
function closeView(){ viewer.style.display="none"; }
function del(i){
    if(isAdmin && confirm("Delete photo?")){
        photos.splice(i,1);
        localStorage.setItem("vaultPhotos",JSON.stringify(photos));
        render();
    }
}
function download(i){
    const a=document.createElement("a");
    a.href=photos[i];
    a.download="photo_"+i;
    a.click();
}
async function share(i){
    if(!navigator.canShare) return alert("Not supported");
    const b=await (await fetch(photos[i])).blob();
    navigator.share({files:[new File([b],"photo.jpg",{type:b.type})]});
}
</script>

</body>
</html>
