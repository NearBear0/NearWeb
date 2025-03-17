<template>
  <div class="min-h-screen bg-gray-50">
    <header class="bg-white shadow">
      <div class="max-w-7xl mx-auto py-6 px-4">
        <h1 class="text-3xl font-bold text-gray-900">Draggable Modal Demo</h1>
      </div>
    </header>
    
    <main class="flex-grow container mx-auto p-4">
      <h2 class="text-xl font-semibold mb-6">Button Customization Options</h2>
      
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        <!-- Using custom button slot -->
        <div class="bg-white p-6 rounded-lg shadow-md">
          <h3 class="text-lg font-medium mb-4">Custom Button Slot</h3>
          <p class="text-gray-600 mb-4">Complete control with custom button slot</p>
          
          <DraggableModal>
            <template #content-1>
              <div class="h-full flex flex-col bg-slate-900 text-white p-4">
                <h2 class="text-xl font-semibold mb-4">Custom Button Slot Example</h2>
                <p class="mb-4">This modal is opened by a fully custom button.</p>
                <div class="bg-slate-800 p-4 rounded mb-4 flex-grow flex items-center justify-center">
                  <p class="text-slate-300 text-center">Custom button with complete control over markup and styling.</p>
                </div>
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
                {{ isOpen ? 'Already Open' : 'Custom Button Slot' }}
              </button>
            </template>
          </DraggableModal>
        </div>
        
        <!-- Using class-based customization -->
        <div class="bg-white p-6 rounded-lg shadow-md">
          <h3 class="text-lg font-medium mb-4">Class-Based Customization</h3>
          <p class="text-gray-600 mb-4">Customize button with classes via props</p>
          
          <DraggableModal
            :buttonClasses="{
              'content-2': 'font-medium py-2 px-6 rounded-full transition-colors m-2 bg-gradient-to-r from-pink-500 to-purple-500 hover:from-pink-600 hover:to-purple-600 text-white flex items-center gap-2 shadow-md'
            }"
            :buttonTexts="{
              'content-2': 'Class-Based Button'
            }"
          >
            <template #content-2>
              <div class="h-full flex flex-col bg-gradient-to-br from-pink-500 to-purple-600 text-white">
                  <SmileOs2/>
              </div>
            </template>
            
            <!-- Custom icon for the button -->
            <template #button-icon-content-2>
              <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M4 2a2 2 0 00-2 2v11a3 3 0 106 0V4a2 2 0 00-2-2H4zm1 14a1 1 0 100-2 1 1 0 000 2zm5-1.757l4.9-4.9a2 2 0 000-2.828L13.485 5.1a2 2 0 00-2.828 0L10 5.757v8.486zM16 18H9.071l6-6H16a2 2 0 012 2v2a2 2 0 01-2 2z" clip-rule="evenodd" />
              </svg>
            </template>
          </DraggableModal>
        </div>
        
        <!-- Using image as button -->
        <div class="bg-white p-6 rounded-lg shadow-md">
          <h3 class="text-lg font-medium mb-4">Image Button</h3>
          <p class="text-gray-600 mb-4">Use an image as a button</p>
          
          <DraggableModal
            :buttonClasses="{
              'image-content-3': 'cursor-pointer inline-block m-2 transition-transform hover:scale-105'
            }"
          >
            <template #content-3>
              <div class="h-full flex flex-col bg-gradient-to-br from-green-400 to-emerald-600 text-white p-4">
                <h2 class="text-xl font-semibold mb-4">Image Button Example</h2>
                <p class="mb-4">This modal is opened by clicking on an image.</p>
                <div class="bg-white/20 backdrop-blur-sm p-4 rounded mb-4 flex-grow flex items-center justify-center">
                  <p class="text-white text-center">Any image or element can be used as a button.</p>
                </div>
              </div>
            </template>
            
            <!-- Image button -->
            <template #button-image-content-3>
              <div class="relative group">
                <img 
                  src="/src/assets/logo.svg" 
                  alt="Open Modal" 
                  class="img-fluid w-32 h-32 rounded-lg shadow-md"
                />
                <div class="absolute inset-0 flex items-center justify-center bg-black bg-opacity-0 group-hover:bg-opacity-30 transition-all duration-300 rounded-lg">
                  <span class="text-white opacity-0 group-hover:opacity-100 font-medium">Open Modal</span>
                </div>
              </div>
            </template>
          </DraggableModal>
        </div>
        
        <!-- Default button -->
        <div class="bg-white p-6 rounded-lg shadow-md">
          <h3 class="text-lg font-medium mb-4">Default Button</h3>
          <p class="text-gray-600 mb-4">Using the default button style</p>
          
          <DraggableModal>
            <template #content-4>
              <div class="h-full flex flex-col bg-white p-4">
                <h2 class="text-xl font-semibold mb-4">Default Button Example</h2>
                <p class="mb-4">This modal uses the default button style.</p>
                <div class="bg-gray-100 p-4 rounded mb-4 flex-grow flex items-center justify-center">
                  <p class="text-gray-700 text-center">No customization needed for basic usage.</p>
                </div>
              </div>
            </template>
          </DraggableModal>
        </div>
      </div>
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