<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>السبورة الذكية — مدرب التنويم الذاتي</title>
  <style>
    :root{
      --primary:#0d47a1;
      --accent:#ffca28;
      --bg:#f5f7fb;
      --card:#ffffff;
      --text:#222;
    }
    body{margin:0;font-family:Tahoma, Arial, sans-serif;background:var(--bg);color:var(--text);direction:rtl}
    header{background:linear-gradient(90deg,var(--primary),#12569b);color:#fff;padding:18px;text-align:center;font-size:20px;font-weight:700}
    .container{max-width:1000px;margin:18px auto;padding:16px}
    .top-row{display:flex;gap:12px;align-items:center;justify-content:space-between;flex-wrap:wrap}
    .card{background:var(--card);border-radius:12px;padding:12px;box-shadow:0 6px 18px rgba(13,71,161,0.08);width:48%}
    .wide{width:100%}
    .title{font-size:18px;margin-bottom:8px;color:var(--primary)}
    canvas{width:100%;height:420px;border-radius:8px;background:#fff;display:block;cursor:crosshair}
    .controls{display:flex;gap:8px;flex-wrap:wrap;align-items:center}
    button{background:var(--primary);color:#fff;border:none;padding:8px 12px;border-radius:8px;cursor:pointer}
    button.secondary{background:#eee;color:#111}
    .session{display:flex;align-items:center;gap:8px;justify-content:space-between;padding:8px 0;border-bottom:1px solid #f0f0f0}
    .session .left{flex:1}
    input[type="file"]{display:none}
    label.filebtn{background:#eee;border-radius:8px;padding:6px 8px;cursor:pointer;color:#111}
    .small{font-size:13px;color:#666}
    footer{margin-top:18px;text-align:center;color:#666;font-size:13px}
    @media(max-width:800px){.card{width:100%}}
    .audio-player{display:flex;gap:8px;align-items:center}
    progress{width:160px}
    .session-title{font-weight:600}
  </style>
</head>
<body>
  <header>🧠 السبورة الذكية — مدرب التنويم الذاتي</header>

  <div class="container">

    <!-- اللوحة -->
    <div class="card wide">
      <div class="title">🖊️ السبورة</div>
      <canvas id="board" width="1200" height="700"></canvas>
      <div style="margin-top:10px" class="controls">
        <button onclick="clearBoard()">مسح</button>
        <button onclick="saveImage()">حفظ الصورة</button>
        <button onclick="downloadAllAsZip()" class="secondary">تحميل كل الملفات (صورة + صوت)</button>
        <div style="margin-left:auto;">
          <label class="filebtn" for="bgMusicFile">رفع موسيقى خلفية</label>
          <input id="bgMusicFile" type="file" accept="audio/*">
          <button id="toggleBg" onclick="toggleBackgroundMusic()">تشغيل الموسيقى</button>
        </div>
      </div>
      <div class="small" style="margin-top:8px">يمكن حفظ صورة السبورة أو تشغيل/إيقاف موسيقى خلفية أثناء الجلسات.</div>
    </div>

    <!-- الجلسات -->
    <div class="card">
      <div class="title">🎧 جلسات التنويم</div>

      <!-- جلسة Template: يتكرر خمس مرات مع ids مختلفة -->
      <div id="sessionsContainer"></div>

      <div class="small" style="margin-top:10px">يمكنك رفع ملف صوتي لكل جلسة من جهازك بالضغط على زر (رفع)، أو وضع رابط خارجي في إعدادات لاحقًا.</div>
    </div>

  </div>

  <footer>حقوقك محفوظة — مدرب التنويم الذاتي · النسخة الشخصية</footer>

<script>
/* ======= بيانات الجلسات (قابلة للتعديل): إذا أردت ملفات جاهزة ضَع روابط هنا =======
   مثال: sessionData[0].defaultUrl = 'https://domain.com/audio/s1.mp3'
   حالياً تركتها فارغة لتستخدم رفع الملفات المحلية أو روابطك. */
const sessionData = [
  { id:'s1', title:'مقدمة التنويم العلمي', defaultUrl:'' },
  { id:'s2', title:'الاسترخاء العميق وتصفية الذهن', defaultUrl:'' },
  { id:'s3', title:'برمجة العقل الباطن بالإيحاء', defaultUrl:'' },
  { id:'s4', title:'السيطرة على الوعي والتركيز', defaultUrl:'' },
  { id:'s5', title:'الخروج الآمن من التنويم', defaultUrl:'' }
];

/* ======= واجهة الجلسات ديناميكيًا ====== */
const sessionsContainer = document.getElementById('sessionsContainer');

function createSessionRow(s){
  const div = document.createElement('div');
  div.className = 'session';
  div.id = 'row-'+s.id;

  const left = document.createElement('div');
  left.className = 'left';

  const title = document.createElement('div');
  title.className = 'session-title';
  title.textContent = s.title;

  // audio element
  const audio = document.createElement('audio');
  audio.id = 'audio-'+s.id;
  audio.controls = false;
  audio.preload = 'metadata';

  // try load saved url from localStorage
  const stored = localStorage.getItem('audioUrl_'+s.id);
  if(stored) audio.src = stored;
  else if(s.defaultUrl) audio.src = s.defaultUrl;

  // small controls area
  const ap = document.createElement('div');
  ap.className = 'audio-player';

  const playBtn = document.createElement('button');
  playBtn.textContent = 'تشغيل';
  playBtn.onclick = () => {
    if(audio.src){
      audio.play();
      playBtn.textContent = 'إيقاف';
    } else alert('لم تقم برفع ملف لهذه الجلسة بعد.');
  };
  audio.addEventListener('pause', ()=> playBtn.textContent='تشغيل');
  audio.addEventListener('ended', ()=> playBtn.textContent='تشغيل');
  audio.addEventListener('play', ()=> playBtn.textContent='إيقاف');

  playBtn.onclick = () => {
    if(!audio.src){ alert('لم يتم اختيار ملف. اضغط رفــع ثم اختر ملف MP3.'); return; }
    if(audio.paused) audio.play(); else audio.pause();
  };

  // progress
  const prog = document.createElement('progress');
  prog.max = 100;
  prog.value = 0;
  audio.addEventListener('timeupdate', ()=>{
    if(audio.duration) prog.value = (audio.currentTime/audio.duration)*100;
  });

  // file input (hidden)
  const fileInput = document.createElement('input');
  fileInput.type='file';
  fileInput.accept='audio/*';
  fileInput.style.display='none';
  fileInput.onchange = (ev)=>{
    const f = ev.target.files[0];
    if(!f) return;
    const url = URL.createObjectURL(f);
    audio.src = url;
    // store blob in sessionStorage? cannot persist blob across sessions.
    // We'll store object URL in localStorage only for the same session. Better: upload file to repo/CDN and set link.
    localStorage.setItem('audioName_'+s.id, f.name);
    // We cannot store blobs permanently in localStorage; but we store file data via FileReader as base64 (not recommended for big files).
    // Instead store a flag that local file was selected in this session:
    localStorage.setItem('audioUrl_'+s.id, url);
    alert('تم تحميل الملف محليًا وسيعمل في هذه النافذة. لحفظه دائمًا ضع رابط الملف على الإنترنت أو ارفعه في مستودعك.');
  };

  // label button to open file selector
  const label = document.createElement('label');
  label.className = 'filebtn';
  label.textContent = 'رفع ملف';
  label.onclick = () => fileInput.click();

  // download current audio link
  const dlbtn = document.createElement('button');
  dlbtn.className='secondary';
  dlbtn.textContent='تحميل';
  dlbtn.onclick = ()=>{
    if(!audio.src){ alert('لا يوجد ملف لتحميل'); return; }
    const a = document.createElement('a');
    a.href = audio.src;
    a.download = (localStorage.getItem('audioName_'+s.id) || s.id+'.mp3');
    a.click();
  };

  // set external url textbox
  const urlBox = document.createElement('input');
  urlBox.type='text';
  urlBox.placeholder='ضع رابط mp3 خارجي هنا ثم اضغط حفظ الرابط';
  urlBox.style.width='60%';
  urlBox.style.margin='6px 6px 0 0';
  urlBox.value = localStorage.getItem('audioUrlExternal_'+s.id) || '';
  const saveUrlBtn = document.createElement('button');
  saveUrlBtn.textContent='حفظ الرابط';
  saveUrlBtn.onclick = ()=>{
    const val = urlBox.value.trim();
    if(!val){ alert('أدخل رابطًا صالحًا'); return; }
    audio.src = val;
    localStorage.setItem('audioUrlExternal_'+s.id, val);
    localStorage.setItem('audioUrl_'+s.id, val);
    alert('تم حفظ الرابط كمصدر دائم لهذه الجلسة في هذا المتصفح.');
  };

  left.appendChild(title);
  left.appendChild(audio);
  left.appendChild(ap);
  ap.appendChild(playBtn);
  ap.appendChild(prog);
  ap.appendChild(label);
  ap.appendChild(dlbtn);
  left.appendChild(document.createElement('br'));
  left.appendChild(urlBox);
  left.appendChild(saveUrlBtn);
  left.appendChild(fileInput);

  div.appendChild(left);
  sessionsContainer.appendChild(div);
}

sessionData.forEach(s => createSessionRow(s));

/* ======= Canvas (السبورة) ====== */
const canvas = document.getElementById('board');
const ctx = canvas.getContext('2d');
let drawing=false;
let lastX=0,lastY=0;

function resizeCanvas(){
  // maintain drawn content? for simplicity keep fixed size as set in html attributes
}
canvas.addEventListener('mousedown', (e)=>{ drawing=true; [lastX,lastY]=[e.offsetX,e.offsetY]; });
canvas.addEventListener('mouseup', ()=>{ drawing=false; ctx.beginPath(); });
canvas.addEventListener('mouseout', ()=>{ drawing=false; ctx.beginPath(); });
canvas.addEventListener('mousemove', (e)=>{ if(!drawing) return; ctx.lineWidth=2; ctx.lineJoin='round'; ctx.lineCap='round'; ctx.strokeStyle='#111'; ctx.beginPath(); ctx.moveTo(lastX,lastY); ctx.lineTo(e.offsetX,e.offsetY); ctx.stroke(); [lastX,lastY]=[e.offsetX,e.offsetY]; });

function clearBoard(){ ctx.clearRect(0,0,canvas.width,canvas.height); }

/* حفظ صورة */
function saveImage(){
  const dataUrl = canvas.toDataURL('image/png');
  const a = document.createElement('a');
  a.href = dataUrl;
  a.download = 'smartboard_'+(new Date().toISOString().slice(0,19).replace(/[:T]/g,'-'))+'.png';
  a.click();
}

/* ======= موسيقى خلفية بسيطة ====== */
let bgAudio = new Audio();
bgAudio.loop = true;
let bgPlaying = false;

document.getElementById('bgMusicFile').addEventListener('change', (e)=>{
  const f = e.target.files[0];
  if(!f) return;
  const url = URL.createObjectURL(f);
  bgAudio.src = url;
  localStorage.setItem('bgMusicName', f.name);
  alert('تم تحميل موسيقى الخلفية محليًا. اضغط تشغيل الموسيقى لتشغيلها.');
});

function toggleBackgroundMusic(){
  if(!bgAudio.src){ alert('لم تحدد موسيقى خلفية. استخدم زر رفع موسيقى.'); return; }
  if(bgPlaying){ bgAudio.pause(); bgPlaying=false; document.getElementById('toggleBg').textContent='تشغيل الموسيقى'; }
  else { bgAudio.play(); bgPlaying=true; document.getElementById('toggleBg').textContent='إيقاف الموسيقى'; }
}

/* ======= تحميل الصورة + ملفات الصوت كـ zip (مبسط) ======
   ملاحظة: لتحضير zip في المتصفح بالكامل يلزم مكتبة JS إضافية (JSZip).
   هنا نستخدم طريقة بسيطة: نطمئن المستخدم ونوفر تحميل للصورة فقط.
*/
function downloadAllAsZip(){
  // بسيط: نحمّل صورة السبورة فقط (لأن الملفات الصوتية محلية أو روابط).
  saveImage();
  alert('تم حفظ صورة السبورة. لتحزيم الصوتيات (zip) أستطيع إضافتها لاحقًا عند طلبك.');
}

/* ======= عند تحميل الصفحة: نعيد ضبط أزرار من localStorage (لو وُجد) ====== */
window.addEventListener('load', ()=>{
  sessionData.forEach(s=>{
    const url = localStorage.getItem('audioUrl_'+s.id) || sessionData.find(x=>x.id===s.id).defaultUrl || '';
    if(url){
      const audio = document.getElementById('audio-'+s.id);
      audio.src = url;
    }
  });

  // إذا أردت، يمكنك وضع روابط مباشرة هنا (مثال):
  // document.getElementById('audio-s1').src = 'https://dl.dropboxusercontent.com/s/xxxxx/s1.mp3';
});
</script>
</body>
</html>
