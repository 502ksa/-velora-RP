<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>وزارة الداخلية - Velora RP</title>

<style>
body{
margin:0;
font-family:Tahoma;
color:#fff;
background: linear-gradient(-45deg,#050814,#0b1220,#0a0e1a,#050816);
background-size:400% 400%;
animation:bg 12s ease infinite;
}

@keyframes bg{
0%{background-position:0% 50%}
50%{background-position:100% 50%}
100%{background-position:0% 50%}
}

header{
background:#0f172a;
padding:16px;
text-align:center;
font-size:22px;
font-weight:bold;
}

.nav{
display:flex;
gap:6px;
justify-content:center;
background:#0b1220;
padding:10px;
flex-wrap:wrap;
}

.nav button{
padding:8px;
border:none;
border-radius:8px;
background:#1f2937;
color:#fff;
cursor:pointer;
}

.nav button.active{background:#2563eb}

.container{max-width:1100px;margin:auto;padding:12px}

.card{
background:#111827cc;
padding:10px;
margin:10px 0;
border-radius:10px;
backdrop-filter: blur(5px);
}

input,select{
width:100%;
padding:10px;
margin:5px 0;
background:#0a0e14;
color:#fff;
border:none;
border-radius:8px;
}

button{padding:8px;border:none;border-radius:8px;cursor:pointer}
.primary{background:#2563eb;color:#fff}
.warn{background:#f59e0b}

.tag{
display:inline-block;
background:#1f2937;
padding:4px 8px;
border-radius:6px;
margin:3px;
font-size:12px;
}

.section{display:none}
.section.active{display:block}
.hidden{display:none}
</style>
</head>

<body>

<header>🏛️ وزارة الداخلية - Velora RP</header>

<!-- LOGIN -->
<div id="loginBox" class="card">
<h3>🔐 تسجيل دخول</h3>
<input id="pass" placeholder="الباسورد">
<button class="primary" onclick="login()">دخول</button>
</div>

<div id="app" class="hidden">

<div class="nav">
<button class="active" onclick="show('army',this)">👮 العسكريين</button>
<button onclick="show('points',this)">⭐ البوينتات</button>
<button onclick="show('warns',this)">⚠ التحذيرات</button>
<button onclick="show('notes',this)">📝 الملاحظات</button>
<button onclick="show('logs',this)">📜 السجل</button>
</div>

<div class="container">

<div class="card">
<h3>➕ إضافة عسكري</h3>

<input id="name" placeholder="اسم">
<input id="gid" placeholder="Game ID">
<input id="disc" placeholder="Discord">

<select id="rank"></select>

<select id="unit">
<option>قوات طوارئ</option>
<option>أمن عام</option>
<option>المرور</option>
<option>التحقيقات</option>
</select>

<button class="primary" onclick="add()">إضافة</button>
</div>

<div class="card">
<h3>🔎 بحث</h3>
<input id="search" placeholder="اسم">
<button onclick="find()">بحث</button>
<div id="result"></div>
</div>

<div id="army" class="section active"></div>
<div id="points" class="section"></div>
<div id="warns" class="section"></div>
<div id="notes" class="section"></div>
<div id="logs" class="section"></div>

</div>
</div>

<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-database-compat.js"></script>

<script>

const firebaseConfig = {
apiKey:"AIzaSyBw9V3yMuiUa5MgurcxW2V1RCImC6eHcGQ",
authDomain:"velora-rp.firebaseapp.com",
databaseURL:"https://velora-rp-default-rtdb.europe-west1.firebasedatabase.app",
projectId:"velora-rp",
storageBucket:"velora-rp.firebasestorage.app",
messagingSenderId:"681356163114",
appId:"1:681356163114:web:c40b8d7fed229ce8e7eb53"
};

firebase.initializeApp(firebaseConfig);
const db=firebase.database();

/* دخول */
function login(){
if(pass.value==="0008"){
loginBox.classList.add("hidden");
app.classList.remove("hidden");
}else alert("خطأ");
}

/* رتب */
const ranks=[
"فريق أول","فريق","لواء","عميد","عقيد","مقدم",
"رائد","نقيب","ملازم أول","ملازم",
"رئيس رقباء","رقيب أول","رقيب","وكيل رقيب","عريف","جندي أول","جندي"
];

window.onload=()=>{
rank.innerHTML=ranks.map(r=>`<option>${r}</option>`).join("");
load();
loadLogs();
};

/* تبويب */
function show(id,btn){
document.querySelectorAll(".section").forEach(s=>s.classList.remove("active"));
document.querySelectorAll(".nav button").forEach(b=>b.classList.remove("active"));
document.getElementById(id).classList.add("active");
btn.classList.add("active");
}

/* سجل */
function addLog(text){
db.ref("logs").push({
text,
time:new Date().toLocaleString("ar")
});
}

/* تحميل */
function load(){
db.ref("players").on("value",snap=>{
let data=snap.val()||{};
let arr=Object.entries(data).map(([id,v])=>({id,...v}));
render(arr);
});
}

/* إضافة */
function add(){

let n=document.getElementById("name").value.trim();
let g=document.getElementById("gid").value.trim();
let d=document.getElementById("disc").value.trim();

db.ref("players").push({
name:n||"بدون اسم",
gid:g||"بدون ID",
disc:d||"لا يوجد",
rank:document.getElementById("rank").value,
unit:document.getElementById("unit").value,
points:0,
warn:0,
notes:[]
});

addLog("تم إضافة عسكري");

name.value="";
gid.value="";
disc.value="";
}

/* ⭐ */
function addPoint(id){
db.ref("players/"+id).once("value",snap=>{
let p=snap.val();
p.points++;
db.ref("players/"+id).set(p);
addLog("نقطة لـ: "+p.name);
});
}

/* ⚠ */
function addWarn(id){
db.ref("players/"+id).once("value",snap=>{
let p=snap.val();
p.warn++;

let note=prompt("اكتب ملاحظة:");
if(note){
if(!p.notes) p.notes=[];
p.notes.push({
text:note,
time:new Date().toLocaleString("ar")
});
}

db.ref("players/"+id).set(p);
addLog("تحذير لـ: "+p.name);
});
}

/* 🧠 عرض + تعديل + حذف */
function render(d){

army.innerHTML=d.map(p=>`
<div class="card">

<b>${p.name}</b><br>

<div class="tag">🎖 ${p.rank}</div>
<div class="tag">🚓 ${p.unit}</div>
<div class="tag">🆔 ${p.gid}</div>

<div class="tag">⭐ ${p.points}</div>
<div class="tag">⚠ ${p.warn}</div>

<button class="primary" onclick="edit('${p.id}')">✏ تعديل</button>
<button class="warn" onclick="removePlayer('${p.id}')">🗑 حذف</button>
<button onclick="addPoint('${p.id}')">⭐</button>
<button onclick="addWarn('${p.id}')">⚠</button>

</div>
`).join("");

points.innerHTML=d.map(p=>`
<div class="card">${p.name} ⭐ ${p.points}</div>
`).join("");

warns.innerHTML=d.map(p=>`
<div class="card">${p.name} ⚠ ${p.warn}</div>
`).join("");

notes.innerHTML=d.map(p=>`
<div class="card">
<b>${p.name}</b><br>
${(p.notes||[]).map(n=>`📝 ${n.text} - ${n.time}`).join("<br>")}
</div>
`).join("");

}

/* ✏ تعديل */
function edit(id){

db.ref("players/"+id).once("value",snap=>{
let p=snap.val();

let n=prompt("الاسم:",p.name);
let g=prompt("Game ID:",p.gid);
let d=prompt("Discord:",p.disc);
let r=prompt("الرتبة:",p.rank);
let u=prompt("الوحدة:",p.unit);

p.name=n;
p.gid=g;
p.disc=d;
p.rank=r;
p.unit=u;

db.ref("players/"+id).set(p);
addLog("تم تعديل: "+p.name);
});
}

/* 🗑 حذف */
function removePlayer(id){

if(!confirm("تأكيد الحذف؟")) return;

db.ref("players/"+id).remove();
addLog("تم حذف عسكري");
}

/* سجل */
function loadLogs(){
db.ref("logs").on("value",snap=>{
let data=snap.val()||{};
let arr=Object.values(data).reverse();

logs.innerHTML=arr.map(l=>`
<div class="card">
📌 ${l.text}<br>
<small>${l.time}</small>
</div>
`).join("");
});

/* بحث */
}
function find(){
db.ref("players").once("value",snap=>{
let d=Object.values(snap.val()||{});
let q=search.value;

let p=d.find(x=>x.name.includes(q));

result.innerHTML=p?`
<div class="card">
<b>${p.name}</b><br>
🎖 ${p.rank}<br>
🚓 ${p.unit}
</div>
`:"غير موجود";
});
}

</script>

</body>
</html>
