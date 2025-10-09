---
layout: page
title: Representing the Search Space
description: Exploration of how information about the world can be encoded in a form that is efficient to search over and interpretable.
img: /assets/img/representation.png
importance: 1
category: themes
related_publications: false
date: 2025-10-08
---

Every form of intelligence can be viewed as **search over a representational space**. The representation defines what is _expressible_ and what can be _generalized_; the _search algorithm_ defines _how_ we move within that space.

A _representation_ R of a search space determines the structural and geometric properties of the domain in which search or learning occurs. More concretely, it defines three interrelated aspects: **(1) expressivity**, **(2) topology or geometry**, and **(3) inductive bias**.

First, **expressivity** specifies what hypotheses, behaviors, or functions are even _representable_ within the space. Formally, it defines the hypothesis class $$\mathcal{H}_R = \{ h \mid h \text{ can be encoded in } R \}$$. In program synthesis, for example, the expressivity is bounded by the grammar of the domain-specific language (DSL); in reinforcement learning (RL), by the set of permissible actions and transition dynamics; and in deep learning, by the architecture and parameterization that determine the family of realizable functions. Thus, the representation defines the _support_ of the search, i.e. what can exist within it.

Second, the representation induces a **topology or geometry**, determining how candidates relate to one another. This topology provides the notion of adjacency or similarity that constrains how search or learning can proceed. In symbolic spaces, proximity may correspond to syntactic edit distance or logical derivability; in latent embedding spaces, to Euclidean or cosine distance; and in action spaces, to similarity of induced state transitions. The topology therefore determines what kinds of _local moves_, _gradients_, or _perturbations_ are meaningful within the search.

Third, the representation embodies an **inductive bias**—a set of assumptions that govern how information generalizes across the topology. Continuous representations typically induce smoothness priors that support interpolation-based generalization, whereas symbolic representations rely on compositional structure to enable combinatorial generalization. In this sense, generalization is not an emergent mystery but a direct consequence of the structural priors embedded in R:

$$

\text{Generalization} \approx \text{Exploitation of structure in } R.
$$

Together, these three dimensions, expressivity, topology, and inductive bias, define the _shape_ of the search space. Search algorithms operate _within_ this shape, learning adapts _traversal_ within it, and representation learning reshapes the topology itself to make useful regions more reachable and semantically coherent.

While representations differ widely across domains, they can be systematically compared along several axes that describe their structural and semantic properties (see Figure below).

