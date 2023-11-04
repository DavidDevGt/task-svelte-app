<script>
  // Importar
  import Swal from "sweetalert2";
  import {
    faPlus,
    faTrashCan,
    faXmark,
  } from "@fortawesome/free-solid-svg-icons";
  import { FontAwesomeIcon } from "@fortawesome/svelte-fontawesome";

  let tasks = [];
  let newTask = "";
  const maxTaskLength = 50; // Establecer un máximo para la longitud de la tarea

  // Función para agregar tareas
  function addTask() {
    if (newTask.trim() === "") {
      Swal.fire({
        icon: "error",
        title: "Oops...",
        text: "La tarea no puede estar vacía!",
      });
      return;
    } else if (newTask.trim().length > maxTaskLength) {
      Swal.fire({
        icon: "warning",
        title: "Tarea demasiado larga",
        text: `La tarea no puede exceder los ${maxTaskLength} caracteres.`,
      });
      return;
    }

    tasks = [...tasks, { id: Date.now(), text: newTask.trim(), done: false }];
    newTask = "";
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
    if (sanitizedText.length > maxTaskLength) {
      Swal.fire({
        icon: "error",
        title: "Error",
        text: `La tarea no puede exceder los ${maxTaskLength} caracteres.`,
      });
      // Encuentra la tarea actual y reestablece su valor a la versión no editada
      const currentTask = tasks.find((task) => task.id === taskId);
      if (currentTask) {
        currentTask.text = currentTask.text.substring(0, maxTaskLength);
      }
      return;
    }

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
              editTask(task.id, event.target.innerText);
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
  /* Variables de colores y fuentes */
  :root {
    --font-main: Inter, system-ui, sans-serif;
    --color-text: #e0e0e0;
    --color-bg: #333;
    --color-link: #646cff;
    --color-link-hover: #535bf2;
    --color-btn: #1a1a1a;
    --color-btn-hover: #646cff;
    --color-task: #1c1c1c;
    --color-delete-btn: #ff6347;
    --color-delete-hover: #ffaaaa;
    --color-disabled: #858585;
  }

  /* Título principal */
  .title-1 {
    font-size: 2rem;
    margin: 1rem 0;
  }

  /* Botones genéricos */
  button {
    padding: 0.5em 1em;
    font-family: var(--font-main);
    cursor: pointer;
    border: none;
    border-radius: 5px;
    transition: background-color 0.25s;
  }

  /* Botón para borrar tareas completadas */
  .clear-completed-btn {
    position: absolute;
    right: 0;
    top: 0;
    background: none;
    color: var(--color-delete-btn);
  }

  /* Input de tareas y botón de añadir */
  .input-group {
    margin-right: 0.5rem;
    display: flex;
    gap: 0.5rem;
    margin-bottom: 1rem;
  }

  .task-input {
    color: #000;
    flex-grow: 1;
    border: 1px solid #ddd;
    border-radius: 4px;
    padding: 0.5em;
  }

  /* Botón para añadir tarea */
  .add-button {
    background-color: var(--color-link);
    color: white;
  }

  .add-button:disabled {
    background-color: var(--color-disabled);
    cursor: not-allowed;
  }

  /* Lista de tareas */
  ul {
    list-style: none;
    padding: 0;
    margin: 0;
  }

  li {
    background: var(--color-task);
    margin-bottom: 0.5rem;
    padding: 0.5rem;
    border-radius: 5px;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  /* Tarea completada */
  .done {
    text-decoration: line-through;
    opacity: 0.5;
  }

  /* Editable y no editable */
  span[contenteditable] {
    border-bottom: 1px solid #ddd;
  }

  span[contenteditable]:focus {
    border-bottom: 2px solid blue;
  }

  .non-editable {
    pointer-events: none;
    border-bottom: none;
  }

  /* Botón de eliminar tarea */
  .delete-task-btn {
    background-color: var(--color-delete-btn);
    color: white;
  }

  .delete-task-btn:hover {
    background-color: var(--color-delete-hover);
  }

  /* Texto de la tarea */
  span {
    max-width: 60%;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  /* Media queries para tema claro y oscuro */
  @media (prefers-color-scheme: light) {
    :root {
      --color-text: #213547;
      --color-bg: #ffffff;
      --color-btn: #f9f9f9;
      --color-link-hover: #747bff;
    }
  }

  /* Media queries para responsividad */
  @media (max-width: 768px) {
    .task-input {
      width: auto;
    }
  }
</style>
