<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Weather Forecast Dashboard</title>

<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap" rel="stylesheet">

<style>
*{
margin:0;
padding:0;
box-sizing:border-box;
font-family:'Poppins',sans-serif;
}

body{
min-height:100vh;
background:linear-gradient(135deg,#0f2027,#203a43,#2c5364);
display:flex;
justify-content:center;
align-items:center;
padding:20px;
}

.container{
width:100%;
max-width:950px;
background:rgba(255,255,255,0.12);
backdrop-filter:blur(20px);
border-radius:25px;
padding:30px;
box-shadow:0 10px 40px rgba(0,0,0,0.3);
color:#fff;
}

.header{
text-align:center;
margin-bottom:30px;
}

.header h1{
font-size:2.7rem;
font-weight:700;
margin-bottom:10px;
}

.header p{
color:#dcdcdc;
}

.search{
display:flex;
gap:12px;
margin-bottom:25px;
}

.search input{
flex:1;
padding:15px;
border:none;
border-radius:12px;
outline:none;
font-size:16px;
}

.search button{
padding:15px 25px;
border:none;
border-radius:12px;
background:#00c6ff;
color:#fff;
cursor:pointer;
font-weight:600;
}

.search button:hover{
background:#0099cc;
}

.datetime{
text-align:center;
margin-bottom:20px;
font-size:15px;
color:#ddd;
}

.weather{
display:none;
}

.top-info{
display:flex;
justify-content:space-between;
align-items:center;
flex-wrap:wrap;
}

.city{
font-size:2rem;
font-weight:700;
}

.temp{
font-size:5rem;
font-weight:700;
}

.icon{
font-size:5rem;
}

.cards{
display:grid;
grid-template-columns:repeat(auto-fit,minmax(180px,1fr));
gap:20px;
margin-top:25px;
}

.card{
background:rgba(255,255,255,0.15);
padding:20px;
border-radius:18px;
text-align:center;
}

.card h3{
font-size:15px;
margin-bottom:10px;
color:#eee;
}

.card p{
font-size:22px;
font-weight:600;
}

.history{
margin-top:30px;
}

.history h3{
margin-bottom:10px;
}

.tags{
display:flex;
flex-wrap:wrap;
gap:10px;
}

.tag{
background:rgba(255,255,255,0.15);
padding:8px 14px;
border-radius:20px;
font-size:14px;
}

.loading{
display:none;
text-align:center;
font-size:18px;
margin-top:20px;
}

@media(max-width:768px){
.temp{
font-size:3.5rem;
}

.city{
font-size:1.5rem;
}

.header h1{
font-size:2rem;
}
}
</style>
</head>
<body>

<div class="container">

<div class="header">
<h1>🌦 Weather Forecast Dashboard</h1>
<p>Get real-time weather updates for cities around the world</p>
</div>

<div class="datetime" id="datetime"></div>

<div class="search">
<input type="text" id="cityInput" placeholder="Enter city name">
<button onclick="getWeather()">Search</button>
</div>

<div class="loading" id="loading">
Loading weather data...
</div>

<div class="weather" id="weather">

<div class="top-info">
<div>
<div class="city" id="city"></div>
<div id="country"></div>
</div>

<div>
<div class="icon" id="icon">☀️</div>
<div class="temp" id="temp"></div>
</div>
</div>

<div class="cards">

<div class="card">
<h3>💧 Humidity</h3>
<p id="humidity">--</p>
</div>

<div class="card">
<h3>🌬 Wind Speed</h3>
<p id="wind">--</p>
</div>

<div class="card">
<h3>📈 Pressure</h3>
<p id="pressure">--</p>
</div>

<div class="card">
<h3>🌡 Feels Like</h3>
<p id="feels">--</p>
</div>

</div>

</div>

<div class="history">
<h3>Recent Searches</h3>
<div class="tags" id="historyTags"></div>
</div>

</div>

<script>

let historyCities=[];

function updateTime(){
document.getElementById("datetime").innerHTML=
new Date().toLocaleString();
}

setInterval(updateTime,1000);
updateTime();

async function getWeather(){

const city=document.getElementById("cityInput").value.trim();

if(city===""){
alert("Please enter a city name");
return;
}

document.getElementById("loading").style.display="block";
document.getElementById("weather").style.display="none";

try{

const geo=await fetch(
`https://geocoding-api.open-meteo.com/v1/search?name=${city}&count=1`
);

const geoData=await geo.json();

if(!geoData.results){
throw new Error("City not found");
}

const lat=geoData.results[0].latitude;
const lon=geoData.results[0].longitude;

const cityName=geoData.results[0].name;
const country=geoData.results[0].country;

const weather=await fetch(
`https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current=temperature_2m,relative_humidity_2m,apparent_temperature,pressure_msl,wind_speed_10m`
);

const data=await weather.json();

document.getElementById("city").innerHTML=cityName;
document.getElementById("country").innerHTML=country;
document.getElementById("temp").innerHTML=
Math.round(data.current.temperature_2m)+"°C";

document.getElementById("humidity").innerHTML=
data.current.relative_humidity_2m+"%";

document.getElementById("wind").innerHTML=
data.current.wind_speed_10m+" km/h";

document.getElementById("pressure").innerHTML=
data.current.pressure_msl+" hPa";

document.getElementById("feels").innerHTML=
Math.round(data.current.apparent_temperature)+"°C";

const temp=data.current.temperature_2m;

let icon="☀️";

if(temp<10) icon="❄️";
else if(temp<20) icon="⛅";
else if(temp<30) icon="🌤️";
else icon="☀️";

document.getElementById("icon").innerHTML=icon;

document.getElementById("loading").style.display="none";
document.getElementById("weather").style.display="block";

if(!historyCities.includes(cityName)){
historyCities.unshift(cityName);

if(historyCities.length>5){
historyCities.pop();
}

showHistory();
}

}catch(error){

document.getElementById("loading").style.display="none";
alert("City not found");

}
}

function showHistory(){

const container=document.getElementById("historyTags");

container.innerHTML="";

historyCities.forEach(city=>{

const span=document.createElement("span");
span.className="tag";
span.innerText=city;

container.appendChild(span);

});

}

</script>

</body>
</html>
