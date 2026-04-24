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
background:#0a0f1e;
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
position:sticky;
top:0;
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
background:#111827;
padding:10px;
margin:10px 0;
border-radius:10px;
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

button{padding:6px 8px;border:none;border-radius:6px;cursor:pointer}
.primary{background:#2563eb;color:#fff}
.warn{background:#f59e0b}
.danger{background:#ef4444}

.section{display:none}
.section.active{display:block}

/* 🔥 تعديل بسيط للعرض */
.idBox{
display:inline-block;
background:#1f2937;
padding:3px 6px;
border-radius:6px;
margin-left:6px;
font-size:12px;
}
</style>
</head>

<body>

<header>🏛️ وزارة الداخلية</header>

<div class="nav">
<button class="active" onclick="show('army',this)">👮 العسكريين</button>
<button onclick="show('points',this)">⭐ النقاط</button>
<button onclick="show('warns',this)">⚠ التحذيرات</button>
<button onclick="show('notes',this)">📝 الملاحظات</button>
</div>

<div class="container">

<div id="army" class="section active">

<div class="card">
<h3>➕ إضافة عسكري</h3>

<input id="name" placeholder="اسم">
<input id="gid" placeholder="Game ID">

<select id="rank"></select>

<select id="unit">
<option>قوات طوارئ</option>
<option>أمن عام</option>
<option>المرور</option>
<option>التحقيقات</option>
</select>

<button class="primary" onclick="add()">إضافة</button>
</div>

<div id="armyList"></div>
</div>

<div id="points" class="section"></div>
<div id="warns" class="section"></div>
<div id="notes" class="section">

<div class="card">
<h3>➕ إضافة ملاحظة</h3>

<select id="noteSelect"></select>
<input id="noteText" placeholder="اكتب الملاحظة">
<button class="primary" onclick="addNote()">إضافة</button>
</div>

<div id="notesList"></div>

</div>

</div>

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

const ranks=[
"فريق أول","فريق","لواء","عميد","عقيد","مقدم",
"رائد","نقيب","ملازم أول","ملازم",
"رقيب أول","رقيب","وكيل رقيب","عريف","جندي أول","جندي"
];

window.onload=()=>{
rank.innerHTML=ranks.map(r=>`<option>${r}</option>`).join("");
load();
};

function show(id,btn){
document.querySelectorAll(".section").forEach(s=>s.classList.remove("active"));
document.querySelectorAll(".nav button").forEach(b=>b.classList.remove("active"));
document.getElementById(id).classList.add("active");
btn.classList.add("active");
}

function load(){
db.ref("players").on("value",snap=>{
let data=snap.val()||{};
let arr=Object.entries(data).map(([id,v])=>({id,...v}));

arr.sort((a,b)=>ranks.indexOf(a.rank)-ranks.indexOf(b.rank));

renderArmy(arr);
renderPoints(arr);
renderWarns(arr);
renderNotes(arr);
fillSelect(arr);
});
}

/* ➕ إضافة */
function add(){

db.ref("players").push({
name:name.value||"بدون اسم",
gid:gid.value||"بدون ID",
rank:rank.value,
unit:unit.value,
points:0,
warn:0,
notes:[]
});

name.value="";
gid.value="";
}

/* ⭐ */
function addPoint(id){
db.ref("players/"+id).once("value",snap=>{
let p=snap.val();

p.points++;

if(p.points>=3){
p.points=0;
let i=ranks.indexOf(p.rank);
if(i>0)p.rank=ranks[i-1];
}

db.ref("players/"+id).set(p);
});
}

/* ⚠ */
function addWarn(id){
db.ref("players/"+id).once("value",snap=>{
let p=snap.val();

p.warn++;

if(p.warn>=3){
p.warn=0;
let i=ranks.indexOf(p.rank);
if(i<ranks.length-1)p.rank=ranks[i+1];
}

db.ref("players/"+id).set(p);
});
}

/* 📝 */
function addNote(){
let id=noteSelect.value;
let text=noteText.value.trim();
if(!id||!text)return;

db.ref("players/"+id).once("value",snap=>{
let p=snap.val();
if(!p.notes)p.notes=[];
p.notes.push({text,time:new Date().toLocaleString("ar")});
db.ref("players/"+id).set(p);
noteText.value="";
});
}

function fillSelect(d){
noteSelect.innerHTML=d.map(p=>`<option value="${p.id}">${p.name}</option>`).join("");
}

/* 👮 عرض العسكريين (تعديل هنا فقط) */
function renderArmy(d){
armyList.innerHTML=d.map(p=>`
<div class="card">

<div>
<span class="idBox">${p.gid}</span>
<b>${p.name}</b>
</div>

🎖 ${p.rank} | 🚓 ${p.unit}<br><br>

<button class="primary" onclick="addPoint('${p.id}')">⭐</button>
<button class="warn" onclick="addWarn('${p.id}')">⚠</button>

</div>
`).join("");
}

function renderPoints(d){
points.innerHTML=d.map(p=>`
<div class="card">${p.name} ⭐ ${p.points}</div>
`).join("");
}

function renderWarns(d){
warns.innerHTML=d.map(p=>`
<div class="card">${p.name} ⚠ ${p.warn}</div>
`).join("");
}

function renderNotes(d){
notesList.innerHTML=d
.filter(p=>p.notes&&p.notes.length>0)
.map(p=>`
<div class="card">
<b>${p.name}</b><br>
${p.notes.map(n=>`📝 ${n.text} - ${n.time}<br>`).join("")}
</div>
`).join("");
}

</script>

</body>
</html>
