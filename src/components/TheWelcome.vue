<template>
  <div>
    <h3>{{ title }}</h3>
    <button @click="$emit('close')">Close Modal</button>
    <button @click="saveAndClose">Save</button>
  </div>
</template>

<script setup>
const emit = defineEmits(['close', 'save'])

const saveAndClose = () => {
  // Do something
  emit('save', { result: 'success' })
  emit('close') // This will close the modal
}
</script>