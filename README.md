# joseluis5528.github.io
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Visor de Excel</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <!-- Bootstrap 5 -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
  <!-- SheetJS -->
  <script src="https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js"></script>
</head>
<body>
<div class="container mt-4">
  <h5 class="mb-3">Detalles del Registro</h5>
  <form>
    <div class="row mb-3">
      <div class="col-md-6">
        <label for="nombre" class="form-label">Nombre</label>
        <input type="text" class="form-control" id="nombre" placeholder="Ingresa tu nombre">
      </div>
      <div class="col-md-6">
        <label for="comentario" class="form-label">Comentario</label>
        <input type="text" class="form-control" id="comentario" placeholder="Escribe un comentario">
      </div>
    </div>

    <div class="row mb-3">
      <div class="col-md-4">
        <label for="tiempo" class="form-label">Tiempo</label>
        <input type="time" class="form-control" id="tiempo">
      </div>
      <div class="col-md-4">
        <label for="fecha" class="form-label">Fecha de Registro</label>
        <input type="date" class="form-control" id="fecha">
      </div>
    </div>

    <button type="submit" class="btn btn-success">Registrar</button>
  </form>
</div>


</body>
</html>
