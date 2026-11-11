<template>
  <div class="barra_comandos" role="toolbar" aria-label="Acciones de archivos">
    <!-- Nuevo -->
    <div class="menu_wrap">
      <button class="cmd" title="Nuevo" aria-label="Nuevo" @click.stop="alternar('nuevo')">
        <font-awesome-icon icon="circle-plus" class="icon-cmd" />
        <span class="etiqueta-main">Nuevo</span>
        <font-awesome-icon icon="chevron-down" class="caret" />
      </button>
      <div v-if="abierto === 'nuevo'" class="menu">
        <button class="plano" @click="emitir('nueva-carpeta')">Carpeta</button>
        <button class="plano" @click="emitir('nuevo-archivo')">Archivo de texto</button>
      </div>
    </div>
 
    <span class="divisor"></span>
 
    <!-- Cortar, copiar, pegar, renombrar, compartir, eliminar -->
    <button
      v-for="a in acciones" :key="a.id"
      class="cmd cmd_icono" :title="a.titulo" :aria-label="a.titulo"
      :disabled="deshabilitado(a.req)" @click="accion(a.id)"
    >
      <font-awesome-icon :icon="a.icono" class="icon-cmd" />
    </button>
 
    <span class="divisor"></span>
 
    <!-- Ordenar / Ver / Filtro -->
    <div v-for="m in menus" :key="m.id" class="menu_wrap">
      <button class="cmd" :title="m.texto" :aria-label="m.texto" @click.stop="alternar(m.id)">
        <font-awesome-icon :icon="m.icono" class="icon-cmd" />
        <span class="etiqueta-sec">{{ m.texto }}</span>
        <font-awesome-icon icon="chevron-down" class="caret" />
      </button>
      <div v-if="abierto === m.id" class="menu">
        <button
          v-for="o in m.opciones" :key="o.id"
          :class="{ activa: m.actual() === o.id }"
          @click="m.elegir(o.id)"
        >{{ o.texto }}</button>
        <button v-if="m.id === 'orden'" :class="{ activa: descendente }" @click="descendente = !descendente">
          Descendente
        </button>
      </div>
    </div>
 
    <button class="cmd cmd_icono" title="Más opciones" aria-label="Más opciones">
      <font-awesome-icon icon="ellipsis" class="icon-cmd" />
    </button>
 
    <span class="espacio"></span>
 
    <button class="cmd" :class="{ encendido: mostrarPrevia }" title="Vista previa" :aria-pressed="mostrarPrevia" @click="mostrarPrevia = !mostrarPrevia">
      <font-awesome-icon icon="table-columns" class="icon-cmd" />
      <span class="etiqueta-main">Vista previa</span>
    </button>
  </div>
</template>
 
<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import {
  vista, orden, descendente, filtro, mostrarPrevia, seleccion, portapapeles,
} from '../store/navegacion.js'
 
// El componente padre (Carpetas.vue o la página) escucha estos eventos
const emit = defineEmits([
  'nueva-carpeta', 'nuevo-archivo', 'cortar', 'copiar', 'pegar', 'renombrar', 'compartir', 'eliminar',
])
const emitir = (e) => { abierto.value = null; emit(e) }
 
const abierto = ref(null)
const alternar = (id) => { abierto.value = abierto.value === id ? null : id }
const cerrar = () => { abierto.value = null }
const alEscape = (e) => e.key === 'Escape' && cerrar()
onMounted(() => { document.addEventListener('click', cerrar); document.addEventListener('keydown', alEscape) })
onBeforeUnmount(() => { document.removeEventListener('click', cerrar); document.removeEventListener('keydown', alEscape) })
 
const acciones = [
  { id: 'cortar',    icono: 'scissors',        titulo: 'Cortar',         req: 'varios' },
  { id: 'copiar',    icono: 'copy',            titulo: 'Copiar',         req: 'varios' },
  { id: 'pegar',     icono: 'paste',           titulo: 'Pegar',          req: 'portapapeles' },
  { id: 'renombrar', icono: 'pen',             titulo: 'Cambiar nombre', req: 'uno' },
  { id: 'compartir', icono: 'share-nodes',     titulo: 'Compartir',      req: 'varios' },
  { id: 'eliminar',  icono: 'trash',           titulo: 'Eliminar',       req: 'varios' },
]
const deshabilitado = (req) =>
  req === 'varios' ? seleccion.value.length === 0
  : req === 'uno' ? seleccion.value.length !== 1
  : !portapapeles.value
