---
layout: page
title: representation
description: 
img: /assets/img/representation.png
importance: 1
category: work
related_publications: false
---
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/representation.png" title="Representation" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

<!-- 
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/1.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/3.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Caption photos easily. On the left, a road goes through a tunnel. Middle, leaves artistically fall in a hipster photoshoot. Right, in another hipster photoshoot, a lumberjack grasps a handful of pine needles.
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    This image can also have a caption. It's like magic.
</div>

You can also put regular text between your rows of images, even citations {% cite einstein1950meaning %}.
Say you wanted to write a bit about your project before you posted the rest of the images.
You describe how you toiled, sweated, _bled_ for your project, and then... you reveal its glory in the next row of images.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    You can also have artistically styled 2/3 + 1/3 images, like these.
</div>

The code is simple.
Just wrap your images with `<div class="col-sm">` and place them inside `<div class="row">` (read more about the <a href="https://getbootstrap.com/docs/4.4/layout/grid/">Bootstrap Grid</a> system).
To make images responsive, add `img-fluid` class to each; for rounded corners and shadows use `rounded` and `z-depth-1` classes.
Here's the code for the last row of images above:

{% raw %}

```html
<div class="row justify-content-sm-center">
  <div class="col-sm-8 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
``` 

{% endraw %}
-->

