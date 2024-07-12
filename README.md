<div class="container-main">
  <div
    class="slider-container"
    (touchstart)="onTouchStart($event)"
    (touchmove)="onTouchMove($event)"
    (touchend)="onTouchEnd()"
  >
    <div class="slider-inner" [style.transform]="transformStyle">
      <div class="slider">
        <div class="card">
          <div class="card-header">Card Title 1</div>
          <div class="card-body">
            <p>This is a simple card with some example content 1.</p>
          </div>
        </div>
      </div>
      <div class="slider">
        <div class="card">
          <div class="card-header">Card Title 2</div>
          <div class="card-body">
            <p>This is a simple card with some example content 2.</p>
          </div>
        </div>
      </div>
      <div class="slider">
        <div class="card">
          <div class="card-header">Card Title 3</div>
          <div class="card-body">
            <p>This is a simple card with some example content 3.</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</div






----------------

.container-main {
  .slider-container {
    // Estilos para el contenedor del slider
    overflow: hidden;
    position: relative;
  }

  .slider-inner {
    // Estilos para el contenedor interno del slider
    display: flex;
    transition: transform 0.5s ease;
  }

  .slider {
    // Estilos para cada slider
    min-width: 100%;
    box-sizing: border-box;
  }

  .card {
    // Estilos para el card
    background-color: #007bff; // Azul
    color: #fff;
    border-radius: 0.25rem;
    box-shadow: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075);
    overflow: hidden;
    margin: 1rem;
    padding: 1rem;

    &-header {
      // Estilos para el encabezado del card
      background-color: darken(#007bff, 10%);
      padding: 0.75rem 1.25rem;
      margin-bottom: 0;
      font-size: 1.25rem;
      font-weight: 500;
    }

    &-body {
      // Estilos para el cuerpo del card
      padding: 1.25rem;
    }
  }
}
