<!DOCTYPE html>
<html lang="it">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Cartomanti Online</title>
<style>
  :root{--green:#16a34a;--amber:#f59e0b;--red:#dc2626;--bg:#f6f7fb;--card:#fff;--txt:#0f172a;--muted:#64748b;--br:#e5e7eb}
  *{box-sizing:border-box} body{margin:0;background:var(--bg);font-family:system-ui,-apple-system,Segoe UI,Roboto,Arial,sans-serif;color:var(--txt)}
  .wrap{max-width:980px;margin:24px auto;padding:0 16px}
  h1{margin:0 0 10px;font-size:24px}
  .legend{font-size:12px;color:var(--muted);margin-bottom:12px}
  .toolbar{display:flex;gap:8px;align-items:center;margin-bottom:12px}
  input,select,button{padding:10px;border:1px solid var(--br);border-radius:10px;background:#fff}
  button{cursor:pointer}
  .stats{display:grid;grid-template-columns:repeat(3,1fr);gap:8px;margin-bottom:10px}
  .stat{background:var(--card);border:1px solid var(--br);border-radius:12px;padding:12px;display:flex;gap:10px;align-items:center}
  .dot{width:10px;height:10px;border-radius:50%}
  .grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(260px,1fr));gap:12px}
  .card{background:var(--card);border:1px solid var(--br);border-radius:14px;padding:12px;box-shadow:0 2px 6px rgba(0,0,0,.05)}
  .row{display:flex;gap:12px;align-items:center}
  .avatar{width:60px;height:60px;border-radius:999px;object-fit:cover;border:2px solid #fff;box-shadow:0 1px 3px rgba(0,0,0,.12)}
  .title{font-weight:700}
  .muted{color:var(--muted);font-size:12px}
  .badge{font-size:12px;padding:4px 10px;border-radius:999px;color:#fff}
  .call{margin-top:10px;width:100%;padding:10px;border:none;border-radius:10px;background:#22c55e;color:#fff;font-weight:700}
</style>
</head>
<body>
<div class="wrap">
  <h1>🔮 Cartomanti Online</h1>
  <div class="legend">💚 Disponibile · 🟧 In pausa · ❤️ In chiamata</div>

  <div class="toolbar">
    <input id="search" placeholder="Cerca nome o codice">
    <select id="spec"><option value="">Tutte le specialità</option></select>
  </div>

  <section class="stats">
    <div class="stat"><span class="dot" style="background:var(--green)"></span><div><div class="muted">Disponibili</div><div id="s-on" style="font:700 20px/1 system-ui">0</div></div></div>
    <div class="stat"><span class="dot" style="background:var(--amber)"></span><div><div class="muted">In pausa</div><div id="s-pa" style="font:700 20px/1 system-ui">0</div></div></div>
    <div class="stat"><span class="dot" style="background:var(--red)"></span><div><div class="muted">In chiamata</div><div id="s-call" style="font:700 20px/1 system-ui">0</div></div></div>
  </section>

  <section id="list" class="grid"></section>
</div>

<script>
/* === 1) METTI QUI I TUOI OPERATORI REALI ===
   - photo: puoi usare "img/CODICE.jpg" se hai caricato le foto nel repo
   - status: "online" | "pause" | "oncall"
*/
const OPERATORI = [
  // ESEMPI: sostituisci con i tuoi
  { name:"MORENA SENSITIVA", code:"330", status:"online", queue:"Sensitiva", photo:"img/330.jpg", phone:"330" },
  { name:"SARAH MEDIUM",     code:"342", status:"oncall", queue:"Medium",    photo:"img/342.jpg", phone:"342" },
  // Aggiungi altre righe qui...
];

// colori/etichette
const LABEL = { online:"Disponibile", pause:"In pausa", oncall:"In chiamata" };
const COLOR = { online:"var(--green)", pause:"var(--amber)", oncall:"var(--red)" };
const PLACEHOLDER = "https://via.placeholder.com/120x120?text=%F0%9F%94%AE";

const el = {
  list: document.getElementById('list'),
  sOn: document.getElementById('s-on'),
  sPa: document.getElementById('s-pa'),
  sCall: document.getElementById('s-call'),
  search: document.getElementById('search'),
  spec: document.getElementById('spec'),
};

function esc(s){return String(s||"").replace(/[&<>"]/g,c=>({"&":"&amp;","<":"&lt;",">":"&gt;","\"":"&quot;"}[c]))}

function render(rows){
  const q = (el.search.value||"").toLowerCase();
  const spec = el.spec.value||"";
  let data = rows.slice();
  if (q) data = data.filter(r => (r.name||"").toLowerCase().includes(q) || String(r.code).includes(q));
  if (spec) data = data.filter(r => (r.queue||"")===spec);

  // stats
  el.sOn.textContent   = data.filter(r=>r.status==="online").length;
  el.sPa.textContent   = data.filter(r=>r.status==="pause").length;
  el.sCall.textContent = data.filter(r=>r.status==="oncall").length;

  // filtro specialità
  const setQ = new Set(rows.map(r=>r.queue).filter(Boolean));
  const prev = el.spec.value;
  el.spec.innerHTML = '<option value="">Tutte le specialità</option>' + Array.from(setQ).map(x=>`<option${x===prev?' selected':''}>${esc(x)}</option>`).join('');

  // cards
  el.list.innerHTML = data.map(r=>{
    const img = r.photo || `img/${r.code}.jpg`;
    return `
      <div class="card">
        <div class="row">
          <img class="avatar" src="${esc(img)}" onerror="this.src='${PLACEHOLDER}'" alt="">
          <div style="flex:1;min-width:0">
            <div class="title">${esc(r.name)}</div>
            <div class="muted">Codice: ${esc(r.code)}${r.queue?' · '+esc(r.queue):''}</div>
          </div>
          <span class="badge" style="background:${COLOR[r.status]||'#94a3b8'}">${LABEL[r.status]||esc(r.status)}</span>
        </div>
        <a class="call" href="tel:${esc(r.phone||r.code)}">📞 Chiama ora</a>
      </div>`;
  }).join('');
}

el.search.addEventListener('input',()=>render(OPERATORI));
el.spec.addEventListener('change',()=>render(OPERATORI));
render(OPERATORI);
</script>
</body>
</html>e 
