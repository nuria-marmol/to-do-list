<script setup>
  import InputTask from './components/InputTask.vue'

  import { ref, onMounted, computed } from 'vue'

  const noTasks = ref(true) // paragraph showing 'No tasks...'
  const allTasks = ref([]) // array with saved tasks
  const newTask = ref('') // input v-model
  const checkbox = ref(false)

  function toggleCheckbox(task) {
      task.checked = checkbox.value
      updateLocalStorage()
  }

  /** 
  * Creating id from written task
  */
  function generateId() {
    return newTask.value.trim().toLowerCase().replace(/ /g, '-')
  }

  function updateLocalStorage() {
    window.localStorage.setItem('list', JSON.stringify(allTasks.value))
  }

  function addTask(task) {
    // Do not add task if input is empty or only has blank spaces
    if (newTask.value.trim() === '') {
      newTask.value = ''
      return
    }
    // Hiding the paragraph 'No tienes tareas...'
    noTasks.value = false
    // Adding the task to the list
    allTasks.value.push({
      id: generateId(),
      title: newTask.value.trim(), // input v-model
      checked: false
    })
    updateLocalStorage()
    // Clearing input
    newTask.value = ''            
  }

  function deleteTask(index) {
    // On that position, remove 1 element
    allTasks.value.splice(index, 1)
    updateLocalStorage()
    // In case we delete all tasks
    if (allTasks.value.length === 0) {
      // Show 'There are no tasks...' paragraph
      noTasks.value = true
    }
  }

  onMounted(() => {
    // Reading localStorage
    const storagedList = JSON.parse(window.localStorage.getItem('list'))
    // If there are storaged tasks already
    if (storagedList?.length > 0) {
      // Hiding 'There are no tasks yet' paragraph
      noTasks.value = false
      // Moving the storaged tasks to the array we are going to loop through
      allTasks.value = storagedList
    }
  })
</script>

<template>
  <header class="header">
    <h1>Tareas</h1>
  </header>

  <main class="tasks">
    <p v-if="noTasks" class="tasks__message">Vaya, parece que esto está vacío.</p>
    <ul v-else class="tasks__list">            
      <li 
        v-for="(task, index) in allTasks"
        v-bind:key="task.id"
        class="tasks__item"
      >
        <div>
            <input
                type="checkbox"
                :id="task.id"
                class="tasks-item__checkbox"                 
                v-model="checkbox"
                @change="toggleCheckbox(task)"
            />

            <!-- Changing background colour when checked -->
            <div 
                class="tasks-item__custom-checkbox"
                :class="{'tasks-item__custom-checkbox--checked': task.checked }"
            />

            <!-- Showing when checked -->
            <img           
                class="tasks-item__checkmark"
                :class="{ 'tasks-item__checkmark--show': task.checked }"       
                src="./assets/checkmarks.png"
                alt="checkmark"
            />
        </div>
        <div class="tasks-item__text-group">

            <!-- Line-through when checked -->
            <span :class="{ done: task.checked }">{{ task.title }}</span>
            <img 
                src="./assets/delete.png"
                alt="Eliminar tarea"
                @click="deleteTask(index)"
            />
        </div>
      </li>      
    </ul>

    <div>   
      <label>

        <!-- Input component -->
        <InputTask
          v-model="newTask"
          @onEnterDoThis="addTask()"
        />
      </label>
    </div>
  </main>
</template>

<style></style>