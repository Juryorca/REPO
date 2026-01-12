<script setup lang="ts">
import { computed } from 'vue'
import { useSlideContext } from '@slidev/client'

const { $clicks } = useSlideContext()
// 0: 初始全满
// 1: add rsp, 0x40
// 2: pop rbp
// 3: ret
// 4: caller cleanup
const step = computed(() => Math.max(0, Math.min(4, Number($clicks.value ?? 0))))

const showLocals = computed(() => step.value < 1)
const showSavedRbp = computed(() => step.value < 2)
const showRet = computed(() => step.value < 3)
const showArgs = computed(() => step.value < 4)
const showRbpPtr = computed(() => step.value < 2)

// 与 Prologue 严格一致的高度数值
const H_ARG = 60
const H_RET = 60
const H_RBP = 60
const H_CALL = 40
const H_LOC = 80

const rspBottom = computed(() => {
  if (step.value === 0) return H_ARG + H_RET + H_RBP + H_CALL + H_LOC
  if (step.value === 1) return H_ARG + H_RET + H_RBP
  if (step.value === 2) return H_ARG + H_RET
  if (step.value === 3) return H_ARG
  return 0
})

const rbpBottom = computed(() => H_ARG + H_RET + H_RBP)
</script>

<template>
  <div class="stack-wrapper">
    <div class="addr-tag">低地址 ↑</div>

    <div class="stack-container">
      <div class="pointer-layer">
        <div class="ptr-item rsp" :style="{ bottom: `${rspBottom}px` }">
          <div class="ptr-label">RSP</div>
          <div class="ptr-line"></div>
        </div>

        <div 
          class="ptr-item rbp" 
          :class="{ 'visible': showRbpPtr }" 
          :style="{ bottom: `${rbpBottom}px` }"
        >
          <div class="ptr-label">RBP</div>
          <div class="ptr-line"></div>
        </div>
      </div>

      <div class="stack-body">
        <div class="slot locals" :style="{ height: showLocals ? H_LOC+'px' : '0px' }">
          <div class="slot-text" v-show="showLocals">
            <span class="main">Local Variables</span>
            <span class="code">sub rsp, 0x40</span>
          </div>
        </div>

        <div class="slot callee" :style="{ height: showLocals ? H_CALL+'px' : '0px' }">
          <div class="slot-text" v-show="showLocals">Callee-saved Regs</div>
        </div>

        <div class="slot rbp-slot" :style="{ height: showSavedRbp ? H_RBP+'px' : '0px' }">
          <div class="slot-text" v-show="showSavedRbp">Saved RBP (old)</div>
        </div>

        <div class="slot ret-slot" :style="{ height: showRet ? H_RET+'px' : '0px' }">
          <div class="slot-text" v-show="showRet">Return Address</div>
        </div>

        <div class="slot arg-slot" :style="{ height: showArgs ? H_ARG+'px' : '0px' }">
          <div class="slot-text" v-show="showArgs">Stack Args (7+)</div>
        </div>
      </div>
    </div>

    <div class="addr-tag">高地址 ↓</div>
  </div>
</template>

<style scoped>
/* 样式直接复刻自 Prologue，确保像素级对齐 */
.stack-wrapper {
  --rsp-color: #00f2ff;
  --rbp-color: #ffde59;
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  max-width: 380px;
}

.addr-tag {
  font-size: 11px;
  font-family: ui-monospace, monospace; /* 强制指定等宽字体 */
  opacity: 0.4;
  margin: 10px 0;
}

.stack-container {
  position: relative;
  width: 240px;
  margin-left: 100px;
}

.pointer-layer {
  position: absolute;
  left: -110px;
  bottom: 0;
  top: 0;
  width: 110px;
  z-index: 20;
}

.ptr-item {
  position: absolute;
  left: 0;
  width: 115px; /* 这里 115px 配合 left -110px，箭头尖端正好压线 5px */
  display: flex;
  align-items: center;
  transition: bottom 0.5s cubic-bezier(0.4, 0, 0.2, 1);
  margin-bottom: -1px;
}

.ptr-label {
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 10px;
  font-weight: 900;
  font-family: ui-monospace, monospace; /* 强制指定等宽字体 */
  border: 1px solid currentColor;
  background: #000;
  box-shadow: 0 4px 10px rgba(0,0,0,0.5);
}

.ptr-line {
  flex-grow: 1;
  height: 2px;
  background: currentColor;
  position: relative;
}

.ptr-line::after {
  content: "";
  position: absolute;
  right: -5px;
  top: -3px;
  border-left: 7px solid currentColor;
  border-top: 4px solid transparent;
  border-bottom: 4px solid transparent;
}

.rsp { color: var(--rsp-color); }
.rbp { color: var(--rbp-color); opacity: 0; pointer-events: none; }
.rbp.visible { opacity: 1; }

.stack-body {
  border: 1px solid rgba(255,255,255,0.2);
  border-radius: 8px;
  background: rgba(255,255,255,0.02);
  overflow: hidden;
}

.slot {
  width: 100%;
  overflow: hidden;
  transition: height 0.5s cubic-bezier(0.4, 0, 0.2, 1);
  display: flex;
  align-items: center;
  padding: 0 15px;
  border-bottom: 1px solid rgba(255,255,255,0.1);
}
.slot:last-child { border-bottom: none; }

.slot-text {
  display: flex;
  flex-direction: column;
}

.main { 
  font-weight: 700; 
  font-size: 15px; 
  font-family: inherit; /* 继承全局字体 */
}
.code { 
  font-size: 11px; 
  color: var(--rsp-color); 
  opacity: 0.8; 
  font-family: ui-monospace, monospace; 
}

.locals { background: rgba(0, 242, 255, 0.08); }
.callee { background: rgba(255, 255, 255, 0.05); }
.rbp-slot { background: rgba(255, 222, 89, 0.08); }
.ret-slot { background: rgba(255, 80, 80, 0.08); }
.arg-slot { background: rgba(255, 165, 0, 0.08); }
</style>