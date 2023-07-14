<script setup>
  import InputText from './components/InputText.vue'

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
  const draggedItemIndex = ref(null) // for drag and drop

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

  /**
  * Adding new task to the list
  */ 
  function addTask() {
    // Do not add task if input is empty or only has blank spaces
    if (newTask.value.trim() === '') {
      newTask.value = ''
    }
    // Hiding the paragraph 'No tienes tareas...'
    noTasks.value = false
    // Adding the task to the list
    allTasks.value.push({
      id: generateId(),
      title: newTask.value.trim(), // input v-model
      checked: false,
      enterAnimation: true,
      leaveAnimation: false
    })
    updateStoragedTasks()
    // Clearing input
    newTask.value = ''
    // Showing All tasks again if the task has been added when seeing Completed
    if (showChecked.value) {
      enableShowAll()
    }        
  }

  function deleteTask(task) {  
     task.enterAnimation = false
     task.leaveAnimation = true
    // Avoid deleting wrong position when seeing Pending or Completed
    const realIndex = allTasks.value.indexOf(task)
    // Wait for the enter animation to end
    setTimeout(function () {
      // On that position, remove 1 element
      allTasks.value.splice(realIndex, 1)
      updateStoragedTasks()
      // In case we delete all tasks
      if (allTasks.value.length === 0) {
        // Show 'There are no tasks...' paragraph
        noTasks.value = true
      }
    }, 500)
  }  

  /**
  * For seeing animation only when manually adding or deleting a task
  * (not when filtering or reading localStorage)
  */
  function preventAutomaticAnimation() {
    allTasks.value.forEach(function (task) {
      task.enterAnimation = false
    })
  }

  /**
  * Enabling filtering by checked tasks
  */
  function enableShowChecked() {
    preventAutomaticAnimation()
    showChecked.value = true
    showAll.value = false
  }

  /**
  * Enabling filtering by pending tasks
  */
  function enableShowPending() {
    preventAutomaticAnimation()
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

  // ------ Drag and drop functions ------
  /**
  * Getting the index of the dragged task on dragstart
  */
  function getDraggedItemIndex(task) {
    // Avoid getting wrong index when seeing Pending or Completed
    draggedItemIndex.value = allTasks.value.indexOf(task)
  }

  /**
  * Changing the order of the dragged task on draggover
  */
  function reorderItems(task) {    
    // Avoid taking wrong position when seeing Pending or Completed
    const realIndex = allTasks.value.indexOf(task)
    // If the user is moving the task on top of another one
    if (realIndex !== draggedItemIndex.value) {
      const draggedItem = allTasks.value[draggedItemIndex.value];
      // Deleting the dragged item on its initial position
      allTasks.value.splice(draggedItemIndex.value, 1);
      // Adding the dragged item on new item's position
      allTasks.value.splice(realIndex, 0, draggedItem);
      /* Now the dragged item should have
      the index of the item that it has replaced */
      draggedItemIndex.value = realIndex;
    }
  }

  /**
  * Resetting the index of the dragged task on dragend
  */
  function resetDraggedItemIndex() {
    draggedItemIndex.value = null
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
      preventAutomaticAnimation()
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
        <InputText          
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

    <div v-if="!noTasks" class="tasks__list">
      <p 
        v-if="zeroPending"
        class="tasks__message"
      >
        No tienes tareas pendientes.
      </p>
      <p 
        v-if="zeroCompleted"
        class="tasks__message"
      >
        No tienes tareas completadas.
      </p>

      <!-- Draggable tasks -->
      <menu>
        <li
          v-for="task in filteredTasks()"
          v-bind:key="task.id"
          class="tasks__item"
          :class="{ 'tasks__item--show': task.enterAnimation, 'tasks__item--hide': task.leaveAnimation }"          
          draggable="true"
          @dragstart="getDraggedItemIndex(task)"            
          @dragover.prevent="reorderItems(task)"                              
          @dragend="resetDraggedItemIndex"
          @drop="updateStoragedTasks"
        >
          <div>
            <label>
              <input
                type="checkbox"
                :id="task.id"
                class="tasks-item__checkbox"                
                v-model="task.checked"
                @change="updateStoragedTasks"     
              />   
            </label>         
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
    </div>

    <div>   
      <label>
        <!-- Input component -->
        <InputText
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