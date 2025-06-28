<html lang="es">
<head>
  <meta charset="UTF-8" />
  <title>Action Tracker con Gráfico de Pareto</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" />
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    .badge-pendiente { background-color: #ffc107; }
    .badge-progreso  { background-color: #0dcaf0; }
    .badge-completado { background-color: #198754; }
    .badge-vencido    { background-color: #dc3545; }
    .img-thumbnail { 
      transition: transform 0.2s; 
      cursor: pointer;
    }
    .img-thumbnail:hover {
      transform: scale(1.05);
    }
    .btn-delete {
      padding: 0.15rem 0.4rem;
      font-size: 0.75rem;
    }
    .chart-container {
      position: relative;
      height: 300px;
      margin-bottom: 1rem;
    }
    #paretoChart {
      margin-top: 10px;
    }
    .analysis-section {
      background-color: #f8f9fa;
      border-radius: 8px;
      padding: 15px;
      margin-bottom: 20px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
    .table-section {
      background-color: white;
      border-radius: 8px;
      padding: 15px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
    .form-section {
      background-color: white;
      border-radius: 8px;
      padding: 20px;
      margin-bottom: 20px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
  </style>
</head>
<body class="bg-light">
  <div class="container my-4">
    <h3 class="text-center mb-4">Action Tracker de Actividades</h3>
    <div id="alertaRegistro" style="position: fixed; top: 20px; right: 20px; z-index: 1055;"></div>
    
    <!-- Modo Administrador -->
    <div class="form-check form-switch text-end mb-3">
      <input class="form-check-input" type="checkbox" id="modoAdmin">
      <label class="form-check-label" for="modoAdmin">Modo Administrador</label>
    </div>

    <!-- Formulario -->
    <div class="form-section">
      <form id="formAccion" class="row g-3">
        <div class="col-md-4">
          <label class="form-label fw-bold">Actividad</label>
          <input type="text" id="actividad" class="form-control" placeholder="Describe la actividad" required />
        </div>
        <div class="col-md-3">
          <label class="form-label fw-bold">Responsable</label>
          <input type="text" id="responsable" class="form-control" placeholder="Nombre del responsable" required />
        </div>
        <div class="col-md-2">
          <label class="form-label fw-bold">Fecha Compromiso</label>
          <input type="date" id="fecha" class="form-control" required />
        </div>
        <div class="col-md-2">
          <label class="form-label fw-bold">Prioridad</label>
          <select id="prioridad" class="form-select">
            <option value="Alta">Alta</option>
            <option value="Media" selected>Media</option>
            <option value="Baja">Baja</option>
          </select>
        </div>
        <div class="col-md-1">
          <label class="form-label fw-bold">Foto</label>
          <input type="file" id="foto" class="form-control" accept="image/*" />
        </div>
        <div class="col-12 text-end">
          <button type="submit" class="btn btn-primary">
            <i class="bi bi-plus-circle me-1"></i> Agregar
          </button>
        </div>
      </form>
    </div>

    <!-- Sección de Análisis -->
    <div class="analysis-section">
      <h5 class="mb-3">Análisis de Pareto</h5>
      <div class="row mb-3">
        <div class="col-md-4">
          <select id="tipoAnalisis" class="form-select" onchange="generarPareto()">
            <option value="responsable">Por Responsable</option>
            <option value="prioridad">Por Prioridad</option>
            <option value="estado">Por Estado</option>
          </select>
        </div>
      </div>
      <div class="chart-container">
        <canvas id="paretoChart"></canvas>
      </div>
    </div>

    <!-- Contadores -->
    <div class="row mb-3">
      <div class="col">
        <span class="badge bg-warning text-dark">Pendiente: <span id="pendientes" class="fw-bold">0</span></span>
        <span class="badge bg-info text-dark">En Proceso: <span id="enProceso" class="fw-bold">0</span></span>
        <span class="badge bg-success">Completado: <span id="completados" class="fw-bold">0</span></span>
        <span class="badge bg-danger">Vencido: <span id="vencidos" class="fw-bold">0</span></span>
      </div>
    </div>

    <!-- Sección de Tabla -->
    <div class="table-section">
      <div class="table-responsive">
        <table class="table table-bordered table-hover align-middle" id="tabla">
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
    </div>
  </div>

  <!-- Scripts -->
  <script>
    // Variables globales
    let paretoChart = null;
    const form = document.getElementById("formAccion");
    const tabla = document.querySelector("#tabla tbody");
    const modoAdmin = document.getElementById("modoAdmin");
    let actividades = JSON.parse(localStorage.getItem("actividades")) || [];
    let adminAutenticado = false;

    // Inicialización
    document.addEventListener('DOMContentLoaded', function() {
      // Establecer fecha mínima como hoy
      document.getElementById('fecha').min = new Date().toISOString().split('T')[0];
      
      // Cargar datos de ejemplo si no hay actividades
      if (actividades.length === 0) {
        cargarDatosEjemplo();
      }
      
      actualizarTabla();
      generarPareto();
    });

    // Función para cargar datos de ejemplo
    function cargarDatosEjemplo() {
      const hoy = new Date();
      const manana = new Date(hoy);
      manana.setDate(hoy.getDate() + 1);
      const pasadoManana = new Date(hoy);
      pasadoManana.setDate(hoy.getDate() + 2);
      
      actividades = [
        {
          actividad: "Revisar informe mensual",
          responsable: "Juan Pérez",
          fecha: manana.toISOString().split('T')[0],
          prioridad: "Alta",
          estado: "Pendiente",
          imagen: ""
        },
        {
          actividad: "Actualizar base de datos",
          responsable: "María Gómez",
          fecha: hoy.toISOString().split('T')[0],
          prioridad: "Media",
          estado: "En Proceso",
          imagen: ""
        },
        {
          actividad: "Preparar presentación",
          responsable: "Carlos Ruiz",
          fecha: pasadoManana.toISOString().split('T')[0],
          prioridad: "Baja",
          estado: "Pendiente",
          imagen: ""
        },
        {
          actividad: "Revisar propuesta comercial",
          responsable: "Juan Pérez",
          fecha: hoy.toISOString().split('T')[0],
          prioridad: "Alta",
          estado: "Completado",
          imagen: ""
        },
        {
          actividad: "Capacitación equipo nuevo",
          responsable: "Ana López",
          fecha: manana.toISOString().split('T')[0],
          prioridad: "Media",
          estado: "Pendiente",
          imagen: ""
        }
      ];
      localStorage.setItem("actividades", JSON.stringify(actividades));
    }

    // Función para generar el gráfico de Pareto
    function generarPareto() {
      const tipo = document.getElementById('tipoAnalisis').value;
      const ctx = document.getElementById('paretoChart').getContext('2d');
      
      // Destruir el gráfico anterior si existe
      if (paretoChart) {
        paretoChart.destroy();
      }

      // Preparar datos según el tipo de análisis seleccionado
      let datos = {};
      actividades.forEach(actividad => {
        const clave = actividad[tipo];
        datos[clave] = (datos[clave] || 0) + 1;
      });

      // Ordenar datos de mayor a menor frecuencia
      const items = Object.entries(datos).sort((a, b) => b[1] - a[1]);
      const labels = items.map(item => item[0]);
      const valores = items.map(item => item[1]);

      // Calcular frecuencias acumuladas para la línea de Pareto
      let acumulado = 0;
      const total = valores.reduce((sum, val) => sum + val, 0);
      const acumulados = valores.map(val => {
        acumulado += val;
        return (acumulado / total) * 100;
      });

      // Crear el gráfico
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
          maintainAspectRatio: false,
          plugins: {
            title: {
              display: true,
              text: `Análisis de Pareto por ${tipo.charAt(0).toUpperCase() + tipo.slice(1)}`,
              font: {
                size: 16
              }
            },
            tooltip: {
              callbacks: {
                label: function(context) {
                  let label = context.dataset.label || '';
                  if (label === 'Porcentaje Acumulado') {
                    label += ': ' + context.raw.toFixed(1) + '%';
                  } else {
                    label += ': ' + context.raw;
                  }
                  return label;
                }
              }
            },
            legend: {
              position: 'top',
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

    // Función para actualizar la tabla
    function actualizarTabla() {
      tabla.innerHTML = "";
      let pend = 0, proc = 0, comp = 0, venc = 0;
      const hoy = new Date().toISOString().split("T")[0];

      actividades.forEach((a, i) => {
        if ((a.estado === "Pendiente" || a.estado === "En Proceso") && a.fecha < hoy) {
          a.estado = "Vencido";
        }

        const badgeClass =
          a.estado === "Pendiente" ? "badge-pendiente" :
          a.estado === "En Proceso" ? "badge-progreso" :
          a.estado === "Completado" ? "badge-completado" :
          "badge-vencido";

        const row = tabla.insertRow();
        row.innerHTML = `
          <td>${a.actividad}</td>
          <td>${a.responsable}</td>
          <td>
            ${modoAdmin.checked && adminAutenticado
              ? `<input type="date" class="form-control form-control-sm" value="${a.fecha}" onchange="modificarFecha(${i}, this.value)">`
              : a.fecha}
          </td>
          <td>
            <span class="badge ${a.prioridad === 'Alta' ? 'bg-danger' : a.prioridad === 'Media' ? 'bg-warning text-dark' : 'bg-secondary'}">
              ${a.prioridad}
            </span>
          </td>
          <td class="text-center">
            <span class="badge ${badgeClass}">${a.estado}</span>
          </td>
          <td class="text-center">
            ${a.imagen 
              ? `<img src="${a.imagen}" class="img-thumbnail" width="60" onclick="verImagen('${a.imagen}')">` 
              : "<span class='text-muted small'>Sin imagen</span>"}
          </td>
          <td class="text-center">
            ${modoAdmin.checked && adminAutenticado
              ? `<div class="d-flex justify-content-center gap-1">
                   <select class="form-select form-select-sm w-75" onchange="modificarEstado(${i}, this.value)">
                     <option ${a.estado === "Pendiente" ? "selected" : ""}>Pendiente</option>
                     <option ${a.estado === "En Proceso" ? "selected" : ""}>En Proceso</option>
                     <option ${a.estado === "Completado" ? "selected" : ""}>Completado</option>
                     <option ${a.estado === "Vencido" ? "selected" : ""}>Vencido</option>
                   </select>
                   <button class="btn btn-danger btn-sm btn-delete" onclick="eliminarActividad(${i})">
                     <i class="bi bi-trash"></i>
                   </button>
                 </div>`
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

      localStorage.setItem("actividades", JSON.stringify(actividades));
    }

    // Función para mostrar alertas
    function mostrarAlerta(mensaje, tipo) {
      const alertaDiv = document.getElementById("alertaRegistro");
      alertaDiv.innerHTML = `
        <div class="alert alert-${tipo} alert-dismissible fade show shadow" role="alert">
          <strong>${tipo === 'success' ? 'Éxito!' : 'Atención!'}</strong> ${mensaje}
          <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
        </div>
      `;
      
      setTimeout(() => {
        const alerta = document.querySelector('.alert');
        if (alerta) {
          alerta.classList.remove('show');
          setTimeout(() => alerta.remove(), 300);
        }
      }, 3000);
    }

    // Funciones para modificar datos
    function modificarEstado(index, nuevoEstado) {
      actividades[index].estado = nuevoEstado;
      mostrarAlerta("Estado actualizado correctamente.", "info");
      actualizarTabla();
    }

    function modificarFecha(index, nuevaFecha) {
      actividades[index].fecha = nuevaFecha;
      mostrarAlerta("Fecha actualizada correctamente.", "info");
      actualizarTabla();
    }

    function eliminarActividad(index) {
      if (confirm("¿Estás seguro de eliminar esta actividad?")) {
        const actividadEliminada = actividades[index].actividad;
        actividades.splice(index, 1);
        mostrarAlerta(`Actividad "${actividadEliminada}" eliminada.`, "danger");
        actualizarTabla();
      }
    }

    // Evento submit del formulario
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
          actividades.push({ actividad, responsable, fecha, prioridad, estado: "Pendiente", imagen: reader.result });
          mostrarAlerta("Actividad registrada correctamente.", "success");
          actualizarTabla();
        };
        reader.readAsDataURL(archivo);
      } else {
        actividades.push({ actividad, responsable, fecha, prioridad, estado: "Pendiente", imagen: "" });
        mostrarAlerta("Actividad registrada correctamente.", "success");
        actualizarTabla();
      }

      form.reset();
    });

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

    function verImagen(src) {
      const modal = new bootstrap.Modal(document.getElementById("modalImagen"));
      document.getElementById("imagenAmpliada").src = src;
      modal.show();
    }
  </script>

  <!-- Modal para imagen ampliada -->
  <div class="modal fade" id="modalImagen" tabindex="-1" aria-hidden="true">
    <div class="modal-dialog modal-lg modal-dialog-centered">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Vista Ampliada</h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
        </div>
        <div class="modal-body d-flex justify-content-center">
          <img src="" id="imagenAmpliada" class="img-fluid rounded" alt="Vista Ampliada">
        </div>
      </div>
    </div>
  </div>

  <!-- Modal de Login para Modo Administrador -->
  <div class="modal fade" id="modalLogin" tabindex="-1" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered">
      <div class="modal-content p-3">
        <h5 class="text-center mb-3">Acceso Administrador</h5>
        <div class="mb-3">
          <label class="form-label">Usuario</label>
          <input type="text" id="usuarioAdmin" class="form-control" placeholder="Ingrese su usuario" />
        </div>
        <div class="mb-4">
          <label class="form-label">Contraseña</label>
          <input type="password" id="passwordAdmin" class="form-control" placeholder="Ingrese su contraseña" />
        </div>
        <div class="text-end">
          <button class="btn btn-secondary btn-sm me-2" onclick="cancelarLogin()">Cancelar</button>
          <button class="btn btn-primary btn-sm" onclick="validarLogin()">Ingresar</button>
        </div>
      </div>
    </div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.0/font/bootstrap-icons.css">
</body>
</html>
