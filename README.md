<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <title>Action Tracker con Firebase</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <!-- Firebase SDK -->
  <script src="https://www.gstatic.com/firebasejs/10.7.2/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.7.2/firebase-firestore-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.7.2/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.7.2/firebase-storage-compat.js"></script>
  <!-- Bootstrap -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" />
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.0/font/bootstrap-icons.css" />
  <!-- Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    :root {
      --primary-color: #2c3e50;
      --secondary-color: #3498db;
      --success-color: #27ae60;
      --warning-color: #f39c12;
      --danger-color: #e74c3c;
    }
    
    body {
      background-color: #f8f9fa;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      padding-top: 20px;
    }
    
    .real-time-clock {
      position: fixed;
      top: 15px;
      left: 15px;
      background: var(--primary-color);
      color: white;
      padding: 10px 20px;
      border-radius: 30px;
      font-size: 1.1rem;
      box-shadow: 0 4px 15px rgba(0,0,0,0.2);
      z-index: 1000;
      display: flex;
      align-items: center;
    }
    
    .real-time-clock i {
      margin-right: 10px;
      font-size: 1.3rem;
    }
    
    .container {
      position: relative;
      max-width: 1400px;
    }
    
    .header-section {
      margin-top: 70px;
      margin-bottom: 30px;
    }
    
    .app-title {
      color: var(--primary-color);
      font-weight: 700;
      text-transform: uppercase;
      letter-spacing: 1px;
      border-bottom: 3px solid var(--secondary-color);
      padding-bottom: 10px;
      display: inline-block;
    }
    
    .badge-pendiente { background-color: var(--warning-color); }
    .badge-progreso  { background-color: var(--secondary-color); }
    .badge-completado { background-color: var(--success-color); }
    .badge-vencido    { background-color: var(--danger-color); }
    
    .badge-new {
      background-color: var(--secondary-color);
      font-size: 0.6rem;
      vertical-align: super;
      margin-left: 5px;
      animation: pulse 2s infinite;
    }
    
    @keyframes pulse {
      0% { opacity: 1; }
      50% { opacity: 0.5; }
      100% { opacity: 1; }
    }
    
    .img-thumbnail { 
      transition: transform 0.2s; 
      cursor: pointer;
      border-radius: 5px;
      border: 1px solid #dee2e6;
    }
    
    .img-thumbnail:hover {
      transform: scale(1.1);
      box-shadow: 0 0 10px rgba(0,0,0,0.2);
    }
    
    .chart-container {
      background: white;
      border-radius: 10px;
      padding: 25px;
      margin-top: 30px;
      box-shadow: 0 5px 15px rgba(0,0,0,0.08);
      border: 1px solid #eaeaea;
    }
    
    .card-section {
      background: white;
      border-radius: 10px;
      padding: 25px;
      box-shadow: 0 5px 15px rgba(0,0,0,0.08);
      border: 1px solid #eaeaea;
      margin-bottom: 25px;
    }
    
    .form-section {
      background: linear-gradient(135deg, #f8f9fa, #e9ecef);
      border-radius: 10px;
      padding: 25px;
      box-shadow: 0 5px 15px rgba(0,0,0,0.08);
      border: 1px solid #eaeaea;
      margin-bottom: 30px;
    }
    
    .status-counters .badge {
      font-size: 0.9rem;
      padding: 8px 15px;
      border-radius: 20px;
      margin-right: 10px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    }
    
    .admin-toggle {
      display: flex;
      align-items: center;
      justify-content: flex-end;
      gap: 10px;
    }
    
    .btn-delete {
      background: var(--danger-color);
      color: white;
      border-radius: 50%;
      width: 30px;
      height: 30px;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 0;
    }
    
    .btn-delete:hover {
      background: #c0392b;
    }
    
    .action-cell {
      display: flex;
      gap: 8px;
    }
    
    .priority-high { color: var(--danger-color); font-weight: bold; }
    .priority-medium { color: var(--warning-color); font-weight: bold; }
    .priority-low { color: var(--success-color); font-weight: bold; }
    
    .table-hover tbody tr:hover {
      background-color: rgba(52, 152, 219, 0.05);
    }
    
    thead th {
      background: var(--primary-color) !important;
      color: white !important;
    }
    
    .modal-content {
      border-radius: 15px;
      overflow: hidden;
    }
    
    .alert-container {
      position: fixed;
      top: 20px;
      right: 20px;
      z-index: 1055;
      width: 350px;
    }

    .firebase-status {
      position: fixed;
      bottom: 15px;
      right: 15px;
      background: #27ae60;
      color: white;
      padding: 8px 15px;
      border-radius: 20px;
      font-size: 0.9rem;
      z-index: 1000;
      display: flex;
      align-items: center;
    }

    .firebase-status i {
      margin-right: 8px;
    }
  </style>
</head>
<body>
  <!-- Reloj en tiempo real -->
  <div class="real-time-clock" id="reloj">
    <i class="bi bi-clock"></i>
    <span id="fecha-hora">Cargando...</span>
  </div>

  <!-- Estado de Firebase -->
  <div class="firebase-status" id="firebaseStatus">
    <i class="bi bi-cloud-check"></i>
    <span>Conectado a Firebase</span>
  </div>

  <div class="container my-4">
    <div class="header-section text-center">
      <h1 class="app-title">Action Tracker con Firebase</h1>
      <p class="text-muted">Gestión profesional de tareas con almacenamiento en la nube</p>
    </div>
    
    <!-- Alertas -->
    <div class="alert-container" id="alertaRegistro"></div>
    
    <!-- Modo Administrador -->
    <div class="admin-toggle mb-4">
      <label class="form-check-label fw-bold" for="modoAdmin">Modo Administrador</label>
      <div class="form-check form-switch">
        <input class="form-check-input" type="checkbox" id="modoAdmin" style="transform: scale(1.3);">
      </div>
    </div>

    <!-- Formulario -->
    <div class="form-section">
      <form id="formAccion" class="row g-3">
        <div class="col-md-4">
          <label class="form-label fw-bold"><i class="bi bi-list-task me-2"></i>Actividad</label>
          <input type="text" id="actividad" class="form-control" placeholder="Describe la actividad" required />
        </div>
        <div class="col-md-3">
          <label class="form-label fw-bold"><i class="bi bi-person me-2"></i>Responsable</label>
          <input type="text" id="responsable" class="form-control" placeholder="Nombre del responsable" required />
        </div>
        <div class="col-md-2">
          <label class="form-label fw-bold"><i class="bi bi-calendar me-2"></i>Fecha Compromiso</label>
          <input type="date" id="fecha" class="form-control" required />
        </div>
        <div class="col-md-2">
          <label class="form-label fw-bold"><i class="bi bi-exclamation-triangle me-2"></i>Prioridad</label>
          <select id="prioridad" class="form-select">
            <option value="Alta">Alta</option>
            <option value="Media" selected>Media</option>
            <option value="Baja">Baja</option>
          </select>
        </div>
        <div class="col-md-1">
          <label class="form-label fw-bold"><i class="bi bi-camera me-2"></i>Foto</label>
          <input type="file" id="foto" class="form-control" accept="image/*" />
        </div>
        <div class="col-12 text-end">
          <button type="submit" class="btn btn-primary px-4">
            <i class="bi bi-plus-circle me-2"></i>Agregar Actividad
          </button>
        </div>
      </form>
    </div>

    <!-- Contadores de estado -->
    <div class="card-section">
      <h5 class="mb-3"><i class="bi bi-bar-chart me-2"></i>Resumen de Actividades</h5>
      <div class="status-counters">
        <span class="badge bg-warning text-dark">Pendiente: <span id="pendientes" class="fw-bold">0</span></span>
        <span class="badge bg-info text-dark">En Proceso: <span id="enProceso" class="fw-bold">0</span></span>
        <span class="badge bg-success">Completado: <span id="completados" class="fw-bold">0</span></span>
        <span class="badge bg-danger">Vencido: <span id="vencidos" class="fw-bold">0</span></span>
      </div>
    </div>

    <!-- Tabla de Actividades -->
    <div class="card-section">
      <div class="d-flex justify-content-between align-items-center mb-3">
        <h5><i class="bi bi-table me-2"></i>Listado de Actividades</h5>
        <div class="text-muted small">Total: <span id="total-actividades">0</span> actividades</div>
      </div>
      <div class="table-responsive">
        <table class="table table-hover align-middle" id="tabla">
          <thead class="table-dark">
            <tr>
              <th>Actividad</th>
              <th>Responsable</th>
              <th>Compromiso</th>
              <th>Prioridad</th>
              <th class="text-center">Estado</th>
              <th class="text-center">Imagen</th>
              <th class="text-center">Acciones</th>
            </tr>
          </thead>
          <tbody></tbody>
        </table>
      </div>
    </div>

    <!-- Gráfico de Pareto -->
    <div class="chart-container">
      <div class="d-flex justify-content-between align-items-center mb-4">
        <h5><i class="bi bi-graph-up me-2"></i>Análisis de Pareto</h5>
        <div class="w-25">
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

  <!-- Scripts -->
  <script>
    // 1. Configuración de Firebase (REEMPLAZA CON TUS PROPIAS CREDENCIALES)
    const firebaseConfig = {
      apiKey: "AIzaSyCQ5gJq5d1Y3Y1z6Y1X3Y1z6Y1X3Y1z6Y1X3",
      authDomain: "action-tracker-5528.firebaseapp.com",
      projectId: "action-tracker-5528",
      storageBucket: "action-tracker-5528.appspot.com",
      messagingSenderId: "123456789012",
      appId: "1:123456789012:web:abcdefghijklmnopqrstuv"
    };

    // Inicializar Firebase
    firebase.initializeApp(firebaseConfig);
    const db = firebase.firestore();
    const auth = firebase.auth();
    const storage = firebase.storage();

    // Variables globales
    let paretoChart = null;
    const form = document.getElementById("formAccion");
    const tabla = document.querySelector("#tabla tbody");
    const modoAdmin = document.getElementById("modoAdmin");
    let actividades = [];
    let adminAutenticado = false;
    let unsubscribe = null;

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

    // Función para mostrar alertas
    function mostrarAlerta(mensaje, tipo) {
      const alerta = document.createElement("div");
      alerta.className = `alert alert-${tipo} alert-dismissible fade show`;
      alerta.role = "alert";
      alerta.innerHTML = `
        <div class="d-flex align-items-center">
          <i class="bi ${tipo === 'success' ? 'bi-check-circle' : tipo === 'danger' ? 'bi-exclamation-triangle' : 'bi-info-circle'} me-2"></i>
          <div>${mensaje}</div>
        </div>
        <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
      `;
      document.getElementById("alertaRegistro").appendChild(alerta);

      setTimeout(() => {
        alerta.classList.remove("show");
        setTimeout(() => alerta.remove(), 300);
      }, 3000);
    }

    // Cargar actividades desde Firebase
    function cargarActividades() {
      unsubscribe = db.collection("actividades")
        .orderBy("fecha", "asc")
        .onSnapshot((snapshot) => {
          actividades = [];
          snapshot.forEach((doc) => {
            const actividad = doc.data();
            actividad.id = doc.id;
            actividades.push(actividad);
          });
          actualizarTabla();
          generarPareto();
        }, (error) => {
          mostrarAlerta("Error cargando actividades: " + error.message, "danger");
        });
    }

    // Guardar actividad en Firebase
    async function guardarActividad(actividad) {
      try {
        // Subir imagen si existe
        if (actividad.archivo) {
          const storageRef = storage.ref();
          const fileRef = storageRef.child(`actividades/${new Date().getTime()}_${actividad.archivo.name}`);
          await fileRef.put(actividad.archivo);
          const imageUrl = await fileRef.getDownloadURL();
          actividad.imagen = imageUrl;
        }

        // Guardar en Firestore
        await db.collection("actividades").add({
          actividad: actividad.actividad,
          responsable: actividad.responsable,
          fecha: actividad.fecha,
          prioridad: actividad.prioridad,
          estado: "Pendiente",
          imagen: actividad.imagen || "",
          timestamp: firebase.firestore.FieldValue.serverTimestamp()
        });

        mostrarAlerta("Actividad guardada en Firebase", "success");
      } catch (error) {
        mostrarAlerta("Error guardando actividad: " + error.message, "danger");
      }
    }

    // Eliminar actividad de Firebase
    async function eliminarActividadFirebase(id) {
      try {
        await db.collection("actividades").doc(id).delete();
        mostrarAlerta("Actividad eliminada", "success");
      } catch (error) {
        mostrarAlerta("Error eliminando actividad: " + error.message, "danger");
      }
    }

    // Actualizar actividad en Firebase
    async function actualizarActividad(id, campo, valor) {
      try {
        await db.collection("actividades").doc(id).update({
          [campo]: valor
        });
      } catch (error) {
        mostrarAlerta("Error actualizando actividad: " + error.message, "danger");
      }
    }

    // Manejar el envío del formulario
    form.addEventListener("submit", async (e) => {
      e.preventDefault();
      const actividad = document.getElementById("actividad").value;
      const responsable = document.getElementById("responsable").value;
      const fecha = document.getElementById("fecha").value;
      const prioridad = document.getElementById("prioridad").value;
      const archivo = document.getElementById("foto").files[0];

      const nuevaActividad = {
        actividad, 
        responsable, 
        fecha, 
        prioridad,
        archivo
      };

      await guardarActividad(nuevaActividad);
      form.reset();
    });

    // Función para actualizar la tabla
    function actualizarTabla() {
      tabla.innerHTML = "";
      const ahora = Date.now();
      const cincoMinutos = 5 * 60 * 1000;
      let totalActividades = 0;
      let pend = 0, proc = 0, comp = 0, venc = 0;

      actividades.forEach((a) => {
        totalActividades++;
        const esNueva = a.timestamp && (ahora - a.timestamp.toDate().getTime()) < cincoMinutos;
        const badgeClass =
          a.estado === "Pendiente" ? "badge-pendiente" :
          a.estado === "En Proceso" ? "badge-progreso" :
          a.estado === "Completado" ? "badge-completado" :
          "badge-vencido";
        
        // Clase de prioridad para colorear el texto
        const prioridadClass = 
          a.prioridad === "Alta" ? "priority-high" :
          a.prioridad === "Media" ? "priority-medium" : "priority-low";

        const row = tabla.insertRow();
        row.innerHTML = `
          <td>
            ${a.actividad}
            ${esNueva ? '<span class="badge badge-new">NEW</span>' : ''}
          </td>
          <td>${a.responsable}</td>
          <td>${a.fecha}</td>
          <td class="${prioridadClass}">${a.prioridad}</td>
          <td class="text-center"><span class="badge ${badgeClass}">${a.estado}</span></td>
          <td class="text-center">
            ${a.imagen ? `<img src="${a.imagen}" class="img-thumbnail" width="60" onclick="verImagen('${a.imagen}')">` : "Sin imagen"}
          </td>
          <td class="text-center action-cell">
            ${modoAdmin.checked && adminAutenticado
              ? `<select class="form-select form-select-sm w-75" onchange="modificarEstado('${a.id}', this.value)">
                  <option ${a.estado === "Pendiente" ? "selected" : ""}>Pendiente</option>
                  <option ${a.estado === "En Proceso" ? "selected" : ""}>En Proceso</option>
                  <option ${a.estado === "Completado" ? "selected" : ""}>Completado</option>
                  <option ${a.estado === "Vencido" ? "selected" : ""}>Vencido</option>
                </select>
                <button class="btn btn-delete" onclick="eliminarActividad('${a.id}')">
                  <i class="bi bi-trash"></i>
                </button>`
              : `<span class="text-muted small">${a.estado}</span>`}
          </td>
        `;

        if (a.estado === "Pendiente") pend++;
        else if (a.estado === "En Proceso") proc++;
        else if (a.estado === "Completado") comp++;
        else if (a.estado === "Vencido") venc++;
      });

      document.getElementById("pendientes").textContent = pend;
      document.getElementById("enProceso").textContent = proc;
      document.getElementById("completados").textContent = comp;
      document.getElementById("vencidos").textContent = venc;
      document.getElementById("total-actividades").textContent = totalActividades;
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
              backgroundColor: 'rgba(41, 128, 185, 0.7)',
              borderColor: 'rgba(41, 128, 185, 1)',
              borderWidth: 1
            },
            {
              label: 'Porcentaje Acumulado',
              data: acumulados,
              type: 'line',
              borderColor: 'rgba(231, 76, 60, 1)',
              backgroundColor: 'rgba(231, 76, 60, 0.1)',
              borderWidth: 3,
              yAxisID: 'y1',
              pointRadius: 5,
              pointBackgroundColor: 'rgba(231, 76, 60, 1)'
            }
          ]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            title: {
              display: true,
              text: `Análisis de Pareto por ${tipo.charAt(0).toUpperCase() + tipo.slice(1)}`,
              font: {
                size: 18,
                weight: 'bold'
              },
              padding: 20
            },
            legend: {
              position: 'top',
              labels: {
                font: {
                  size: 14
                }
              }
            },
            tooltip: {
              backgroundColor: 'rgba(0, 0, 0, 0.8)',
              titleFont: {
                size: 14
              },
              bodyFont: {
                size: 13
              },
              padding: 12,
              cornerRadius: 6
            }
          },
          scales: {
            y: {
              beginAtZero: true,
              title: {
                display: true,
                text: 'Cantidad de Actividades',
                font: {
                  size: 14,
                  weight: 'bold'
                }
              },
              ticks: {
                font: {
                  size: 12
                }
              }
            },
            y1: {
              position: 'right',
              beginAtZero: true,
              max: 100,
              title: {
                display: true,
                text: 'Porcentaje Acumulado',
                font: {
                  size: 14,
                  weight: 'bold'
                }
              },
              grid: {
                drawOnChartArea: false
              },
              ticks: {
                font: {
                  size: 12
                },
                callback: function(value) {
                  return value + '%';
                }
              }
            },
            x: {
              ticks: {
                font: {
                  size: 12
                }
              }
            }
          }
        }
      });
    }

    // Función para eliminar actividad (solo admin)
    async function eliminarActividad(id) {
      if (adminAutenticado) {
        if (confirm("¿Estás seguro de eliminar esta actividad?")) {
          await eliminarActividadFirebase(id);
        }
      } else {
        mostrarAlerta("Acceso denegado. Solo administradores pueden eliminar actividades.", "warning");
      }
    }

    // Función para modificar estado
    async function modificarEstado(id, nuevoEstado) {
      await actualizarActividad(id, "estado", nuevoEstado);
      mostrarAlerta("Estado actualizado correctamente.", "info");
    }

    // Función para ver imagen ampliada
    function verImagen(src) {
      const modal = new bootstrap.Modal(document.getElementById("modalImagen"));
      document.getElementById("imagenAmpliada").src = src;
      modal.show();
    }

    // Funciones para el modo administrador
    modoAdmin.addEventListener("change", () => {
      if (modoAdmin.checked) {
        if (!adminAutenticado) {
          modoAdmin.checked = false;
          document.getElementById("usuarioAdmin").value = "";
          document.getElementById("passwordAdmin").value = "";
          const modal = new bootstrap.Modal(document.getElementById("modalLogin"));
          modal.show();
          setTimeout(() => {
            document.getElementById("usuarioAdmin").focus();
          }, 500);
        } else {
          actualizarTabla();
        }
      } else {
        adminAutenticado = false;
        actualizarTabla();
      }
    });

    // Autenticación con Firebase
    async function validarLogin() {
      const usuario = document.getElementById("usuarioAdmin").value;
      const password = document.getElementById("passwordAdmin").value;

      try {
        // Iniciar sesión con Firebase Authentication
        const userCredential = await auth.signInWithEmailAndPassword(usuario, password);
        
        // Verificar si el usuario es administrador
        if (userCredential.user) {
          adminAutenticado = true;
          modoAdmin.checked = true;
          bootstrap.Modal.getInstance(document.getElementById("modalLogin")).hide();
          mostrarAlerta("Modo administrador activado", "success");
          actualizarTabla();
        }
      } catch (error) {
        mostrarAlerta("Credenciales incorrectas: " + error.message, "danger");
      }
    }

    function cancelarLogin() {
      modoAdmin.checked = false;
      bootstrap.Modal.getInstance(document.getElementById("modalLogin")).hide();
    }

    // Inicializar Firebase y cargar datos
    firebase.auth().onAuthStateChanged((user) => {
      if (user) {
        adminAutenticado = true;
        modoAdmin.checked = true;
        mostrarAlerta("Sesión de administrador activa", "success");
      } else {
        adminAutenticado = false;
      }
    });

    // Iniciar la aplicación
    document.addEventListener('DOMContentLoaded', function() {
      cargarActividades();
    });
  </script>

  <!-- Modal para imagen ampliada -->
  <div class="modal fade" id="modalImagen" tabindex="-1" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered modal-lg">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Vista Ampliada</h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
        </div>
        <div class="modal-body d-flex justify-content-center p-4">
          <img src="" id="imagenAmpliada" class="img-fluid rounded" alt="Vista Ampliada">
        </div>
      </div>
    </div>
  </div>

  <!-- Modal de Login para Modo Administrador -->
  <div class="modal fade" id="modalLogin" tabindex="-1" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered">
      <div class="modal-content p-4">
        <div class="text-center mb-4">
          <i class="bi bi-shield-lock" style="font-size: 3rem; color: #2c3e50;"></i>
          <h4 class="mt-2">Acceso Administrador</h4>
          <p class="text-muted">Ingrese sus credenciales para continuar</p>
        </div>
        <div class="mb-3">
          <label class="form-label">Usuario</label>
          <div class="input-group">
            <span class="input-group-text"><i class="bi bi-person"></i></span>
            <input type="email" id="usuarioAdmin" class="form-control" placeholder="admin@ejemplo.com" required />
          </div>
        </div>
        <div class="mb-4">
          <label class="form-label">Contraseña</label>
          <div class="input-group">
            <span class="input-group-text"><i class="bi bi-lock"></i></span>
            <input type="password" id="passwordAdmin" class="form-control" placeholder="********" required />
          </div>
        </div>
        <div class="text-end">
          <button class="btn btn-secondary me-2" onclick="cancelarLogin()">Cancelar</button>
          <button class="btn btn-primary" onclick="validarLogin()">Ingresar</button>
        </div>
      </div>
    </div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
