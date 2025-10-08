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

<!-- ===== Interactive Representation Space (data-driven) ===== -->
<style>
  .rep-space-wrap{max-width:900px;margin:2rem auto;aspect-ratio:1/1;position:relative;border-radius:12px;overflow:hidden;border:1px solid rgba(0,0,0,.12);box-shadow:0 6px 24px rgba(0,0,0,.06)}
  /* Four-corner gradient (saturated). Lower saturation by changing filter. */
  .rep-space-bg{position:absolute;inset:0;background:
    radial-gradient(120% 120% at 0%   0%,   #00C853 0% 40%, transparent 60%),
    radial-gradient(120% 120% at 100% 0%,   #E53935 0% 40%, transparent 60%),
    radial-gradient(120% 120% at 0%   100%, #1E88E5 0% 40%, transparent 60%),
    radial-gradient(120% 120% at 100% 100%, #FDD835 0% 40%, transparent 60%),
    radial-gradient(120% 120% at 50%  50%,  rgba(255,255,255,.15), rgba(255,255,255,0) 70%);filter:saturate(100%)}
  .rep-axis{position:absolute;color:rgba(0,0,0,.75);font-weight:600;font-size:.95rem;text-shadow:0 1px 0 rgba(255,255,255,.6)}
  .rep-axis.tl{top:.5rem;left:.75rem} .rep-axis.tr{top:.5rem;right:.75rem}
  .rep-axis.bl{bottom:.5rem;left:.75rem} .rep-axis.br{bottom:.5rem;right:.75rem}
  .rep-axis.lt{top:.75rem;left:.75rem;transform:translate(-.4rem,0)}
  .rep-axis.rt{top:.75rem;right:.75rem;transform:translate(.4rem,0)}
  .rep-node{--size:16px;position:absolute;transform:translate(-50%,-50%);width:var(--size);height:var(--size);border-radius:999px;border:2px solid rgba(255,255,255,.9);box-shadow:0 2px 10px rgba(0,0,0,.18);cursor:pointer}
  .rep-node-label{position:absolute;transform:translate(-50%,calc(-50% - 18px));white-space:nowrap;font-size:.95rem;font-weight:600;color:rgba(0,0,0,.9);text-shadow:0 1px 0 rgba(255,255,255,.7);pointer-events:none}
  .rep-card{position:absolute;min-width:240px;max-width:360px;background:rgba(255,255,255,.95);backdrop-filter:blur(4px);border:1px solid rgba(0,0,0,.08);border-radius:10px;padding:.75rem .9rem;box-shadow:0 10px 30px rgba(0,0,0,.18);z-index:5;display:none}
  .rep-card h4{margin:0 0 .35rem 0;font-size:1rem} .rep-card ul{margin:0;padding-left:1rem}
  .rep-card a{color:#0d47a1;text-decoration:none} .rep-card a:hover{text-decoration:underline}
  .chip{display:inline-block;width:.85em;height:.85em;border-radius:50%;margin-right:.4em;vertical-align:baseline}
</style>

<div class="rep-space-wrap" id="repSpace">
  <div class="rep-space-bg"></div>
  <!-- Side/corner labels -->
  <div class="rep-axis tl">Concrete</div>
  <div class="rep-axis tr">Abstract</div>
  <div class="rep-axis bl">Explicit</div>
  <div class="rep-axis br">Implicit</div>
  <div class="rep-axis lt">Discrete</div>
  <div class="rep-axis rt">Grounded</div>
  <div class="rep-card" id="repCard"></div>
</div>

<script>
(function(){
  const DATA_URL = "{{ '/assets/json/representation-nodes.json' | relative_url }}";
  const wrap = document.getElementById('repSpace');
  const card = document.getElementById('repCard');

  function makeNode(node){
    const dot = document.createElement('div');
    dot.className = 'rep-node';
    dot.style.left = node.x + '%';
    dot.style.top  = node.y + '%';
    dot.style.background = node.color || '#555';
    if(node.size){ dot.style.setProperty('--size', node.size + 'px'); }

    const label = document.createElement('div');
    label.className = 'rep-node-label';
    label.textContent = node.label ?? 'Untitled';
    label.style.left = node.x + '%';
    label.style.top  = node.y + '%';

    const show = (evt)=>{
      const rect = wrap.getBoundingClientRect();
      const px = rect.left + (node.x/100)*rect.width;
      const py = rect.top  + (node.y/100)*rect.height;
      card.innerHTML = `
        <h4><span class="chip" style="background:${node.color || '#555'}"></span>${node.label ?? 'Untitled'}</h4>
        ${
          Array.isArray(node.items) && node.items.length
          ? `<ul>${node.items.map(p=>`<li><a href="${p.url}" target="_blank" rel="noopener">${p.title}</a></li>`).join('')}</ul>`
          : `<div>No links yet.</div>`
        }
      `;
      const ox = 18, oy = -18;
      card.style.left = (px - rect.left + ox) + 'px';
      card.style.top  = (py - rect.top  + oy) + 'px';
      card.style.display = 'block';
    };
    const hide = ()=> card.style.display = 'none';

    dot.addEventListener('mouseenter', show);
    dot.addEventListener('mouseleave', hide);
    dot.addEventListener('click', show);   // tap support
    wrap.addEventListener('click', (e)=>{ if(!e.target.classList.contains('rep-node')) hide(); });

    wrap.appendChild(dot);
    wrap.appendChild(label);
  }

  async function init(){
    try{
      const res = await fetch(DATA_URL, {cache:'no-store'});
      const data = await res.json();
      (data || []).forEach(makeNode);
    }catch(err){
      console.error('Failed loading nodes:', err);
      // Fallback: show helpful message
      card.style.display = 'block';
      card.style.left = '12px';
      card.style.top = '12px';
      card.innerHTML = `<strong>Could not load nodes.</strong><br><code>${DATA_URL}</code>`;
    }
  }
  init();
})();
</script>
<!-- ===== /Interactive Representation Space ===== -->