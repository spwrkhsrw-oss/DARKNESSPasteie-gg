<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>DarKNessPastie</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: Consolas, monospace; }
    html,body { height: 100%; }
    body { background: #0b0e14; color: #c9d1d9; display:flex; flex-direction:column; min-height:100vh; }

    header {
      background: linear-gradient(90deg,#02040a,#071026);
      padding: 1rem 1.2rem;
      display:flex;
      justify-content:space-between;
      align-items:center;
      border-bottom:1px solid #1f2937;
    }
    header h1 { color:#38bdf8; font-size:1.25rem; letter-spacing:0.6px; }
    header span { color:#64748b; font-size:0.85rem; }

    main { max-width:1100px; margin:1.2rem auto; padding:0 1rem; width:100%; flex:1 0 auto; }

    .toolbar { display:flex; gap:0.6rem; align-items:center; margin-bottom:0.8rem; }
    .toolbar select, .toolbar button, .inline-btn {
      background:#020617; color:#e5e7eb; border:1px solid #1f2937;
      padding:0.5rem 0.8rem; border-radius:8px; cursor:pointer; font-family:inherit;
      transition: transform 0.15s ease, box-shadow 0.15s ease;
    }
    .toolbar button.primary { background:#38bdf8; color:#020617; font-weight:700; border:none; }
    .btn-ghost { background:transparent; border:1px solid #334155; color:#cbd5e1; padding:0.35rem 0.6rem; border-radius:6px; }

    textarea {
      width:100%; min-height:380px; background:#020617; color:#e5e7eb;
      border:1px solid #1f2937; border-radius:10px; padding:1rem; resize:vertical;
      font-size:0.95rem; line-height:1.6; white-space:pre-wrap; overflow-wrap:break-word;
      transition: border-color 0.2s ease;
    }
    textarea:focus { border-color: #38bdf8; box-shadow: 0 0 8px rgba(56,189,248,0.12); }

    .info { margin-top:0.8rem; display:flex; justify-content:space-between; color:#94a3b8; font-size:0.85rem; }
    .panel { margin-top:0.8rem; padding:0.8rem; border-radius:10px; background:linear-gradient(#020617,#060718); border:1px solid #111827; color:#c9d1d9; }
    .links { display:flex; gap:0.5rem; flex-wrap:wrap; align-items:center; margin-top:0.5rem; }
    .small { font-size:0.85rem; color:#94a3b8; }

    #viewer { display:none; }
    #viewer-content { width:100%; min-height:380px; background:#020617; color:#e5e7eb; border:1px solid #1f2937; border-radius:10px; padding:1.5rem; font-size:1rem; line-height:1.7; white-space:pre-wrap; overflow-wrap:break-word; box-shadow: 0 4px 20px rgba(0,0,0,0.2); }

    .copy-btn { padding:0.45rem 0.7rem; border-radius:6px; border:none; cursor:pointer; }
    .copy-success { background:#10b981; color:#021; }

    footer { margin-top:1.6rem; padding:1rem; text-align:center; color:#475569; border-top:1px solid #1f2937; font-size:0.85rem; }

    @media (max-width:720px) { .toolbar { flex-direction:column; align-items:stretch; } .actions { margin-left:0; } }
  </style>
</head>
<body>

<header>
  <div>
    <h1>DarKNessPastie</h1>
    <div style="font-size:0.8rem;color:#64748b">Dark Paste • Secure • Fast</div>
  </div>
  <div class="small">Share links: clean /p/&lt;id&gt; (local) or /s/&lt;encoded&gt; (anyone)</div>
</header>

<main id="app">
  <div class="toolbar">
    <select id="lang">
      <option value="text">Plain Text</option>
      <option value="html">HTML</option>
      <option value="javascript">JavaScript</option>
      <option value="python">Python</option>
      <option value="css">CSS</option>
      <option value="json">JSON</option>
      <option value="bash">Bash</option>
    </select>

    <div style="display:flex; gap:0.5rem; align-items:center;">
      <button id="createBtn" class="primary">Create Paste</button>
      <button id="shareBtn" class="btn-ghost" title="Create a shareable link (content is embedded in the URL)">Share (anyone)</button>
      <button id="updateBtn" class="btn-ghost" title="Update existing paste (if loaded locally)">Update</button>
      <button id="deleteBtn" class="btn-ghost" title="Delete paste from local storage">Delete</button>
    </div>

    <div class="actions" style="margin-left:auto; display:flex; gap:0.5rem;">
      <button id="copyBtn" class="btn-ghost">Copy</button>
      <button id="pasteBtn" class="btn-ghost" title="Replace editor with clipboard content">Paste ⌘V</button>
    </div>
  </div>

  <textarea id="paste" placeholder="Paste your code or text here..."></textarea>

  <div id="viewer">
    <div id="viewer-content"></div>
  </div>

  <div class="panel" id="metaPanel" style="display:none;">
    <div><strong id="pasteIdLabel"></strong> <span class="small" id="langLabel"></span></div>
    <div class="links" id="linksArea"></div>
    <div class="small" id="createdAtLabel"></div>
  </div>

  <div class="panel" id="listPanel" style="margin-top:0.8rem;">
    <div style="display:flex; justify-content:space-between; align-items:center;">
      <div><strong>Saved local pastes</strong> <span class="small"> (stored in your browser)</span></div>
      <div><button id="clearAll" class="btn-ghost">Clear all</button></div>
    </div>
    <div id="list" style="margin-top:0.6rem; display:flex; gap:0.6rem; flex-wrap:wrap;"></div>
  </div>

  <div class="info">
    <span id="info-status">Ready</span>
    <span class="small">Public • No expiration • Use <code>/s/&lt;encoded&gt;</code> to share with anyone</span>
  </div>
</main>

<footer>
  <p>© 2026 DarKNessPastie — Dark themed paste service</p>
</footer>

<script>
(function(){
  // --- Helpers ---
  const $ = id => document.getElementById(id);
  const nowISO = () => new Date().toISOString();
  const generateId = () => (Date.now().toString(36) + Math.random().toString(36).slice(2,8)).slice(0,12);

  // URL-safe Base64 encode/decode for Unicode strings
  function encodeForUrl(str){
    try{
      const b64 = btoa(unescape(encodeURIComponent(str)));
      return b64.replace(/\+/g,'-').replace(/\//g,'_').replace(/=+$/,'');
    }catch(e){
      // fallback — may fail on very large input
      return encodeURIComponent(str);
    }
  }
  function decodeFromUrl(s){
    try{
      s = s.replace(/-/g,'+').replace(/_/g,'/');
      // add padding
      while(s.length % 4) s += '=';
      return decodeURIComponent(escape(atob(s)));
    }catch(e){
      try { return decodeURIComponent(s); } catch(e2){ return null; }
    }
  }

  function getBaseForPath(){
    const p = location.pathname;
    const pathPrefix = p.endsWith('/') ? p : p + '/';
    return location.origin + pathPrefix;
  }

  function getRoot(){
    // base url up to the file (used for constructing /s/ and /p/ links). If hosted on GitHub Pages, should work with 404 fallback.
    const parts = location.pathname.split('/').filter(Boolean);
    if(parts.length === 0) return location.origin + '/';
    // remove any last segment that looks like an id when in view mode — prefer using origin + path up to the directory
    const last = parts[parts.length-1];
    if(last.length <= 24 && /^[A-Za-z0-9_-]+$/.test(last)){
      parts.pop();
    }
    return location.origin + '/' + parts.join('/') + (parts.length ? '/' : '');
  }

  // Storage helpers
  function savePasteObj(obj){
    try{ localStorage.setItem('paste_' + obj.id, JSON.stringify(obj)); return true; }
    catch(e){ console.error(e); alert('Failed to save (localStorage may be full)'); return false; }
  }
  function getPasteObj(id){
    try{ const raw = localStorage.getItem('paste_' + id); return raw ? JSON.parse(raw) : null; }
    catch(e){ return null; }
  }
  function removePaste(id){ localStorage.removeItem('paste_' + id); }

  // UI refs
  const pasteEl = $('paste');
  const createBtn = $('createBtn');
  const shareBtn = $('shareBtn');
  const updateBtn = $('updateBtn');
  const deleteBtn = $('deleteBtn');
  const copyBtn = $('copyBtn');
  const pasteBtn = $('pasteBtn');
  const langSel = $('lang');
  const metaPanel = $('metaPanel');
  const pasteIdLabel = $('pasteIdLabel');
  const langLabel = $('langLabel');
  const linksArea = $('linksArea');
  const createdAtLabel = $('createdAtLabel');
  const status = $('info-status');
  const list = $('list');
  const clearAll = $('clearAll');
  const viewer = $('viewer');
  const viewerContent = $('viewer-content');

  function setStatus(t){ status.textContent = t; }

  function refreshSavedList(){
    list.innerHTML = '';
    const keys = Object.keys(localStorage).filter(k=>k.startsWith('paste_')).sort().reverse();
    if(!keys.length){ list.innerHTML = '<div class="small">No saved pastes in this browser.</div>'; return; }
    keys.forEach(k=>{
      try{
        const obj = JSON.parse(localStorage.getItem(k));
        const btn = document.createElement('button');
        btn.className = 'inline-btn'; btn.textContent = obj.id + ' · ' + (obj.lang||'text');
        btn.title = (obj.content||'').slice(0,200);
        btn.onclick = ()=>{ window.location.href = getRoot() + 'p/' + obj.id; };
        list.appendChild(btn);
      }catch(e){console.warn('bad paste',k);}    });
  }

  function renderMetaFor(obj, mode){
    if(!obj){ metaPanel.style.display='none'; return; }
    metaPanel.style.display='block';
    pasteIdLabel.textContent = 'Paste: ' + obj.id;
    langLabel.textContent = ' • ' + (obj.lang||'text');
    createdAtLabel.textContent = 'Created: ' + new Date(obj.createdAt).toLocaleString();
    linksArea.innerHTML = '';

    const root = getRoot();
    const localView = root + 'p/' + obj.id;
    const localRaw = root + 'r/' + obj.id;
    const shareEncoded = root + 's/' + encodeForUrl(obj.content);

    addLink('Open view', localView);
    addLink('Open raw', localRaw);
    addButton('Copy view', ()=>copyToClipboard(localView));
    addButton('Copy raw', ()=>copyToClipboard(localRaw));
    addButton('Copy share link', ()=>copyToClipboard(shareEncoded));
    if(mode === 'view') addButton('Edit paste', ()=>{ window.location.href = root + '#'+obj.id; });
    addButton('Copy content', ()=>copyToClipboard(obj.content));
    addButton('Delete', ()=>{ if(confirm('Delete paste '+obj.id+'?')){ removePaste(obj.id); setStatus('Deleted '+obj.id); refreshSavedList(); metaPanel.style.display='none'; } });

    function addLink(text, href){ const a = document.createElement('a'); a.href = href; a.className='linkish'; a.textContent = text; a.target='_blank'; linksArea.appendChild(a); }
    function addButton(text, fn){ const b = document.createElement('button'); b.className='btn-ghost'; b.textContent = text; b.onclick = fn; linksArea.appendChild(b); }
  }

  async function copyToClipboard(text){ try{ await navigator.clipboard.writeText(text); setStatus('Copied to clipboard'); }catch(e){ prompt('Copy', text); } }

  // Create paste locally
  createBtn.onclick = ()=>{
    const content = pasteEl.value || '';
    if(!content.trim()){ alert('Paste cannot be empty'); return; }
    const id = generateId();
    const obj = { id, content, lang: langSel.value, createdAt: nowISO() };
    if(savePasteObj(obj)){
      setStatus('Saved locally: '+id);
      refreshSavedList();
      // navigate to clean local url
      window.history.pushState({}, '', getRoot() + 'p/' + id);
      renderMetaFor(obj, 'edit');
    }
  };

  // Share link (embed content in URL) — works for anyone
  shareBtn.onclick = ()=>{
    const content = pasteEl.value || '';
    if(!content.trim()){ alert('Paste cannot be empty'); return; }
    const encoded = encodeForUrl(content);
    const link = getRoot() + 's/' + encoded;
    copyToClipboard(link);
    setStatus('Share link created and copied');
  };

  // Update current local paste (if loaded via /p/<id> or hash)
  updateBtn.onclick = ()=>{
    const id = detectIdFromLocation()?.id || (location.hash||'').replace('#','');
    if(!id) return alert('No paste id to update. Open a local paste first.');
    const existing = getPasteObj(id);
    if(!existing) return alert('Local paste not found: '+id);
    existing.content = pasteEl.value; existing.lang = langSel.value; existing.updatedAt = nowISO();
    if(savePasteObj(existing)){ setStatus('Updated: '+id); renderMetaFor(existing,'edit'); refreshSavedList(); }
  };

  // Delete
  deleteBtn.onclick = ()=>{
    const id = detectIdFromLocation()?.id || (location.hash||'').replace('#','');
    if(!id) return alert('No paste id to delete.');
    if(!confirm('Delete paste '+id+' from local storage?')) return;
    removePaste(id); setStatus('Deleted '+id); refreshSavedList(); metaPanel.style.display='none'; window.history.pushState({},'', getRoot());
  };

  copyBtn.onclick = ()=>copyToClipboard(pasteEl.value || '');
  pasteBtn.onclick = async ()=>{ try{ pasteEl.value = await navigator.clipboard.readText(); setStatus('Pasted from clipboard'); }catch(e){ alert('Unable to read clipboard'); } };

  clearAll.onclick = ()=>{ if(!confirm('Clear ALL local pastes?')) return; Object.keys(localStorage).filter(k=>k.startsWith('paste_')).forEach(k=>localStorage.removeItem(k)); refreshSavedList(); setStatus('Cleared all'); metaPanel.style.display='none'; window.history.pushState({},'', getRoot()); };

  // Detect id or encoded content from path or hash
  function detectIdFromLocation(){
    const segs = location.pathname.split('/').filter(Boolean);
    // /s/<encoded>
    const sIndex = segs.indexOf('s');
    if(sIndex !== -1 && segs.length > sIndex+1) return { type:'share', encoded: segs[sIndex+1], id: null };
    // /p/<id>
    const pIndex = segs.indexOf('p');
    if(pIndex !== -1 && segs.length > pIndex+1) return { type:'local', id: segs[pIndex+1] };
    // /r/<id>
    const rIndex = segs.indexOf('r');
    if(rIndex !== -1 && segs.length > rIndex+1) return { type:'raw', id: segs[rIndex+1] };
    // fallback: last path segment might be an id
    if(segs.length===1 && /^[A-Za-z0-9_-]{4,24}$/.test(segs[0])) return { type:'local', id:segs[0] };
    // hash-based (edit)
    const h = (location.hash||'').replace('#',''); if(h) return { type:'hash', id:h };
    return null;
  }

  function showViewOnly(text, opts={raw:false, id:null}){
    // Hide editor
    $('.toolbar')?.replaceWith?.($('.toolbar') || document.querySelector('.toolbar'));
    document.querySelector('.toolbar').style.display='none';
    pasteEl.style.display='none';
    $('listPanel').style.display='none';
    viewer.style.display='block';
    viewerContent.textContent = text;
    // meta panel with links
    const fakeObj = { id: opts.id || '(share)', content: text, lang: langSel.value, createdAt: nowISO() };
    renderMetaFor(fakeObj, 'view');
    metaPanel.style.display='block';
  }

  function showRaw(text){
    // Replace page with raw pre (minimized chrome)
    document.documentElement.style.height='100%'; document.body.style.margin='0'; document.body.style.background='#030409';
    document.body.innerHTML = '';
    const pre = document.createElement('pre');
    pre.style.whiteSpace='pre-wrap'; pre.style.wordBreak='break-word'; pre.style.padding='1rem'; pre.style.margin='0'; pre.style.fontFamily='monospace'; pre.style.fontSize='14px'; pre.style.color='#e6eef6'; pre.style.background='#030409'; pre.textContent = text;
    document.body.appendChild(pre);
  }

  // Router: run on load and popstate
  function router(){
    const detected = detectIdFromLocation();
    if(!detected){ // editor mode
      document.querySelector('.toolbar').style.display='flex'; pasteEl.style.display='block'; $('listPanel').style.display='block'; viewer.style.display='none'; metaPanel.style.display='none'; document.title='DarKNessPastie'; setStatus('Ready'); refreshSavedList(); return;
    }
    if(detected.type==='share'){
      const decoded = decodeFromUrl(detected.encoded);
      if(decoded===null){ document.body.innerHTML = '<div style="padding:2rem;color:#f87171">Invalid share link</div>'; return; }
      showViewOnly(decoded, { id: '(shared)' }); document.title = 'Shared paste — DarKNessPastie'; setStatus('Viewing shared paste'); return;
    }
    if(detected.type==='local'){
      const obj = getPasteObj(detected.id);
      if(!obj){ document.body.innerHTML = '<div style="padding:2rem;color:#f87171">Local paste not found: '+detected.id+'</div>'; return; }
      showViewOnly(obj.content, { id: obj.id }); document.title = obj.id + ' — DarKNessPastie'; setStatus('Viewing: '+obj.id); return;
    }
    if(detected.type==='raw'){
      const obj = getPasteObj(detected.id);
      if(!obj){ document.body.innerHTML = '<div style="padding:2rem;color:#f87171">Raw paste not found: '+detected.id+'</div>'; return; }
      showRaw(obj.content); return;
    }
    if(detected.type==='hash'){
      // Edit mode using hash id
      const obj = getPasteObj(detected.id);
      if(!obj){ setStatus('No local paste found for id: '+detected.id); return; }
      pasteEl.value = obj.content; langSel.value = obj.lang || 'text'; renderMetaFor(obj,'edit'); setStatus('Editing: '+detected.id); refreshSavedList(); return;
    }
  }

  window.addEventListener('popstate', router);
  window.addEventListener('DOMContentLoaded', ()=>{ refreshSavedList(); router(); });
  window.addEventListener('keydown', (ev)=>{ if((ev.ctrlKey||ev.metaKey) && ev.key==='Enter'){ ev.preventDefault(); createBtn.click(); } if((ev.ctrlKey||ev.metaKey) && (ev.key==='s' || ev.key==='S')){ ev.preventDefault(); updateBtn.click(); } });

})();
</script>

</body>
</html>
