
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Registro de Actividades</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <!-- Bootstrap 5 -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>

<div class="container mt-4">
  <h4 class="mb-3">Detalles del Registro</h4>
  <form id="form-registro">
    <div class="row mb-3">
      <div class="col-md-6">
        <label for="nombre" class="form-label">Nombre</label>
        <input type="text" class="form-control" id="nombre" required>
      </div>
      <div class="col-md-6">
        <label for="comentario" class="form-label">Comentario</label>
        <input type="text" class="form-control" id="comentario" required>
      </div>
    </div>

    <div class="row mb-3">
      <div class="col-md-4">
        <label for="tiempo" class="form-label">Tiempo</label>
        <input type="time" class="form-control" id="tiempo" required>
      </div>
      <div class="col-md-4">
        <label for="fecha" class="form-label">Fecha de Registro</label>
        <input type="date" class="form-control" id="fecha" required>
      </div>
    </div>

    <button type="submit" class="btn btn-success">Registrar</button>
  </form>

  <hr class="my-4">

  <h5>Historial de Registros</h5>
  <table class="table table-bordered" id="tabla-registros">
    <thead class="table-light">
      <tr>
        <th>Nombre</th>
        <th>Comentario</th>
        <th>Tiempo</th>
        <th>Fecha</th>
      </tr>
    </thead>
    <tbody>
      <!-- Se agregarán los registros aquí -->
    </tbody>
  </table>
</div>

<!-- Script JavaScript -->
<script>
  document.addEventListener('DOMContentLoaded', () => {
    const formulario = document.getElementById('form-registro');
    const tabla = document.querySelector('#tabla-registros tbody');
    const claveLocal = 'registrosActividades';

    // Cargar registros guardados al iniciar
    const registros = JSON.parse(localStorage.getItem(claveLocal)) || [];
    registros.forEach(agregarFila);

    // Evento al enviar formulario
    formulario.addEventListener('submit', e => {
      e.preventDefault();

      const nuevo = {
        nombre: document.getElementById('nombre').value,
        comentario: document.getElementById('comentario').value,
        tiempo: document.getElementById('tiempo').value,
        fecha: document.getElementById('fecha').value
      };

      registros.push(nuevo);
      localStorage.setItem(claveLocal, JSON.stringify(registros));
      agregarFila(nuevo);
      formulario.reset();
    });

    // Insertar una fila en la tabla
    function agregarFila(data) {
      const fila = document.createElement('tr');
      fila.innerHTML = `
        <td>${data.nombre}</td>
        <td>${data.comentario}</td>
        <td>${data.tiempo}</td>
        <td>${data.fecha}</td>
      `;
      tabla.appendChild(fila);
    }
  });
</script>

</body>
</html>
