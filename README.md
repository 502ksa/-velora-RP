<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>وزارة الداخلية - Velora RP</title>

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
width:100%;padding:10px;margin:5px 0;
background:#0a0e14;color:#fff;border:none;border-radius:8px;
}

button{padding:8px;border:none;border-radius:8px;cursor:pointer}
.primary{background:#2563eb;color:#fff}
.warn{background:#f59e0b}

.tag{display:inline-block;background:#1f2937;padding:4px 8px;border-radius:6px;margin:3px;font-size:12px}

.section{display:none}
.section.active{display:block}
.hidden{display:none}
</style>
</head>

<body>

<header>🏛️ وزارة الداخلية - Velora RP</header>

<div class="container">

<div class="card">
<h3>➕ إضافة ملاحظة تجريبية</h3>
<input id="testNote" placeholder="اكتب ملاحظة">
<button class="primary" onclick="testAddNote()">إضافة (اختبار)</button>
</div>

<div id="notesBox" class="card"></div>

</div>

<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-database-compat.js"></script>

<script>

/* 🔥 Firebase */
const firebaseConfig = {
  apiKey: "AIzaSyBw9V3yMuiUa5MgurcxW2V1RCImC6eHcGQ",
  authDomain: "velora-rp.firebaseapp.com",
  databaseURL: "https://velora-rp-default-rtdb.europe-west1.firebasedatabase.app",
  projectId: "velora-rp",
  storageBucket: "velora-rp.firebasestorage.app",
  messagingSenderId: "681356163114",
  appId: "1:681356163114:web:c40b8d7fed229ce8e7eb53"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.database();

/* 🧠 إضافة ملاحظة احترافية */
function testAddNote(){

let text = testNote.value.trim();
if(!text) return;

db.ref("notes").push({
text:text,
time:new Date().toLocaleString("ar"),
by:"مسؤول"
});

testNote.value="";
}

/* 📜 عرض الملاحظات */
db.ref("notes").on("value",snap=>{
let data = snap.val() || {};
let arr = Object.values(data).reverse();

notesBox.innerHTML = arr.map(n=>`
<div class="card">
📝 ${n.text}<br>
<small>⏰ ${n.time}</small>
</div>
`).join("");
});

</script>

</body>
</html>
