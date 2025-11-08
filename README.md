<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>GoPaste - App</title>
  <style>
    :root{--bg:#0f1720;--card:#0b1220;--muted:#9aa4b2;--accent:#06f;--mono:#dbe7ff}
    html,body{height:100%}
    body{margin:0;font-family:Inter,ui-sans-serif,system-ui,-apple-system,"Segoe UI",Roboto, "Helvetica Neue",Arial;background:linear-gradient(180deg,#071021 0%,#071827 100%);color:#e6eef8;display:flex;align-items:flex-start;justify-content:center;padding:36px}
    .container{width:980px;max-width:95%}
    header{display:flex;align-items:center;gap:14px;margin-bottom:18px}
    .logo{font-weight:700;font-size:20px;color:var(--accent)}
    .sub{color:var(--muted);font-size:13px}
    .card{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));border-radius:10px;padding:18px;box-shadow:0 6px 20px rgba(2,6,23,0.6)}
    .row{display:flex;gap:12px}
    label{display:block;font-size:13px;color:var(--muted);margin-bottom:6px}
    input[type=text]{width:100%;padding:10px;border-radius:6px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:var(--mono)}
    textarea{width:100%;min-height:260px;padding:12px;border-radius:8px;border:1px dashed rgba(255,255,255,0.04);background:transparent;color:var(--mono);resize:vertical;font-family:ui-monospace, SFMono-Regular, Menlo, Monaco, "Roboto Mono", "Courier New", monospace;font-size:13px}
    .controls{display:flex;gap:8px;align-items:center;margin-top:10px}
    button{background:var(--accent);color:#021025;border:0;padding:9px 12px;border-radius:8px;cursor:pointer;font-weight:600}
    button.ghost{background:transparent;border:1px solid rgba(255,255,255,0.04);color:var(--muted);font-weight:500}
    .meta{display:flex;gap:10px;align-items:center;color:var(--muted);font-size:13px;margin-top:10px}
    .paste-view{white-space:pre-wrap;word-break:break-word;background:#06111b;padding:16px;border-radius:8px;font-family:ui-monospace, SFMono-Regular, Menlo, Monaco, "Roboto Mono", monospace;font-size:13px;color:var(--mono);line-height:1.45;border:1px solid rgba(255,255,255,0.02)}
    .toolbar{display:flex;gap:8px;align-items:center;margin-top:8px}
    a.linklike{color:var(--accent);text-decoration:none;font-weight:600}
    .note{color:var(--muted);font-size:13px;margin-top:8px}
    footer{color:var(--muted);font-size:12px;margin-top:14px;text-align:center}
    @media (max-width:640px){header{flex-direction:column;align-items:flex-start;gap:6px}}
  </style>
</head>
<body>
  <div class="container">
    <header>
      <div class="logo">GoPaste</div>
      <div class="sub">A Pastebin Inspired App</div>
    </header>

    <main class="card" id="app">
      <!-- Create form -->
      <section id="createSection">
        <label for="title">Title (optional)</label>
        <input id="title" type="text" placeholder="My snippet or note" />

        <label for="paste">Paste text</label>
        <textarea id="paste" placeholder="Paste your code or text here..."></textarea>

        <div class="controls">
          <button id="createBtn">Create Paste</button>
          <button id="clearBtn" class="ghost" type="button">Clear</button>
          <div style="flex:1"></div>
          <div class="meta">Made By Team Magnets</div>
        </div>

        <div id="shareBox" style="display:none;margin-top:10px">
          <label>Share link</label>
          <input id="shareUrl" type="text" readonly />
          <div class="note">Open the link to view the paste. Use the Copy or Raw buttons there.</div>
        </div>
      </section>

      <!-- View paste -->
      <section id="viewSection" style="display:none">
        <div style="display:flex;justify-content:space-between;align-items:center">
          <div>
            <div id="viewTitle" style="font-weight:700;font-size:16px"></div>
            <div id="viewMeta" class="meta"></div>
          </div>
          <div class="toolbar">
            <button id="copyBtn">Copy</button>
            <button id="rawBtn" class="ghost">Raw</button>
            <button id="backBtn" class="ghost">New Paste</button>
          </div>
        </div>

        <div id="pasteContent" class="paste-view" style="margin-top:12px"></div>
      </section>
    </main>

    <footer>GoPaste — Made By Team Magnets </footer>
  </div>

  <script>
    // Simple client-side paste store using localStorage
    const qs = new URLSearchParams(location.search);
    const idParam = qs.get('id');
    const app = document.getElementById('app');
    const createSection = document.getElementById('createSection');
    const viewSection = document.getElementById('viewSection');
    const pasteInput = document.getElementById('paste');
    const titleInput = document.getElementById('title');
    const createBtn = document.getElementById('createBtn');
    const clearBtn = document.getElementById('clearBtn');
    const shareBox = document.getElementById('shareBox');
    const shareUrl = document.getElementById('shareUrl');
    const viewTitle = document.getElementById('viewTitle');
    const viewMeta = document.getElementById('viewMeta');
    const pasteContent = document.getElementById('pasteContent');
    const copyBtn = document.getElementById('copyBtn');
    const rawBtn = document.getElementById('rawBtn');
    const backBtn = document.getElementById('backBtn');

    function genId(){
      return Date.now().toString(36) + '-' + Math.random().toString(36).slice(2,9);
    }

    function storePaste(id, data){
      localStorage.setItem('gopaste:' + id, JSON.stringify(data));
    }
    function loadPaste(id){
      const raw = localStorage.getItem('gopaste:' + id);
      return raw ? JSON.parse(raw) : null;
    }

    createBtn.addEventListener('click', ()=>{
      const text = pasteInput.value.trim();
      if(!text){ alert('Paste is empty'); pasteInput.focus(); return; }
      const id = genId();
      const data = { title: titleInput.value.trim() || 'Untitled', body: text, created: Date.now() };
      storePaste(id, data);
      const url = location.origin + location.pathname + '?id=' + encodeURIComponent(id);
      shareBox.style.display = 'block';
      shareUrl.value = url;
      shareUrl.select();
      try { navigator.clipboard.writeText(url); } catch(e){}
    });

    clearBtn.addEventListener('click', ()=>{ titleInput.value=''; pasteInput.value=''; pasteInput.focus(); shareBox.style.display='none'; });

    function showView(id){
      const data = loadPaste(id);
      if(!data){ alert('Paste not found'); history.replaceState(null,'',location.pathname); return; }
      createSection.style.display = 'none';
      viewSection.style.display = 'block';
      viewTitle.textContent = data.title;
      viewMeta.textContent = 'ID: ' + id + ' · ' + new Date(data.created).toLocaleString();
      pasteContent.textContent = data.body;
      copyBtn.focus();
      // set actions
      copyBtn.onclick = async ()=>{
        try{
          await navigator.clipboard.writeText(data.body);
          copyBtn.textContent = 'Copied';
          setTimeout(()=>copyBtn.textContent='Copy',1200);
        }catch(e){
          alert('Copy failed — please select and copy manually.');
        }
      };
      rawBtn.onclick = ()=>{
        // create a blob with type text/plain and open it in a new tab to emulate "raw"
        const blob = new Blob([data.body], { type: 'text/plain;charset=utf-8' });
        const url = URL.createObjectURL(blob);
        window.open(url, '_blank');
        setTimeout(()=>URL.revokeObjectURL(url), 30000);
      };
      backBtn.onclick = ()=>{
        history.replaceState(null,'',location.pathname);
        viewSection.style.display='none';
        createSection.style.display='block';
      };
    }

    // If page opened with an id param, show view
    if(idParam){
      showView(idParam);
    }

    // Bonus: allow opening share URL by pasting an id in the input
    shareUrl.addEventListener('click', ()=>shareUrl.select());
  </script>
</body>
</html>
