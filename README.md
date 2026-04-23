# -وزارة الداخلية

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
background:#0b0f17;
color:white;
}

header{
background:#111827;
padding:18px;
text-align:center;
border-bottom:2px solid #2563eb;
}

.container{
max-width:900px;
margin:auto;
padding:15px;
}

.card{
background:#161b22;
padding:15px;
margin:10px 0;
border-radius:10px;
border:1px solid #222c3a;
}

input,textarea{
width:100%;
padding:10px;
margin:6px 0;
border:none;
border-radius:8px;
background:#0a0e14;
color:white;
}

button{
width:100%;
padding:12px;
margin-top:8px;
border:none;
border-radius:8px;
background:#2563eb;
color:white;
font-weight:bold;
cursor:pointer;
}

.small{
font-size:12px;
opacity:0.7;
}

.hidden{display:none;}

.tag{
display:inline-block;
padding:4px 8px;
background:#1f2937;
border-radius:6px;
margin:2px;
font-size:12px;
}

.warn{
color:#fbbf24;
}

.bad{
color:#ef4444;
}

.good{
color:#22c55e;
}

hr{opacity:0.2;}

</style>
</head>

<body>

<header>
🏛️ وزارة الداخلية - Velora RP
<div class="small">نظام إدارة الموارد البشرية</div>
</header>

<div class="container">

<!-- تسجيل دخول مسؤول -->
<div class="card">
<h3>🔐 دخول المسؤول</h3>
<button onclick="login()">تسجيل دخول</button>
</div>

<!-- تقديم -->
<div class="card">
<h3>📄 نظام التقديم</h3>

<input id="name" placeholder="الاسم">
<input id="age" placeholder="العمر">
<input id="discord" placeholder="Discord ID">
<input id="game" placeholder="Game ID">
<textarea id="exp" placeholder="الخبرات"></textarea>

<button onclick="send()">إرسال طلب</button>
<p id="msg"></p>
</div>

<!-- لوحة الوزارة -->
<div id="panel" class="hidden">

<div class="card">
<h3>👮 لوحة وزارة الداخلية</h3>
<div id="list"></div>
</div>

</div>

</div>

<script>

let logged=false;

/* تسجيل دخول مسؤول */
function login(){

let pass=prompt("ادخل كلمة السر");

if(pass!="0008"){
alert("❌ خطأ");
return;
}

logged=true;
panel.classList.remove("hidden");
load();

}

/* إرسال طلب */
function send(){

let data={
name:name.value,
age:age.value,
discord:discord.value,
game:game.value,
exp:exp.value,
status:"pending",
warnings:0,
notes:[]
};

if(!data.name || !data.age || !data.discord || !data.game || !data.exp){
alert("عبي كل البيانات");
return;
}

let arr=JSON.parse(localStorage.getItem("apps")||"[]");
arr.push(data);
localStorage.setItem("apps",JSON.stringify(arr));

msg.innerText="✔ تم إرسال الطلب";

}

/* تحميل */
function load(){

let arr=JSON.parse(localStorage.getItem("apps")||"[]");

let html="";

arr.forEach((a,i)=>{

html+=`
<div class="card">

<h3>👤 ${a.name}</h3>

<div class="tag">🎂 ${a.age}</div>
<div class="tag">🎮 ${a.game}</div>
<div class="tag">💬 ${a.discord}</div>

<p>📄 ${a.exp}</p>

<p class="${a.status=='accepted'?'good':a.status=='rejected'?'bad':'warn'}">
📊 الحالة: ${a.status}
</p>

<p>⚠ التحذيرات: ${a.warnings}</p>

<button onclick="accept(${i})">✔ قبول</button>
<button onclick="reject(${i})">✖ رفض</button>
<button onclick="warn(${i})">⚠ تحذير</button>
<button onclick="note(${i})">📝 إضافة ملاحظة</button>

<hr>

<div id="notes_${i}"></div>

</div>
`;
});

list.innerHTML=html;

/* عرض الملاحظات */
arr.forEach((a,i)=>{
let n="";
a.notes.forEach(t=>{
n+=`<div class="tag">📝 ${t}</div>`;
});
let el=document.getElementById("notes_"+i);
if(el) el.innerHTML=n;
});

}

/* قبول */
function accept(i){
let arr=JSON.parse(localStorage.getItem("apps"));
arr[i].status="accepted";
localStorage.setItem("apps",JSON.stringify(arr));
load();
}

/* رفض */
function reject(i){
let arr=JSON.parse(localStorage.getItem("apps"));
arr[i].status="rejected";
localStorage.setItem("apps",JSON.stringify(arr));
load();
}

/* تحذير */
function warn(i){
let arr=JSON.parse(localStorage.getItem("apps"));
arr[i].warnings++;
localStorage.setItem("apps",JSON.stringify(arr));
load();
}

/* ملاحظة */
function note(i){

let text=prompt("اكتب الملاحظة");

if(!text) return;

let arr=JSON.parse(localStorage.getItem("apps"));
arr[i].notes.push(text);

localStorage.setItem("apps",JSON.stringify(arr));
load();

}

</script>

</body>
</html>
