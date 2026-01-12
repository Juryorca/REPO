<script setup>
import { ref, computed } from 'vue'

const step = ref(0)
const stackData = [
  { addr: '0x7fffffffe000', val: '0x7ffff7bc35a2', label: 'pop rdi; ret (in libc)' },
  { addr: '0x7fffffffe008', val: '0x7ffff7db1d53', label: '"/bin/sh" addr (in libc)' },
  { addr: '0x7fffffffe010', val: '0x7ffff7a3b060', label: 'system() @libc' },
  { addr: '0x7fffffffe018', val: '0x000000000000', label: 'dummy' },
]

const states = [
  { rip: '0x4011fb', rspIdx: 0, rdi: '0x0', note: '当前函数返回，即将跳转到攻击者布置的首个 gadget。' },
  { rip: '0x7ffff7bc35a2', rspIdx: 1, rdi: '0x0', note: '执行 ret：将栈顶的 pop rdi 地址弹出到 RIP。' },
  { rip: '0x7ffff7bc35a3', rspIdx: 2, rdi: '0x7ffff7db1d53', note: '执行 pop rdi：从栈中取出 /bin/sh 地址并存入 RDI。' },
  { rip: '0x7ffff7a3b060', rspIdx: 3, rdi: '0x7ffff7db1d53', note: '执行 ret：跳转至 system 函数，此时 RDI 已指向 /bin/sh。' }
]

const currentState = computed(() => states[step.value])
const next = () => { if (step.value < states.length - 1) step.value++ }
const reset = () => step.value = 0
</script>

<template>
  <div class="p-[2px] rounded-xl bg-gradient-to-r from-cyan-500 via-purple-600 to-blue-500 animate-gradient-x shadow-[0_0_20px_rgba(124,58,237,0.5)] max-w-5xl mx-auto overflow-hidden">
    
    <div class="font-mono text-[14px] bg-[#0d0d0d] p-4 rounded-lg w-full">
      
      <div class="flex gap-3 mb-4 items-center border-b border-gray-800 pb-3">
        <button @click="next" v-if="step < 3" 
          class="px-6 py-1.5 bg-gradient-to-r from-blue-600 to-indigo-600 hover:from-blue-500 hover:to-indigo-500 text-white rounded font-bold transition-all active:scale-95 shadow-[0_0_15px_rgba(37,99,235,0.4)]">
          STEP (NI)
        </button>
        <button @click="reset" class="px-4 py-1.5 bg-[#222] hover:bg-[#333] text-gray-300 rounded border border-gray-700 transition-all">
          RESTART
        </button>
        <div v-if="step === 3" class="text-transparent bg-clip-text bg-gradient-to-r from-green-400 to-cyan-400 font-bold ml-4 animate-pulse uppercase text-lg italic tracking-widest">
          [ Shell Pop! ]
        </div>
      </div>

      <div class="flex gap-6 items-start">
        <div class="flex-1 space-y-4">
          <div class="bg-black/50 p-3 rounded border border-blue-500/30 shadow-[0_0_15px_rgba(0,0,0,0.5)]">
            <div class="text-cyan-400 font-bold text-xs mb-2 border-b border-gray-900 italic">REGISTERS</div>
            <div class="space-y-1">
              <div class="flex justify-between items-center text-sm">
                <span class="text-red-500 font-bold w-12 text-left uppercase drop-shadow-[0_0_5px_rgba(239,68,68,0.5)]">RDI</span>
                <span :class="step >= 2 ? 'text-yellow-400 font-bold drop-shadow-[0_0_8px_rgba(250,204,21,0.6)]' : 'text-gray-600'">{{ currentState.rdi }}</span>
              </div>
              <div class="flex justify-between items-center text-sm">
                <span class="text-red-500 font-bold w-12 text-left uppercase">RSP</span>
                <span class="text-green-500 font-bold">{{ stackData[currentState.rspIdx].addr }}</span>
              </div>
              <div class="flex justify-between items-center text-sm pt-1 border-t border-gray-900 mt-1 font-bold">
                <span class="text-red-500 w-12 text-left uppercase">RIP</span>
                <span class="text-green-400 font-black drop-shadow-[0_0_8px_rgba(74,222,128,0.6)]">{{ currentState.rip }}</span>
              </div>
            </div>
          </div>

          <div class="bg-black/50 p-3 rounded border border-purple-500/20 shadow-inner">
            <div class="text-purple-400 font-bold text-xs mb-2 border-b border-gray-900 italic">DISASM (Next)</div>
            <div class="space-y-1">
              <div v-for="(instr, idx) in ['ret', 'pop rdi', 'ret', 'system']" :key="idx"
                :class="['px-2 py-0.5 rounded transition-all duration-300 border-l-2', 
                         step === idx ? 'bg-indigo-900/40 border-indigo-500 shadow-[0_0_15px_rgba(99,102,241,0.3)]' : 'opacity-20 border-transparent']">
                <span class="text-yellow-400 font-bold">{{ instr }}</span>
              </div>
            </div>
          </div>
        </div>

        <div class="flex-[1.5] bg-[#080808] p-4 rounded border border-gray-800 min-h-[300px] relative shadow-[inset_0_0_20px_rgba(0,0,0,1)]">
          <div class="text-cyan-500 font-bold text-xs mb-4 italic tracking-widest">STACK VIEW</div>
          
          <div class="relative border-x-2 border-b-2 border-gray-800 mx-auto w-[22rem]">
            <div v-for="(item, i) in stackData" :key="i" 
                 :class="['h-14 border-t border-gray-800 flex items-center px-4 transition-all duration-700', 
                          i < currentState.rspIdx ? 'opacity-5 blur-[1px] translate-y-[-5px]' : 'bg-[#121212]']">
              <div class="text-[11px] text-gray-600 mr-4 tabular-nums w-24 text-left">{{ item.addr }}</div>
              <div class="flex flex-col">
                <span :class="['text-sm font-bold tabular-nums transition-colors', i === currentState.rspIdx ? 'text-white drop-shadow-[0_0_5px_rgba(255,255,255,0.5)]' : 'text-gray-500']">{{ item.val }}</span>
                <span class="text-[10px] text-blue-400/80 font-medium">← {{ item.label }}</span>
              </div>
            </div>

            <div class="absolute -left-16 w-14 transition-all duration-500 cubic-bezier(0.34, 1.56, 0.64, 1)" 
                 :style="{ top: (currentState.rspIdx * 56 + 18) + 'px' }">
              <div class="text-red-500 font-black text-xs flex items-center group">
                <span class="animate-pulse drop-shadow-[0_0_8px_rgba(239,68,68,0.8)] font-black">RSP</span>
                <span class="ml-1 text-xl translate-y-[-1px] drop-shadow-[0_0_8px_rgba(239,68,68,0.8)]">➔</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="mt-4 p-3 bg-black/80 border-l-4 border-cyan-500 flex gap-3 items-start rounded-r shadow-[0_0_15px_rgba(6,182,212,0.1)]">
        <span class="text-cyan-500 font-black whitespace-nowrap pt-0.5 uppercase text-xs tracking-tighter">行为：</span>
        <span class="text-gray-200 text-[14px] leading-relaxed italic">{{ currentState.note }}</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
@keyframes gradient-x {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}
.animate-gradient-x {
  background-size: 200% 200%;
  animation: gradient-x 4s ease infinite;
}
.transition-all {
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}
</style>