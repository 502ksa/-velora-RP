<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>وزارة الداخلية</title>

<style>
body{margin:0;font-family:Tahoma;background:#070b14;color:#fff}

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

.card{background:#111827;padding:10px;margin:10px 0;border-radius:10px}

input,select{
width:100%;
padding:10px;
margin:5px 0;
background:#0a0e14;
color:#fff;
border:none;
border-radius:8px
}

button{padding:8px;border:none;border-radius:8px;cursor:pointer}
.primary{background:#2563eb;color:#fff}
.warn{background:#f59e0b}
.danger{background:#ef4444}

.tag{display:inline-block;background:#1f2937;padding:4px 8px;border-radius:6px;margin:3px;font-size:12px}

.section{display:none}
.section.active{display:block}
</style>
</head>

<body>

<header>🏛️ وزارة الداخلية</header>

<!-- NAV -->
<div class="nav">
<button class="active" onclick="show('army',this)">👮 العسكريين</button>
<button onclick="show('promo',this)">⭐ الترقيات</button>
<button onclick="show('warns',this)">⚠ التحذيرات</button>
<button onclick="show('notes',this)">📝 الملاحظات</button>
<button onclick="show('logs',this)">📜 السجل</button>
</div>

<div class="container">

<!-- إضافة -->
<div class="card">
<h3>➕ إضافة عسكري</h3>

<input id="name" placeholder="اسم">
<input id="gid" placeholder="Game ID">
<input id="disc" placeholder="Discord (اختياري)">

<select id="rank"></select>

<select id="unit">
<option>قوات طوارئ</option>
<option>أمن عام</option>
</select>

<button class="primary" onclick="add()">إضافة</button>
</div>

<!-- بحث -->
<div class="card">
<h3>🔎 بحث</h3>
<input id="search" placeholder="اسم">
<button onclick="find()">بحث</button>
<div id="result"></div>
</div>

<!-- العسكريين -->
<div id="army" class="section active"></div>

<!-- الترقيات -->
<div id="promo" class="section"></div>

<!-- التحذيرات -->
<div id="warns" class="section"></div>

<!-- الملاحظات -->
<div id="notes" class="section"></div>

<!-- السجل -->
<div id="logs" class="section"></div>

</div>

<script>

const ranks=[
"جندي","جندي أول","عريف","وكيل رقيب","رقيب","رقيب أول",
"رئيس رقباء","ملازم","ملازم أول","نقيب","رائد","مقدم",
"عقيد","عميد","لواء","فريق","فريق أول"
];

const colors={
"جندي":"#6b7280",
"عريف":"#22c55e",
"رقيب":"#f59e0b",
"ملازم":"#3b82f6",
"نقيب":"#a855f7",
"مقدم":"#eab308",
"عقيد":"#ef4444",
"لواء":"#111827",
"فريق أول":"#000"
};

function get(){return JSON.parse(localStorage.getItem("data")||"[]")}
function save(d){localStorage.setItem("data",JSON.stringify(d))}

function log(t){
let l=JSON.parse(localStorage.getItem("logs")||"[]");
l.unshift(new Date().toLocaleString()+" - "+t);
localStorage.setItem("logs",JSON.stringify(l));
}

/* تبويب */
function show(id,btn){

document.querySelectorAll(".section").forEach(s=>s.classList.remove("active"));
document.querySelectorAll(".nav button").forEach(b=>b.classList.remove("active"));

document.getElementById(id).classList.add("active");
btn.classList.add("active");

render();
}

/* init */
window.onload=()=>{
rank.innerHTML=ranks.map(r=>`<option>${r}</option>`).join("");
render();
}

/* إضافة */
function add(){

let d=get();

if(!name.value || !gid.value){
return alert("لازم الاسم + Game ID");
}

d.push({
name:name.value,
gid:gid.value,
disc:disc.value||"لا يوجد",
rank:rank.value,
unit:unit.value,
points:0,
warn:0,
notes:[]
});

save(d);
render();
}

/* عرض */
function render(){

let d=get();

d.sort((a,b)=>ranks.indexOf(b.rank)-ranks.indexOf(a.rank));

army.innerHTML=d.map((p,i)=>`
<div class="card">
<b>${p.name}</b>
<div class="tag" style="background:${colors[p.rank]}">${p.rank}</div>
<div class="tag">🚓 ${p.unit}</div>
<div class="tag">🆔 ${p.gid}</div>
</div>
`).join("");

promo.innerHTML=d.map(p=>`
<div class="card">${p.name} - ⭐ ${p.points}</div>
`).join("");

warns.innerHTML=d.map(p=>`
<div class="card">${p.name} - ⚠ ${p.warn}</div>
`).join("");

notes.innerHTML=d.map(p=>`
<div class="card">${p.name} - 📝 ${p.notes?.length||0}</div>
`).join("");

logs.innerHTML=(JSON.parse(localStorage.getItem("logs")||"[]"))
.slice(0,30)
.map(x=>`<div class="card">${x}</div>`).join("");
}

/* بحث */
function find(){

let d=get();
let q=search.value.trim();

let p=d.find(x=>x.name.includes(q));

if(!p)return result.innerHTML="❌ غير موجود";

result.innerHTML=`
<div class="card">
<b>${p.name}</b><br>
🎖 ${p.rank}<br>
🚓 ${p.unit}
</div>
`;
}

</script>

</body>
</html>