const accion = (id) => {
  if (id === 'cortar' || id === 'copiar') {
    portapapeles.value = { modo: id, items: [...seleccion.value] }
  }
  emit(id)
}
 
const menus = [
  { id: 'orden', texto: 'Ordenar', icono: 'arrow-down-wide-short',
    opciones: [{ id: 'nombre', texto: 'Nombre' }, { id: 'fecha', texto: 'Fecha' }],
    actual: () => orden.value, elegir: (v) => { orden.value = v; cerrar() } },
  { id: 'vista', texto: 'Ver', icono: 'table-cells-large',
    opciones: [{ id: 'iconos', texto: 'Iconos' }, { id: 'lista', texto: 'Lista' }],
    actual: () => vista.value, elegir: (v) => { vista.value = v; cerrar() } },
  { id: 'filtro', texto: 'Filtro', icono: 'filter',
    opciones: [{ id: 'todo', texto: 'Todo' }, { id: 'carpetas', texto: 'Carpetas' }, { id: 'archivos', texto: 'Archivos' }],
    actual: () => filtro.value, elegir: (v) => { filtro.value = v; cerrar() } },
]
</script>
 
<style scoped>
* { font-family: 'Courier New', monospace; }
 
.barra_comandos {
  display: flex;
  align-items: center;
  gap: 2px;
  height: 44px;
  padding: 0 12px;
  border-radius: 10px;
  border: 1px solid rgba(131, 100, 255, 0.15);
  background: rgba(131, 100, 255, 0.05);
  container-type: inline-size;
}
 
/* Si la barra no cabe, primero se ocultan los textos de Ordenar, Ver y Filtro,
   y después los de Nuevo y Vista previa. Los íconos y el tooltip se mantienen. */
@container (max-width: 880px) {
  .etiqueta-sec { display: none; }
}
@container (max-width: 700px) {
  .etiqueta-main { display: none; }
}
 
.cmd {
  display: flex;
  align-items: center;
  gap: 8px;
  height: 32px;
  padding: 0 10px;
  border: 1px solid transparent;
  border-radius: 6px;
  background: transparent;
  color: #d4c4ff;
  font-size: 0.85rem;
  white-space: nowrap;
  flex-shrink: 0;
  cursor: pointer;
  transition: all 0.3s ease;
}
.cmd_icono { padding: 0; width: 32px; justify-content: center; }
 
.cmd:hover:not(:disabled) {
  background: rgba(131, 100, 255, 0.15);
  border-color: rgba(131, 100, 255, 0.3);
}
.cmd:focus-visible { outline: 2px solid #b388ff; }
.cmd:disabled { opacity: 0.35; cursor: default; }
.cmd.encendido { background: rgba(131, 100, 255, 0.22); border-color: rgba(131, 100, 255, 0.4); }
 
.icon-cmd { color: #b388ff; font-size: 14px; }
.caret { color: #6a4e9a; font-size: 10px; }
 
.divisor { width: 1px; height: 20px; margin: 0 6px; background: rgba(131, 100, 255, 0.25); }
.espacio { flex: 1; }
 
.menu_wrap { position: relative; }
.menu {
  position: absolute;
  top: 38px;
  left: 0;
  z-index: 50;
  min-width: 170px;
  padding: 4px;
  border-radius: 8px;
  border: 1px solid rgba(131, 100, 255, 0.3);
  background: #12062a;
  box-shadow: 0 8px 28px rgba(0, 0, 0, 0.6), 0 0 20px rgba(131, 100, 255, 0.1);
}
.menu button {
  display: block;
  width: 100%;
  padding: 7px 10px;
  border: 0;
  border-radius: 5px;
  background: none;
  color: #d4c4ff;
  font-size: 0.85rem;
  text-align: left;
  cursor: pointer;
}
.menu button::before { content: '○ '; color: #6a4e9a; }
.menu button.plano::before { content: none; }
.menu button.activa::before { content: '● '; color: #b388ff; }
.menu button:hover { background: rgba(131, 100, 255, 0.15); }
</style>
 