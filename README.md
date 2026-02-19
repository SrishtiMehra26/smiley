<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Smiley Feeding Game</title>
  <style>
    html,body { height:100%; margin:0; font-family:Segoe UI, Roboto, Arial; background:#f0f6ff; }
    .wrap { display:flex; flex-direction:column; align-items:center; padding:18px; }
    canvas { background:linear-gradient(#eef6ff,#ffffff); border:1px solid #cbdff5; border-radius:8px; touch-action:none; }
    .controls { margin-top:12px; display:flex; gap:8px; align-items:center; }
    button { padding:8px 12px; border-radius:6px; border:1px solid #9fb8e6; background:#dff0ff; cursor:pointer; }
    p { margin:8px 0 0 0; color:#334; }
    @media (max-width:700px){ canvas{ width:100%; height:auto; } }
  </style>
</head>
<body>
  <div class="wrap">
    <h2>Feed the Smiley</h2>
    <canvas id="c" width="900" height="600" aria-label="Smiley feeding game"></canvas>
    <div class="controls">
      <button id="reset">Reset</button>
      <div style="color:#334;">Drag the chocolate or broccoli onto the smiley to change its expression.</div>
    </div>
    <p id="stateText"></p>
  </div>

  <script>
    const canvas = document.getElementById('c');
    const ctx = canvas.getContext('2d');
    const stateText = document.getElementById('stateText');
    const resetBtn = document.getElementById('reset');

    // Smiley properties
    const smiley = {
      x: canvas.width / 2,
      y: canvas.height / 2,
      r: 150,
      expression: 'neutral' // 'neutral' | 'happy' | 'sad'
    };

    // Items: chocolate and broccoli
    const items = {
      chocolate: {
        type: 'chocolate',
        x: canvas.width - 220,
        y: 180,
        w: 90,
        h: 130,
        color: '#5a2f1a'
      },
      broccoli: {
        type: 'broccoli',
        x: canvas.width - 220,
        y: 350,
        w: 90,
        h: 90,
        color: '#3b8b3b'
      }
    };

    let dragging = null;
    let dragOffset = {x:0,y:0};

    function reset() {
      smiley.expression = 'neutral';
      items.chocolate.x = canvas.width - 220;
      items.chocolate.y = 180;
      items.broccoli.x = canvas.width - 220;
      items.broccoli.y = 350;
      updateStateText();
    }

    function updateStateText(){
      if (smiley.expression === 'neutral') stateText.textContent = 'Expression: Neutral';
      else if (smiley.expression === 'happy') stateText.textContent = 'Expression: Happy 😊';
      else if (smiley.expression === 'sad') stateText.textContent = 'Expression: Sad ☹️';
    }

    function draw() {
      ctx.clearRect(0,0,canvas.width,canvas.height);

      // Draw instructions overlay
      ctx.save();
      ctx.fillStyle = 'rgba(255,255,255,0.6)';
      ctx.fillRect(10,10,340,48);
      ctx.fillStyle = '#235';
      ctx.font = '14px system-ui, Arial';
      ctx.fillText('Drag the chocolate or broccoli onto the smiley to feed it.', 18, 36);
      ctx.restore();

      drawSmiley(smiley);
      drawChocolate(items.chocolate);
      drawBroccoli(items.broccoli);

      requestAnimationFrame(draw);
    }

    function drawSmiley(s) {
      // body
      ctx.save();
      ctx.beginPath();
      const grd = ctx.createRadialGradient(s.x - s.r*0.3, s.y - s.r*0.4, s.r*0.1, s.x, s.y, s.r);
      grd.addColorStop(0, '#fff89a');
      grd.addColorStop(0.5, '#fff067');
      grd.addColorStop(1, '#f5c400');
      ctx.fillStyle = grd;
      ctx.arc(s.x, s.y, s.r, 0, Math.PI*2);
      ctx.fill();

      ctx.lineWidth = 4;
      ctx.strokeStyle = '#e0b400';
      ctx.stroke();

      // eyes
      const eyeOffsetX = 55;
      const eyeOffsetY = -25;
      ctx.fillStyle = '#111';
      ctx.beginPath();
      ctx.ellipse(s.x - eyeOffsetX, s.y + eyeOffsetY, 14, 18, 0, 0, Math.PI*2);
      ctx.fill();
      ctx.beginPath();
      ctx.ellipse(s.x + eyeOffsetX, s.y + eyeOffsetY, 14, 18, 0, 0, Math.PI*2);
      ctx.fill();

      // eyebrows depend on expression
      ctx.strokeStyle = '#333';
      ctx.lineWidth = 6;
      ctx.lineCap = 'round';
      if (s.expression === 'neutral') {
        // flat eyebrows
        ctx.beginPath();
        ctx.moveTo(s.x - eyeOffsetX - 22, s.y + eyeOffsetY - 30);
        ctx.lineTo(s.x - eyeOffsetX + 22, s.y + eyeOffsetY - 30);
        ctx.stroke();
        ctx.beginPath();
        ctx.moveTo(s.x + eyeOffsetX - 22, s.y + eyeOffsetY - 30);
        ctx.lineTo(s.x + eyeOffsetX + 22, s.y + eyeOffsetY - 30);
        ctx.stroke();
      } else if (s.expression === 'happy') {
        // raised outer edges
        ctx.beginPath();
        ctx.moveTo(s.x - eyeOffsetX - 22, s.y + eyeOffsetY - 34);
        ctx.quadraticCurveTo(s.x - eyeOffsetX, s.y + eyeOffsetY - 42, s.x - eyeOffsetX + 22, s.y + eyeOffsetY - 34);
        ctx.stroke();
        ctx.beginPath();
        ctx.moveTo(s.x + eyeOffsetX - 22, s.y + eyeOffsetY - 34);
        ctx.quadraticCurveTo(s.x + eyeOffsetX, s.y + eyeOffsetY - 42, s.x + eyeOffsetX + 22, s.y + eyeOffsetY - 34);
        ctx.stroke();
      } else if (s.expression === 'sad') {
        // slanted inward (concern)
        ctx.beginPath();
        ctx.moveTo(s.x - eyeOffsetX - 18, s.y + eyeOffsetY - 18);
        ctx.lineTo(s.x - eyeOffsetX + 18, s.y + eyeOffsetY - 36);
        ctx.stroke();
        ctx.beginPath();
        ctx.moveTo(s.x + eyeOffsetX - 18, s.y + eyeOffsetY - 36);
        ctx.lineTo(s.x + eyeOffsetX + 18, s.y + eyeOffsetY - 18);
        ctx.stroke();
      }

      // mouth based on expression
      ctx.strokeStyle = '#111';
      ctx.lineWidth = 8;
      ctx.lineCap = 'round';
      if (s.expression === 'neutral') {
        ctx.beginPath();
        ctx.moveTo(s.x - 55, s.y + 60);
        ctx.lineTo(s.x + 55, s.y + 60);
        ctx.stroke();
      } else if (s.expression === 'happy') {
        ctx.beginPath();
        ctx.arc(s.x, s.y + 40, 60, 0.15 * Math.PI, 0.85 * Math.PI);
        ctx.stroke();
      } else if (s.expression === 'sad') {
        ctx.beginPath();
        ctx.arc(s.x, s.y + 90, 60, 1.15 * Math.PI, 1.85 * Math.PI, true);
        ctx.stroke();
      }

      ctx.restore();
    }

    function drawChocolate(c) {
      ctx.save();
      // shadow
      ctx.fillStyle = 'rgba(0,0,0,0.12)';
      ctx.fillRect(c.x+6, c.y + c.h - 8, c.w, 12);

      // body
      ctx.fillStyle = c.color;
      roundRect(ctx, c.x, c.y, c.w, c.h, 8, true, false);

      // chocolate stripes
      ctx.fillStyle = '#6f3b23';
      for (let i=0;i<4;i++) {
        ctx.fillRect(c.x + 10 + i*18, c.y + 12, 12, c.h - 24);
      }

      // wrapper band
      ctx.fillStyle = '#b36b3b';
      ctx.fillRect(c.x, c.y + c.h - 28, c.w, 20);

      // tiny shine
      ctx.fillStyle = 'rgba(255,255,255,0.18)';
      ctx.fillRect(c.x + 8, c.y + 8, 14, 6);

      ctx.restore();
    }

    function drawBroccoli(b) {
      ctx.save();
      // stalk
      ctx.fillStyle = '#7ea77e';
      roundRect(ctx, b.x + b.w/2 - 12, b.y + b.h - 10, 24, 18, 6, true, false);

      // crowns - three clustered circles
      const cx = b.x + b.w/2;
      const cy = b.y + b.h/2 - 6;
      const greens = ['#4BAF4B', '#3a8b3a', '#6bcb6b'];
      ctx.fillStyle = greens[0];
      circle(ctx, cx, cy-8, 26, true);
      ctx.fillStyle = greens[1];
      circle(ctx, cx-26, cy+8, 22, true);
      ctx.fillStyle = greens[2];
      circle(ctx, cx+26, cy+8, 22, true);

      // little highlight
      ctx.fillStyle = 'rgba(255,255,255,0.12)';
      circle(ctx, cx+8, cy-12, 8, true);

      ctx.restore();
    }

    // Utilities
    function roundRect(ctx, x, y, w, h, r, fill, stroke) {
      if (r === undefined) r = 5;
      ctx.beginPath();
      ctx.moveTo(x + r, y);
      ctx.arcTo(x + w, y,   x + w, y + h, r);
      ctx.arcTo(x + w, y + h, x, y + h, r);
      ctx.arcTo(x, y + h,   x, y,   r);
      ctx.arcTo(x, y,       x + w, y,   r);
      ctx.closePath();
      if (fill) ctx.fill();
      if (stroke) ctx.stroke();
    }

    function circle(ctx, x, y, r, fill) {
      ctx.beginPath();
      ctx.arc(x, y, r, 0, Math.PI*2);
      if (fill) ctx.fill();
      else ctx.stroke();
    }

    // Pointer / mouse handling (works for mouse and touch)
    canvas.addEventListener('pointerdown', (e) => {
      const p = getPointerPos(e);
      for (let key of ['chocolate', 'broccoli']) {
        const it = items[key];
        if (pointInRect(p.x, p.y, it.x, it.y, it.w, it.h)) {
          dragging = it;
          dragOffset.x = p.x - it.x;
          dragOffset.y = p.y - it.y;
          // bring to front by cloning ordering (no z-order in canvas, but we can choose draw order)
          // We'll draw chocolate and broccoli in the same order always; no layering change required
          canvas.setPointerCapture(e.pointerId);
          break;
        }
      }
    });

    canvas.addEventListener('pointermove', (e) => {
      if (!dragging) return;
      const p = getPointerPos(e);
      dragging.x = p.x - dragOffset.x;
      dragging.y = p.y - dragOffset.y;
    });

    canvas.addEventListener('pointerup', (e) => {
      if (!dragging) return;
      const p = getPointerPos(e);
      // compute center of dragging object
      const cx = dragging.x + dragging.w/2;
      const cy = dragging.y + dragging.h/2;
      const dx = cx - smiley.x;
      const dy = cy - smiley.y;
      const dist = Math.sqrt(dx*dx + dy*dy);

      if (dist < smiley.r - 30) {
        // considered "fed"
        if (dragging.type === 'chocolate') {
          smiley.expression = 'happy';
        } else if (dragging.type === 'broccoli') {
          smiley.expression = 'sad';
        }
      } else {
        // not fed — do nothing to expression (remains as it was)
      }

      updateStateText();
      dragging = null;
      try { canvas.releasePointerCapture(e.pointerId); } catch(e){}
    });

    function getPointerPos(e) {
      const rect = canvas.getBoundingClientRect();
      return {
        x: (e.clientX - rect.left) * (canvas.width / rect.width),
        y: (e.clientY - rect.top) * (canvas.height / rect.height)
      };
    }

    function pointInRect(px, py, rx, ry, rw, rh) {
      return px >= rx && px <= rx + rw && py >= ry && py <= ry + rh;
    }

    // Reset button
    resetBtn.addEventListener('click', reset);

    // Initialize
    reset();
    draw();

    // Make canvas resize friendly for small screens while keeping internal resolution
    function resizeCanvasToDisplaySize() {
      const displayWidth  = Math.floor(canvas.clientWidth * devicePixelRatio);
      const displayHeight = Math.floor(canvas.clientHeight * devicePixelRatio);
      if (canvas.width !== displayWidth || canvas.height !== displayHeight) {
        canvas.width = Math.max(600, displayWidth);
        canvas.height = Math.max(400, displayHeight);
        // Recalculate smiley center if desired — keep center after resize:
        smiley.x = canvas.width / 2;
        smiley.y = canvas.height / 2;
        // reposition items to a safe right area if they were off-screen:
        items.chocolate.x = Math.min(items.chocolate.x, canvas.width - 120);
        items.broccoli.x = Math.min(items.broccoli.x, canvas.width - 120);
      }
    }

    // Optional: keep checking for canvas CSS size changes
    const ro = new ResizeObserver(resizeCanvasToDisplaySize);
    ro.observe(canvas);
  </script>
</body>
</html>

