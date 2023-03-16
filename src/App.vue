<script setup>
  import InputComponent from './components/InputComponent.vue'

  import { ref, onMounted } from 'vue'

  const toBeChanged = ref(false) // set to true when editing title
  const listTitle = ref('Tareas')
  const newListTitle = ref('') // input v-model
  const noTasks = ref(true) // paragraph showing 'No tasks...'
  const zeroPending = ref(true)
  const zeroCompleted = ref(false)
  const allTasks = ref([]) // array with saved tasks
  const showChecked = ref(false) // for filtering
  const showAll = ref(true) // for filtering
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

  function generateId() {
    // Milliseconds since 1st January 1970
    return Date.now()
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

  function deleteTask(task) {
    // Avoid deleting wrong position when seeing Pending or Completed
    const realIndex = allTasks.value.indexOf(task)
    // On that position, remove 1 element
    allTasks.value.splice(realIndex, 1)
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

  /**
  * Enabling filtering by checked tasks
  */
  function enableShowChecked() {
    showChecked.value = true
    showAll.value = false
  }

  /**
  * Enabling filtering by pending tasks
  */
  function enableShowPending() {
    showChecked.value = false
    showAll.value = false
  }

  function enableShowAll() {
    showAll.value = true
    showChecked.value = false
  }

  // Function in v-for
  function filteredTasks() {  
    // When the user wants to filter by checked tasks
    if (showChecked.value) {    
      const checkedTasks = allTasks.value.filter(function (task) {
        return task.checked
      })

      if (checkedTasks.length === 0) {
        zeroPending.value = false
        zeroCompleted.value = true
      }

      // Avoid showing 'No pending tasks' paragraph
      zeroPending.value = false
      return checkedTasks

    // Filter by pending tasks
    } else if (!showChecked.value && !showAll.value) {
      const pendingTasks = allTasks.value.filter(function (task) {
        return !task.checked
      })

      if (pendingTasks.length === 0) {
        zeroCompleted.value = false
        zeroPending.value = true
      }
      
      // Avoid showing 'No completed tasks' paragraph
      zeroCompleted.value = false
      return pendingTasks
    }
      // For showing all tasks
      zeroCompleted.value = false
      zeroPending.value = false
      return allTasks.value
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
        <InputComponent          
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

    <menu v-if="!noTasks" class="tasks__filter">
      <li 
        class="tasks-filter__item" 
        :class="{ 'tasks-filter__item--active': showAll }"
        @click="enableShowAll()"
      >Todas</li>
      <li>|</li>
      <li 
        class="tasks-filter__item" 
        :class="{ 'tasks-filter__item--active': !showChecked && !showAll }"
        @click="enableShowPending()"
      >Pendientes</li>
      <li>|</li>
      <li 
        class="tasks-filter__item" 
        :class="{ 'tasks-filter__item--active': showChecked }"
        @click="enableShowChecked"
      >Completadas</li>
    </menu>

    <menu v-if="!noTasks" class="tasks__list">
      <li 
        v-if="zeroPending"
        class="tasks__message"
      >
        No tienes tareas pendientes.
      </li>
      <li 
        v-if="zeroCompleted"
        class="tasks__message"
      >
        No tienes tareas completadas.
      </li>       
      <li
        v-for="(task, index) in filteredTasks()"
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
                @click="deleteTask(task)"
            />
        </div>
      </li>      
    </menu>

    <div>   
      <label>
        <!-- Input component -->
        <InputComponent
          forTask
          inputPlaceholder="Escribe una tarea y pulsa Enter"
          v-model="newTask"
          @onEnterDoThis="addTask()"
        />
      </label>
    </div>
  </main>
</template>

<style></style>