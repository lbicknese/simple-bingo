<script setup lang="ts">
import { useStorage } from '@vueuse/core'
import { Chance } from 'chance'

class Ball {
  key!: string
  value!: number
}

const chance = new Chance()

const b = [1, 15]
const i = [16, 30]
const n = [31, 45]
const g = [46, 60]
const o = [61, 75]

const balls: Ball[] = []

for (let index = b[0]; index <= b[1]; index++) {
  balls.push({ key: 'B', value: index })
}

for (let index = i[0]; index <= i[1]; index++) {
  balls.push({ key: 'I', value: index })
}

for (let index = n[0]; index <= n[1]; index++) {
  balls.push({ key: 'N', value: index })
}

for (let index = g[0]; index <= g[1]; index++) {
  balls.push({ key: 'G', value: index })
}

for (let index = o[0]; index <= o[1]; index++) {
  balls.push({ key: 'O', value: index })
}

const state = useStorage<{ calls: Ball[]; balls: Ball[] }>('game', {
  calls: [],
  balls: [...balls],
})

function called(ball: Ball) {
  return !!state.value.calls.find((b) => b.key === ball.key && b.value === ball.value)
}

function call() {
  if (state.value.balls.length <= 0) {
    return
  }

  const ball = chance.pickone(state.value.balls)
  state.value = {
    calls: [ball, ...state.value.calls],
    balls: state.value.balls.filter((b) => b.key !== ball.key || b.value !== ball.value),
  }
}

function reset() {
  if (window.confirm('Reset game? This will clear the board')) {
    state.value = {
      calls: [],
      balls: [...balls],
    }
  }
}
</script>

<template>
  <div class="flex flex-wrap gap-1 h-[100vh] p-2">
    <div class="flex gap-0.5 portrait:flex-auto justify-between">
      <div>
        <div class="text-center landscape:text-[5vh]/tight portrait:text-[4vw]/tight">B</div>
        <div class="grid gap-0.5">
          <div
            v-for="ball in balls.filter((b) => b.key === 'B')"
            class="circle ball bg-red-500 opacity-50"
            :class="{
              'opacity-100': called(ball),
            }"
          >
            <span>{{ ball.value }}</span>
          </div>
        </div>
      </div>
      <div>
        <div class="text-center landscape:text-[5vh]/tight portrait:text-[4vw]/tight">I</div>
        <div class="grid gap-0.5">
          <div
            v-for="ball in balls.filter((b) => b.key === 'I')"
            class="circle ball bg-blue-500 opacity-50"
            :class="{
              'opacity-100': called(ball),
            }"
          >
            <span>{{ ball.value }}</span>
          </div>
        </div>
      </div>
      <div>
        <div class="text-center landscape:text-[6vh]/tight portrait:text-[5vw]/tight">N</div>
        <div class="grid gap-0.5">
          <div
            v-for="ball in balls.filter((b) => b.key === 'N')"
            class="circle ball bg-amber-500 opacity-50"
            :class="{
              'opacity-100': called(ball),
            }"
          >
            <span>{{ ball.value }}</span>
          </div>
        </div>
      </div>
      <div>
        <div class="text-center landscape:text-[6vh]/tight portrait:text-[5vw]/tight">G</div>
        <div class="grid gap-0.5">
          <div
            v-for="ball in balls.filter((b) => b.key === 'G')"
            class="circle ball bg-green-500 opacity-50"
            :class="{
              'opacity-100': called(ball),
            }"
          >
            <span>{{ ball.value }}</span>
          </div>
        </div>
      </div>
      <div>
        <div class="text-center landscape:text-[6vh]/tight portrait:text-[5vw]/tight">O</div>
        <div class="grid gap-0.5">
          <div
            v-for="ball in balls.filter((b) => b.key === 'O')"
            class="circle ball bg-purple-500 opacity-50"
            :class="{
              'opacity-100': called(ball),
            }"
          >
            <span>{{ ball.value }}</span>
          </div>
        </div>
      </div>
    </div>
    <div class="flex flex-col gap-0.5 items-center-safe flex-auto">
      <div class="landscape:text-[6dvh]/tight portrait:text-[5dvw]/tight">Last Call</div>
      <div
        class="circle last"
        :class="{
          'bg-red-500': state.calls[0]?.key === 'B',
          'bg-blue-500': state.calls[0]?.key === 'I',
          'bg-amber-500': state.calls[0]?.key === 'N',
          'bg-green-500': state.calls[0]?.key === 'G',
          'bg-purple-500': state.calls[0]?.key === 'O',
        }"
      >
        <span>{{ state.calls[0]?.key }}{{ state.calls[0]?.value }}</span>
      </div>
      <div class="flex gap-1">
        <div
          v-for="ball in state.calls.slice(1, 3)"
          class="circle previous"
          :class="{
            'bg-red-500': ball.key === 'B',
            'bg-blue-500': ball.key === 'I',
            'bg-amber-500': ball.key === 'N',
            'bg-green-500': ball.key === 'G',
            'bg-purple-500': ball.key === 'O',
          }"
        >
          {{ ball.key }}{{ ball.value }}
        </div>
      </div>
      <div class="text-2xl">Calls: {{ state.calls.length }}</div>
      <div class="flex-auto"></div>
      <div class="flex gap-1 w-full text-2xl">
        <button type="button" class="rounded bg-rose-500 px-4 py-1 cursor-pointer" @click="reset">
          Reset
        </button>
        <button
          type="button"
          class="rounded bg-emerald-500 px-4 py-1 cursor-pointer w-full"
          @click="call"
        >
          Call
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.grid {
  display: grid;
  grid-template:
    'ball-1 ball-9'
    'ball-2 ball-10'
    'ball-3 ball-11'
    'ball-4 ball-12'
    'ball-5 ball-13'
    'ball-6 ball-14'
    'ball-7 ball-15'
    'ball-8 .';
}

.circle {
  aspect-ratio: 1/1;
  border-radius: calc(infinity * 1px);
  display: flex;
  justify-content: safe center;
  align-items: safe center;
  line-height: 0;
}

@media (orientation: landscape) {
  .ball {
    width: 10dvh;
    font-size: 5dvh;
  }

  .last {
    width: 20dvw;
    font-size: 10dvw;
  }

  .previous {
    width: 10dvw;
    font-size: 5dvw;
  }
}

@media (orientation: portrait) {
  .ball {
    width: 8dvw;
    font-size: 4dvw;
  }

  .last {
    width: 20dvh;
    font-size: 10dvh;
  }

  .previous {
    width: 10dvh;
    font-size: 5dvh;
  }
}
</style>
