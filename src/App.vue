<template>
  <div class="min-h-screen bg-slate-300">

    <nav class="w-full bg-white shadow p-4">
        <div class="container mx-auto flex items-center">
            <div class="w-12 h-12 bg-gray-300"></div> <!-- Espacio para el logo -->
            <h1 class="text-3xl font-bold text-gray-900 ml-4">Nearweb</h1>
        </div>
    </nav>
    
    <main class="flex-grow container mx-auto p-4 grid grid-cols-3 md:grid-cols-6 lg:grid-cols-8 gap-4">
      
      <DraggableModal>
            <template #content-1>
              <div class="h-full flex flex-col bg-slate-300 text-white">
                  <SmileOs2/>
              </div>
            </template>
            
            <!-- Custom button slot -->
            <template #button-content-1="{ open, isOpen }">
              <button
                @click="open"
                :disabled="isOpen"
                class="flex items-center gap-2 bg-slate-800 hover:bg-slate-700 text-white px-4 py-2 rounded-lg shadow-md transition-colors disabled:opacity-50 disabled:cursor-not-allowed"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                  <path d="M10 12a2 2 0 100-4 2 2 0 000 4z" />
                  <path fill-rule="evenodd" d="M.458 10C1.732 5.943 5.522 3 10 3s8.268 2.943 9.542 7c-1.274 4.057-5.064 7-9.542 7-4.478 0-8.268-2.943-9.542-7z" clip-rule="evenodd" />
                </svg>
                {{ isOpen ? 'Abierto...' : 'SmileOs 2.0' }}
              </button>
            </template>
        </DraggableModal>
    </main>
  </div>
</template>

<script>
import DraggableModal from './components/DraggableModal.vue'
import SmileOs2 from './components/SmileOs2.vue';

export default {
  name: 'App',
  components: {
    DraggableModal,
    SmileOs2
  }
}
</script>