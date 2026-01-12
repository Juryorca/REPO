<template>
  <div class="frame">
    <div class="addr addr-top">
      <span class="chip">栈顶（低地址）</span><span class="arrow">↑</span>
    </div>

    <div class="stack">
      <div class="slot loc">local variables + padding</div>
      <div class="slot callee">callee-saved regs</div>
      <div class="slot rbp">saved rbp</div>
      <div class="slot ret">return address</div>
      <div class="slot arg">stack args (7+)</div>
    </div>

    <div class="addr addr-bot">
      <span class="chip">栈底（高地址）</span><span class="arrow">↓</span>
    </div>
  </div>
</template>

<style scoped>
/* 画布宽度：别太宽 */
.frame{
  width: 420px;
  max-width: 76vw;
  margin: 0 auto;
}

/* 上下标注 */
.addr{
  display:flex;
  align-items:center;
  justify-content:center;
  gap:10px;
  margin: 10px 0;
  opacity:.85;
}
.chip{
  padding: 6px 12px;
  border-radius: 999px;
  border: 1px solid rgba(255,255,255,.14);
  background: rgba(255,255,255,.06);
  font-size: 14px;
  letter-spacing: .02em;
}
.arrow{ font-size: 18px; }

/* 栈主体 */
.stack{
  border-radius: 16px;
  overflow: hidden;
  border: 1px solid rgba(255,255,255,.14);
  background: rgba(255,255,255,.04);
  box-shadow: 0 18px 60px rgba(0,0,0,.45);
  max-height: 68vh;
}

/* 每格 */
.slot{
  padding: 16px 18px;
  border-top: 1px solid rgba(255,255,255,.10);
  font-size: 22px;
  font-weight: 800;
  line-height: 1.15;
}
.slot:first-child{ border-top:none; }

/* 配色（克制） */
.loc   { background: rgba(120, 255, 180, 0.12); }
.callee{ background: rgba(200, 200, 255, 0.08); }
.rbp   { background: rgba(120, 200, 255, 0.12); }
.ret   { background: rgba(255, 80, 80, 0.14); }
.arg   { background: rgba(255, 200, 80, 0.12); }

@media (max-width: 900px){
  .frame{ width: 360px; }
  .slot{ font-size: 20px; padding: 14px 16px; }
}
</style>
