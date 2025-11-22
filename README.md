<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>www.ramith.com • Calculator • World Clock • Weather • Music</title>
  <style>
    :root{--bg:#0f1724;--card:#0b1220;--muted:#9aa6b2;--accent:#67e8f9}
    *{box-sizing:border-box;font-family:Inter,system-ui,Segoe UI,Roboto,Arial}
    body{margin:0;background:linear-gradient(180deg,#071024 0%, #071633 100%);color:#e6eef6;min-height:100vh;display:flex;align-items:center;justify-content:center;padding:24px}
    .grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(320px,1fr));gap:18px;width:1100px;max-width:100%}
    .card{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));padding:18px;border-radius:12px;box-shadow:0 6px 18px rgba(2,6,23,0.6);}
    h1{font-size:18px;margin:0 0 12px}

    /* Calculator */
    .calc{display:flex;flex-direction:column;gap:12px}
    .display{background:rgba(255,255,255,0.03);padding:12px;border-radius:8px;text-align:right;font-size:28px;min-height:48px}
    .keys{display:grid;grid-template-columns:repeat(4,1fr);gap:8px}
    button.key{padding:14px;border-radius:8px;border:0;background:rgba(255,255,255,0.03);color:inherit;font-size:16px;cursor:pointer}
    button.key.operator{background:rgba(103,232,249,0.12)}
    button.key.equal{grid-column:span 2;background:var(--accent);color:#052023}

    /* Clock */
    .clock-line{display:flex;gap:8px;align-items:center;flex-wrap:wrap}
    select, input[type=text]{padding:8px;border-radius:8px;border:0;background:rgba(255,255,255,0.03);color:inherit}
    .time{font-size:28px;font-weight:600}

    /* Weather */
    .weather-row{display:flex;gap:12px;align-items:center}
    .weather-box{min-width:120px}

    /* Music */
    .playlist{max-height:220px;overflow:auto;margin-top:8px}
    .track{display:flex;justify-content:space-between;padding:6px;border-radius:6px;margin-bottom:6px;background:rgba(255,255,255,0.02);cursor:pointer}
    .controls{display:flex;gap:8px;align-items:center;margin-top:8px}
    input[type=range]{width:100%}

    footer{margin-top:12px;font-size:12px;color:var(--muted)}
  </style>
</head>
<body>
  <div class="grid">

    <!-- Calculator -->
    <div class="card">
      <h1>Calculator</h1>
      <div class="calc">
        <div id="calcDisplay" class="display">0</div>
        <div class="keys" id="keys"></div>
      </div>
      <footer>Supports mouse & keyboard. Use Enter for equals and Backspace to delete.</footer>
    </div>

    <!-- World Clock -->
    <div class="card">
      <h1>World Clock</h1>
      <div class="clock-line">
        <label for="tzSelect">Timezone / City (IANA timezone or leave blank for common list):</label>
        <select id="tzSelect" style="min-width:230px"></select>
        <input id="tzInput" type="text" placeholder="Or type timezone like Europe/London" />
      </div>
      <div style="margin-top:12px">
        <div class="time" id="clockTime">--:--:--</div>
        <div id="clockDate" style="color:var(--muted)"></div>
      </div>
      <footer>Pick from the list or paste an IANA timezone (e.g. Asia/Colombo, America/New_York).</footer>
    </div>

    <!-- Weather -->
    <div class="card">
      <h1>Weather</h1>
      <div style="display:flex;gap:8px;align-items:center;flex-wrap:wrap">
        <input id="cityInput" type="text" placeholder="Enter city name (e.g. Colombo)" />
        <button id="fetchWeather">Get Weather</button>
      </div>
      <div style="margin-top:12px;display:flex;gap:12px;align-items:center">
        <div class="weather-box">
          <div id="weatherMain" style="font-weight:600">--</div>
          <div id="weatherTemp" style="font-size:22px">-- °C</div>
          <div id="weatherDesc" style="color:var(--muted)"></div>
        </div>
        <div style="flex:1">
          <div style="display:flex;gap:8px;align-items:center">
            <div style="flex:1">
              <div style="font-size:12px;color:var(--muted)">Humidity</div>
              <div id="weatherHumidity">--</div>
            </div>
            <div style="flex:1">
              <div style="font-size:12px;color:var(--muted)">Wind</div>
              <div id="weatherWind">--</div>
            </div>
          </div>
        </div>
      </div>
      <footer>To make this work, open the HTML and set your OpenWeatherMap API key in the code (see instructions below).</footer>
    </div>

    <!-- Music Player -->
    <div class="card">
      <h1>Music Player</h1>
      <div>
        <input id="filePicker" type="file" accept="audio/*" multiple />
        <button id="addUrlBtn">Add audio URL</button>
        <input id="urlInput" type="text" placeholder="Paste direct audio URL and press Add" style="width:100%;margin-top:6px" />
        <div class="playlist" id="playlist"></div>
        <div class="controls">
          <button id="prevBtn">Prev</button>
          <button id="playBtn">Play</button>
          <button id="nextBtn">Next</button>
          <div style="flex:1">
            <input id="progress" type="range" min="0" max="100" value="0" />
          </div>
          <input id="volume" type="range" min="0" max="1" step="0.01" value="0.8" style="width:120px" />
        </div>
        <audio id="player"></audio>
      </div>
      <footer>Drag multiple local audio files into the file selector or paste direct audio URLs.</footer>
    </div>

  </div>

  <script>
    // ================= Calculator =================
    (function(){
      const display = document.getElementById('calcDisplay');
      const keysEl = document.getElementById('keys');
      const buttons = ['7','8','9','/','4','5','6','*','1','2','3','-','0','.','%','+','C','<-','^','='];
      let expr = '';

      function safeEval(s){
        try{
          if(!s) return '';
          s = s.replace(/\^/g,'**');
          if(/[a-zA-Z]/.test(s)) throw 'Invalid';
          return Function('return ('+s+')')();
        }catch(e){return 'Error';}
      }

      function handleKey(k){
        if(k==='C'){expr=''; display.textContent='0'; return;}
        if(k==='<-'){expr=expr.slice(0,-1); display.textContent=expr||'0'; return;}
        if(k==='='){ const r=safeEval(expr); display.textContent=r; expr=String(r); return;}
        expr+=k; display.textContent=expr;
      }

      buttons.forEach(b=>{
        const btn = document.createElement('button'); btn.textContent=b; btn.className='key';
        if(['/','*','-','+','%','^'].includes(b)) btn.classList.add('operator');
        if(b==='=') btn.classList.add('equal');
        btn.addEventListener('click',()=>handleKey(b));
        keysEl.appendChild(btn);
      });

      window.addEventListener('keydown',e=>{
        if(e.key==='Enter') handleKey('=');
        else if(e.key==='Backspace') handleKey('<-');
        else if(e.key.match(/[0-9.+\-*/%^]/)) handleKey(e.key);
      });
    })();

    // ================= World Clock =================
    (function(){
      const tzSelect=document.getElementById('tzSelect');
      const tzInput=document.getElementById('tzInput');
      const clockTime=document.getElementById('clockTime');
      const clockDate=document.getElementById('clockDate');
      const zones=['UTC','Europe/London','Europe/Paris','Asia/Colombo','Asia/Kolkata','Asia/Tokyo','America/New_York'];
      zones.forEach(z=>{const o=document.createElement('option'); o.value=z; o.textContent=z; tzSelect.appendChild(o);});
      function updateClock(){
        let tz=tzInput.value.trim()||tzSelect.value||Intl.DateTimeFormat().resolvedOptions().timeZone;
        try{
          const now=new Date();
          const parts=new Intl.DateTimeFormat('en-GB',{timeZone:tz,hour:'2-digit',minute:'2-digit',second:'2-digit',hour12:false}).formatToParts(now);
          const dateParts=new Intl.DateTimeFormat('en-GB',{timeZone:tz,weekday:'short',year:'numeric',month:'short',day:'2-digit'}).format(now);
          const hh=parts.find(p=>p.type==='hour').value;
          const mm=parts.find(p=>p.type==='minute').value;
          const ss=parts.find(p=>p.type==='second').value;
          clockTime.textContent=`${hh}:${mm}:${ss}`;
          clockDate.textContent=dateParts+' — '+tz;
        }catch(e){clockTime.textContent='Invalid timezone'; clockDate.textContent='Try an IANA timezone';}
      }
      tzSelect.addEventListener('change',()=>{tzInput.value='';updateClock();});
      tzInput.addEventListener('change',updateClock);
      setInterval(updateClock,1000);
      updateClock();
    })();

    // ================= Weather =================
    (function(){
      const btn=document.getElementById('fetchWeather');
      const cityInput=document.getElementById('cityInput');
      const main=document.getElementById('weatherMain');
      const temp=document.getElementById('weatherTemp');
      const desc=document.getElementById('weatherDesc');
      const hum=document.getElementById('weatherHumidity');
      const wind=document.getElementById('weatherWind');
      const OWM_API_KEY='YOUR_OPENWEATHERMAP_API_KEY_HERE';

      async function fetchWeather(city){
        if(!OWM_API_KEY || OWM_API_KEY.includes('YOUR_')){alert('Set your OpenWeatherMap API key in the HTML'); return;}
        try{
          main.textContent='Loading...'; temp.textContent='--'; desc.textContent=''; hum.textContent='--'; wind.textContent='--';
          const res=await fetch(`https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(city)}&units=metric&appid=${OWM_API_KEY}`);
          if(!res.ok) throw new Error('City not found');
          const data=await res.json();
          main.textContent=data.name+', '+(data.sys?.country||'');
          temp.textContent=Math.round(data.main.temp)+' °C';
          desc.textContent=data.weather?.[0]?.description||'';
          hum.textContent=(data.main.humidity||'--')+'%';
          wind.textContent=(data.wind.speed||'--')+' m/s';
        }catch(e){main.textContent='Error'; temp.textContent='--'; desc.textContent=e.message;}
      }
      btn.addEventListener('click',()=>{const city=cityInput.value.trim(); if(city) fetchWeather(city);});
    })();

    // ================= Music Player =================
    (function(){
      const filePicker=document.getElementById('filePicker');
      const playlistEl=document.getElementById('playlist');
      const player=document.getElementById('player');
      const playBtn=document.getElementById('playBtn');
      const prevBtn=document.getElementById('prevBtn');
      const nextBtn=document.getElementById('nextBtn');
      const progress=document.getElementById('progress');
      const volume=document.getElementById('volume');
      const addUrlBtn=document.getElementById('addUrlBtn');
      const urlInput=document.getElementById('urlInput');
      let tracks=[]; let current=-1;

      function renderPlaylist(){
        playlistEl.innerHTML='';
        tracks.forEach((t,i)=>{
          const el=document.createElement('div'); el.className='track';
          el.textContent=t.name;
          el.addEventListener('click',()=>playIndex(i));
          playlistEl.appendChild(el);
        });
      }

      function addTrackFromFile(f){
        const url=URL.createObjectURL(f);
        tracks.push({name:f.name,src:url});
        renderPlaylist();
        if(current===-1) playIndex(0);
      }
      function addTrackFromUrl(src){
        const name=src.split('/').pop().split('?')[0]||src;
        tracks.push({name,src});
        renderPlaylist();
        if(current===-1) playIndex(0);
      }

      filePicker.addEventListener('change',e=>{Array.from(e.target.files).forEach(addTrackFromFile);});
      addUrlBtn.addEventListener('click',()=>{const v=urlInput.value.trim(); if(v){addTrackFromUrl(v); urlInput.value='';}});

      function playIndex(i){
        if(i<0||i>=tracks.length) return;
        current=i;
        player.src=tracks[i].src;
        player.play();
        playBtn.textContent='Pause';
      }
      playBtn.addEventListener('click',()=>{if(player.paused){if(!player.src&&tracks.length)playIndex(0);else player.play();playBtn.textContent='Pause'}else{player.pause();playBtn.textContent='Play'}});
      prevBtn.addEventListener('click',()=>{if(tracks.length) playIndex((current-1+tracks.length)%tracks.length)});
      nextBtn.addEventListener('click',()=>{if(tracks.length) playIndex((current+1)%tracks.length)});
      player.addEventListener('timeupdate',()=>{if(player.duration){progress.max=100; progress.value=Math.floor((player.currentTime/player.duration)*100);}});
      progress.addEventListener('input',()=>{if(player.duration) player.currentTime=(progress.value/100)*player.duration;});
      volume.addEventListener('input',()=>{player.volume=volume.value});
      player.addEventListener('ended',()=>{nextBtn.click()});
    })();
  </script>

  <!-- Save as .html and open in browser. Set your OpenWeatherMap API key in the JS above -->
</body>
</html>
