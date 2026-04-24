<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>وزارة الداخلية</title>

<style>
body{margin:0;font-family:Tahoma;background:#070b14;color:#fff}
header{background:#0f172a;padding:16px;text-align:center;font-size:22px;font-weight:bold}

.nav{display:flex;gap:6px;justify-content:center;background:#0b1220;padding:10px}
.nav button{padding:8px;border:none;border-radius:8px;background:#1f2937;color:#fff;cursor:pointer}
.nav button.active{background:#2563eb}

.container{padding:12px;max-width:1100px;margin:auto}

.card{background:#111827;padding:10px;margin:10px 0;border-radius:10px;border:1px solid #1f2a44}

input,textarea{
width:100%;padding:10px;margin:5px 0;
background:#0a0e14;color:#fff;border:none;border-radius:8px}

button{padding:8px;border:none;border-radius:8px;cursor:pointer}
.primary{background:#2563eb;color:#fff}
.warn{background:#f59e0b}
.danger{background:#ef4444}
.gray{background:#374151}

.tag{display:inline-block;background:#1f2937;padding:4px 8px;border-radius:6px;margin:3px;font-size:12px}

.hidden{display:none}

.log{
font-size:12px;background:#000;
padding:5px;margin:4px;border-radius:6px
}

.section{display:none}
.section.active{display:block}

</style>
</head>

<body>

<header>🏛️ وزارة الداخلية</header>

<div class="nav">
<button onclick="show('army')" class="active">👮 العسكريين</button>
<button onclick="show('logs')">📜 السجلات</button>
<button onclick="show('stats')">📊 الإحصائيات</button>
</div>

<div class="container">

<div class="card">
<h3>➕ إضافة عسكري</h3>
<input id="name" placeholder="اسم العسكري">
<input id="id" placeholder="Game ID (إجباري)">
<input id="discord" placeholder="Discord (اختياري)">
<button class="primary" onclick="add()">إضافة</button>
</div>

<div class="card">
<h3>🔎 بحث</h3>
<input id="search" placeholder="اكتب الاسم">
<button onclick="find()">بحث</button>
<div id="result"></div>
</div>

<!-- العسكريين -->
<div id="army" class="section active"></div>

<!-- السجلات -->
<div id="logs" class="section">
<div class="card">
<h3>⭐ سجل البوينتات</h3>
<div id="logPoints"></div>
</div>

<div class="card">
<h3>⚠ سجل التحذيرات</h3>
<div id="logWarn"></div>
</div>

<div class="card">
<h3>📝 سجل الملاحظات</h3>
<div id="logNotes"></div>
</div>
</div>

<!-- الإحصائيات -->
<div id="stats" class="section"></div>

</div>

<script>

const ranks=[
"جندي","جندي أول","عريف","وكيل رقيب","رقيب","رقيب أول",
"رئيس رقباء","ملازم","ملازم أول","نقيب","رائد","مقدم",
"عقيد","عميد","لواء","فريق","فريق أول"
];

function get(){return JSON.parse(localStorage.getItem("data")||"[]")}
function save(d){localStorage.setItem("data",JSON.stringify(d))}

function logs(type,text){
let l=JSON.parse(localStorage.getItem("logs")||"{}");

if(!l.points)l.points=[];
if(!l.warn)l.warn=[];
if(!l.notes)l.notes=[];

l[type].unshift(new Date().toLocaleString()+" - "+text);

localStorage.setItem("logs",JSON.stringify(l));
}

function show(x){
document.querySelectorAll(".section").forEach(s=>s.classList.remove("active"));
document.getElementById(x).classList.add("active");
render();
}

/* إضافة */
function add(){
let d=get();

if(!name.value || !id.value){
return alert("الاسم + Game ID مطلوب");
}

d.push({
name:name.value,
id:id.value,
discord:discord.value||"لا يوجد",
rank:"جندي",
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

army.innerHTML=d.map((p,i)=>`
<div class="card">
<b>${p.name}</b>

<div class="tag">🆔 ${p.id}</div>
<div class="tag">💬 ${p.discord}</div>
<div class="tag">🎖 ${p.rank}</div>
<div class="tag">⭐ ${p.points}</div>
<div class="tag">⚠ ${p.warn}</div>

<div>
<button class="primary" onclick="point(${i})">⭐ بوينت</button>
<button class="warn" onclick="warn(${i})">⚠ تحذير</button>
<button onclick="note(${i})">📝 ملاحظة</button>
<button class="danger" onclick="del(${i})">🗑 حذف</button>
</div>
</div>
`).join("");


/* السجلات */
let l=JSON.parse(localStorage.getItem("logs")||"{}");

logPoints.innerHTML=(l.points||[]).slice(0,20).map(x=>`<div class="log">${x}</div>`).join("");
logWarn.innerHTML=(l.warn||[]).slice(0,20).map(x=>`<div class="log">${x}</div>`).join("");
logNotes.innerHTML=(l.notes||[]).slice(0,20).map(x=>`<div class="log">${x}</div>`).join("");


/* احصائيات */
stats.innerHTML="عدد العسكريين: "+d.length;
}

/* بحث */
function find(){
let d=get();
let q=search.value.trim();

let p=d.find(x=>x.name.includes(q));

if(!p)return result.innerHTML="❌ غير موجود";

let i=d.indexOf(p);

result.innerHTML=`
<div class="card">
<b>${p.name}</b>

<div>🆔 ${p.id}</div>
<div>💬 ${p.discord}</div>

<button class="primary" onclick="point(${i})">⭐ بوينت</button>
<button class="warn" onclick="warn(${i})">⚠ تحذير</button>
<button onclick="note(${i})">📝 ملاحظة</button>
</div>
`;
}

/* بوينت */
function point(i){
let d=get();
d[i].points++;

logs("points",d[i].name);

if(d[i].points>=3){
let idx=ranks.indexOf(d[i].rank);
if(idx<ranks.length-1)d[i].rank=ranks[idx+1];
d[i].points=0;
}

save(d);render();
}

/* تحذير */
function warn(i){
let d=get();
d[i].warn++;

logs("warn",d[i].name);

if(d[i].warn>=3){
let idx=ranks.indexOf(d[i].rank);
if(idx>0)d[i].rank=ranks[idx-1];
d[i].warn=0;
}

save(d);render();
}

/* ملاحظة */
function note(i){
let t=prompt("اكتب الملاحظة");
if(!t)return;

let d=get();
d[i].notes.push(t);

logs("notes",d[i].name+" - "+t);

save(d);render();
}

/* حذف */
function del(i){
let d=get();
if(!confirm("متأكد؟ 🚨"))return;

d.splice(i,1);
save(d);
render();
}

</script>

</body>
</html>
