<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>وزارة الداخلية</title>

<style>
body{margin:0;font-family:Tahoma;background:#070b14;color:#fff}
header{
background:linear-gradient(90deg,#0f172a,#1e3a8a);
padding:18px;text-align:center;
font-size:22px;font-weight:bold;
border-bottom:2px solid #2563eb;
}

.sub{font-size:12px;opacity:.7}

.container{padding:12px;max-width:1100px;margin:auto}

.card{
background:#111827;
border:1px solid #1f2a44;
border-radius:12px;
padding:12px;
margin:10px 0;
box-shadow:0 0 10px rgba(0,0,0,.4);
}

input,select{
width:100%;
padding:10px;
margin:6px 0;
background:#0a0e14;
border:none;
border-radius:8px;
color:#fff;
}

button{
padding:8px 10px;
border:none;
border-radius:8px;
cursor:pointer;
font-weight:bold;
}

.primary{background:#2563eb;color:#fff}
.warn{background:#f59e0b}
.danger{background:#ef4444}
.gray{background:#374151}

.tag{
display:inline-block;
padding:4px 8px;
border-radius:6px;
margin:3px;
font-size:12px;
background:#1f2937;
}

.grid{
display:grid;
grid-template-columns:repeat(auto-fit,minmax(250px,1fr));
gap:10px;
}

.log{
font-size:12px;
background:#000;
padding:6px;
margin:4px 0;
border-radius:6px;
opacity:.9;
}

.big{
font-size:14px;
font-weight:bold;
}

</style>
</head>

<body>

<header>
🏛️ وزارة الداخلية
<div class="sub">نظام الرصد والمتابعة الرسمي</div>
</header>

<div class="container">

<div class="card" id="loginBox">
<h3>🔐 دخول الوزارة</h3>
<input id="pass" placeholder="أدخل كلمة المرور">
<button class="primary" onclick="login()">دخول</button>
</div>

<div id="app" style="display:none">

<div class="grid">

<!-- إضافة -->
<div class="card">
<h3>➕ إضافة عسكري</h3>
<input id="name" placeholder="الاسم">
<input id="id" placeholder="ID">
<select id="rank"></select>
<select id="unit">
<option>Patrol</option>
<option>Traffic</option>
<option>SWAT</option>
<option>Investigation</option>
</select>
<button class="primary" onclick="add()">إضافة</button>
</div>

<!-- بحث -->
<div class="card">
<h3>🔎 بحث الوزارة</h3>
<input id="searchName" placeholder="اسم العسكري">
<button class="primary" onclick="find()">بحث</button>
<div id="result"></div>
</div>

<!-- إحصائيات -->
<div class="card">
<h3>📊 إحصائيات الوزارة</h3>
<div id="stats"></div>
</div>

</div>

<!-- القائمة -->
<div id="list"></div>

<!-- السجل -->
<div class="card">
<h3>📜 سجل الوزارة</h3>
<div id="logs"></div>
</div>

</div>

</div>

<script>

const PASS="0008";

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
"فريق أول":"#000000"
};

function login(){
if(pass.value!==PASS)return alert("خطأ");
loginBox.style.display="none";
app.style.display="block";
init();
render();
}

function init(){
rank.innerHTML=ranks.map(r=>`<option>${r}</option>`).join("");
}

function get(){return JSON.parse(localStorage.getItem("data")||"[]")}
function save(d){localStorage.setItem("data",JSON.stringify(d))}

function log(t){
let l=JSON.parse(localStorage.getItem("logs")||"[]");
l.unshift(new Date().toLocaleString()+" - "+t);
localStorage.setItem("logs",JSON.stringify(l));
}

function add(){
let d=get();
d.push({
name:name.value,
id:id.value,
rank:rank.value,
unit:unit.value,
points:0,
warn:0
});
log("إضافة "+name.value);
save(d);
render();
}

function render(){
let d=get();

d.sort((a,b)=>ranks.indexOf(b.rank)-ranks.indexOf(a.rank));

list.innerHTML=d.map((p,i)=>`
<div class="card">
<b class="big">${p.name}</b>
<div class="tag" style="background:${colors[p.rank]||'#1f2937'}">${p.rank}</div>
<div class="tag">${p.unit}</div>
<div class="tag">⭐ ${p.points}</div>
<div class="tag">⚠ ${p.warn}</div>

<div>
<button class="primary" onclick="point(${i})">⭐ بوينت</button>
<button class="warn" onclick="warn(${i})">⚠ تحذير</button>
<button class="gray" onclick="edit(${i})">✏️ تعديل</button>
</div>
</div>
`).join("");

/* stats */
let s={};
d.forEach(x=>s[x.rank]=(s[x.rank]||0)+1);

stats.innerHTML=
"👥 عدد المواطنين: "+d.length+"<br><br>"+
Object.keys(s).map(k=>k+" : "+s[k]).join("<br>");

/* logs */
let l=JSON.parse(localStorage.getItem("logs")||"[]");
logs.innerHTML=l.slice(0,20).map(x=>`<div class="log">${x}</div>`).join("");
}

/* بحث */
function find(){
let d=get();
let p=d.find(x=>x.name.includes(searchName.value));
if(!p)return result.innerHTML="❌ غير موجود";

let i=d.indexOf(p);

result.innerHTML=`
<div class="card">
<b>${p.name}</b>
<div>${p.rank} - ${p.unit}</div>

<button class="primary" onclick="point(${i})">⭐ بوينت</button>
<button class="warn" onclick="warn(${i})">⚠ تحذير</button>
<button onclick="edit(${i})">✏️ تعديل</button>
</div>
`;
}

/* بوينت */
function point(i){
let d=get();
d[i].points++;

log("بوينت "+d[i].name);

if(d[i].points>=3){
let idx=ranks.indexOf(d[i].rank);
if(idx<ranks.length-1){
d[i].rank=ranks[idx+1];
log("ترقية "+d[i].name);
}
d[i].points=0;
}

save(d);render();find();
}

/* تحذير */
function warn(i){
let d=get();
d[i].warn++;

log("تحذير "+d[i].name);

if(d[i].warn>=3){
let idx=ranks.indexOf(d[i].rank);
if(idx>0){
d[i].rank=ranks[idx-1];
log("تنزيل "+d[i].name);
}
d[i].warn=0;
}

save(d);render();find();
}

/* تعديل */
function edit(i){
let d=get();
let p=d[i];

let n=prompt("الاسم",p.name);
if(n===null)return;

let u=prompt("يونت",p.unit);
let r=prompt("رقم الرتبة:\n"+ranks.map((x,i)=>i+"-"+x).join("\n"),ranks.indexOf(p.rank));

p.name=n;
p.unit=u;
p.rank=ranks[r];

log("تعديل "+p.name);

save(d);render();find();
}

</script>

</body>
</html>
