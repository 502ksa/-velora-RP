<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>وزارة الداخلية</title>

<style>
body{margin:0;font-family:Tahoma;background:#070b14;color:#fff}
header{background:#0f172a;padding:16px;text-align:center;font-size:22px;font-weight:bold}

.nav{display:flex;gap:6px;justify-content:center;background:#0b1220;padding:10px;flex-wrap:wrap}
.nav button{padding:8px;border:none;border-radius:8px;background:#1f2937;color:#fff;cursor:pointer}
.nav button.active{background:#2563eb}

.container{max-width:1100px;margin:auto;padding:12px}

.card{background:#111827;padding:10px;margin:10px 0;border-radius:10px}

input,select{
width:100%;padding:10px;margin:5px 0;
background:#0a0e14;color:#fff;border:none;border-radius:8px}

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

<div class="nav">
<button class="active" onclick="show('army',this)">👮 العسكريين</button>
<button onclick="show('logs',this)">📜 السجل</button>
</div>

<div class="container">

<div class="card">
<h3>➕ إضافة عسكري</h3>

<input id="name" placeholder="اسم">
<input id="gid" placeholder="Game ID">
<input id="disc" placeholder="Discord (اختياري)">

<select id="rank"></select>

<select id="unit">
<option>قوات طوارئ</option>
<option>أمن عام</option>
<option>المرور</option>
<option>التحقيقات</option>
</select>

<button class="primary" onclick="add()">إضافة</button>
</div>

<div id="army" class="section active"></div>
<div id="logs" class="section"></div>

</div>

<script>

const ranks=[
"فريق أول","فريق","لواء","عميد","عقيد","مقدم",
"رائد","نقيب","ملازم أول","ملازم",
"رئيس رقباء","رقيب أول","رقيب","وكيل رقيب","عريف","جندي أول","جندي"
];

function get(){return JSON.parse(localStorage.getItem("data")||"[]")}
function save(d){localStorage.setItem("data",JSON.stringify(d))}

/* init */
window.onload=()=>{
rank.innerHTML=ranks.map(r=>`<option>${r}</option>`).join("");
render();
}

/* تبويب */
function show(id,btn){
document.querySelectorAll(".section").forEach(s=>s.classList.remove("active"));
document.querySelectorAll(".nav button").forEach(b=>b.classList.remove("active"));
document.getElementById(id).classList.add("active");
btn.classList.add("active");
render();
}

/* إضافة */
function add(){

let d=get();

let nameVal=name.value.trim();
let gidVal=gid.value.trim();
let discVal=disc.value.trim();

if(!nameVal || !gidVal){
alert("لازم الاسم + Game ID");
return;
}

d.push({
name:nameVal,
gid:gidVal,
disc:discVal||"لا يوجد",
rank:rank.value,
unit:unit.value,
points:0,
warn:0
});

save(d);

name.value="";
gid.value="";
disc.value="";

render();
}

/* عرض العسكريين */
function render(){

let d=get();

/* ترتيب حسب الرتبة من الأعلى للأقل */
d.sort((a,b)=>ranks.indexOf(a.rank)-ranks.indexOf(b.rank));

army.innerHTML=d.map((p,i)=>`
<div class="card">

<b>${p.name}</b>

<div class="tag">🎖 ${p.rank}</div>
<div class="tag">🚓 ${p.unit}</div>
<div class="tag">🆔 ${p.gid}</div>
<div class="tag">💬 ${p.disc}</div>

<div class="tag">⭐ ${p.points}</div>
<div class="tag">⚠ ${p.warn}</div>

</div>
`).join("");

/* السجل الكامل */
logs.innerHTML=d.map(p=>`
<div class="card">

<b>${p.name}</b><br>

🎖 الرتبة: ${p.rank}<br>
🚓 اليونت: ${p.unit}<br>
🆔 ID: ${p.gid}<br>
💬 Discord: ${p.disc}<br>
⭐ بوينت: ${p.points}<br>
⚠ تحذير: ${p.warn}

</div>
`).join("");
}

</script>

</body>
</html>
