<script setup lang="ts">
import { useStorage } from '@vueuse/core'
import { Chance } from 'chance'
import { onMounted, ref, useTemplateRef } from 'vue'

const chance = new Chance()

// const socket = new WebSocket('ws://localhost:8080')

let identifier = localStorage.getItem('identifier')
if (identifier === null) {
  identifier = chance.guid()
  localStorage.setItem('identifier', identifier)
}

class Ball {
  key!: string
  value!: number
}

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

const state = useStorage<{ calls: Ball[]; balls: Ball[]; room: string | undefined }>('game', {
  calls: [],
  balls: [...balls],
  room: undefined,
})

const joinRoomDialog = useTemplateRef<HTMLDialogElement>('joinRoomDialog')

const newRoomName = ref<string>('')

const owner = ref<boolean>(false)

function called(ball: Ball) {
  return !!state.value.calls.find((b) => b.key === ball.key && b.value === ball.value)
}

function call() {
  if (state.value.balls.length <= 0) {
    return
  }

  const ball = chance.pickone(state.value.balls)
  state.value = {
    ...state.value,
    calls: [ball, ...state.value.calls],
    balls: state.value.balls.filter((b) => b.key !== ball.key || b.value !== ball.value),
  }

  // socket.send(JSON.stringify({ type: 'update', identifier, payload: state.value }))
}

function reset() {
  if (window.confirm('Reset game? This will clear the board.')) {
    _reset()
    // socket.send(JSON.stringify({ type: 'update', identifier, payload: state.value }))
  }
}

function _reset() {
  state.value = {
    ...state.value,
    calls: [],
    balls: [...balls],
  }
}

function newRoom() {
  if (window.confirm('Create a new room? This will clear the board.')) {
    // socket.send(JSON.stringify({ type: 'leave', identifier, payload: state.value }))
    state.value.room = chance.radio()
    _reset()
    // socket.send(JSON.stringify({ type: 'create', identifier, payload: state.value }))
  }
}

function openJoinRoomDialog() {
  joinRoomDialog.value?.showModal()
}

function joinRoom() {
  if (newRoomName.value.trim().length === 0) {
    return
  }
  owner.value = false
  // socket.send(JSON.stringify({ type: 'leave', identifier, payload: state.value }))
  state.value.room = newRoomName.value.toLocaleUpperCase()
  newRoomName.value = ''
  joinRoomDialog.value?.close()
  // socket.send(JSON.stringify({ type: 'join', identifier, payload: state.value }))
}

function cancelJoinRoom() {
  newRoomName.value = ''
  joinRoomDialog.value?.close()
}

onMounted(() => {
  if (state.value.room === undefined || state.value.room.trim() === '') {
    state.value.room = chance.radio()
  }

  // Connection opened
  // socket.addEventListener('open', (event) => {
  //   socket.send(JSON.stringify({ type: 'create', identifier, payload: state.value }))
  // })

  // Listen for messages
  // socket.addEventListener('message', (event) => {
  //   const data = JSON.parse(event.data)

  //   if (data.type === 'join') {
  //     socket.send(JSON.stringify({ type: 'update', identifier, payload: state.value }))
  //   } else if (data.type === 'update') {
  //     state.value = data.payload
  //   } else if (data.type === 'identify') {
  //     owner.value = true
  //   }
  // })
})
</script>

<template>
  <div class="flex flex-wrap gap-1 h-[100vh] p-2">
    <div class="flex gap-0.5 portrait:flex-auto justify-between">
      <div>
        <div class="text-center landscape:text-[5vh]/tight portrait:text-[4vw]/tight">B</div>
        <div class="grid gap-0.5">
          <div
            v-for="ball in balls.filter((b) => b.key === 'B')"
            :key="`${ball.key}-${ball.value}`"
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
            :key="`${ball.key}-${ball.value}`"
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
            :key="`${ball.key}-${ball.value}`"
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
            :key="`${ball.key}-${ball.value}`"
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
            :key="`${ball.key}-${ball.value}`"
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
      <div class="text-lg flex gap-1 items-center">
        <span>Room: {{ state.room }}</span>
        <button
          type="button"
          class="circle bg-gray-400 size-8 cursor-pointer"
          aria-label="Join a Room"
          @click="openJoinRoomDialog"
        >
          <span class="text-2xl/0">&#9901;</span>
        </button>
        <button
          type="button"
          class="circle bg-rose-400 size-8 cursor-pointer"
          aria-label="New Room"
          @click="newRoom"
        >
          <span class="text-2xl/0">&plus;</span>
        </button>
      </div>
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
      <TransitionGroup class="flex gap-1" name="list" tag="div">
        <div
          v-for="ball in state.calls.slice(1, 3)"
          :key="`${ball.key}-${ball.value}`"
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
      </TransitionGroup>
      <div class="text-xl">Calls: {{ state.calls.length }}</div>
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
  <dialog ref="joinRoomDialog" class="fixed m-auto rounded p-2">
    <div class="flex flex-col gap-1">
      <h2 class="text-xl">Join a Room</h2>
      <div class="flex gap-1">
        <label for="roomName">Room Name</label>
        <input
          type="text"
          name="roomName"
          class="outline p-1 outline-gray-600 rounded"
          v-model="newRoomName"
        />
      </div>
      <div class="flex">
        <button type="button" class="border border-gray-600 rounded px-2" @click="cancelJoinRoom">
          Cancel
        </button>
        <div class="flex-auto"></div>
        <button type="button" class="bg-emerald-500 rounded px-2" @click="joinRoom">Join</button>
      </div>
    </div>
  </dialog>
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
  transition: all 0.6s ease-out;
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
    width: 8dvw;
    font-size: 4dvw;
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

.list-enter-active.list-leave-active {
  transition: all 0.6s ease-out;
}
.list-leave-active {
  transition: all 0.6s ease-out;
}
.list-enter-from {
  opacity: 0;
  transform: translateY(-30px);
}

.list-leave-to {
  opacity: 0;
  width: 0;
  overflow: hidden;
  transform: translateX(30px);
}

#joinRoomDialog {
  position: fixed;
  margin: auto;
}
</style>
