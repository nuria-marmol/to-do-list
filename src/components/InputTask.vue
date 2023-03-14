<script setup>
 defineProps({
    modelValue: String
 })
</script>

<template>
    <input
        type="text"
        class="input"
        placeholder="Añade una nueva tarea"
        :value="modelValue"
        @input="$emit('update:modelValue', $event.target.value)"
        @keyup.enter="$emit('onEnterDoThis')"
    />
</template>

<style></style>