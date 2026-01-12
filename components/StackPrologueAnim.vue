<script setup lang="ts">
import { computed } from 'vue'
import { useSlideContext } from '@slidev/client'

const { $clicks } = useSlideContext()
const step = computed(() => Math.max(0, Math.min(5, Number($clicks.value ?? 0))))

// 状态开关
const openArgs = computed(() => step.value >= 1)
const openRet = computed(() => step.value >= 2)
const openSavedRbp = computed(() => step.value >= 3)
const showRbpPtr = computed(() => step.value >= 4)
const openLocals = computed(() => step.value >= 5)

// 精确高度定义 (px)
const H_ARG = 60
const H_RET = 60
const H_RBP = 60
const H_CALL = 40
const H_LOC = 80

// 计算 RSP 距离【栈底】的高度
const rspBottom = computed(() => {
  let h = 0
  if (step.value === 1) h = H_ARG
  if (step.value === 2) h = H_ARG + H_RET
  // Step 3 (push rbp) 和 Step 4 (mov rbp, rsp) 时，RSP 都在 saved rbp 的顶端
  if (step.value >= 3 && step.value < 5) h = H_ARG + H_RET + H_RBP
  if (step.value === 5) h = H_ARG + H_RET + H_RBP + H_CALL + H_LOC
  return h
})

// 计算 RBP 距离【栈底】的高度
// 核心修正：指向 Saved RBP 的顶端 = 参数高 + 返回地址高 + RBP格自身高
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
        <div class="slot locals" :style="{ height: openLocals ? H_LOC+'px' : '0px' }">
          <div class="slot-text" v-show="openLocals">
            <span class="main">Local Variables</span>
            <span class="code">sub rsp, 0x40</span>
          </div>
        </div>

        <div class="slot callee" :style="{ height: openLocals ? H_CALL+'px' : '0px' }">
          <div class="slot-text" v-show="openLocals">Callee-saved Regs</div>
        </div>

        <div class="slot rbp-slot" :style="{ height: openSavedRbp ? H_RBP+'px' : '0px' }">
          <div class="slot-text" v-show="openSavedRbp">Saved RBP (old)</div>
        </div>

        <div class="slot ret-slot" :style="{ height: openRet ? H_RET+'px' : '0px' }">
          <div class="slot-text" v-show="openRet">Return Address</div>
        </div>

        <div class="slot arg-slot" :style="{ height: openArgs ? H_ARG+'px' : '0px' }">
          <div class="slot-text" v-show="openArgs">Stack Args (7+)</div>
        </div>
      </div>
    </div>

    <div class="addr-tag">高地址 ↓</div>
  </div>
</template>

<style scoped>
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
  font-family: ui-monospace, monospace;
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
  width: 115px;
  display: flex;
  align-items: center;
  transition: bottom 0.5s cubic-bezier(0.4, 0, 0.2, 1);
  margin-bottom: -1px; /* 对齐边框缝隙 */
}

.ptr-label {
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 10px;
  font-weight: 900;
  font-family: ui-monospace, monospace;
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
  animation: fadeIn 0.3s forwards;
}

.main { font-weight: 700; font-size: 15px; }
.code { font-size: 11px; color: var(--rsp-color); opacity: 0.8; font-family: monospace; }

.locals { background: rgba(0, 242, 255, 0.08); }
.callee { background: rgba(255, 255, 255, 0.05); }
.rbp-slot { background: rgba(255, 222, 89, 0.08); }
.ret-slot { background: rgba(255, 80, 80, 0.08); }
.arg-slot { background: rgba(255, 165, 0, 0.08); }

@keyframes fadeIn {
  from { opacity: 0; transform: translateX(5px); }
  to { opacity: 1; transform: translateX(0); }
}
</style>