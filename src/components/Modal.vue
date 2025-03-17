<script setup>
import { ref, defineProps, defineEmits, watch } from "vue";

const props = defineProps({
  modalId: { type: Number, required: true },
  title: { type: String, required: true },
  content: { type: Object, required: true },
  zIndex: { type: Number, required: true },
  width: { type: Number, required: true },
  height: { type: Number, required: true },
  posX: { type: Number, required: true },
  posY: { type: Number, required: true }
});

const emit = defineEmits(["close", "minimize", "focus"]);

// Estado del modal
const isMaximized = ref(false);
const modalWidth = ref(props.width);
const modalHeight = ref(props.height);
const posX = ref(props.posX);
const posY = ref(props.posY);
const initialState = ref({ width: props.width, height: props.height, x: props.posX, y: props.posY });

// Llevar modal al frente
const bringToFront = () => emit("focus", props.modalId);

// Maximizar y restaurar modal
const toggleMaximize = () => {
  if (isMaximized.value) {
    modalWidth.value = initialState.value.width;
    modalHeight.value = initialState.value.height;
    posX.value = initialState.value.x;
    posY.value = initialState.value.y;
  } else {
    initialState.value = { width: modalWidth.value, height: modalHeight.value, x: posX.value, y: posY.value };
    modalWidth.value = window.innerWidth;
    modalHeight.value = window.innerHeight;
    posX.value = 0;
    posY.value = 0;
  }
  isMaximized.value = !isMaximized.value;
};

// Mover modal
const startDrag = (event) => {
  if (isMaximized.value) return;
  const startX = event.clientX - posX.value;
  const startY = event.clientY - posY.value;

  const onMouseMove = (e) => {
    let newX = e.clientX - startX;
    let newY = e.clientY - startY;

    // Restringir dentro de la pantalla
    newX = Math.max(0, Math.min(window.innerWidth - modalWidth.value, newX));
    newY = Math.max(0, Math.min(window.innerHeight - modalHeight.value, newY));

    posX.value = newX;
    posY.value = newY;
  };

  const onMouseUp = () => {
    window.removeEventListener("mousemove", onMouseMove);
    window.removeEventListener("mouseup", onMouseUp);
  };

  window.addEventListener("mousemove", onMouseMove);
  window.addEventListener("mouseup", onMouseUp);
};

// Redimensionar modal
const startResize = (event) => {
  event.stopPropagation();
  const startWidth = modalWidth.value;
  const startHeight = modalHeight.value;
  const startX = event.clientX;
  const startY = event.clientY;

  const onMouseMove = (e) => {
    modalWidth.value = Math.max(300, startWidth + (e.clientX - startX));
    modalHeight.value = Math.max(200, startHeight + (e.clientY - startY));
  };

  const onMouseUp = () => {
    window.removeEventListener("mousemove", onMouseMove);
    window.removeEventListener("mouseup", onMouseUp);
  };

  window.addEventListener("mousemove", onMouseMove);
  window.addEventListener("mouseup", onMouseUp);
};
</script>

<template>
  <div 
    class="fixed bg-white rounded-lg shadow-lg flex flex-col overflow-hidden"
    :style="{ width: modalWidth + 'px', height: modalHeight + 'px', top: posY + 'px', left: posX + 'px', zIndex }"
    @mousedown="bringToFront"
  >
    <div class="flex justify-between items-center bg-slate-400 text-white cursor-grab" @mousedown="startDrag">
      <h2 class="ml-5 select-none">{{ title }}</h2>
      <div>
        <button @click="emit('minimize', { id: modalId, title, content })" class="text-white p-2 px-4 hover:bg-slate-500 rounded invisible md:visible">🗕</button>
        <button @click="toggleMaximize" class="text-white p-2 px-4 hover:bg-slate-500 rounded invisible md:visible">🗖</button>
        <button @click="emit('close')" class="text-white p-2 px-4 hover:bg-red-400 rounded">✕</button>
      </div>
    </div>

    <div class="flex-grow h-full w-full flex">
        <component :is="content" class="w-full h-full" />
    </div>


    <div class="absolute bottom-0 right-0 w-4 h-4 bg-gray-400 cursor-se-resize" @mousedown="startResize"></div>
  </div>
</template>
