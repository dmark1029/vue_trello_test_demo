<script setup>
import { reactive, computed, onBeforeMount } from 'vue'
import { useStore } from 'vuex'
import axios from 'axios'
import KanbanBoard from './components/kanban/KanbanBoard.vue'
import CloseIcon from '@/components/icons/CloseIcon.vue'
import SaveIcon from '@/components/icons/SaveIcon.vue'

const store = useStore()
const state = reactive({
  is_editing_title: false,
  temp_title: null,
})
const payload = computed(() => {
  return store.getters['vuello/getVuelloDatas']
})

onBeforeMount(async () => {
  const data = store.getters['vuello/getVuelloDatas']
  if (!data) {
    await axios.get('/sample-data.json').then(({ data }) => {
      store.dispatch('vuello/setVuello', data)
    })
  }
})

const handleEditTitle = (type) => {
  if (type === 'edit') {
    state.is_editing_title = true
    state.temp_title = payload.value.title
  } else if (type === 'save') {
    state.is_editing_title = false
    payload.value.last_modified = new Date().toLocaleString('en-GB')
    store.dispatch('vuello/setVuello', payload.value)
  } else {
    state.is_editing_title = false
    payload.value.title = state.temp_title
  }
}
</script>

<template>
  <div v-if="payload" class="flex h-screen flex-col p-4 bg-blue-400">
    <div class="flex justify-between">
      <div>
        <Transition name="fade" mode="out-in">
          <div v-if="!state.is_editing_title">
            <span
              class="rounded-md px-2 text-3xl font-bold transition-all duration-300 ease-in-out hover:cursor-pointer hover:bg-slate-200"
              @click="handleEditTitle('edit')"
            >
              {{ payload.title }}
            </span>
          </div>
          <div v-else class="flex place-items-center">
            <input
              v-model="payload.title"
              type="text"
              class="block w-[230px] rounded-lg border border-gray-300 bg-gray-50 p-2 text-xl text-gray-900 transition duration-300 ease-in-out focus:border-blue-500 focus:ring-blue-500"
              placeholder="Add Board Title"
              @keypress.enter="handleEditTitle('save')"
            />
            <div class="ml-2 flex place-items-center justify-center">
              <SaveIcon
                height="30px"
                class="mr-2 cursor-pointer rounded-full bg-blue-500 p-1 text-white hover:bg-blue-700"
                @click="handleEditTitle('save')"
              />
              <CloseIcon
                height="30px"
                class="cursor-pointer rounded-full p-1 text-red-500 hover:bg-red-600 hover:text-white"
                @click="handleEditTitle('cancel')"
              />
            </div>
          </div>
        </Transition>
        <h3 class="my-2 px-2 text-sm">
          Last Modified : {{ payload.last_modified }}
        </h3>
      </div>
    </div>
    <KanbanBoard
      :payload="payload"
    />
  </div>
  <!-- Container Modal -->
</template>
