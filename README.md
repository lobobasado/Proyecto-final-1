# Proyecto-final-1
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Gestor de Tareas Grupales con Ruleta</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f0f0f0;
      padding: 20px;
      text-align: center;
    }

    .formulario, .login {
      margin-bottom: 20px;
      display: flex;
      flex-direction: column;
      align-items: center;
    }

    input, select {
      padding: 10px;
      margin: 5px;
      width: 90%;
      max-width: 300px;
    }

    button {
      padding: 10px 15px;
      background-color: #00a86b;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      margin: 5px;
    }

    ul {
      list-style-type: none;
      padding: 0;
    }

    li {
      background: #fff;
      margin: 10px auto;
      padding: 10px;
      width: 95%;
      border-radius: 8px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
      text-align: left;
    }

    #estudiantesList {
      margin-top: 10px;
      font-style: italic;
    }

    .admin-only {
      display: none;
    }

    @media (max-width: 600px) {
      input {
        width: 90%;
      }
      li {
        width: 95%;
      }
    }
  </style>
</head>
<body>
  <h1>Gestor de Tareas Grupales con Ruleta</h1>

  <div class="login" id="loginForm">
    <input type="password" id="pin" placeholder="Ingresa tu PIN (4 dígitos)" maxlength="4">
    <input type="text" id="nombreUsuario" placeholder="ingresa tu nombre (estudiante)">
    <button onclick="verificarPIN()">Ingresar</button>
  </div>

  <div class="formulario admin-only" id="adminForm">
    <input type="text" id="nuevoEstudiante" placeholder="Nombre del estudiante">
    <button onclick="agregarEstudiante()">Agregar Estudiante</button>

    <input type="text" id="tarea" placeholder="Descripción de la tarea">
    <button onclick="asignarTarea()">Girar Ruleta y Asignar</button>
  </div>

  <div id="estudiantesList"></div>
  <ul id="listaTareas"></ul>

  <script>
    let estudiantes = [];
    let tareas = [];
    let rolActual = "";
    let usuarioActual = "";

    const PIN_USUARIO = "1234";
    const PIN_ADMIN = "0101";

    const lista = document.getElementById("listaTareas");
    const estudiantesDiv = document.getElementById("estudiantesList");

    // Cargar desde localStorage
    window.onload = () => {
      estudiantes = JSON.parse(localStorage.getItem("estudiantes")) || [];
      tareas = JSON.parse(localStorage.getItem("tareas")) || [];
      actualizarListaEstudiantes();
    };

    function guardarEnStorage() {
      localStorage.setItem("estudiantes", JSON.stringify(estudiantes));
      localStorage.setItem("tareas", JSON.stringify(tareas));
    }

    function verificarPIN() {
      const pin = document.getElementById("pin").value;
      const nombre = document.getElementById("nombreUsuario").value.trim();

      if (pin === PIN_ADMIN) {
        rolActual = "admin";
        document.getElementById("loginForm").style.display = "none";
        document.querySelectorAll(".admin-only").forEach(e => e.style.display = "flex");
        actualizarListaEstudiantes();
        mostrarTareasGuardadas();
      } else if (pin === PIN_USUARIO && nombre) {
        rolActual = "usuario";
        usuarioActual = nombre;
        document.getElementById("loginForm").style.display = "none";
        actualizarListaEstudiantes();
        mostrarTareasGuardadas();
      } else {
        alert("PIN incorrecto o falta el nombre.");
      }
    }

    function agregarEstudiante() {
      const nombre = document.getElementById("nuevoEstudiante").value.trim();
      if (nombre) {
        estudiantes.push(nombre);
        actualizarListaEstudiantes();
        document.getElementById("nuevoEstudiante").value = "";
        guardarEnStorage();
      }
    }

    function actualizarListaEstudiantes() {
      if (rolActual === "admin") {
        if (estudiantes.length === 0) {
          estudiantesDiv.innerHTML = "<strong>No hay estudiantes disponibles.</strong>";
        } else {
          estudiantesDiv.innerHTML = "<strong>Estudiantes:</strong> " + estudiantes.join(", ");
        }
      } else {
        estudiantesDiv.innerHTML = "";
      }
    }

    function asignarTarea() {
      const tarea = document.getElementById("tarea").value.trim();
      if (!tarea) {
        alert("Primero ingresa una tarea.");
        return;
      }

      if (estudiantes.length === 0) {
        alert("No hay estudiantes disponibles.");
        return;
      }

      const indice = Math.floor(Math.random() * estudiantes.length);
      const responsable = estudiantes[indice];

      const tareaObj = {
        descripcion: tarea,
        responsable: responsable,
        comentarios: [],
        completada: false
      };

      tareas.push(tareaObj);
      estudiantes.splice(indice, 1);

      guardarEnStorage();
      actualizarListaEstudiantes();
      document.getElementById("tarea").value = "";
      mostrarTareasGuardadas();
    }

    function mostrarTareasGuardadas() {
      lista.innerHTML = "";

      tareas.forEach((tarea, index) => {
        if (rolActual === "usuario" && tarea.responsable !== usuarioActual) return;

        const item = document.createElement("li");
        item.innerHTML = `
          <strong>${tarea.descripcion}</strong> - <em>${tarea.responsable}</em><br>
          ${rolActual === "admin" ? `
            <button onclick="marcarCompletado(${index})">✔ Completada</button>
            <button onclick="eliminarTarea(${index})">🗑 Eliminar</button>
          ` : ""}
          <button onclick="agregarComentario(${index})">💬 Comentar</button>
          <div class="comentarios">
            ${tarea.comentarios.map(c => `<p>💬 ${c}</p>`).join("")}
          </div>
        `;
        if (tarea.completada) {
          item.style.backgroundColor = "#c8f7c5";
        }
        lista.appendChild(item);
      });
    }

    function marcarCompletado(index) {
      tareas[index].completada = true;
      guardarEnStorage();
      mostrarTareasGuardadas();
    }

    function agregarComentario(index) {
      const comentario = prompt("Escribe tu comentario:");
      if (comentario) {
        tareas[index].comentarios.push(comentario);
        guardarEnStorage();
        mostrarTareasGuardadas();
      }
    }

    function eliminarTarea(index) {
      tareas.splice(index, 1);
      guardarEnStorage();
      mostrarTareasGuardadas();
    }
  </script>
</body>
</html>

