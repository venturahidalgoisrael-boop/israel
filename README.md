# 100 mexicanos dijeron
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>100 Mexicanos Dijeron - Influencia de las Amistades</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
margin:0;
font-family:Arial, Helvetica, sans-serif;
background:linear-gradient(135deg,#0f2027,#203a43,#2c5364);
color:white;
text-align:center;
overflow-x:hidden;
}

h1{margin-top:15px;font-size:32px;}
#game{width:90%;max-width:1000px;margin:auto;background:#111;padding:20px;border-radius:20px;box-shadow:0 0 30px #000;}
.tablero{display:grid;grid-template-columns:1fr 1fr;margin-top:20px;gap:10px;}
.celda{
background:#222;
padding:15px;
border-radius:10px;
font-size:18px;
transition:0.4s;
}
.celda.revelada{
background:#00b894;
transform:scale(1.05);
}
button{
padding:10px 20px;
margin-top:10px;
font-size:16px;
border:none;
border-radius:8px;
cursor:pointer;
background:#0984e3;
color:white;
}
button:hover{background:#74b9ff;}
input{
padding:10px;
font-size:16px;
border-radius:6px;
border:none;
width:60%;
max-width:300px;
}
#equipos{display:flex;justify-content:space-around;margin-top:10px;}
.equipo{
padding:10px 20px;
border-radius:10px;
background:#222;
}
.activo{border:2px solid #ffd700;}
#errorBox{font-size:40px;color:red;height:50px;}
#timer{font-size:20px;margin:10px;}
#mensaje{font-size:18px;height:30px;}
#finalScreen{
display:none;
position:fixed;
top:0;
left:0;
width:100%;
height:100%;
background:#000;
color:white;
justify-content:center;
align-items:center;
flex-direction:column;
z-index:999;
}
canvas{position:fixed;top:0;left:0;pointer-events:none;}
</style>
</head>

<body>

<h1>🎤 100 Mexicanos Dijeron</h1>
<h3>Influencia de las Amistades en la Conducta</h3>

<div id="game">

<div id="equipos">
<div id="team1" class="equipo activo">Equipo A: <span id="score1">0</span></div>
<div id="team2" class="equipo">Equipo B: <span id="score2">0</span></div>
</div>

<h2 id="ronda"></h2>
<h3 id="pregunta"></h3>
<div id="timer">⏳ 30</div>

<input type="text" id="respuesta">
<br>
<button onclick="verificar()">Responder</button>

<div id="errorBox"></div>
<div id="mensaje"></div>

<div class="tablero" id="tablero"></div>

<br>
<button onclick="siguienteRonda()">Siguiente Ronda</button>

</div>

<div id="finalScreen">
<h1 id="ganador"></h1>
<h2 id="estadisticas"></h2>
<h3>Reflexión:</h3>
<p>Las amistades influyen en nuestras decisiones. Elegir bien nuestro entorno puede definir nuestro futuro.</p>
</div>

<canvas id="confetti"></canvas>

<script>

let rondaActual=0;
let turno=1;
let score1=0;
let score2=0;
let errores=0;
let tiempo=30;
let intervalo;
let correctas=0;

const rondas=[
{
titulo:"RONDA 1",
pregunta:"Algo malo que una amistad puede influir a hacer.",
doble:false,
respuestas:{
"consumir drogas":40,
"faltar a clases":25,
"mentir":15,
"pelear":10,
"robar":10
}
},
{
titulo:"RONDA 2",
pregunta:"Algo positivo que tus amigos pueden motivarte a hacer.",
doble:false,
respuestas:{
"estudiar":30,
"hacer ejercicio":25,
"cumplir metas":20,
"ser responsable":15,
"ayudar":10
}
},
{
titulo:"RONDA 3",
pregunta:"Razón por la que alguien cambia con amigos.",
doble:false,
respuestas:{
"presión social":35,
"encajar":30,
"miedo al rechazo":20,
"imitar":10,
"aceptación":5
}
},
{
titulo:"RONDA 4",
pregunta:"Algo que un joven hace para encajar.",
doble:false,
respuestas:{
"cambiar forma de vestir":30,
"hablar diferente":25,
"probar alcohol":20,
"actuar distinto":15,
"ocultar gustos":10
}
},
{
titulo:"RONDA 5",
pregunta:"Señal de influencia negativa.",
doble:false,
respuestas:{
"malas calificaciones":30,
"aislamiento":25,
"agresividad":20,
"mentiras":15,
"problemas escolares":10
}
},
{
titulo:"RONDA 6",
pregunta:"Algo que demuestra buenas amistades.",
doble:false,
respuestas:{
"apoyo":30,
"respeto":25,
"confianza":20,
"motivación":15,
"lealtad":10
}
},
{
titulo:"RONDA 7 PUNTOS DOBLES",
pregunta:"Decisión importante influida por amigos.",
doble:true,
respuestas:{
"elegir carrera":30,
"relaciones":25,
"sustancias":20,
"actividades escolares":15,
"proyectos personales":10
}
}
];

function cargarRonda(){
errores=0;
tiempo=30;
document.getElementById("errorBox").innerHTML="";
document.getElementById("mensaje").innerHTML="";
document.getElementById("ronda").innerHTML=rondas[rondaActual].titulo;
document.getElementById("pregunta").innerHTML=rondas[rondaActual].pregunta;
let tablero=document.getElementById("tablero");
tablero.innerHTML="";
let i=1;
for(let r in rondas[rondaActual].respuestas){
tablero.innerHTML+=`<div class="celda" id="celda${i}">${i}. ?????</div>`;
i++;
}
iniciarTimer();
}

function verificar(){
let input=document.getElementById("respuesta").value.toLowerCase();
let ronda=rondas[rondaActual];
let index=0;
let found=false;
for(let r in ronda.respuestas){
index++;
if(input===r){
found=true;
let puntos=ronda.respuestas[r];
if(ronda.doble)puntos*=2;
if(turno===1){score1+=puntos;}else{score2+=puntos;}
document.getElementById("score1").innerHTML=score1;
document.getElementById("score2").innerHTML=score2;
document.getElementById("celda"+index).classList.add("revelada");
document.getElementById("celda"+index).innerHTML=index+". "+r+" ("+puntos+")";
document.getElementById("mensaje").innerHTML="✅ Correcto!";
correctas++;
sonido(800);
break;
}
}
if(!found){
errores++;
document.getElementById("errorBox").innerHTML+="❌";
sonido(200);
if(errores>=3){
cambiarTurno();
}
}
document.getElementById("respuesta").value="";
}

function cambiarTurno(){
turno=turno===1?2:1;
document.getElementById("team1").classList.toggle("activo");
document.getElementById("team2").classList.toggle("activo");
errores=0;
document.getElementById("errorBox").innerHTML="";
}

function siguienteRonda(){
if(rondaActual<rondas.length-1){
rondaActual++;
cargarRonda();
}else{
finalJuego();
}
}

function iniciarTimer(){
clearInterval(intervalo);
intervalo=setInterval(()=>{
tiempo--;
document.getElementById("timer").innerHTML="⏳ "+tiempo;
if(tiempo<=0){
clearInterval(intervalo);
cambiarTurno();
}
},1000);
}

function finalJuego(){
document.getElementById("game").style.display="none";
document.getElementById("finalScreen").style.display="flex";
let ganador=score1>score2?"Equipo A":"Equipo B";
document.getElementById("ganador").innerHTML="🏆 Gana "+ganador;
document.getElementById("estadisticas").innerHTML="Respuestas correctas: "+correctas;
confetti();
}

function sonido(freq){
let ctx=new (window.AudioContext||window.webkitAudioContext)();
let osc=ctx.createOscillator();
osc.type="square";
osc.frequency.value=freq;
osc.connect(ctx.destination);
osc.start();
setTimeout(()=>osc.stop(),200);
}

function confetti(){
let canvas=document.getElementById("confetti");
canvas.width=window.innerWidth;
canvas.height=window.innerHeight;
let ctx=canvas.getContext("2d");
let pieces=[];
for(let i=0;i<150;i++){
pieces.push({x:Math.random()*canvas.width,y:Math.random()*canvas.height,r:Math.random()*6+2,d:Math.random()*50});
}
setInterval(()=>{
ctx.clearRect(0,0,canvas.width,canvas.height);
ctx.fillStyle="yellow";
pieces.forEach(p=>{
ctx.beginPath();
ctx.arc(p.x,p.y,p.r,0,Math.PI*2);
ctx.fill();
p.y+=2;
if(p.y>canvas.height)p.y=0;
});
},20);
}

cargarRonda();

</script>

</body>
</html>
