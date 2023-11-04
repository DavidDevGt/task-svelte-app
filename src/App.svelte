<script>
  // Importa los iconos individuales que necesitas
  import { faPlus } from "@fortawesome/free-solid-svg-icons";
  import { faTrashCan } from "@fortawesome/free-solid-svg-icons";
  import { faXmark } from "@fortawesome/free-solid-svg-icons";
  // Importa el componente FontAwesomeIcon
  import { FontAwesomeIcon } from "@fortawesome/svelte-fontawesome";

  let tasks = [];
  let newTask = "";

  function addTask() {
    if (newTask.trim() !== "") {
      tasks = [...tasks, { id: Date.now(), text: newTask, done: false }];
      newTask = "";
    }
  }

  function removeTask(taskId) {
    tasks = tasks.filter((task) => task.id !== taskId);
  }

  function toggleDone(taskId) {
    tasks = tasks.map((task) =>
      task.id === taskId ? { ...task, done: !task.done } : task
    );
  }

  // Asegúrate de que newText no contenga HTML
  function sanitizeString(str) {
    return str.replace(/<\/?[^>]+(>|$)/g, "");
  }

  // Función para editar la descripción de la tarea
  function editTask(taskId, newText) {
    const sanitizedText = sanitizeString(newText);
    tasks = tasks.map((task) =>
      task.id === taskId && !task.done ? { ...task, text: sanitizedText } : task
    );
  }
  // Función para obtener solo las tareas no completadas
  function clearCompleted() {
    tasks = tasks.filter((task) => !task.done);
  }
  $: isAddButtonDisabled = newTask.trim() === "";
</script>

<main class="main-width">
  <div class="header">
    <h1 class="title-1">Mi Lista de Tareas</h1>
    {#if tasks.length > 0}
      <button class="clear-completed-btn" on:click={clearCompleted}>
        <FontAwesomeIcon icon={faTrashCan} />
      </button>
    {/if}
  </div>
  <div class="input-group">
    <input
      class="task-input"
      type="text"
      bind:value={newTask}
      on:keyup={(event) => event.key === "Enter" && addTask()}
      placeholder="¿Qué necesitas hacer?"
    />
    <button
      class="add-button small-button"
      on:click={addTask}
      disabled={isAddButtonDisabled}
    >
      <FontAwesomeIcon icon={faPlus} />
    </button>
  </div>

  <ul>
    {#each tasks as task (task.id)}
      <li class:done={task.done}>
        <input
          type="checkbox"
          checked={task.done}
          on:change={() => toggleDone(task.id)}
        />
        <!-- Añade una condición al atributo contenteditable -->
        <span
          contenteditable={!task.done}
          on:input={(event) => {
            if (event.target instanceof HTMLElement) {
              editTask(task.id, event.target.textContent);
            }
          }}
          class:non-editable={task.done}
        >
          {task.text}
        </span>
        <button class="delete-task-btn" on:click={() => removeTask(task.id)}>
          <FontAwesomeIcon icon={faXmark} />
        </button>
      </li>
    {/each}
  </ul>
</main>

<style>
  .done {
    text-decoration: line-through;
  }
  /* Estilos adicionales para editar las tareas */
  span[contenteditable] {
    display: inline-block;
    min-width: 100px;
    margin-right: 10px;
    border-bottom: 1px solid #ddd; /* Indica que el elemento es editable */
    word-break: break-word; /* Asegura que las palabras se rompan si es necesario */
  }
  span[contenteditable]:focus {
    outline: none;
    border-bottom: 2px solid blue; /* Destaca el elemento al editarlo */
  }
  /* Estilos para el botón pequeño */
  .small-button {
    padding: 0.4em 0.8em; /* Ajusta el padding para hacer el botón más pequeño */
  }
  /* Estilos para las tareas que no son editables */
  .non-editable {
    pointer-events: none; /* Deshabilita eventos del ratón para evitar la edición */
    border-bottom: none; /* Quita el borde que indica edición */
  }
  .header {
    position: relative;
    padding-right: 40px; /* Asegúrate de tener suficiente espacio para el botón */
  }
  .title-1 {
    width: 100%;
  }

  .clear-completed-btn {
    position: absolute;
    right: 0;
    top: 0;
    padding: 0.5em 1em;
  }

  .delete-task-btn {
    background-color: #ffcccc; /* Rojo claro para el fondo */
    color: #ff0000; /* Rojo para el ícono */
    border: none;
    margin-left: 10px;
    cursor: pointer;
  }
  .delete-task-btn:hover {
    background-color: #ffaaaa; /* Un poco más oscuro al pasar el mouse */
  }

  /* Truncamiento del texto para tareas con mucho texto */
  span {
    display: inline-block;
    max-width: 60%; /* o el máximo que prefieras */
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  /* Deshabilitar el botón si no hay texto en la entrada */
  .add-button:disabled {
    background-color: #ccc;
    cursor: not-allowed;
  }
</style>
