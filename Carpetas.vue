<template>
    <div class="carpetas-contenedor col-8">
        <BarraComandos
            class="barra-comandos"
            @nueva-carpeta="nuevaCarpeta"
            @nuevo-archivo="nuevoArchivo"
            @pegar="pegar"
            @renombrar="iniciarRenombre(seleccion[0])"
            @compartir="compartir"
            @eliminar="eliminar"
        />
 
        <div class="carpetas" @click.self="limpiarSeleccion">
            <h2 class="texto">Carpetas</h2>
 
            <svg width="0" height="0" style="position:absolute">
                <linearGradient id="grad-carpeta" x1="0%" y1="0%" x2="100%" y2="100%">
                    <stop offset="0%" stop-color="#8364ff" />
                    <stop offset="100%" stop-color="#b388ff" />
                </linearGradient>
            </svg>
 
            <div class="grid-carpetas" :class="{ lista: vista === 'lista' }" @click.self="limpiarSeleccion">
                <div
                    v-for="carpeta in carpetasVisibles"
                    :key="carpeta.nombre"
                    class="item-carpeta"
                    :class="{
                        abierta: carpetaAbierta?.nombre === carpeta.nombre,
                        seleccionada: seleccion.includes(carpeta.nombre),
                        cortada: portapapeles?.modo === 'cortar' && portapapeles.items.includes(carpeta.nombre),
                    }"
                    @mouseenter="carpetaActiva = carpeta"
                    @mouseleave="carpetaActiva = null"
                    @click.stop="seleccionar($event, carpeta.nombre)"
                    @dblclick="abrir(carpeta)"
                >
                    <svg class="icono-carpeta" viewBox="0 0 24 24" fill="none">
                        <path
                            d="M3 6a2 2 0 0 1 2-2h4l2 2h8a2 2 0 0 1 2 2v9a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V6z"
                            stroke="url(#grad-carpeta)"
                            stroke-width="1.5"
                        />
                    </svg>
                    <input
                        v-if="renombrando === carpeta.nombre"
                        v-model="nombreNuevo"
                        v-focus
                        class="input-renombrar"
                        @click.stop
                        @dblclick.stop
                        @keydown.enter.stop="confirmarRenombre"
                        @keydown.esc.stop="renombrando = null"
                        @blur="confirmarRenombre"
                    />
                    <p v-else class="nombre-carpeta">{{ carpeta.nombre }}</p>
                </div>
            </div>
        </div>
 
        <div v-if="mostrarPrevia" class="contenido">
            <h2 class="texto">Vista previa</h2>
 
            <template v-if="previa">
                <p class="subtitulo-preview">{{ previa.nombre }} — {{ previa.archivos.length }} elementos</p>
 
                <div class="contenido-items" :class="{ lista: vista === 'lista' }">
                    <div v-for="archivo in archivosPrevia" :key="archivo" class="item-archivo">
 
                        <img
                            v-if="esRutaReal(archivo)"
                            :src="archivo"
                            class="miniatura-real"
                            alt="vista previa"
                        />
 
                        <svg v-else-if="tipoArchivo(archivo) === 'imagen'" class="icono-archivo icono-imagen" viewBox="0 0 24 24" fill="none">
                            <rect x="3" y="4" width="18" height="16" rx="2" stroke="currentColor" stroke-width="1.5"/>
                            <circle cx="8.5" cy="9.5" r="1.5" stroke="currentColor" stroke-width="1.5"/>
                            <path d="M4 17l5-5 3.5 3.5L17 11l3 3" stroke="currentColor" stroke-width="1.5"/>
                        </svg>
 
                        <svg v-else-if="tipoArchivo(archivo) === 'pdf'" class="icono-archivo icono-pdf" viewBox="0 0 24 24" fill="none">
                            <path d="M6 2h9l4 4v16H6V2z" stroke="currentColor" stroke-width="1.5"/>
                            <path d="M15 2v4h4" stroke="currentColor" stroke-width="1.5"/>
                            <text x="7.5" y="17" font-size="6" fill="currentColor" font-weight="bold">PDF</text>
                        </svg>
 
                        <svg v-else-if="tipoArchivo(archivo) === 'audio'" class="icono-archivo icono-audio" viewBox="0 0 24 24" fill="none">
                            <path d="M9 18V5l12-2v13" stroke="currentColor" stroke-width="1.5"/>
                            <circle cx="9" cy="18" r="3" stroke="currentColor" stroke-width="1.5"/>
                            <circle cx="21" cy="16" r="3" stroke="currentColor" stroke-width="1.5"/>
                        </svg>
 
                        <svg v-else-if="tipoArchivo(archivo) === 'carpeta'" class="icono-archivo icono-subcarpeta" viewBox="0 0 24 24" fill="none">
                            <path d="M3 6a2 2 0 0 1 2-2h4l2 2h8a2 2 0 0 1 2 2v9a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V6z" stroke="currentColor" stroke-width="1.5"/>
                        </svg>
 
                        <svg v-else class="icono-archivo icono-generico" viewBox="0 0 24 24" fill="none">
                            <path d="M6 2h9l4 4v16H6V2z" stroke="currentColor" stroke-width="1.5"/>
                            <path d="M15 2v4h4" stroke="currentColor" stroke-width="1.5"/>
                            <path d="M9 13h6M9 16h6" stroke="currentColor" stroke-width="1.2"/>
                        </svg>
 
                        <span class="nombre-archivo">{{ nombreVisible(archivo) }}</span>
                    </div>
                </div>
            </template>
            <template v-else>
                <p class="placeholder-preview">Pasa el mouse sobre una carpeta para ver su contenido</p>
            </template>
        </div>
 
        <ModalCompartir
            :visible="compartirAbierto"
            :rutas="rutasSeleccion"
            @cerrar="compartirAbierto = false"
            @aviso="avisar"
        />
 
        <div v-if="aviso" class="aviso" role="status">{{ aviso }}</div>
    </div>
