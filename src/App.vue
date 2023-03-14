<script setup>
  import InputTask from './components/InputTask.vue'

  import { ref, onMounted } from 'vue'

  const toBeChanged = ref(false) // set to true when editing title
  const listTitle = ref('Tareas')
  const newListTitle = ref('') // input v-model
  const noTasks = ref(true) // paragraph showing 'No tasks...'
  const allTasks = ref([]) // array with saved tasks
  const newTask = ref('') // input v-model
  const checkbox = ref(false)

  /**
  * Enabling list title edit
  */
  function enableEdit() {
    toBeChanged.value = true
    // Clearing input
    newListTitle.value = ''  
  }

  function cancelEdit() {
    toBeChanged.value = false
  }

  function changeListTitle() {
    // Do not add task if input is empty or only has blank spaces
    if (newListTitle.value.trim() === '') {
      newListTitle.value = ''
      return
    }
    listTitle.value = newListTitle.value.trim()
    toBeChanged.value = false
    // Saving new list title
    window.localStorage.setItem('title', listTitle.value)
  }

  /** 
  * Creating id from written task
  */
  function generateId() {
    return newTask.value.trim().toLowerCase().replace(/ /g, '-')
  }

  function updateStoragedTasks() {
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
    updateStoragedTasks()
    // Clearing input
    newTask.value = ''            
  }

  function deleteTask(index) {
    // On that position, remove 1 element
    allTasks.value.splice(index, 1)
    updateStoragedTasks()
    // In case we delete all tasks
    if (allTasks.value.length === 0) {
      // Show 'There are no tasks...' paragraph
      noTasks.value = true
    }
  }

  function toggleCheckbox(task) {
      task.checked = checkbox.value
      updateStoragedTasks()
  }

  onMounted(() => {
    // Reading localStorage
    const storagedList = JSON.parse(window.localStorage.getItem('list'))
    const storagedTitle = window.localStorage.getItem('title')
    // If there are storaged tasks already
    if (storagedList?.length > 0) {
      // Hiding 'There are no tasks yet' paragraph
      noTasks.value = false
      // Moving the storaged tasks to the array we are going to loop through
      allTasks.value = storagedList
    }
    // If there is a storaged title already
    if (storagedTitle) {
      listTitle.value = storagedTitle
    }
  })
</script>

<template>
  <header class="header">
    <h1 v-if="!toBeChanged">{{ listTitle }}</h1>
    <!-- Showing this div only when editing title -->
    <div v-else>
      <label>
        <!-- Input component -->
        <InputTask          
          forTitle
          inputPlaceholder="Nuevo título"
          v-model="newListTitle"          
        />
      </label>
    </div>
    <div class="header__icons">
      <img 
        v-if="!toBeChanged"
        src="./assets/icons/pencil.png" 
        alt="Editar título de la lista" 
        @click="enableEdit()" 
      />
      <img
        v-if="toBeChanged"
        src="./assets/icons/pencil.png" 
        alt="Editar título de la lista" 
        @click="changeListTitle()"
      />
      <img
        v-if="toBeChanged"
        class="header-icons__last"
        src="./assets/icons/x.png" 
        alt="Cancelar edición" 
        @click="cancelEdit()"
      />
    </div>
  </header>

  <main class="tasks">
    <p 
      v-if="noTasks" 
      class="tasks__message"
    >
      Vaya, esto está vacío. Pero que no cunda el pánico: en cuanto añadas una tarea, se mostrará aquí.
    </p>
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
                src="./assets/icons/checkmarks.png"
                alt="checkmark"
            />
        </div>
        <div class="tasks-item__text-group">
            <!-- Line-through when checked -->
            <span :class="{ done: task.checked }">{{ task.title }}</span>
            <img 
                src="./assets/icons/delete.png"
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
          forTask
          inputPlaceholder="Añade una nueva tarea"
          v-model="newTask"
          @onEnterDoThis="addTask()"
        />
      </label>
    </div>
  </main>
</template>

<style></style>