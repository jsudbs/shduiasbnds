<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<title>FIB HR Bewerbungs­gespräch</title>

<style>
body{
  margin:0;
  background:#0d1117;
  font-family:Segoe UI, Arial, sans-serif;
  color:#e6edf3;
}
.container{
  max-width:950px;
  margin:30px auto;
  background:#161b22;
  padding:30px;
  border-radius:12px;
  box-shadow:0 0 25px rgba(0,0,0,.6);
}
h1,h2,h3{margin-top:0}
label{margin-top:14px;display:block;font-weight:600}
input,textarea{
  width:100%;
  padding:8px;
  margin-top:6px;
  border-radius:6px;
  border:none;
  background:#0d1117;
  color:#fff;
  user-select:text!important;
}
textarea{resize:vertical}
button{
  margin-top:15px;
  padding:10px 18px;
  border:none;
  border-radius:6px;
  font-weight:600;
  cursor:pointer;
}
.primary{background:#238636;color:#fff}
.secondary{background:#30363d;color:#fff}
.danger{background:#da3633;color:#fff}
button:disabled{opacity:.5;cursor:not-allowed}
.timer{float:right;font-weight:600}
.question{
  background:#0d1117;
  padding:20px;
  border-radius:10px;
  margin-top:20px;
  user-select:none;
}
.answers label{
  display:inline-block;
  margin-right:15px;
  font-weight:normal;
}
.hidden{display:none}
.footer{
  margin-top:40px;
  font-size:11px;
  opacity:.6;
  text-align:right;
}
</style>
</head>

<body>
<div class="container">

<h1>FIB – HR Bewerbungs­gespräch
<span class="timer" id="timer">00:00</span>
</h1>

<div id="start">
<label>Bewerber – Name</label>
<input id="bewerberName">

<label>Bewerber – ID</label>
<input id="bewerberID">

<button class="primary" onclick="startInterview()">Gespräch starten</button>
</div>

<div id="interview" class="hidden">
<div class="question">
<h3 id="qTitle"></h3>
<p id="qText"></p>
<p><b>Musterlösung:</b> <span id="qAnswer"></span></p>

<div class="answers">
<label><input type="radio" name="rate" value="1"> Richtig</label>
<label><input type="radio" name="rate" value="0.5"> Halb</label>
<label><input type="radio" name="rate" value="0"> Falsch</label>
</div>

<button class="primary" onclick="nextQ()">Weiter</button>
<button class="danger" onclick="finish()">Gespräch beenden</button>
</div>
</div>

<div id="end" class="hidden">
<h2>Auswertung</h2>
<div id="summary"></div>

<label>HR-Kommentar</label>
<textarea id="hrComment"></textarea>

<label>
HR-Namen (max. 3 – nur HR-Liste Bewerbungs­gespräche)
</label>
<input id="hrNames" placeholder="Name 1, Name 2, Name 3">

<button class="primary" onclick="makePDF()">PDF erstellen</button>
</div>

<div class="footer">
Erstellt von Silas Vaught<br>
Aktualisiert am 14.01.2026
</div>

</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

<script>
/* ===== FRAGEN – ALLE ===== */
const questions=[
{q:"Wer ist die höchste Instanz im Staat und Oberbefehlshaber aller Streitkräfte?",a:"Die Regierung; Oberbefehlshaber ist der Gouverneur."},
{q:"Was versteht man unter einem Korruptionsdelikt?",a:"Jede Straftat, die von einem Beamten begangen wird."},
{q:"Welche Pflicht hat ein Beamter bei Kenntnis eines Korruptionsdelikts?",a:"Meldung bei der FIBCO oder Justiz."},
{q:"Dürfen Sie eine Weisung Ihres Vorgesetzten ablehnen?",a:"Ja, bei Unzuständigkeit oder Gesetzesverstoß."},
{q:"Wie lange besteht die Amtsverschwiegenheit?",a:"Auch nach Dienstende."},
{q:"Was passiert bei rechtskräftiger Verurteilung eines Beamten?",a:"Sofortige Beendigung des Dienstverhältnisses."},
{q:"Wie wird das Wiedereinstellungsverbot berechnet?",a:"72 Stunden pro Wanted."},
{q:"Was ist die maximale interne Disziplinarbuße?",a:"100.000 $."},
{q:"Welche Konsequenz haben rassistische Äußerungen?",a:"Kündigung."},
{q:"Dürfen eigene Familienmitglieder verfolgt werden?",a:"Nein, Befangenheit."},
{q:"Wann müssen Rechte verlesen werden?",a:"Nach Fesselung, spätestens vor Strafakte."},
{q:"Was passiert bei fehlender Rechteverlesung?",a:"Akte löschen, Person entlassen."},
{q:"Wie lange müssen Bodycam-Aufnahmen aufbewahrt werden?",a:"48 Stunden."},
{q:"Welche Interaktionen müssen aufgezeichnet werden?",a:"Alle bis zur letzten Interaktion."},
{q:"Wann muss ein Dienstausweis vorgezeigt werden?",a:"Bei Verlangen durch Justiz/FIB."},
{q:"Was passiert bei Nicht-Ausweisen?",a:"Strafbarkeit."},
{q:"Dürfen Ermittlungen gegen Immunitätsträger geführt werden?",a:"Nein, nur Beweise sammeln."},
{q:"Wer kann parlamentarische Immunität aufheben?",a:"Regierung oder Parlamentsmehrheit."},
{q:"Wann darf Privatgrund betreten werden?",a:"Mit Befehl oder Gefahr im Verzug."},
{q:"Was ist Amtsmissbrauch?",a:"Missbrauch der Stellung für Vorteil."},
{q:"Wann ist eine Schusswaffe illegal?",a:"Ohne Seriennummer oder Staatseigentum außer Dienst."},
{q:"Dürfen Dienstwaffen privat geführt werden?",a:"Nein."},
{q:"Dürfen Hieb- und Stichwaffen ohne Lizenz getragen werden?",a:"Ja."},
{q:"Was ist bei Notwehr mit illegaler Waffe zu tun?",a:"Sofortige Meldung an Exekutive."},
{q:"Dürfen FIB-Agenten illegal handeln?",a:"Ja, verdeckt, danach Vernichtung."},
{q:"Wie lange ist ein Haftbefehl gültig?",a:"30 Tage."},
{q:"Was passiert bei rechtswidrigen Beweisen?",a:"Beweis ist nichtig."},
{q:"Was ist Meineid?",a:"Vorsätzliche Falschaussage unter Eid."},
{q:"Unterschied Straftat / Ordnungswidrigkeit?",a:"Haft vs. Geldbuße."},
{q:"Verjährung der meisten Delikte?",a:"72 Stunden."}
];

/* ===== VARS ===== */
let i=0,start,end,timerInt;
let answers=[],logs=[];

/* ===== START ===== */
function startInterview(){
  start=new Date();
  logs.push({event:"Start",time:0});
  document.getElementById("start").classList.add("hidden");
  document.getElementById("interview").classList.remove("hidden");
  timerInt=setInterval(updateTimer,1000);
  showQ();
}

function updateTimer(){
  let s=Math.floor((new Date()-start)/1000);
  let m=Math.floor(s/60); s%=60;
  document.getElementById("timer").innerText=
  String(m).padStart(2,"0")+":"+String(s).padStart(2,"0");
}

function showQ(){
  document.getElementById("qTitle").innerText="Frage "+(i+1);
  document.getElementById("qText").innerText=questions[i].q;
  document.getElementById("qAnswer").innerText=questions[i].a;
  document.querySelectorAll("input[name=rate]").forEach(r=>r.checked=false);
}

function nextQ(){
  const sel=document.querySelector("input[name=rate]:checked");
  if(!sel){alert("Bitte Bewertung auswählen");return;}
  let sec=Math.floor((new Date()-start)/1000);
  answers.push(Number(sel.value));
  logs.push({event:"Weiter",frage:i+1,sekunde:sec});
  i++;
  if(i>=questions.length){finish();return;}
  showQ();
}

function finish(){
  clearInterval(timerInt);
  end=new Date();
  let sec=Math.floor((end-start)/1000);
  logs.push({event:"Ende",sekunde:sec});
  document.getElementById("interview").classList.add("hidden");
  document.getElementById("end").classList.remove("hidden");

  let sum=answers.reduce((a,b)=>a+b,0);
  let pct=Math.round((sum/answers.length)*100);
  document.getElementById("summary").innerHTML=
  `<b>Fragen:</b> ${answers.length}<br>
   <b>Punkte:</b> ${sum}<br>
   <b>Ergebnis:</b> ${pct}%`;
}

/* ===== PDF ===== */
function makePDF(){
  const {jsPDF}=window.jspdf;
  const pdf=new jsPDF();
  let y=10;

  pdf.setFont("helvetica","bold");
  pdf.text("FIB HR Bewerbungs­gespräch",10,y); y+=8;
  pdf.setFont("helvetica","normal");
  pdf.text(`Bewerber: ${bewerberName.value} | ID: ${bewerberID.value}`,10,y); y+=6;
  pdf.text(`Start: ${start.toLocaleString()}`,10,y); y+=6;
  pdf.text(`Ende: ${end.toLocaleString()}`,10,y); y+=10;

  answers.forEach((a,idx)=>{
    pdf.setFont("helvetica","bold");
    pdf.text(`Frage ${idx+1}`,10,y); y+=5;
    pdf.setFont("helvetica","normal");
    pdf.text(questions[idx].q,12,y); y+=6;
    pdf.text("Antwort: "+questions[idx].a,12,y); y+=6;
    pdf.text("Bewertung: "+a,12,y); y+=8;
    if(y>270){pdf.addPage();y=10;}
  });

  pdf.addPage();
  pdf.setFont("helvetica","bold");
  pdf.text("Auswertung",10,10);
  pdf.setFont("helvetica","normal");
  pdf.text(document.getElementById("summary").innerText,10,18);

  pdf.text("HR-Kommentar:",10,40);
  pdf.text(hrComment.value||"-",10,48);

  pdf.text("HR-Namen:",10,80);
  pdf.text(hrNames.value,10,88);

  pdf.addPage();
  pdf.setFont("helvetica","bold");
  pdf.text("Logs",10,10);
  pdf.setFont("helvetica","normal");
  let ly=18;
  logs.forEach(l=>{
    pdf.text(JSON.stringify(l),10,ly);
    ly+=6;
    if(ly>270){pdf.addPage();ly=10;}
  });

  pdf.save(`${bewerberName.value}_${bewerberID.value}.pdf`);
}

/* ===== SCHUTZ + LOG ===== */
document.addEventListener("contextmenu",e=>{
  e.preventDefault();
  logs.push({event:"Rechtsklick",sekunde:Math.floor((new Date()-start)/1000)});
});
document.addEventListener("keydown",e=>{
  if(e.ctrlKey||e.key==="F12"){
    e.preventDefault();
    logs.push({event:"Shortcut",sekunde:Math.floor((new Date()-start)/1000)});
  }
});
</script>

</body>
</html>