</template>
 
<script setup>
import { ref, computed, watch, onMounted, onBeforeUnmount } from 'vue'
import monje from '../assets/logo.png'
import BarraComandos from './BarraComandos.vue'
import ModalCompartir from './ModalCompartir.vue'
import { rutaActual, carpetaAbiertaNombre } from '../store/ruta.js'
import {
    irA, atras, adelante, subir, recargar, recargas,
    vista, orden, descendente, filtro, busqueda, mostrarPrevia,
    seleccion, portapapeles,
} from '../store/navegacion.js'
 
const BASE = '/home/usuario/'
 
const carpetaActiva = ref(null)
 
// Datos de ejemplo en memoria. Las acciones de la barra modifican este arreglo.
const carpetas = ref([
    { nombre: 'Documentos', fecha: '2025-03-12', archivos: ['Informe.docx', 'Presupuesto.xlsx', 'Notas.txt'] },
    { nombre: 'Proyectos',  fecha: '2025-05-02', archivos: ['proyecto-web/', 'proyecto-app/', 'README.md'] },
    { nombre: 'Multimedia', fecha: '2025-01-20', archivos: ['video1.mp4', 'cancion.mp3', monje] },
    { nombre: 'Descargas',  fecha: '2025-06-08', archivos: ['instalador.exe', 'manual.pdf', 'reporte.pdf'] },
    { nombre: 'Imagenes',   fecha: '2025-04-15', archivos: ['vacaciones.jpg', 'perfil.png', 'wallpaper.jpg'] },
    { nombre: 'Archivo',    fecha: '2024-11-30', archivos: ['backup_2025.zip', 'antiguo.docx'] },
])
 
const carpetaAbierta = computed(() =>
    carpetas.value.find(c => c.nombre === carpetaAbiertaNombre.value) || null
)
 
// Qué carpeta se muestra en la vista previa
const previa = computed(() =>
    carpetaAbierta.value
    || carpetaActiva.value
    || (seleccion.value.length === 1 ? carpetas.value.find(c => c.nombre === seleccion.value[0]) : null)
)
 
/* ---------- Mantener carpetaAbiertaNombre sincronizado con la ruta ---------- */
// Así Atrás, Adelante, Subir y las migas de la barra superior abren o cierran la carpeta correcta.
watch(rutaActual, (r) => {
    const ultimo = r.split('/').filter(Boolean).pop()
    const esCarpeta = r === BASE + ultimo && carpetas.value.some(c => c.nombre === ultimo)
    carpetaAbiertaNombre.value = esCarpeta ? ultimo : null
}, { immediate: true })
 
/* ---------- Orden, filtro y búsqueda ---------- */
const carpetasVisibles = computed(() => {
    const q = busqueda.value.trim().toLowerCase()
    const lista = carpetas.value.filter(c => !q || c.nombre.toLowerCase().includes(q))
    lista.sort(orden.value === 'fecha'
        ? (a, b) => b.fecha.localeCompare(a.fecha)
        : (a, b) => a.nombre.localeCompare(b.nombre))
    return descendente.value ? lista.reverse() : lista
})
 
