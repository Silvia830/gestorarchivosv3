<template>
  <div class="barra_superior col-12">
    <div class="titulo_costado">
      <img src="/logo.png" class="logo">
      <h1 class="titulo">EstudiOS</h1>
      <button class="boton_control"> 
        <font-awesome-icon icon="arrow-left" class="icon-control" />
      </button>
      <button class="boton_control">
        <font-awesome-icon icon="arrow-right" class="icon-control" />
      </button>
    </div>
    <div class="rutas">
    <p>Ruta : {{ rutaActual }}</p>
</div>
    <div class="buscador">
      <font-awesome-icon icon="search" class="icon-search" />
      <input type="text" placeholder="Buscar..." class="buscar">
    </div>
    <div class="botones_superior">
      <button class="botones_accion"> 
        <font-awesome-icon icon="minus" class="icon-window" />
      </button>
      <button class="botones_accion">
        <font-awesome-icon icon="square" class="icon-window" />
      </button>
      <button class="botones_accion">
        <font-awesome-icon icon="times" class="icon-window" />
      </button>
    </div>
  </div>
</template>

<script setup>
import { rutaActual } from '../store/ruta.js'
</script>

<style scoped>
* {
  font-family: 'Courier New', monospace;
}

.barra_superior {
  display: flex;
  align-items: center;
  height: 56px;
 
  
  border-radius: 10px;
  
  backdrop-filter: blur(10px);
}

.titulo_costado {
  display: flex;
  width: 25%;
  align-items: center;
  gap: 8px;
}

.logo {
  margin-left: 16px;
  width: 36px;
  height: 36px;
  filter: drop-shadow(0 0 10px rgba(131, 100, 255, 0.3));
}

.titulo {
  color: #d4c4ff;
  font-size: 1.3rem;
  font-weight: bold;
  display: flex;
  align-items: center;
  padding-left: 8px;
  text-shadow: 0 0 20px rgba(131, 100, 255, 0.3);
  letter-spacing: 1px;
}

.boton_control {
  width: 32px;
  height: 32px;
  margin-left: 8px;
  border: 1px solid transparent;
  background: transparent;
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
}

.boton_control:hover {
  background: rgba(131, 100, 255, 0.15);
  border-color: rgba(131, 100, 255, 0.3);
  box-shadow: 0 0 20px rgba(131, 100, 255, 0.1);
  transform: scale(1.05);
}

.icon-control {
  color: #8a7aaa;
  font-size: 16px;
  transition: all 0.3s ease;
}

.boton_control:hover .icon-control {
  color: #b388ff;
  text-shadow: 0 0 20px rgba(131, 100, 255, 0.5);
}

.rutas {
  width: 35%;
  height: 100%;
  display: flex;
  align-items: center;
  color: #8a7aaa;
  padding-left: 20px;
}

.rutas p {
  font-size: 0.85rem;
  letter-spacing: 0.5px;
  color: #8a7aaa;
  border-left: 2px solid rgba(131, 100, 255, 0.2);
  padding-left: 16px;
}

.buscador {
  width: 30%;
  height: 100%;
  display: flex;
  align-items: center;
  position: relative;
}

.icon-search {
  position: absolute;
  left: 16px;
  color: #6a4e9a;
  font-size: 14px;
}

.buscar {
  width: 100%;
  height: 36px;
  border-radius: 8px;
  background: rgba(131, 100, 255, 0.05);
  border: 1px solid rgba(131, 100, 255, 0.15);
  font-size: 0.9rem;
  padding-left: 40px;
  padding-right: 16px;
  color: #d4c4ff;
  transition: all 0.3s ease;
  font-family: 'Courier New', monospace;
}

.buscar::placeholder {
  color: #6a4e9a;
  font-size: 0.85rem;
}

.buscar:focus {
  outline: none;
  background: rgba(131, 100, 255, 0.08);
  border-color: rgba(131, 100, 255, 0.4);
  box-shadow: 0 0 30px rgba(131, 100, 255, 0.1);
}

.buscar:focus + .icon-search {
  color: #b388ff;
}

.botones_superior {
  width: 15%;
  height: 100%;
  display: flex;
  justify-content: flex-end;
  align-items: center;
  gap: 4px;
  padding-right: 12px;
}

.botones_accion {
  width: 32px;
  height: 32px;
  background: transparent;
  border: 1px solid transparent;
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
}

.botones_accion:hover {
  background: rgba(131, 100, 255, 0.1);
  border-color: rgba(131, 100, 255, 0.2);
}

.botones_accion:nth-child(3):hover {
  background: rgba(255, 0, 0, 0.15);
  border-color: rgba(255, 0, 0, 0.3);
  box-shadow: 0 0 20px rgba(255, 0, 0, 0.1);
}

.icon-window {
  color: #8a7aaa;
  font-size: 14px;
  transition: all 0.3s ease;
}

.botones_accion:hover .icon-window {
  color: #d4c4ff;
}

.botones_accion:nth-child(3):hover .icon-window {
  color: #ff6b6b;
  text-shadow: 0 0 20px rgba(255, 0, 0, 0.3);
}

/* Scrollbar */
::-webkit-scrollbar {
  width: 4px;
}

::-webkit-scrollbar-track {
  background: rgba(131, 100, 255, 0.05);
}

::-webkit-scrollbar-thumb {
  background: linear-gradient(180deg, #8364ff, #b388ff);
  border-radius: 2px;
  box-shadow: 0 0 15px rgba(131, 100, 255, 0.3);
}
</style>