<!-- ===== Representation Compass (SVG, radial / octagon) ===== -->
<style>
  .repc-wrap { max-width: 900px; margin: 2rem auto; }
  .repc-figure {
    position: relative; aspect-ratio: 1/1; width: 100%;
    border-radius: 16px; overflow: hidden;
    box-shadow: 0 6px 24px rgba(0,0,0,.06);
    border: 1px solid rgba(0,0,0,.08);
  }
  /* Conic gradient background; lower opacity if you prefer subtle */
  .repc-bg {
    position: absolute; inset: 0;
    background:
      conic-gradient(from -90deg,
        #00C853 0deg,    /* Concrete */
        #00C853 22.5deg,
        #2E7D32 45deg,   /* Discrete (use a greenish tone) */
        #1E88E5 90deg,   /* Explicit (cool blue) */
        #7E57C2 135deg,  /* Implicit (lavender) */
        #C62828 180deg,  /* Abstract (red) */
        #9C27B0 225deg,  /* Ungrounded (purple) */
        #F9A825 270deg,  /* Continuous (yellow/orange) */
        #00C853 360deg
      );
    opacity: 0.18; /* tweak for taste, or set to 0 to drop colors */
  }
  .repc-svg { position: absolute; inset: 0; }
  .repc-label { font: 600 13px/1.15 system-ui, -apple-system, Segoe UI, Roboto, sans-serif; fill: rgba(0,0,0,.85); text-anchor: middle; }
  .repc-tick  { stroke: rgba(0,0,0,.18); stroke-width: 1; }
  .repc-ring  { stroke: rgba(0,0,0,.10); stroke-width: 1; fill: none; }
  .repc-node  { stroke: #fff; stroke-width: 2; }
  .repc-node-label { font: 600 13px/1.2 system-ui, -apple-system, Segoe UI, Roboto, sans-serif; fill: rgba(0,0,0,.92); }
  .repc-card {
    position:absolute; padding:.6rem .8rem; min-width:220px; max-width:360px;
    background: rgba(255,255,255,.96); border:1px solid rgba(0,0,0,.08); border-radius:10px;
    box-shadow:0 10px 30px rgba(0,0,0,.18); display:none; z-index:10;
  }
  .repc-card h4{ margin:.1rem 0 .4rem 0; font-size:15px; }
  .repc-card ul{ margin:0; padding-left:1rem; }
  .repc-card a { color:#0d47a1; text-decoration:none; }
  .repc-card a:hover { text-decoration:underline; }
</style>

<div class="repc-wrap">
  <div class="repc-figure" id="repc">
    <div class="repc-bg"></div>
    <svg class="repc-svg" viewBox="0 0 1000 1000" preserveAspectRatio="none" id="repcSvg"></svg>
    <div class="repc-card" id="repcCard"></div>
  </div>
</div>

<script>
(function(){
  const DATA_URL = "{{ '/assets/representation-nodes.json' | relative_url }}";
  const svg = document.getElementById('repcSvg');
  const box = svg.viewBox.baseVal; // 0..1000
  const cx = 500, cy = 500, R = 430; // center & radius
  const CARD = document.getElementById('repcCard');
  const FIG  = document.getElementById('repc');

  // 8 compass poles clockwise from top (Abstract at 0°)
  const POLES = [
    {name:'Abstract',    ang:  -90},
    {name:'Ungrounded',  ang:  -45},
    {name:'Implicit',    ang:    0},
    {name:'Continuous',  ang:   45},
    {name:'Concrete',    ang:   90},
    {name:'Grounded',    ang:  135},
    {name:'Explicit',    ang:  180},
    {name:'Discrete',    ang: -135}
  ];
  const toRad = d => (d*Math.PI)/180;

  // ----- helpers -----
  function polToXY(rad, angDeg){
    const t = toRad(angDeg);
    return { x: cx + rad*Math.cos(t), y: cy + rad*Math.sin(t) };
  }
  function line(x1,y1,x2,y2, cls){
    const el = document.createElementNS('http://www.w3.org/2000/svg', 'line');
    el.setAttribute('x1', x1); el.setAttribute('y1', y1);
    el.setAttribute('x2', x2); el.setAttribute('y2', y2);
    el.setAttribute('class', cls);
    return el;
  }
  function circle(cx_, cy_, r, cls, fill){
    const el = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
    el.setAttribute('cx', cx_); el.setAttribute('cy', cy_); el.setAttribute('r', r);
    el.setAttribute('class', cls); if(fill) el.setAttribute('fill', fill);
    return el;
  }
  function text(x,y, str, cls, anchor='middle'){
    const t = document.createElementNS('http://www.w3.org/2000/svg', 'text');
    t.setAttribute('x', x); t.setAttribute('y', y); t.setAttribute('class', cls);
    t.setAttribute('text-anchor', anchor); t.textContent = str; return t;
  }

  // ----- grid (rings + axis ticks + labels) -----
  for(let r=R; r>=R*0.25; r-=R*0.25){
    svg.appendChild(circle(cx,cy,r,'repc-ring'));
  }
  POLES.forEach(p=>{
    const a = polToXY(R, p.ang);
    svg.appendChild(line(cx,cy,a.x,a.y,'repc-tick'));
    const lab = polToXY(R+28, p.ang);
    svg.appendChild(text(lab.x, lab.y, p.name, 'repc-label'));
  });

  // ----- projection: from 4 scalars to 2D -----
  // We form a vector sum of the four “top” directions (opposites implied).
  const DIR = {
    Abstract:     {x: Math.cos(toRad(-90)),  y: Math.sin(toRad(-90))},
    Ungrounded:   {x: Math.cos(toRad(-45)),  y: Math.sin(toRad(-45))},
    Implicit:     {x: Math.cos(toRad(0)),    y: Math.sin(toRad(0))},
    Continuous:   {x: Math.cos(toRad(45)),   y: Math.sin(toRad(45))}
    // explicit, discrete, grounded, concrete are 1 - corresponding score
  };
  function project(n){
    const a = n.abstract    ?? 0.5;
    const u = n.ungrounded  ?? 0.5;
    const i = n.implicit    ?? 0.5;
    const c = n.continuous  ?? 0.5;
    // add opposites implicitly:
    const ex = 1 - i, dis = 1 - c, gr = 1 - u, con = 1 - a;

    // vector sum across all eight (weights times unit vectors)
    // use the same four directions plus subtract opposites by adding in opposing vectors
    let vx = 0, vy = 0;
    const add = (w, angDeg)=>{ vx += w*Math.cos(toRad(angDeg)); vy += w*Math.sin(toRad(angDeg)); };
    add(a,  -90);   add(u,  -45);   add(i,   0);   add(c,   45);
    add(con,  90);  add(gr, 135);   add(ex, 180);  add(dis,-135);

    // normalize magnitude to [0, R*0.9] for readability
    const mag = Math.hypot(vx, vy) || 1;
    const scale = (R*0.9) / (4); // conservative scaling
    const px = cx + (vx/mag) * (mag*scale/Math.SQRT2);
    const py = cy + (vy/mag) * (mag*scale/Math.SQRT2);
    return {x:px, y:py};
  }

  // ----- tooltip -----
  function showCard(node, px, py){
    CARD.innerHTML = `
      <h4><span style="display:inline-block;width:.8em;height:.8em;border-radius:50%;background:${node.color};margin-right:.4em"></span>${node.label}</h4>
      ${
        Array.isArray(node.items) && node.items.length
        ? `<ul>${node.items.map(d=>`<li><a target="_blank" rel="noopener" href="${d.url}">${d.title}</a></li>`).join('')}</ul>`
        : `<div>No links yet.</div>`
      }`;
    // position near the point
    const rect = FIG.getBoundingClientRect();
    CARD.style.left = ( (px/1000)*rect.width + 16 ) + 'px';
    CARD.style.top  = ( (py/1000)*rect.height - 16 ) + 'px';
    CARD.style.display = 'block';
  }
  function hideCard(){ CARD.style.display = 'none'; }

  // ----- load data -----
  fetch(DATA_URL, {cache:'no-store'})
    .then(r=>r.json())
    .then(nodes=>{
      nodes.forEach(n=>{
        const p = project(n);
        const dot = circle(p.x, p.y, 9, 'repc-node', n.color || '#555');
        dot.style.cursor = 'pointer';
        dot.addEventListener('mouseenter', ()=>showCard(n, p.x, p.y));
        dot.addEventListener('mouseleave', hideCard);
        dot.addEventListener('click',      ()=>showCard(n, p.x, p.y));
        svg.appendChild(dot);

        const label = text(p.x, p.y - 16, n.label, 'repc-node-label');
        svg.appendChild(label);
      });
      // click anywhere to dismiss
      FIG.addEventListener('click', (e)=>{ if(e.target.tagName!=='circle') hideCard(); });
    })
    .catch(err=>{
      console.error(err);
      CARD.style.display='block';
      CARD.style.left='12px'; CARD.style.top='12px';
      CARD.innerHTML = `<strong>Could not load nodes</strong>`;
    });
})();
</script>
<!-- ===== /Representation Compass ===== -->