// El filtro (Todo / Carpetas / Archivos) se aplica al contenido de la vista previa
const archivosPrevia = computed(() => {
    if (!previa.value) return []
    const q = busqueda.value.trim().toLowerCase()
    const lista = previa.value.archivos.filter(a => {
        const esCarpeta = tipoArchivo(a) === 'carpeta'
        if (filtro.value === 'carpetas' && !esCarpeta) return false
        if (filtro.value === 'archivos' && esCarpeta) return false
        return !q || nombreVisible(a).toLowerCase().includes(q)
    })
    lista.sort((a, b) => nombreVisible(a).localeCompare(nombreVisible(b)))
    return descendente.value ? lista.reverse() : lista
})
 
/* ---------- Selección ---------- */
function seleccionar(e, nombre) {
    if (e.ctrlKey || e.metaKey) {
        seleccion.value = seleccion.value.includes(nombre)
            ? seleccion.value.filter(n => n !== nombre)
            : [...seleccion.value, nombre]
    } else {
        seleccion.value = [nombre]
    }
}
const limpiarSeleccion = () => { seleccion.value = [] }
 
function abrir(carpeta) {
    carpetaAbiertaNombre.value = carpeta.nombre
    irA(BASE + carpeta.nombre)
}
 
/* ---------- Avisos ---------- */
const aviso = ref('')
let temporizador
function avisar(texto) {
    aviso.value = texto
    clearTimeout(temporizador)
    temporizador = setTimeout(() => { aviso.value = '' }, 2000)
}
 
/* ---------- Acciones de la barra de comandos ---------- */
function nombreLibre(base) {
    let nombre = base, n = 2
    while (carpetas.value.some(c => c.nombre.toLowerCase() === nombre.toLowerCase())) nombre = `${base} ${n++}`
    return nombre
}
 
function nuevaCarpeta() {
    const nombre = nombreLibre('Nueva carpeta')
    carpetas.value.push({ nombre, fecha: new Date().toISOString().slice(0, 10), archivos: [] })
    seleccion.value = [nombre]
    iniciarRenombre(nombre) // igual que el Explorador: queda lista para escribir el nombre
}
 
function nuevoArchivo() {
    if (!carpetaAbierta.value) return avisar('Abre una carpeta (doble clic) para crear el archivo ahí')
    const existentes = carpetaAbierta.value.archivos
    let nombre = 'Nuevo documento.txt', n = 2
    while (existentes.includes(nombre)) nombre = `Nuevo documento ${n++}.txt`
    existentes.push(nombre)
    avisar(`Creado: ${nombre}`)
}
 
function pegar() {
    const p = portapapeles.value
    if (!p) return
    const destino = carpetaAbierta.value
    if (!destino) return avisar('Abre una carpeta (doble clic) para pegar ahí')
    const nombres = p.items.filter(n => n !== destino.nombre)
    nombres.forEach(n => { if (!destino.archivos.includes(n + '/')) destino.archivos.push(n + '/') })
    if (p.modo === 'cortar') {
        carpetas.value = carpetas.value.filter(c => !nombres.includes(c.nombre))
        portapapeles.value = null
    }
    avisar(`Pegado en ${destino.nombre}`)
}
 
const renombrando = ref(null)
const nombreNuevo = ref('')
function iniciarRenombre(nombre) {
    if (!nombre) return
    renombrando.value = nombre
    nombreNuevo.value = nombre
}
function confirmarRenombre() {
    const viejo = renombrando.value
    if (!viejo) return
    renombrando.value = null
    const nuevo = nombreNuevo.value.trim()
    if (!nuevo || nuevo === viejo) return
    if (carpetas.value.some(c => c.nombre.toLowerCase() === nuevo.toLowerCase())) {
        return avisar('Ya existe una carpeta con ese nombre')
    }
    carpetas.value.find(c => c.nombre === viejo).nombre = nuevo
    if (carpetaAbiertaNombre.value === viejo) irA(BASE + nuevo)
    seleccion.value = [nuevo]
}
 
const compartirAbierto = ref(false)
const rutasSeleccion = computed(() => seleccion.value.map(n => BASE + n))
function compartir() {
    if (seleccion.value.length) compartirAbierto.value = true
}
 
function eliminar() {
    const cuantos = seleccion.value.length
    if (!cuantos) return
    const borradas = [...seleccion.value]
    carpetas.value = carpetas.value.filter(c => !borradas.includes(c.nombre))
    if (borradas.includes(carpetaAbiertaNombre.value)) irA('/home/usuario')
    seleccion.value = []
    avisar(`${cuantos} elemento(s) eliminado(s)`)
}
 
