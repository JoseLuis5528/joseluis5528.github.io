<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <title>Action Tracker con Reloj</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" />
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    .badge-pendiente { background-color: #ffc107; }
    .badge-progreso  { background-color: #0dcaf0; }
    .badge-completado { background-color: #198754; }
    .badge-vencido    { background-color: #dc3545; }
    .badge-new {
      background-color: #0d6efd;
      font-size: 0.6rem;
      vertical-align: super;
      margin-left: 5px;
    }
    .img-thumbnail { 
      transition: transform 0.2s; 
      cursor: pointer;
    }
    .img-thumbnail:hover {
      transform: scale(1.05);
    }
    .chart-container {
      background: white;
      border-radius: 8px;
      padding: 20px;
      margin-top: 30px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
    canvas {
      width: 100% !important;
      height: 400px !important;
    }
    .real-time-clock {
      position: absolute;
      top: 15px;
      left: 15px;
      background: rgba(0, 0, 0, 0.7);
      color: white;
      padding: 8px 15px;
      border-radius: 20px;
      font-size: 1.1rem;
      box-shadow: 0 2px 10px rgba(0,0,0,0.3);
      z-index: 1000;
    }
    .container {
      position: relative;
    }
    .header-section {
      margin-top: 50px;
    }
  </style>
</head>
<body class="bg-light">
  <!-- Reloj en tiempo real -->
  <div class="real-time-clock" id="reloj">
    <span id="fecha-hora">Cargando...</span>
  </div>

  <div class="container my-4">
    <div class="header-section">
      <h3 class="text-center mb-3">Action Tracker de Actividades</h3>
      <div id="alertaRegistro" style="position: fixed; top: 20px; right: 20px; z-index: 1055;"></div>
    </div>
    
    <!-- Modo Administrador -->
    <div class="form-check form-switch text-end mb-3">
      <input class="form-check-input" type="checkbox" id="modoAdmin">
      <label class="form-check-label" for="modoAdmin">Modo Administrador</label>
    </div>

    <!-- Formulario -->
    <form id="formAccion" class="row g-3 mb-4 bg-white p-3 rounded shadow">
      <div class="col-md-4">
        <label class="form-label">Actividad</label>
        <input type="text" id="actividad" class="form-control" required />
      </div>
      <div class="col-md-3">
        <label class="form-label">Responsable</label>
        <input type="text" id="responsable" class="form-control" required />
      </div>
      <div class="col-md-2">
        <label class="form-label">Fecha Compromiso</label>
        <input type="date" id="fecha" class="form-control" required />
      </div>
      <div class="col-md-2">
        <label class="form-label">Prioridad</label>
        <select id="prioridad" class="form-select">
          <option value="Alta">Alta</option>
          <option value="Media">Media</option>
          <option value="Baja">Baja</option>
        </select>
      </div>
      <div class="col-md-1">
        <label class="form-label">Foto</label>
        <input type="file" id="foto" class="form-control" accept="image/*" />
      </div>
      <div class="col-12 text-end">
        <button type="submit" class="btn btn-primary">Agregar</button>
      </div>
    </form>

    <!-- Tabla de Actividades -->
    <div class="table-responsive bg-white p-3 rounded shadow">
      <table class="table table-bordered table-striped align-middle" id="tabla">
        <thead class="table-dark text-center">
          <tr>
            <th>Actividad</th>
            <th>Responsable</th>
            <th>Compromiso</th>
            <th>Prioridad</th>
            <th>Estado</th>
            <th>Imagen</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody></tbody>
      </table>
    </div>

    <!-- Gráfico de Pareto -->
    <div class="chart-container shadow">
      <h5 class="mb-4">Análisis de Pareto</h5>
      <div class="row mb-3">
        <div class="col-md-4">
          <select id="tipoAnalisis" class="form-select" onchange="generarPareto()">
            <option value="responsable">Por Responsable</option>
            <option value="prioridad">Por Prioridad</option>
            <option value="estado">Por Estado</option>
          </select>
        </div>
      </div>
      <canvas id="paretoChart"></canvas>
    </div>
  </div>
  <script type="module">
  // Import the functions you need from the SDKs you need
  import { initializeApp } from "https://www.gstatic.com/firebasejs/11.9.1/firebase-app.js";
  import { getAnalytics } from "https://www.gstatic.com/firebasejs/11.9.1/firebase-analytics.js";
  // TODO: Add SDKs for Firebase products that you want to use
  // https://firebase.google.com/docs/web/setup#available-libraries

  // Your web app's Firebase configuration
  // For Firebase JS SDK v7.20.0 and later, measurementId is optional
  const firebaseConfig = {
    apiKey: "AIzaSyD6m97z4Z0HDzVIS5ojAoVfmesqCUxy5KU",
    authDomain: "action-tracker-3ccd8.firebaseapp.com",
    projectId: "action-tracker-3ccd8",
    storageBucket: "action-tracker-3ccd8.firebasestorage.app",
    messagingSenderId: "820818816895",
    appId: "1:820818816895:web:d0f7565d2c3cb5d257344a",
    measurementId: "G-6KB5BTJZK3"
  };

  // Initialize Firebase
  const app = initializeApp(firebaseConfig);
  const analytics = getAnalytics(app);
  
  db.ref("actividades").on("value", snapshot => {
  actividades = [];
  snapshot.forEach(child => {
    actividades.push(child.val());
  });
  actualizarTabla();
});
</script>

  <!-- Scripts -->
  <script>
    // Variables globales
    let paretoChart = null;
    const form = document.getElementById("formAccion");
    const tabla = document.querySelector("#tabla tbody");
    const modoAdmin = document.getElementById("modoAdmin");
    let actividades = JSON.parse(localStorage.getItem("actividades")) || [];
    let adminAutenticado = false;

    // Función para actualizar el reloj en tiempo real
    function actualizarReloj() {
      const ahora = new Date();
      const opciones = { 
        weekday: 'long', 
        year: 'numeric', 
        month: 'long', 
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit',
        second: '2-digit',
        hour12: true
      };
      
      const fechaHora = ahora.toLocaleDateString('es-ES', opciones);
      document.getElementById("fecha-hora").textContent = fechaHora;
    }
    
    // Actualizar el reloj cada segundo
    setInterval(actualizarReloj, 1000);
    actualizarReloj(); // Inicializar inmediatamente

    // Inicialización
    document.addEventListener('DOMContentLoaded', function() {
      if (actividades.length === 0) {
        cargarDatosEjemplo();
      }
      actualizarTabla();
      generarPareto();
    });

    function cargarDatosEjemplo() {
      const hoy = new Date();
      const manana = new Date(hoy);
      manana.setDate(hoy.getDate() + 1);
      
      actividades = [
        {
          actividad: "Revisar informe mensual",
          responsable: "Juan Pérez",
          fecha: manana.toISOString().split('T')[0],
          prioridad: "Alta",
          estado: "Pendiente",
          imagen: "",
          timestamp: Date.now() - 3600000
        },
        {
          actividad: "Actualizar base de datos",
          responsable: "María Gómez",
          fecha: hoy.toISOString().split('T')[0],
          prioridad: "Media",
          estado: "En Proceso",
          imagen: "",
          timestamp: Date.now() - 1800000
        },
        {
          actividad: "Preparar presentación",
          responsable: "Carlos Ruiz",
          fecha: manana.toISOString().split('T')[0],
          prioridad: "Baja",
          estado: "Pendiente",
          imagen: "",
          timestamp: Date.now()
        }
      ];
      localStorage.setItem("actividades", JSON.stringify(actividades));
    }

    // Manejar el envío del formulario
    form.addEventListener("submit", e => {
      e.preventDefault();
      const actividad = document.getElementById("actividad").value;
      const responsable = document.getElementById("responsable").value;
      const fecha = document.getElementById("fecha").value;
      const prioridad = document.getElementById("prioridad").value;
      const archivo = document.getElementById("foto").files[0];

      if (archivo) {
        const reader = new FileReader();
        reader.onload = () => {
          actividades.push({
            actividad, 
            responsable, 
            fecha, 
            prioridad, 
            estado: "Pendiente", 
            imagen: reader.result,
            timestamp: Date.now()
          });
          mostrarAlerta("Actividad registrada correctamente.", "success");
          actualizarTabla();
          generarPareto();
        };
        reader.readAsDataURL(archivo);
      } else {
        actividades.push({
          actividad, 
          responsable, 
          fecha, 
          prioridad, 
          estado: "Pendiente", 
          imagen: "",
          timestamp: Date.now()
        });
        mostrarAlerta("Actividad registrada correctamente.", "success");
        actualizarTabla();
        generarPareto();
      }

      form.reset();
    });

    // Función para actualizar la tabla
    function actualizarTabla() {
      tabla.innerHTML = "";
      const ahora = Date.now();
      const cincoMinutos = 5 * 60 * 1000;

      actividades.forEach((a, i) => {
        const esNueva = (ahora - a.timestamp) < cincoMinutos;
        const badgeClass =
          a.estado === "Pendiente" ? "badge-pendiente" :
          a.estado === "En Proceso" ? "badge-progreso" :
          a.estado === "Completado" ? "badge-completado" :
          "badge-vencido";

        const row = tabla.insertRow();
        row.innerHTML = `
          <td>
            ${a.actividad}
            ${esNueva ? '<span class="badge badge-new">New</span>' : ''}
          </td>
          <td>${a.responsable}</td>
          <td>
            ${modoAdmin.checked && adminAutenticado
              ? `<input type="date" class="form-control form-control-sm" value="${a.fecha}" onchange="modificarFecha(${i}, this.value)">`
              : a.fecha}
          </td>
          <td>${a.prioridad}</td>
          <td class="text-center"><span class="badge ${badgeClass}">${a.estado}</span></td>
          <td class="text-center">
            ${a.imagen ? `<img src="${a.imagen}" width="60" style="cursor:pointer" onclick="verImagen('${a.imagen}')">` : "Sin imagen"}
          </td>
          <td class="text-center">
            ${modoAdmin.checked && adminAutenticado
              ? `<select class="form-select form-select-sm" onchange="modificarEstado(${i}, this.value)">
                  <option ${a.estado === "Pendiente" ? "selected" : ""}>Pendiente</option>
                  <option ${a.estado === "En Proceso" ? "selected" : ""}>En Proceso</option>
                  <option ${a.estado === "Completado" ? "selected" : ""}>Completado</option>
                  <option ${a.estado === "Vencido" ? "selected" : ""}>Vencido</option>
                </select>`
              : `<span class="text-muted small">${a.estado}</span>`}
          </td>
        `;
      });

      localStorage.setItem("actividades", JSON.stringify(actividades));
    }

    // Función para generar el gráfico de Pareto
    function generarPareto() {
      const tipo = document.getElementById('tipoAnalisis').value;
      const ctx = document.getElementById('paretoChart').getContext('2d');
      
      if (paretoChart) {
        paretoChart.destroy();
      }

      let datos = {};
      actividades.forEach(actividad => {
        const clave = actividad[tipo];
        datos[clave] = (datos[clave] || 0) + 1;
      });

      const items = Object.entries(datos).sort((a, b) => b[1] - a[1]);
      const labels = items.map(item => item[0]);
      const valores = items.map(item => item[1]);

      let acumulado = 0;
      const total = valores.reduce((sum, val) => sum + val, 0);
      const acumulados = valores.map(val => {
        acumulado += val;
        return (acumulado / total) * 100;
      });

      paretoChart = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: labels,
          datasets: [
            {
              label: 'Frecuencia',
              data: valores,
              backgroundColor: 'rgba(54, 162, 235, 0.7)',
              borderColor: 'rgba(54, 162, 235, 1)',
              borderWidth: 1
            },
            {
              label: 'Porcentaje Acumulado',
              data: acumulados,
              type: 'line',
              borderColor: 'rgba(255, 99, 132, 1)',
              backgroundColor: 'rgba(255, 99, 132, 0.1)',
              borderWidth: 2,
              yAxisID: 'y1'
            }
          ]
        },
        options: {
          responsive: true,
          plugins: {
            title: {
              display: true,
              text: `Análisis de Pareto por ${tipo.charAt(0).toUpperCase() + tipo.slice(1)}`
            }
          },
          scales: {
            y: {
              beginAtZero: true,
              title: {
                display: true,
                text: 'Cantidad de Actividades'
              }
            },
            y1: {
              position: 'right',
              beginAtZero: true,
              max: 100,
              title: {
                display: true,
                text: 'Porcentaje Acumulado'
              },
              grid: {
                drawOnChartArea: false
              }
            }
          }
        }
      });
    }

    // Funciones auxiliares
    function mostrarAlerta(mensaje, tipo) {
      const alerta = document.createElement("div");
      alerta.className = `alert alert-${tipo} alert-dismissible fade show`;
      alerta.role = "alert";
      alerta.innerHTML = `
        ${mensaje}
        <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
      `;
      document.getElementById("alertaRegistro").appendChild(alerta);

      setTimeout(() => {
        alerta.classList.remove("show");
        alerta.classList.add("hide");
        setTimeout(() => alerta.remove(), 300);
      }, 3000);
    }

    function modificarEstado(index, nuevoEstado) {
      actividades[index].estado = nuevoEstado;
      actualizarTabla();
      generarPareto();
    }

    function modificarFecha(index, nuevaFecha) {
      actividades[index].fecha = nuevaFecha;
      actualizarTabla();
      generarPareto();
    }

    function verImagen(src) {
      const modal = new bootstrap.Modal(document.getElementById("modalImagen"));
      document.getElementById("imagenAmpliada").src = src;
      modal.show();
    }

    function validarLogin() {
      const usuario = document.getElementById("usuarioAdmin").value;
      const password = document.getElementById("passwordAdmin").value;

      if (usuario === "Admin" && password === "Admin1234") {
        adminAutenticado = true;
        modoAdmin.checked = true;
        bootstrap.Modal.getInstance(document.getElementById("modalLogin")).hide();
        mostrarAlerta("Modo administrador activado", "success");
        actualizarTabla();
      } else {
        mostrarAlerta("Credenciales incorrectas", "danger");
      }
    }

    function cancelarLogin() {
      modoAdmin.checked = false;
      bootstrap.Modal.getInstance(document.getElementById("modalLogin")).hide();
    }
  </script>

  <!-- Modal para imagen ampliada -->
  <div class="modal fade" id="modalImagen" tabindex="-1" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered">
      <div class="modal-content">
        <img src="" id="imagenAmpliada" class="img-fluid" alt="Vista Ampliada">
      </div>
    </div>
  </div>

  <!-- Modal de Login para Modo Administrador -->
  <div class="modal fade" id="modalLogin" tabindex="-1" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered">
      <div class="modal-content p-3">
        <h5 class="text-center">Acceso Administrador</h5>
        <div class="mb-2">
          <label class="form-label">Usuario</label>
          <input type="text" id="usuarioAdmin" class="form-control" />
        </div>
        <div class="mb-3">
          <label class="form-label">Contraseña</label>
          <input type="password" id="passwordAdmin" class="form-control" />
        </div>
        <div class="text-end">
          <button class="btn btn-secondary btn-sm me-2" onclick="cancelarLogin()">Cancelar</button>
          <button class="btn btn-primary btn-sm" onclick="validarLogin()">Ingresar</button>
        </div>
      </div>
    </div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