<!-- ===== Representation Compass (semantic placement + blended colors) ===== -->
<style>
  .repc-wrap { max-width: 900px; margin: 2rem auto; }
  .repc-figure {
    position: relative; aspect-ratio: 1/1; width: 100%;
    border-radius: 18px; overflow: hidden;
    box-shadow: 0 6px 24px rgba(0,0,0,.06);
    background: #fff;
  }
  .repc-bg {
    position: absolute; inset: 0;
    background:
      conic-gradient(from -90deg,
        #E53935 0deg,   #9C27B0 45deg,  #1E88E5 90deg,  #26C6DA 135deg,
        #2E7D32 180deg, #C0CA33 225deg, #FDD835 270deg, #FB8C00 315deg, #E53935 360deg
      );
    opacity: .18; pointer-events: none;
  }
  .repc-svg { position: absolute; inset: 0; }
  .repc-axis  { stroke: rgba(0,0,0,.14); stroke-width: 1; }
  .repc-octo  { fill: none; stroke: rgba(0,0,0,.28); stroke-width: 1.25; }
  .repc-label { font: 600 13px/1.15 system-ui, -apple-system, Segoe UI, Roboto, sans-serif; fill: rgba(0,0,0,.85); text-anchor: middle; }
  .repc-node  { stroke: #fff; stroke-width: 2; }
  .repc-node-label { font: 600 13px/1.2 system-ui, -apple-system, Segoe UI, Roboto, sans-serif; fill: rgba(0,0,0,.92); }
  .repc-card {
    position:absolute; padding:.6rem .8rem; min-width:220px; max-width:360px;
    background: rgba(255,255,255,.98); border:1px solid rgba(0,0,0,.08); border-radius:10px;
    box-shadow:0 10px 30px rgba(0,0,0,.18); display:none; z-index:10; pointer-events:auto;
  }
  .repc-card h4{ margin:.1rem 0 .4rem 0; font-size:15px; }
  .repc-card ul{ margin:0; padding-left:1rem; }
  .repc-card a { color:#0d47a1; text-decoration:none; }
  .repc-card a:hover { text-decoration:underline; }
</style>

<div class="repc-wrap">
  <div class="repc-figure" id="repc">
    <div class="repc-bg" id="repcBg"></div>
    <svg class="repc-svg" viewBox="0 0 1000 1000" preserveAspectRatio="none" id="repcSvg"></svg>
    <div class="repc-card" id="repcCard"></div>
  </div>
</div>

<script>
(function(){
  const DATA_URL = "{{ '/assets/json/representation_nodes.json' | relative_url }}";
  const svg  = document.getElementById('repcSvg');
  const bg   = document.getElementById('repcBg');
  const FIG  = document.getElementById('repc');
  const CARD = document.getElementById('repcCard');

  const cx = 500, cy = 500, R = 430;  // drawing radius
  const toRad = d => (d*Math.PI)/180, toDeg = r => r*180/Math.PI;

  // Poles (clockwise from top)
  const POLES = [
    {name:'Abstract',    ang:-90,  color:'#E53935'},
    {name:'Ungrounded',  ang:-45,  color:'#9C27B0'},
    {name:'Implicit',    ang:  0,  color:'#1E88E5'},
    {name:'Continuous',  ang: 45,  color:'#26C6DA'},
    {name:'Concrete',    ang: 90,  color:'#2E7D32'},
    {name:'Grounded',    ang:135,  color:'#C0CA33'},
    {name:'Explicit',    ang:180,  color:'#FDD835'},
    {name:'Discrete',    ang:-135, color:'#FB8C00'}
  ];
  const poleIndex = Object.fromEntries(POLES.map((p,i)=>[p.name, i]));
  const poleByName = n => POLES[poleIndex[n]];

  // ===== Helpers =====
  function el(name, attrs){ const e=document.createElementNS('http://www.w3.org/2000/svg', name); for(const k in attrs) e.setAttribute(k, attrs[k]); return e; }
  function text(x,y,str,cls,anchor='middle'){ const t=el('text',{x,y,'text-anchor':anchor}); if(cls)t.setAttribute('class',cls); t.textContent=str; return t; }
  function hash32(str){ let h=2166136261>>>0; for(let i=0;i<str.length;i++){ h^=str.charCodeAt(i); h=Math.imul(h,16777619);} h^=h>>>13; h=Math.imul(h,0x5bd1e995); h^=h>>>15; return h>>>0; }
  function rng(seed){ let s=(seed||1)>>>0; return ()=>{ s^=s<<13; s^=s>>>17; s^=s<<5; s>>>=0; return (s>>>0)/0x100000000; }; }
  function clamp(n,min,max){ return Math.max(min, Math.min(max, n)); }
  function lerp(a,b,t){ return a+(b-a)*t; }
  function hexToRgb(h){ const m=/^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(h); return m?{r:parseInt(m[1],16),g:parseInt(m[2],16),b:parseInt(m[3],16)}:null; }
  function rgbToHex({r,g,b}){ const h=n=>n.toString(16).padStart(2,'0'); return '#'+h(r)+h(g)+h(b); }
  function blend(c1,c2,t){ const a=hexToRgb(c1), b=hexToRgb(c2); return rgbToHex({r:Math.round(lerp(a.r,b.r,t)), g:Math.round(lerp(a.g,b.g,t)), b:Math.round(lerp(a.b,b.b,t))}); }

  // ===== Layout control (angle + ring) =====
  // Angle: prefer explicit node.pos; else pole; else infer from scores.
  function wrapAngle(a){ while(a<=-180) a+=360; while(a>180) a-=360; return a; }
  function interpAngle(a1,a2,t){ // shortest-arc interpolation
    let d = wrapAngle(a2 - a1);
    return a1 + d*t;
  }
  function getAngle(n){
    if (n.pos && typeof n.pos.angleDeg === 'number') return n.pos.angleDeg;
    if (n.pos && Array.isArray(n.pos.between) && n.pos.between.length===2){
      const A = poleByName(n.pos.between[0])?.ang;
      const B = poleByName(n.pos.between[1])?.ang;
      const t = clamp(n.pos.t ?? 0.5, 0, 1);
      if (typeof A==='number' && typeof B==='number') return interpAngle(A, B, t);
    }
    if (n.pole && poleIndex[n.pole]!=null) return POLES[poleIndex[n.pole]].ang;

    // legacy: infer from scores if present
    const a=n.abstract??0.5, u=n.ungrounded??0.5, i=n.implicit??0.5, c=n.continuous??0.5;
    const scores = {Abstract:a, Ungrounded:u, Implicit:i, Continuous:c, Concrete:1-a, Grounded:1-u, Explicit:1-i, Discrete:1-c};
    let best='Implicit', v=-1; for(const k in scores){ if(scores[k]>v){ best=k; v=scores[k]; } }
    return POLES[poleIndex[best]].ang;
  }

  // Rings: neat, meaning-free circles; small jitter to avoid overlaps
  const RINGS = { center: R*0.08, inner: R*0.38, mid: R*0.58, outer: R*0.78 };
  function getRingR(n){
    const name = n.pos?.ring || 'mid';
    return RINGS[name] ?? RINGS.mid;
  }

  function nodeColor(n){
    if (n.color) return n.color;
    if (n.pos && Array.isArray(n.pos.between) && n.pos.between.length===2){
      const A = poleByName(n.pos.between[0]), B = poleByName(n.pos.between[1]);
      if (A && B) return blend(A.color, B.color, clamp(n.pos.t ?? 0.5,0,1));
    }
    if (n.pole && poleIndex[n.pole]!=null) return POLES[poleIndex[n.pole]].color;
    return '#555';
  }

  // ===== Build octagon once; use same points for stroke and clip =====
  const octAngles = [-90,-45,0,45,90,135,180,-135];
  function octoPoints(r){ return octAngles.map(a=>[cx + r*Math.cos(toRad(a)), cy + r*Math.sin(toRad(a))]); }
  const oct = octoPoints(R);
  svg.appendChild(el('polygon', {points: oct.map(p=>p.join(',')).join(' '), class:'repc-octo'}));
  bg.style.clipPath = 'polygon(' + oct.map(([x,y]) => `${(x/1000*100).toFixed(2)}% ${(y/1000*100).toFixed(2)}%`).join(',') + ')';

  // Axes + labels
  POLES.forEach(p=>{
    const end = { x: cx + R*Math.cos(toRad(p.ang)), y: cy + R*Math.sin(toRad(p.ang)) };
    svg.appendChild(el('line', {x1:cx, y1:cy, x2:end.x, y2:end.y, class:'repc-axis'}));
    const lab = { x: cx + (R+28)*Math.cos(toRad(p.ang)), y: cy + (R+28)*Math.sin(toRad(p.ang)) };
    svg.appendChild(text(lab.x, lab.y, p.name, 'repc-label'));
  });

  // Tooltip (sticky)
  let hideT=null, locked=false;
  function showCard(node, px, py, color){
    CARD.innerHTML = `
      <h4><span style="display:inline-block;width:.8em;height:.8em;border-radius:50%;background:${color};margin-right:.4em"></span>${node.label}</h4>
      ${Array.isArray(node.items)&&node.items.length ? `<ul>${node.items.map(d=>`<li><a target="_blank" rel="noopener" href="${d.url}">${d.title}</a></li>`).join('')}</ul>` : `<div>No links yet.</div>`}
    `;
    const rect = FIG.getBoundingClientRect();
    let left=(px/1000)*rect.width+16, top=(py/1000)*rect.height-16;
    CARD.style.left=left+'px'; CARD.style.top=top+'px'; CARD.style.display='block';
    const cRect = CARD.getBoundingClientRect();
    CARD.style.left = clamp(left, 8, rect.width - cRect.width - 8)+'px';
    CARD.style.top  = clamp(top,  8, rect.height - cRect.height - 8)+'px';
  }
  function hideCard(){ CARD.style.display='none'; }
  function hideCardDeferred(ms=180){ clearTimeout(hideT); hideT=setTimeout(()=>{ if(!locked) hideCard(); }, ms); }
  CARD.addEventListener('mouseenter', ()=> clearTimeout(hideT));
  CARD.addEventListener('mouseleave', ()=> { if(!locked) hideCardDeferred(); });
  CARD.addEventListener('click', e=> e.stopPropagation());
  document.addEventListener('keydown', e=>{ if(e.key==='Escape'){ locked=false; hideCard(); }});

  // ===== Load + render =====
  fetch(DATA_URL, {cache:'no-store'})
    .then(r=>r.json())
    .then(nodes=>{
      nodes.forEach(n=>{
        const ang = getAngle(n);
        const R0  = getRingR(n);
        const seed = hash32(String(n.label||'')), rand = rng(seed);
        const rJ = R*0.02*(rand()*2-1);            // tiny radial jitter (no meaning)
        const aJ = 3*(rand()*2-1);                 // tiny angular jitter (no meaning)
        const rad = R0 + rJ;
        const th  = toRad(ang + aJ);
        const x = cx + rad*Math.cos(th), y = cy + rad*Math.sin(th);
        const fill = nodeColor(n);

        const dot = el('circle', {cx:x, cy:y, r:9, class:'repc-node', fill});
        dot.style.cursor='pointer';
        dot.addEventListener('mouseenter', ()=>{ if(!locked) showCard(n, x, y, fill); });
        dot.addEventListener('mouseleave', (e)=>{ if(!CARD.contains(e.relatedTarget) && !locked) hideCardDeferred(); });
        dot.addEventListener('click', (e)=>{ e.stopPropagation(); locked=true; showCard(n, x, y, fill); });

        svg.appendChild(dot);
        svg.appendChild(text(x, y-16, n.label, 'repc-node-label'));
      });

      FIG.addEventListener('click', (e)=>{ if(e.target.tagName!=='circle' && !CARD.contains(e.target)){ locked=false; hideCard(); } });
      FIG.addEventListener('touchstart', (e)=>{ if(e.target.tagName!=='circle' && !CARD.contains(e.target)){ locked=false; hideCard(); } }, {passive:true});
    })
    .catch(err=>{
      console.error(err);
      CARD.style.display='block'; CARD.style.left='12px'; CARD.style.top='12px';
      CARD.innerHTML = `<strong>Could not load nodes</strong>`;
    });
})();
</script>
<!-- ===== /Representation Compass ===== -->

These axes, **concreteness**, **explicitness**, **grounding**, and **discreteness**, are often correlated but conceptually distinct. _Concreteness_ refers to the **content** of a representation: how directly its elements correspond to observable phenomena or sensorimotor data. _Explicitness_, by contrast, refers to **form**: whether the internal structure of the representation is accessible, interpretable, and directly manipulable. These two are related but not equivalent. For instance, a neural world model in reinforcement learning can be _concrete but implicit_: it encodes raw pixel-level dynamics without any symbolic decomposition. Conversely, a symbolic algebra system is _abstract but explicit_: it manipulates purely conceptual objects such as variables or group elements through rule-based transformations. _Grounding_ captures the degree of coupling between representational states and external referents, an action space in RL is tightly grounded in the environment’s causal structure, whereas a language model’s latent space is only indirectly grounded through statistical co-occurrence in text. Finally, _discreteness_ characterizes the topology of the representational domain: symbolic systems inhabit discrete combinatorial spaces, while neural or embedding-based systems operate over continuous manifolds that support differentiable optimization. Considering these axes jointly allows one to situate any representational system—symbolic, neural, or hybrid—within a shared analytical space, clarifying how different approaches constrain what can be represented, related, and generalized.

### Big challenges

The central challenge in representation design is to construct spaces that are simultaneously efficient to search, stable to learn, and interpretable to analyze. Each of these desiderata pulls against the others. Representations that enable rapid optimization, such as dense continuous embeddings, often obscure their internal semantics; representations that make structure explicit, such as symbolic programs, are interpretable but hard to explore due to combinatorial explosion. Bridging these regimes without loss of tractability or fidelity remains an open problem.

A first difficulty concerns structure and interpretability. Continuous representations support smooth similarity and differentiable optimization but provide no explicit account of what each dimension encodes. Small perturbations in latent space can produce large, incoherent changes in behavior, and there is no systematic way to impose known constraints, such as invariances, causal relations, or compositional rules, without full retraining. Symbolic representations, on the other hand, make structure explicit but require discrete search procedures whose complexity grows exponentially with representational depth. Hybrid, neuro-symbolic systems attempt to combine these advantages but introduce new failure modes at the interface: it is unclear how to propagate learning signals across discrete boundaries, how to assign responsibility for errors between neural and symbolic components, and how to maintain consistent objectives during joint optimization.

A second challenge concerns grounding. Many representational systems are learned from proxy signals, text, static images, or logged trajectories, without access to the causal or interactive structure of the environments they are meant to model. Such representations are statistically adequate yet operationally shallow: they predict correlations but cannot support controlled intervention, planning, or tool use. Acquiring grounded representations requires interactive or interventional data, which are costly to obtain and difficult to evaluate safely and reproducibly. Recent multimodal models, such as DINO or CLIP, partially address this through cross-modal grounding, but such statistical alignment does not yet yield representations that support causal prediction or physical reasoning.

A third difficulty lies in diagnosis and evaluation. Existing metrics, such as downstream task accuracy, provide weak evidence about the internal adequacy of a representation (i.e., whether a representation captures the correct factors of variation). They do not capture properties such as disentanglement, robustness to perturbation, or the capacity to manipulate representational components predictably. As a result, systems may perform well empirically while relying on brittle shortcuts or entangled factors that fail under distribution shift.

Ultimately, the choice of representation defines the structure of the search space: which regions are reachable, how boundaries between valid and invalid solutions are formed, and whether rare, high-value solutions can be discovered efficiently. The closer the representation reflects the underlying structure of the problem, the more likely search is to uncover novel yet valid outcomes rather than repeatedly rediscovering known ones. Designing such well-aligned representations remains a central open challenge for scalable and interpretable intelligence.

The landscape illustrated below, spanning infeasible, nearly-feasible, conventional, suboptimal, and novel regions, can be understood as a projection of the representational space into the domain of possible artifacts. In this view, the representation determines how easily search can move from failure to success, and whether truly creative solutions remain within reach.

<div class="artifact-diagram">
  <svg viewBox="0 0 960 680" role="img" aria-labelledby="cap">
    <title id="cap">Valid (green, upper-right) vs Invalid (red, lower-left) with BL→TR frontier.</title>
    <defs>
      <style>
        .frame{fill:white;stroke:#111;stroke-width:2}
        .h1{font:700 26px/1 Inter, system-ui, -apple-system, Segoe UI, Roboto, sans-serif;fill:#111}
        .p {font:400 15px/1.35 Inter, system-ui, -apple-system, Segoe UI, Roboto, sans-serif;fill:#333}
        .soft{fill:rgba(0,0,0,.06);stroke:rgba(0,0,0,.12)}
        .note{font:600 14px/1 Inter, system-ui, -apple-system, Segoe UI, Roboto, sans-serif; fill:#0b70d1}
      </style>
      <pattern id="nearBandDots" patternUnits="userSpaceOnUse" width="6" height="6">
        <circle cx="3" cy="3" r="1" fill="#6b7280" />
      </pattern>
    </defs>

    <!-- frame -->
    <rect x="40" y="40" width="880" height="600" rx="8" class="frame"/>

    <!-- BACKGROUNDS aligned to BL→TR frontier -->
    <!-- Invalid (lower-left triangle) -->
    <polygon points="40,40 40,640 920,640" fill="#fde8e8"/>
    <!-- Valid (upper-right triangle) -->
    <polygon points="40,40 920,40 920,640" fill="#e7f8ef"/>

    <!-- labels -->
    <!-- Valid (top-right) -->
    <text x="740" y="110" class="h1">Valid</text>
    <text x="740" y="135" class="p">Meets problem constraints</text>

    <!-- Invalid (bottom-left) -->
    <text x="70"  y="610" class="h1">Invalid</text>
    <text x="70"  y="635" class="p">Violates constraints or fails tests</text>

    <!-- Dashed diagonal (TL -> BR, negative slope) -->
    <line x1="60" y1="60" x2="900" y2="620"
          stroke="#374151" stroke-width="4" stroke-dasharray="10 8"/>

    <!-- Dotted band offset slightly toward the Invalid side (bottom-right) -->
    <line x1="70" y1="70" x2="910" y2="630"
          stroke="url(#nearBandDots)" stroke-width="16"/>

    <!-- subspaces (kept clearly in the Valid / upper-right area) -->
    <!-- Common-sense — left of the green area -->
    <g transform="translate(350,130)">
      <ellipse cx="0" cy="0" rx="120" ry="56" class="soft"/>
      <text x="-78" y="-6" class="h1" style="font-size:20px">Common-sense</text>
      <text x="-78" y="16" class="p">obvious, trivial to find</text>
    </g>

    <!-- Valid but sub-optimal — centered -->
    <g transform="translate(600,250)">
      <ellipse cx="0" cy="0" rx="130" ry="60" class="soft"/>
      <text x="-112" y="-6" class="h1" style="font-size:20px">Sub-Optimal</text>
      <text x="-112" y="16" class="p">works, not the best</text>
    </g>

    <!-- Creative / Frontier — right side -->
    <g transform="translate(750,400)">
      <ellipse cx="0" cy="0" rx="120" ry="70" class="soft"/>
      <text x="-92" y="-6" class="h1" style="font-size:20px">Creative</text>
      <text x="-92" y="14" class="p">rare, hard to find</text>
    </g>

    <!-- Legend -->
    <g transform="translate(50,300)">
      <rect width="260" height="130" rx="10" fill="white" stroke="#d1d5db"/>
      <text x="14" y="26" style="font:600 18px Inter,system-ui;fill:#111">Legend</text>
      <line x1="18" y1="50" x2="62" y2="50" stroke="#374151" stroke-width="4" stroke-dasharray="10 8"/>
      <text x="72" y="54" style="font:14px Inter,system-ui;fill:#333">Validity frontier</text>
      <line x1="18" y1="74" x2="62" y2="74" stroke="url(#nearBandDots)" stroke-width="10"/>
      <text x="72" y="78" style="font:14px Inter,system-ui;fill:#333">Near-valid band</text>
      <circle cx="40" cy="98" r="5" fill="#0f172a"/>
      <text x="72" y="102" style="font:14px Inter,system-ui;fill:#333">Candidate</text>
      <!-- Target star -->
      <text x="30" y="125" style="font:700 18px Inter,system-ui; fill:#111;">★</text>
      <text x="72" y="125" style="font:14px Inter,system-ui;fill:#333">Target creative solution</text>
    </g>

    <!-- Cleaner arrowhead for step links -->
    <defs>
      <marker id="arrowStep" markerWidth="12" markerHeight="8" refX="10" refY="4" orient="auto">
        <polygon points="0 0, 12 4, 0 8" fill="#d92c2c"/>
      </marker>
    </defs>

    <!-- FEWER, BIGGER SWINGS → tighten near goal -->
    <!-- Remove any old polyline/trajectory before pasting this -->
    <g id="trajectory" stroke="#111" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">
      <!-- step links (arrow only at the end of each segment) -->
      <line x1="320" y1="620" x2="240" y2="480" marker-end="url(#arrowStep)" stroke="#d92c2c"/>
      <line x1="240" y1="480" x2="425" y2="360" marker-end="url(#arrowStep)" stroke="#d92c2c"/>
      <line x1="425" y1="360" x2="470" y2="405" marker-end="url(#arrowStep)" stroke="#d92c2c"/>
      <line x1="470" y1="405" x2="535" y2="330" marker-end="url(#arrowStep)" stroke="#d92c2c"/>
      <line x1="535" y1="330" x2="615" y2="380" marker-end="url(#arrowStep)" stroke="#d92c2c"/>
      <line x1="615" y1="380" x2="700" y2="305" marker-end="url(#arrowStep)" stroke="#d92c2c"/>
      <line x1="700" y1="305" x2="780" y2="340" marker-end="url(#arrowStep)" stroke="#d92c2c"/>
    </g>

    <!-- Candidate dots (slightly shrinking) -->
    <g id="trajectory-dots" fill="#d92c2c" stroke="white" stroke-width="0.6" opacity="0.95">
      <circle cx="320" cy="620" r="5.0"/>
      <circle cx="240" cy="480" r="5.0"/>
      <circle cx="425" cy="360" r="4.9"/>
      <circle cx="470" cy="405" r="4.8"/>
      <circle cx="535" cy="330" r="4.6"/>
      <circle cx="615" cy="380" r="4.4"/>
      <circle cx="700" cy="305" r="4.2"/>
      <g id="target-star">
      <!-- subtle halo so it pops on any background -->
      <circle cx="785" cy="355" r="12" fill="rgba(255,255,255,0.7)"/>
      <text x="785" y="360" text-anchor="middle"
            style="font:700 22px Inter,system-ui; fill:#d92c2c;">★</text>
    </g>
    </g>

    <!-- (Optional) tiny, faint guide under the steps -->
    <polyline points="320,620 240,480 425,360 470,405 535,330 615,380 700,305"
              fill="none" stroke="#111" stroke-opacity="0.18" stroke-width="1.2" stroke-dasharray="2 6"/>
  </svg>
</div>
<style>
  .artifact-diagram { margin: 1rem 0 0; }
  .artifact-diagram svg { width: 100%; height: auto; display:block; }
</style>

### Open Questions
1.	Structured continuity: How can discrete structure—graphs, programs, mathematical symbols—be embedded within continuous models while preserving differentiability and computational efficiency?
For example: representing a scene graph linking objects and relations (“cube–on–table”) inside a neural latent space that still supports gradient-based learning and compositional reasoning.

2. Stable interfaces: How can information flow reliably between neural and symbolic components without loss, ambiguity, or instability?
For instance: a visual encoder that outputs symbolic predicates (“red”, “cube”, “on”) to a planner that must generate a manipulation sequence. How can gradients or feedback be propagated across this boundary so that both modules remain consistent and trainable?

3. Grounded semantics: How can representations acquire causal and actionable meaning from limited multimodal or partially supervised data, beyond statistical co-occurrence?
For example: a vision–language model that not only labels “a ball rolling” but can predict what will happen if the slope or friction changes—capturing causal dependencies rather than descriptive correlations.

4. Diagnostic formalism: What kinds of formal diagnostics can characterize representational adequacy?
For instance: developing metrics that detect when a learned representation preserves logical equivalence, or compositional semantics under transformation.