watch(recargas, () => avisar('Actualizado'))
 
// Directiva local: enfoca y selecciona el texto del input de renombrar
const vFocus = { mounted: (el) => { el.focus(); el.select() } }
 
/* ---------- Atajos de teclado ---------- */
function alTeclado(e) {
    if (['INPUT', 'TEXTAREA'].includes(e.target.tagName)) return
    if (e.altKey && e.key === 'ArrowLeft') atras()
    if (e.altKey && e.key === 'ArrowRight') adelante()
    if (e.altKey && e.key === 'ArrowUp') subir()
    if (e.key === 'F5') { e.preventDefault(); recargar() }
    if (e.key === 'F2' && seleccion.value.length === 1) iniciarRenombre(seleccion.value[0])
    if (e.key === 'Delete') eliminar()
    if ((e.ctrlKey || e.metaKey) && seleccion.value.length) {
        if (e.key === 'c') portapapeles.value = { modo: 'copiar', items: [...seleccion.value] }
        if (e.key === 'x') portapapeles.value = { modo: 'cortar', items: [...seleccion.value] }
    }
    if ((e.ctrlKey || e.metaKey) && e.key === 'v') pegar()
}
onMounted(() => document.addEventListener('keydown', alTeclado))
onBeforeUnmount(() => document.removeEventListener('keydown', alTeclado))
 
/* ---------- Utilidades de archivos ---------- */
function tipoArchivo(nombre) {
    if (nombre.endsWith('/')) return 'carpeta'
    const ext = nombre.split('.').pop().toLowerCase()
    if (['jpg', 'jpeg', 'png', 'webp', 'gif'].includes(ext)) return 'imagen'
    if (ext === 'pdf') return 'pdf'
    if (['mp3', 'wav', 'ogg'].includes(ext)) return 'audio'
    return 'archivo'
}
 
// Una ruta real es una imagen importada (monje) o un data:, no una subcarpeta como "proyecto-web/"
function esRutaReal(archivo) {
    return (archivo.includes('/') && !archivo.endsWith('/')) || archivo.startsWith('data:')
}
 
function nombreDesdeRuta(ruta) {
    return ruta.split('/').pop()
}
 
function nombreVisible(archivo) {
    return esRutaReal(archivo) ? nombreDesdeRuta(archivo) : archivo
}
</script>
 
<style scoped>
* {
    font-family: 'Courier New', monospace;
}
 
.carpetas-contenedor {
    position: relative;
    height: 100%;
    display: flex;
    flex-direction: column;
    padding: 0 20px;
}
 
.barra-comandos {
    flex-shrink: 0;
    margin-bottom: 12px;
}
 
