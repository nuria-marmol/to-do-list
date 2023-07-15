<script setup>
defineProps({    
    forTask: {
        type: Boolean,
        default: false
    },
    forTitle: {
        type: Boolean,
        default: false
    },
    inputPlaceholder: String,
    modelValue: String
})
</script>

<template>
    <input        
        type="text"
        name="task-to-do"
        class="input"
        :class="{ 'input--task': forTask, 'input--title': forTitle }"
        :placeholder="inputPlaceholder"
        :value="modelValue"
        @input="$emit('update:modelValue', $event.target.value)"
        @keyup.enter="$emit('onEnterDoThis')"
    />
</template>

<style></style>