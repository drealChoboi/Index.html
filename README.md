# Index.html
<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Unofficial Union Build Allocation Checker</title>
  <meta name="description" content="Estimate allocations based on XP using a transparent, configurable formula. Unofficial community tool." />
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    :root{--bg:#0b1020;--card:#12172a;--muted:#8b9bb0;--accent:#6ee7b7}
    html,body{height:100%}
    body{font-family:Inter,system-ui,Segoe UI,Roboto,Arial,Helvetica,sans-serif;background:linear-gradient(180deg,#0b1020 0%,#0b0f1a 100%);color:#e6eef8}
    .glass{background:rgba(18,23,42,.75);backdrop-filter: blur(10px)}
    .mono{font-family: ui-monospace,SFMono-Regular,Menlo,Monaco,Consolas,"Liberation Mono","Courier New",monospace}
    input,textarea,select{background:#0c1224}
    .kbd{border:1px solid #24314d;border-radius:.5rem;padding:.1rem .4rem}
  </style>
</head>
<body class="min-h-full">
  <header class="sticky top-0 z-30 border-b border-white/10 bg-[#0b1020]/80 backdrop-blur">
    <div class="max-w-6xl mx-auto px-4 py-3 flex items-center gap-3">
      <div class="w-2 h-2 rounded-full bg-emerald-400 animate-pulse"></div>
      <h1 class="text-lg sm:text-xl font-semibold">Unofficial Union Build Allocation Checker</h1>
      <span class="text-xs sm:text-sm text-slate-400">by community • transparent • configurable</span>
      <div class="ml-auto flex items-center gap-2 text-xs text-slate-400">
        <a class="underline decoration-dotted" href="#about">About</a>
        <a class="underline decoration-dotted" href="#how">Method</a>
        <a class="underline decoration-dotted" href="#disclaimer">Disclaimer</a>
      </div>
    </div>
  </header>  <main class="max-w-6xl mx-auto p-4 sm:p-6 space-y-6">
    <!-- Controls -->
    <section class="grid grid-cols-1 lg:grid-cols-3 gap-4">
      <!-- Left: Config -->
      <div class="glass rounded-2xl p-4 sm:p-6 space-y-4 border border-white/10">
        <div class="flex items-center justify-between gap-3">
          <h2 class="text-base sm:text-lg font-semibold">Config</h2>
          <button id="btnReset" class="text-xs px-3 py-1 rounded-full border border-white/10 hover:bg-white/5">Reset</button>
        </div><div class="grid grid-cols-2 gap-3">
      <label class="text-sm">Total Pool (tokens)
        <input id="totalPool" type="number" class="mt-1 w-full rounded-xl px-3 py-2 border border-white/10" value="100000" />
      </label>
      <label class="text-sm">Decimals (display)
        <input id="decimals" type="number" class="mt-1 w-full rounded-xl px-3 py-2 border border-white/10" value="2" />
      </label>
      <label class="text-sm">Weight Alpha (0.5–1.0)
        <input id="alpha" type="number" step="0.01" class="mt-1 w-full rounded-xl px-3 py-2 border border-white/10" value="0.7" />
      </label>
      <label class="text-sm">XP Floor (min)
        <input id="xpFloor" type="number" class="mt-1 w-full rounded-xl px-3 py-2 border border-white/10" value="1" />
      </label>
      <label class="text-sm">XP Cap (max)
        <input id="xpCap" type="number" class="mt-1 w-full rounded-xl px-3 py-2 border border-white/10" value="100000" />
      </label>
      <label class="text-sm">Per-User Min
        <input id="minAlloc" type="number" class="mt-1 w-full rounded-xl px-3 py-2 border border-white/10" value="0" />
      </label>
      <label class="text-sm">Per-User Max
        <input id="maxAlloc" type="number" class="mt-1 w-full rounded-xl px-3 py-2 border border-white/10" value="1000" />
      </label>
      <label class="text-sm">Role Boost (%)
        <input id="roleBoost" type="number" class="mt-1 w-full rounded-xl px-3 py-2 border border-white/10" value="20" />
      </label>
    </div>

    <div class="text-xs text-slate-400 leading-relaxed">
      <p>Common approach used by many airdrops: apply <span class="font-semibold">diminishing returns</span> with a power-law (XP<sup>α</sup>), cap extremes, then distribute pool proportionally. Optional role boosts for testers/mods/etc.</p>
    </div>

    <div class="flex gap-2">
      <button id="btnSaveCfg" class="px-3 py-2 rounded-xl bg-emerald-500/20 hover:bg-emerald-500/30 border border-emerald-400/40 text-emerald-200">Save Config</button>
      <button id="btnShareCfg" class="px-3 py-2 rounded-xl border border-white/10 hover:bg-white/5">Share Link</button>
    </div>
  </div>

  <!-- Middle: Data -->
  <div class="glass rounded-2xl p-4 sm:p-6 space-y-3 border border-white/10 lg:col-span-2">
    <div class="flex flex-wrap items-center gap-3">
      <h2 class="text-base sm:text-lg font-semibold">Paste CSV or Upload</h2>
      <input id="file" type="file" accept=".csv,.txt" class="text-xs" />
      <button id="btnSample" class="text-xs px-3 py-1 rounded-full border border-white/10 hover:bg-white/5">Load Sample</button>
      <span class="text-xs text-slate-400">Format: <span class="mono">address,xp,role(optional)</span></span>
    </div>
    <textarea id="csv" rows="8" placeholder="0xabc...,1234,mod\n0xdef...,987,tester" class="w-full rounded-2xl px-3 py-3 border border-white/10 mono"></textarea>
    <div class="flex flex-wrap items-center gap-2">
      <button id="btnCompute" class="px-3 py-2 rounded-xl bg-emerald-500/20 hover:bg-emerald-500/30 border border-emerald-400/40 text-emerald-200">Compute Allocation</button>
      <button id="btnExport" class="px-3 py-2 rounded-xl border border-white/10 hover:bg-white/5">Export CSV</button>
      <span id="summary" class="text-xs text-slate-400"></span>
    </div>

    <div class="overflow-auto border border-white/10 rounded-2xl">
      <table id="table" class="min-w-full text-sm">
        <thead class="bg-white/5">
          <tr>
            <th class="text-left px-3 py-2">#</th>
            <th class="text-left px-3 py-2">Address</th>
            <th class="text-right px-3 py-2">XP</th>
            <th class="text-right px-3 py-2">Weight</th>
            <th class="text-right px-3 py-2">Boost</th>
            <th class="text-right px-3 py-2">Allocation</th>
          </tr>
        </thead>
        <tbody id="tbody"></tbody>
      </table>
    </div>
  </div>
</section>

<!-- Checker -->
<section class="glass rounded-2xl p-4 sm:p-6 space-y-3 border border-white/10">
  <div class="flex flex-wrap items-center gap-3">
    <h2 class="text-base sm:text-lg font-semibold">Check My Address</h2>
    <input id="addr" placeholder="0x... or username" class="flex-1 rounded-xl px-3 py-2 border border-white/10 mono" />
    <button id="btnCheck" class="px-3 py-2 rounded-xl border border-white/10 hover:bg-white/5">Find</button>
    <button id="btnLink" class="px-3 py-2 rounded-xl border border-white/10 hover:bg-white/5">Copy Deep Link</button>
    <span id="checkOut" class="text-sm text-slate-300"></span>
  </div>
  <p class="text-xs text-slate-400">Paste your dataset first. The deep link encodes <span class="mono">address</span> so you can share a one-click checker.</p>
</section>

<!-- About/Method/Disclaimer -->
<section id="about" class="grid grid-cols-1 lg:grid-cols-3 gap-4">
  <div class="glass rounded-2xl p-4 sm:p-6 space-y-2 border border-white/10">
    <h3 class="font-semibold">About</h3>
    <p class="text-sm text-slate-300">This is a community-made, <span class="font-semibold">unofficial</span> estimator for Union Build-style XP distributions. You provide the data; the app shows transparent math.</p>
  </div>
  <div id="how" class="glass rounded-2xl p-4 sm:p-6 space-y-2 border border-white/10">
    <h3 class="font-semibold">Method</h3>
    <ol class="list-decimal list-inside text-sm text-slate-300 space-y-1">
      <li>Clamp XP to <span class="mono">[XP_FLOOR, XP_CAP]</span>.</li>
      <li>Compute weight <span class="mono">w = XP^α</span> with 0.5 ≤ α ≤ 1.0 (diminishing returns).</li>
      <li>Apply role boosts (e.g., tester/mod/ambassador) as a multiplicative factor.</li>
      <li>Distribute total pool proportionally: <span class="mono">alloc = pool * w_i / Σw</span>.</li>
      <li>Clamp per-user allocation to <span class="mono">[MIN, MAX]</span>; re-normalize excess if needed.</li>
    </ol>
  </div>
  <div id="disclaimer" class="glass rounded-2xl p-4 sm:p-6 space-y-2 border border-white/10">
    <h3 class="font-semibold">Disclaimer</h3>
    <p class="text-sm text-slate-300">This tool is not affiliated with any project and provides estimates only. Final allocations, if any, are determined solely by the project team.</p>
  </div>
</section>

  </main>  <footer class="max-w-6xl mx-auto px-4 py-8 text-xs text-slate-500">
    <p>Made with ❤️ by the community. No data is uploaded; everything runs in your browser.</p>
  </footer>  <script>
    // --- Utilities ---
    const $ = (id) => document.getElementById(id)
    const clamp = (v, lo, hi) => Math.max(lo, Math.min(hi, v))
    const fmt = (n, d=2) => Number(n).toLocaleString(undefined,{maximumFractionDigits:d})

    function parseCSV(text){
      const rows = []
      const lines = text.split(/\r?\n/).map(l=>l.trim()).filter(Boolean)
      for(const line of lines){
        const [addressRaw,xpRaw,roleRaw] = line.split(',')
        if(!addressRaw||!xpRaw) continue
        const address = addressRaw.trim()
        const xp = Number(xpRaw.trim())
        const role = (roleRaw||'').trim().toLowerCase()
        if(Number.isFinite(xp)) rows.push({address,xp,role})
      }
      return rows
    }

    function boostForRole(role, basePct){
      if(!role) return 0
      const map = {
        'tester': basePct,
        'mod': basePct,
        'ambassador': basePct,
        'translator': basePct/2,
        'designer': basePct/2,
        'host': basePct/2
      }
      return map[role] ?? 0
    }

    function compute(rows, cfg){
      const {pool, alpha, floor, cap, minAlloc, maxAlloc, roleBoostPct} = cfg
      // 1) Clamp XP + weights
      let items = rows.map(r=>{
        const xpC = clamp(r.xp, floor, cap)
        const wBase = Math.pow(xpC, alpha)
        const boostPct = boostForRole(r.role, roleBoostPct)
        const w = wBase * (1 + boostPct/100)
        return {...r, xpC, wBase, boostPct, w}
      })
      const sumW = items.reduce((a,b)=>a+b.w,0)
      if(sumW === 0) return {items:[], sumW:0, sumAlloc:0}

      // 2) Initial proportional allocation
      items = items.map((it)=> ({...it, alloc: pool * (it.w / sumW)}))

      // 3) Clamp per-user
      let excessHi = 0, deficitLo = 0
      items = items.map(it=>{
        let a = it.alloc
        if(a > maxAlloc){ excessHi += (a - maxAlloc); a = maxAlloc }
        if(a < minAlloc){ deficitLo += (minAlloc - a); a = minAlloc }
        return {...it, alloc:a}
      })

      // 4) Re-normalize: redistribute excess to those below max, subtract to fill deficits
      function redistribute(delta, dir){
        // dir: +1 to add extra to under-max; -1 to take from above-min
        const eligible = items.filter(it=> dir>0 ? it.alloc < maxAlloc : it.alloc > minAlloc)
        const sumWElig = eligible.reduce((a,b)=>a+b.w,0)
        if(sumWElig <= 0) return
        for(const it of eligible){
          const share = delta * (it.w / sumWElig)
          it.alloc = clamp(it.alloc + dir*share, minAlloc, maxAlloc)
        }
      }
      if(excessHi>0) redistribute(excessHi, +1)
      if(deficitLo>0) redistribute(deficitLo, -1)

      const sumAlloc = items.reduce((a,b)=>a+b.alloc,0)
      return {items, sumW, sumAlloc}
    }

    function renderTable(items, decimals){
      const tbody = $('tbody')
      tbody.innerHTML = ''
      items.forEach((it,i)=>{
        const tr = document.createElement('tr')
        tr.className = i%2? 'bg-white/0' : 'bg-white/0'
        tr.innerHTML = `
          <td class="px-3 py-2 text-slate-400">${i+1}</td>
          <td class="px-3 py-2 mono">${it.address}${it.role?` <span class='text-xs text-slate-400'>(${it.role})</span>`:''}</td>
          <td class="px-3 py-2 text-right">${fmt(it.xp,0)}</td>
          <td class="px-3 py-2 text-right">${fmt(it.w,4)}</td>
          <td class="px-3 py-2 text-right">${it.boostPct? ('+'+it.boostPct+'%'):'-'}</td>
          <td class="px-3 py-2 text-right font-semibold">${fmt(it.alloc,decimals)}</td>
        `
        tbody.appendChild(tr)
      })
    }

    function exportCSV(items){
      const header = 'address,xp,role,weight,boostPct,allocation'\
      + '\n'
      const body = items.map(it=>[
        it.address,it.xp,it.role||'',it.w.toFixed(6),it.boostPct,(+it.alloc.toFixed(8))
      ].join(',')).join('\n')
      const blob = new Blob([header+body],{type:'text/csv'})
      const url = URL.createObjectURL(blob)
      const a = document.createElement('a')
      a.href = url
      a.download = 'allocations.csv'
      a.click()
      URL.revokeObjectURL(url)
    }

    // --- State & Events ---
    function getCfg(){
      return {
        pool: Number($('totalPool').value)||0,
        alpha: clamp(Number($('alpha').value)||0.7, 0.5, 1.0),
        floor: Number($('xpFloor').value)||0,
        cap: Number($('xpCap').value)||1e9,
        minAlloc: Number($('minAlloc').value)||0,
        maxAlloc: Number($('maxAlloc').value)||1e12,
        roleBoostPct: Number($('roleBoost').value)||0,
        decimals: clamp(Number($('decimals').value)||2,0,8)
      }
    }

    function setCfg(cfg){
      $('totalPool').value = cfg.pool
      $('alpha').value = cfg.alpha
      $('xpFloor').value = cfg.floor
      $('xpCap').value = cfg.cap
      $('minAlloc').value = cfg.minAlloc
      $('maxAlloc').value = cfg.maxAlloc
      $('roleBoost').value = cfg.roleBoostPct
      $('decimals').value = cfg.decimals
    }

    function saveCfg(){
      const cfg = getCfg()
      localStorage.setItem('ub_cfg', JSON.stringify(cfg))
    }
    function loadCfg(){
      const raw = localStorage.getItem('ub_cfg')
      if(raw){ try{ setCfg(JSON.parse(raw)) }catch(e){} }
    }

    function shareLink(){
      const cfg = getCfg()
      const params = new URLSearchParams()
      params.set('cfg', btoa(unescape(encodeURIComponent(JSON.stringify(cfg)))))
      const url = location.origin + location.pathname + '#' + params.toString()
      navigator.clipboard.writeText(url)
      alert('Shareable link copied to clipboard. Paste to share.')
    }

    function importCfgFromURL(){
      if(location.hash.includes('cfg=')){
        const qs = new URLSearchParams(location.hash.slice(1))
        const cfgStr = decodeURIComponent(escape(atob(qs.get('cfg'))))
        try{ setCfg(JSON.parse(cfgStr)) }catch(e){}
      }
      if(location.hash.includes('address=')){
        const qs = new URLSearchParams(location.hash.slice(1))
        $('addr').value = qs.get('address')
      }
    }

    function copyDeepLinkForAddress(){
      const addr = $('addr').value.trim()
      const base = location.origin + location.pathname
      const qs = new URLSearchParams(location.hash.slice(1))
      qs.set('address', addr)
      const url = base + '#' + qs.toString()
      navigator.clipboard.writeText(url)
      alert('Deep link copied to clipboard.')
    }

    function checkAddress(items){
      const q = $('addr').value.trim().toLowerCase()
      const hit = items.find(it=> it.address.toLowerCase()===q || it.address.toLowerCase().includes(q))
      if(!hit){ $('checkOut').textContent = 'No match in current dataset.'; return }
      $('checkOut').innerHTML = `Found: <span class='mono'>${hit.address}</span> → <span class='font-semibold'>${fmt(hit.alloc, getCfg().decimals)}</span>`
    }

    // Sample dataset (fake for demo)
    const SAMPLE = `0x1111111111111111111111111111111111111111,12000,tester\n0x2222222222222222222222222222222222222222,5400,\n0x3333333333333333333333333333333333333333,2500,mod\n0x4444444444444444444444444444444444444444,300,\n0x5555555555555555555555555555555555555555,50,ambassador`

    let lastItems = []

    $('btnSample').onclick = ()=> { $('csv').value = SAMPLE }

    $('file').onchange = (e)=>{
      const file = e.target.files?.[0]
      if(!file) return
      const reader = new FileReader()
      reader.onload = ()=>{ $('csv').value = reader.result }
      reader.readAsText(file)
    }

    $('btnCompute').onclick = ()=>{
      const rows = parseCSV($('csv').value)
      const cfg = getCfg()
      const {items, sumW, sumAlloc} = compute(rows, cfg)
      lastItems = items
      renderTable(items, cfg.decimals)
      $('summary').textContent = `${items.length} users • Σw=${fmt(sumW,4)} • distributed=${fmt(sumAlloc,cfg.decimals)}`
    }

    $('btnExport').onclick = ()=>{
      if(!lastItems.length) return alert('Compute first.')
      exportCSV(lastItems)
    }

    $('btnSaveCfg').onclick = saveCfg
    $('btnShareCfg').onclick = shareLink
    $('btnReset').onclick = ()=>{ localStorage.removeItem('ub_cfg'); setCfg({pool:100000,alpha:0.7,floor:1,cap:100000,minAlloc:0,maxAlloc:1000,roleBoostPct:20,decimals:2}) }

    $('btnCheck').onclick = ()=>{ if(!lastItems.length) return alert('Compute first.'); checkAddress(lastItems) }
    $('btnLink').onclick = copyDeepLinkForAddress

    // Init
    loadCfg(); importCfgFromURL()
  </script></body>
</html>
