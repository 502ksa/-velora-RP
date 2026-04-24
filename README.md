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
animation:bg 10s ease infinite;
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
position:sticky;
top:0;
z-index:999;
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

button{padding:6px 8px;border:none;border-radius:6px;cursor:pointer}
.primary{background:#2563eb;color:#fff}
.warn{background:#f59e0b}
.danger{background:#ef4444}

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
</style>
</head>

<body>

<header>🏛️ وزارة الداخلية - Velora RP</header>

<div class="nav">
<button class="active" onclick="show('army',this)">👮 العسكريين</button>
<button onclick="show('points',this)">⭐ النقاط</button>
<button onclick="show('warns',this)">⚠ التحذيرات</button>
<button onclick="show('notes',this)">📝 الملاحظات</button>
</div>

<div class="container">

<!-- العسكريين -->
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

<!-- النقاط -->
<div id="points" class="section"></div>

<!-- التحذيرات -->
<div id="warns" class="section"></div>

<!-- الملاحظات -->
<div id="notes" class="section">

<div id="notesList"></div>

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

const ranks=[
"فريق أول","فريق","لواء","عميد","عقيد","مقدم",
"رائد","نقيب","ملازم أول","ملازم",
"رقيب أول","رقيب","وكيل رقيب","عريف","جندي أول","جندي"
];

window.onload=()=>{
rank.innerHTML=ranks.map(r=>`<option>${r}</option>`).join("");
load();
};

/* تبويب */
function show(id,btn){
document.querySelectorAll(".section").forEach(s=>s.classList.remove("active"));
document.querySelectorAll(".nav button").forEach(b=>b.classList.remove("active"));
document.getElementById(id).classList.add("active");
btn.classList.add("active");
}

/* تحميل */
function load(){
db.ref("players").on("value",snap=>{
let data=snap.val()||{};
let arr=Object.entries(data).map(([id,v])=>({id,...v}));

renderArmy(arr);
renderPoints(arr);
renderWarns(arr);
renderNotes(arr);
});
}

/* إضافة */
function add(){

db.ref("players").push({
name:name.value?.trim()||"بدون اسم",
gid:gid.value?.trim()||"بدون ID",
rank:rank.value,
unit:unit.value,
points:0,
warn:0,
notes:[]
});

name.value="";
gid.value="";
}

/* ⭐ نقاط */
function addPoint(id){
db.ref("players/"+id).once("value",snap=>{
let p=snap.val()||{};

p.points=(p.points||0)+1;

if(p.points>=3){
p.points=0;
let i=ranks.indexOf(p.rank);
if(i>0)p.rank=ranks[i-1];
}

db.ref("players/"+id).set(p);
});
}

/* ⚠ تحذير */
function addWarn(id){
db.ref("players/"+id).once("value",snap=>{
let p=snap.val()||{};

p.warn=(p.warn||0)+1;

if(p.warn>=3){
p.warn=0;
let i=ranks.indexOf(p.rank);
if(i<ranks.length-1)p.rank=ranks[i+1];
}

db.ref("players/"+id).set(p);
});
}

/* 📝 ملاحظات */
function renderNotes(d){
notesList.innerHTML = d
.filter(p => p.notes && p.notes.length > 0)
.map(p => `
<div class="card">
<b>${p.name}</b><br>

${p.notes.map((n,i)=>`
📝 ${n.text} - ${n.time}
<button class="primary" onclick="editNote('${p.id}',${i})">تعديل</button>
<button class="danger" onclick="deleteNote('${p.id}',${i})">حذف</button><br>
`).join("")}

</div>
`).join("");
}

/* تعديل عسكري */
function editPlayer(id){
db.ref("players/"+id).once("value",snap=>{
let p=snap.val();

p.name=prompt("اسم",p.name)||p.name;
p.gid=prompt("ID",p.gid)||p.gid;

db.ref("players/"+id).set(p);
});
}

/* حذف عسكري */
function deletePlayer(id){
if(confirm("حذف؟")){
db.ref("players/"+id).remove();
}
}

/* تعديل ملاحظة */
function editNote(pid,index){
db.ref("players/"+pid).once("value",snap=>{
let p=snap.val();

let t=prompt("تعديل الملاحظة",p.notes[index].text);

if(t){
p.notes[index].text=t;
db.ref("players/"+pid).set(p);
}
});
}

/* حذف ملاحظة */
function deleteNote(pid,index){
db.ref("players/"+pid).once("value",snap=>{
let p=snap.val();

p.notes.splice(index,1);

db.ref("players/"+pid).set(p);
});
}

/* عرض بسيط */
function renderArmy(d){
armyList.innerHTML=d.map(p=>`
<div class="card">
<b>${p.name}</b><br>
🎖 ${p.rank} | 🚓 ${p.unit}
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

</script>

</body>
</html>