.carpetas {
    flex: 1;
    min-height: 0;
    background: linear-gradient(180deg, #0a0015 0%, #1a0030 50%, #0a0015 100%);
    border: 1px solid rgba(131, 100, 255, 0.15);
    border-radius: 8px;
    box-shadow: 0 0 30px rgba(131, 100, 255, 0.05);
    padding: 16px 20px;
    margin-bottom: 12px;
    overflow-y: auto;
}
 
.contenido {
    flex: 1;
    min-height: 0;
    background: linear-gradient(180deg, #0a0015 0%, #1a0030 50%, #0a0015 100%);
    border: 1px solid rgba(131, 100, 255, 0.15);
    border-radius: 8px;
    box-shadow: 0 0 30px rgba(131, 100, 255, 0.05);
    padding: 16px 20px;
    overflow-y: auto;
}
 
.texto {
    font-size: 0.8rem;
    padding-bottom: 10px;
    border-bottom: 2px solid rgba(131, 100, 255, 0.3);
    color: #8a6eff;
    text-transform: uppercase;
    letter-spacing: 3px;
    text-shadow: 0 0 10px rgba(131, 100, 255, 0.3);
    margin-bottom: 12px;
}
 
.grid-carpetas {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
    gap: 16px;
}
 
.grid-carpetas.lista {
    grid-template-columns: 1fr;
    gap: 4px;
}
 
.item-carpeta {
    --glow: rgba(131, 100, 255, 0.7);
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 8px;
    border: 1px solid transparent;
    border-radius: 8px;
    cursor: pointer;
    transition: background 0.3s ease, border-color 0.3s ease, opacity 0.3s ease;
}
 
.lista .item-carpeta {
    flex-direction: row;
    gap: 12px;
    padding: 6px 10px;
}
 
.item-carpeta.seleccionada {
    background: rgba(131, 100, 255, 0.15);
    border-color: rgba(131, 100, 255, 0.4);
}
 
.item-carpeta.cortada {
    opacity: 0.45;
}
 
.icono-carpeta {
    width: 56px;
    height: 56px;
    filter: drop-shadow(0 0 6px var(--glow));
    animation: pulso 3s ease-in-out infinite;
    transition: filter 0.3s ease, transform 0.3s ease;
}
 
.lista .icono-carpeta {
    width: 24px;
    height: 24px;
}
 
.item-carpeta:hover .icono-carpeta {
    --glow: rgba(179, 136, 255, 0.9);
    transform: scale(1.12);
    animation-play-state: paused;
}
 
.item-carpeta.abierta .icono-carpeta {
    --glow: rgba(179, 136, 255, 1);
    filter: drop-shadow(0 0 14px var(--glow));
}
 
.item-carpeta.abierta .nombre-carpeta {
    color: #b388ff;
}
 
@keyframes pulso {
    50% { filter: drop-shadow(0 0 11px var(--glow)); }
}
 
@media (prefers-reduced-motion: reduce) {
    .icono-carpeta { animation: none; }
}
 
.nombre-carpeta {
    margin-top: 8px;
    color: #e8e8ff;
    font-size: 11px;
    text-transform: uppercase;
    letter-spacing: 1px;
    text-align: center;
}
 
.lista .nombre-carpeta {
    margin-top: 0;
    text-align: left;
}
 
.input-renombrar {
    width: 100%;
    margin-top: 8px;
    padding: 2px 4px;
    border: 1px solid rgba(131, 100, 255, 0.5);
    border-radius: 4px;
    background: rgba(131, 100, 255, 0.1);
    color: #d4c4ff;
    font-size: 11px;
    text-align: center;
    outline: none;
}
 
.lista .input-renombrar {
    margin-top: 0;
    text-align: left;
}
 
.subtitulo-preview {
    color: rgba(232, 232, 255, 0.5);
    font-size: 11px;
    margin-bottom: 14px;
    text-transform: uppercase;
    letter-spacing: 1px;
}
 
.contenido-items {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(90px, 1fr));
    gap: 18px;
}
 
.contenido-items.lista {
    grid-template-columns: 1fr;
    gap: 6px;
}
 
.item-archivo {
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
}
 
.lista .item-archivo {
    flex-direction: row;
    gap: 12px;
    text-align: left;
}
 
.icono-archivo {
    width: 40px;
    height: 40px;
}
 
.lista .icono-archivo {
    width: 24px;
    height: 24px;
}
 
.icono-imagen { color: #b388ff; }
.icono-pdf { color: #ff6e6e; }
.icono-audio { color: #6ee7ff; }
.icono-subcarpeta { color: #8364ff; }
.icono-generico { color: #a89bd0; }
 
.miniatura-real {
    width: 48px;
    height: 48px;
    object-fit: cover;
    border-radius: 6px;
    border: 1px solid rgba(131, 100, 255, 0.4);
    box-shadow: 0 0 10px rgba(131, 100, 255, 0.3);
}
 
.lista .miniatura-real {
    width: 24px;
    height: 24px;
}
 
.nombre-archivo {
    margin-top: 6px;
    font-size: 10.5px;
    color: rgba(232, 232, 255, 0.75);
    word-break: break-word;
}
 
.lista .nombre-archivo {
    margin-top: 0;
}
 
.placeholder-preview {
    color: rgba(232, 232, 255, 0.35);
    font-size: 12px;
    font-style: italic;
}
 
.aviso {
    position: absolute;
    left: 50%;
    bottom: 20px;
    transform: translateX(-50%);
    padding: 8px 14px;
    border: 1px solid rgba(131, 100, 255, 0.4);
    border-radius: 8px;
    background: #12062a;
    color: #d4c4ff;
    font-size: 0.8rem;
    box-shadow: 0 0 20px rgba(131, 100, 255, 0.2);
}
 
.carpetas::-webkit-scrollbar,
.contenido::-webkit-scrollbar {
    width: 4px;
}
 
.carpetas::-webkit-scrollbar-track,
.contenido::-webkit-scrollbar-track {
    background: rgba(131, 100, 255, 0.02);
}
 
.carpetas::-webkit-scrollbar-thumb,
.contenido::-webkit-scrollbar-thumb {
    background: linear-gradient(180deg, #8364ff, #b388ff);
    border-radius: 2px;
    box-shadow: 0 0 15px rgba(131, 100, 255, 0.2);
}
</style>