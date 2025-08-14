# Wheather-of-Pakistan-to-all-cities-
<!DOCTYPE html>
<html lang="ur" dir="rtl">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>پاکستان کے شہروں کا موسم</title>
  <style>
    body{font-family: system-ui,-apple-system,Segoe UI,Roboto,Noto Naskh Arabic,Arial; background:#f5f7fb; margin:0; padding:24px;}
    .card{max-width:560px; margin:auto; background:#fff; padding:24px; border-radius:16px; box-shadow:0 10px 30px rgba(0,0,0,.06)}
    h1{margin:0 0 16px; font-size:26px}
    label{display:block; margin:12px 0 6px}
    input{width:100%; padding:12px 14px; border:1px solid #d0d7e1; border-radius:10px; font-size:16px}
    button{margin-top:12px; width:100%; padding:12px 14px; border:0; border-radius:12px; font-size:16px; cursor:pointer; background:#1355d4; color:#fff}
    .muted{color:#6b7280; margin-top:8px; font-size:14px}
    .error{color:#b91c1c; margin-top:12px}
    .result{margin-top:16px; line-height:1.9}
    .row{display:flex; gap:10px; flex-wrap:wrap}
    .chip{background:#f1f5f9; border-radius:999px; padding:6px 10px; font-size:14px; cursor:pointer}
  </style>
</head>
<body>
  <div class="card">
    <h1>موسم معلوم کریں (پاکستان)</h1>

    <label for="city">شہر کا نام لکھیں</label>
    <input id="city" placeholder="کراچی / Karachi" value="کراچی" />

    <div class="row" id="quick">
      <span class="chip">کراچی</span>
      <span class="chip">Lahore</span>
      <span class="chip">اسلام آباد</span>
      <span class="chip">Quetta</span>
      <span class="chip">Peshawar</span>
      <span class="chip">Multan</span>
    </div>

    <button id="go">موسم دکھائیں</button>
    <div class="muted">Hint: Urdu ya English dono chale گا. Extra spaces/harakat se farq نہیں پڑے گا۔</div>
    <div id="msg" class="error" hidden></div>
    <div id="out" class="result"></div>
  </div>

<script>
const $ = (s)=>document.querySelector(s);
const msg = $("#msg"), out = $("#out"), cityIn = $("#city");

document.getElementById("quick").addEventListener("click", (e)=>{
  if(e.target.classList.contains("chip")){
    cityIn.value = e.target.textContent.trim();
    fetchWeather();
  }
});

$("#go").addEventListener("click", fetchWeather);
cityIn.addEventListener("keydown", (e)=>{ if(e.key==="Enter") fetchWeather(); });

async function fetchWeather(){
  const qRaw = cityIn.value.normalize("NFC").trim();
  msg.hidden = true; msg.textContent = ""; out.innerHTML = "";

  if(!qRaw){ showErr("براہِ کرم شہر کا نام درج کریں۔"); return; }

  try{
    // 1) Geocoding (Open-Meteo) — API key not required
    const geoURL = new URL("https://geocoding-api.open-meteo.com/v1/search");
    geoURL.searchParams.set("name", qRaw);
    geoURL.searchParams.set("count", "10");
    geoURL.searchParams.set("language", "ur");
    geoURL.searchParams.set("format", "json");

    const gRes = await fetch(geoURL);
    if(!gRes.ok) throw new Error("Geocoding failed");
    const g = await gRes.json();

    // filter to Pakistan only
    const hits = (g.results||[]).filter(r => (r.country_code==="PK" || /Pakistan/i.test(r.country||"")));
    if(hits.length===0){ showErr("شہر نہیں ملا۔ دوبارہ کوشش کریں یا انگریزی ہجوں میں لکھیں۔"); return; }

    // choose best match (highest population or first)
    hits.sort((a,b)=> (b.population||0)-(a.population||0));
    const best = hits[0];

    // 2) Current weather
    const wxURL = new URL("https://api.open-meteo.com/v1/forecast");
    wxURL.searchParams.set("latitude", best.latitude);
    wxURL.searchParams.set("longitude", best.longitude);
    wxURL.searchParams.set("current_weather", "true");
    wxURL.searchParams.set("timezone", "auto");

    const wRes = await fetch(wxURL);
    if(!wRes.ok) throw new Error("Weather fetch failed");
    const w = await wRes.json();

    const c = w.current_weather;
    const name = `${best.name}${best.admin1? "، "+best.admin1: ""} (${best.country_code})`;

    out.innerHTML = `
      <div><strong>${name}</strong></div>
      <div>درجۂ حرارت: <strong>${Math.round(c.temperature)}°C</strong></div>
      <div>ہوا کی رفتار: ${c.windspeed} km/h</div>
      <div>ہوا کا رُخ: ${degToDir(c.winddirection)} (${Math.round(c.winddirection)}°)</div>
      <div>وقت: ${new Date(c.time).toLocaleString()}</div>
    `;
  }catch(e){
    console.error(e);
    showErr("سرور سے رابطہ نہیں ہو سکا۔ انٹرنیٹ چیک کریں اور دوبارہ کوشش کریں۔");
  }
}

function showErr(t){ msg.textContent = t; msg.hidden = false; }
function degToDir(d){
  const dirs = ["N","NNE","NE","ENE","E","ESE","SE","SSE","S","SSW","SW","WSW","W","WNW","NW","NNW"];
  return dirs[Math.round(((d%360)+360)%360/22.5)%16];
}
</script>
</body>
</html>
