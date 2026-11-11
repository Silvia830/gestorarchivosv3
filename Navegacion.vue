<template>
    <div class="navegacion col-4">
        <div class="navegar">
            <h2 class="textos">Navegación</h2>
            <div
                v-for="item in navegaciones"
                :key="item.nombre"
                class="nav-item"
                :class="{ active: item.activo }"
                @click="marcarActivo(item)"
            >
                <font-awesome-icon :icon="item.icono" class="nav-icon" />
                <span>{{ item.nombre }}</span>
            </div>
        </div>
 
        <div class="carpetas">
            <h2 class="textos">Bibliotecas</h2>
 
            <svg width="0" height="0" style="position:absolute">
                <linearGradient id="grad-biblioteca" x1="0%" y1="0%" x2="100%" y2="100%">
                    <stop offset="0%" stop-color="#8364ff" />
                    <stop offset="100%" stop-color="#b388ff" />
                </linearGradient>
            </svg>
 
            <div class="lista-bibliotecas">
                <div
                    v-for="biblioteca in bibliotecas"
                    :key="biblioteca.nombre"
                    class="item-biblioteca"
                    :class="{ activa: carpetaAbiertaNombre === biblioteca.nombre }"
                    @click="abrirBiblioteca(biblioteca)"
                >
                    <svg class="icono-biblioteca" viewBox="0 0 24 24" fill="none">
                        <path
                            :d="biblioteca.icono"
                            stroke="currentColor"
                            stroke-width="1.5"
                        />
                    </svg>
                    <span class="nombre-biblioteca">{{ biblioteca.nombre }}</span>
                </div>
            </div>
        </div>
 
        <div class="dispositivos">
            <h2 class="textos">Dispositivos</h2>
 
            <div class="lista-dispositivos">
                <div
                    v-for="d in dispositivos"
                    :key="d.id"
                    class="item-dispositivo"
                    :class="{ activa: rutaActual === d.montaje }"
                    tabindex="0"
                    @click="irA(d.montaje)"
                    @keydown.enter="irA(d.montaje)"
                >
                    <svg class="icono-dispositivo" viewBox="0 0 24 24" fill="none">
                        <path
                            :d="iconosDispositivo[d.tipo]"
                            stroke="currentColor"
                            stroke-width="1.5"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                        />
                    </svg>
 
                    <div class="info-dispositivo">
                        <span class="nombre-dispositivo">{{ d.nombre }}</span>
                        <div
                            class="barra-uso"
                            role="progressbar"
                            :aria-valuenow="porcentaje(d)"
                            aria-valuemin="0"
                            aria-valuemax="100"
                            :aria-label="'Espacio usado en ' + d.nombre"
                        >
                            <div
                                class="barra-uso-relleno"
                                :class="{ lleno: porcentaje(d) > 85 }"
                                :style="{ width: porcentaje(d) + '%' }"
                            ></div>
                        </div>
                        <span class="detalle-dispositivo">{{ d.libre }} GB libres de {{ d.total }} GB</span>
                    </div>
 
                    <button
                        v-if="d.extraible"
                        class="boton-expulsar"
                        title="Expulsar"
                        aria-label="Expulsar"
                        @click.stop="expulsar(d)"
                    >
                        <svg viewBox="0 0 24 24" fill="none">
                            <path d="M12 5l7 8H5l7-8zM5 17h14" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" />
                        </svg>
                    </button>
                </div>
            </div>
        </div>
    </div>
</template>
 
<script setup>
import { ref, watch } from "vue"
import { rutaActual, carpetaAbiertaNombre } from '../store/ruta.js'
import { irA } from '../store/navegacion.js'
 
const navegaciones = ref([
    { nombre: "Inicio", icono: "home", ruta: "/home/usuario", activo: true },
    { nombre: "Recientes", icono: "clock", ruta: "/home/usuario/Recientes", activo: false },
    { nombre: "Favoritos", icono: "star", ruta: "/home/usuario/Favoritos", activo: false },
    { nombre: "Papelera", icono: "trash", ruta: "/home/usuario/Papelera", activo: false }
])
 
const marcarActivo = (item) => {
    carpetaAbiertaNombre.value = null // Inicio/Recientes/etc no abren carpeta en Vista Previa
    irA(item.ruta)
}
 
// El resaltado sigue a la ruta, así también cambia con Atrás, Adelante y Subir
watch(rutaActual, (r) => {
    navegaciones.value.forEach(nav => nav.activo = nav.ruta === r)
}, { immediate: true })
 
const bibliotecas = ref([
    { nombre: 'Documentos', icono: 'M3 6a2 2 0 0 1 2-2h4l2 2h8a2 2 0 0 1 2 2v9a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V6z' },
    { nombre: 'Imagenes', icono: 'M4 5h16a1 1 0 0 1 1 1v12a1 1 0 0 1-1 1H4a1 1 0 0 1-1-1V6a1 1 0 0 1 1-1zM3 16l5-5 4 4 5-6 4 5' },
    { nombre: 'Musica', icono: 'M9 18V5l12-2v13M9 18a3 3 0 1 1-6 0 3 3 0 0 1 6 0zM21 16a3 3 0 1 1-6 0 3 3 0 0 1 6 0z' },
    { nombre: 'Videos', icono: 'M4 5h12a1 1 0 0 1 1 1v3l4-2v10l-4-2v3a1 1 0 0 1-1 1H4a1 1 0 0 1-1-1V6a1 1 0 0 1 1-1z' },
])
 
const abrirBiblioteca = (biblioteca) => {
    carpetaAbiertaNombre.value = biblioteca.nombre // abre el contenido en Vista Previa
    irA('/home/usuario/' + biblioteca.nombre)
}
 
/* ---------- Dispositivos ---------- */
// Datos de ejemplo (tamaños en GB). En Debian los reales salen de los puntos de montaje:
//   "/"                       -> disco del sistema (equivale al C: de Windows)
//   "/home"                   -> partición personal, si está separada
//   "/media/<usuario>/<etiqueta>" -> USB y discos externos
// Con Tauri conviene pedirlos al backend (por ejemplo con el crate "sysinfo")
// y reemplazar este arreglo con: dispositivos.value = await invoke('listar_dispositivos')
const dispositivos = ref([
    { id: 'raiz', nombre: 'Sistema de archivos', tipo: 'sistema', montaje: '/', total: 250, libre: 120, extraible: false },
    { id: 'home', nombre: 'Personal', tipo: 'disco', montaje: '/home', total: 500, libre: 310, extraible: false },
    { id: 'usb', nombre: 'USB_ESTUDIO', tipo: 'usb', montaje: '/media/usuario/USB_ESTUDIO', total: 32, libre: 4, extraible: true },
])
 
const iconosDispositivo = {
    sistema: 'M4 15h16a1 1 0 0 1 1 1v3a1 1 0 0 1-1 1H4a1 1 0 0 1-1-1v-3a1 1 0 0 1 1-1zM5.5 7L3 15M18.5 7L21 15M5.5 7h13M17 17.5h.01',
    disco: 'M4 15h16a1 1 0 0 1 1 1v3a1 1 0 0 1-1 1H4a1 1 0 0 1-1-1v-3a1 1 0 0 1 1-1zM5.5 7L3 15M18.5 7L21 15M5.5 7h13M17 17.5h.01',
    usb: 'M8 2h8v6H8zM6 8h12v11a3 3 0 0 1-3 3H9a3 3 0 0 1-3-3V8zM10.5 4.5h.01M13.5 4.5h.01',
}
 
const porcentaje = (d) => Math.round(((d.total - d.libre) / d.total) * 100)
 
function expulsar(d) {
    // Aquí iría la llamada al backend (en Debian: udisksctl unmount + power-off)
    dispositivos.value = dispositivos.value.filter(x => x.id !== d.id)
    if (rutaActual.value.startsWith(d.montaje)) irA('/home/usuario')
}
</script>
 
<style scoped>
* {
    color: white;
    font-family: 'Courier New', monospace;
}
 
.navegacion {
    height: 100vh;
    display: flex;
    flex-direction: column;
    background: linear-gradient(180deg, #0a0015 0%, #1a0030 50%, #0a0015 100%);
    border-right: 2px solid rgba(131, 100, 255, 0.3);
    box-shadow: 4px 0 30px rgba(131, 100, 255, 0.1);
}
 
.navegar {
    flex-shrink: 0;
    width: 100%;
    padding-bottom: 8px;
}
 
/* Bibliotecas: ocupa el espacio sobrante. El título queda fijo y solo la lista hace scroll */
.carpetas {
    flex: 1 1 0;
    min-height: 0;
    width: 100%;
    display: flex;
    flex-direction: column;
    overflow: hidden;
}
 
.dispositivos {
    flex-shrink: 0;
    width: 100%;
    max-height: 40%;
    overflow-y: auto;
    padding-bottom: 12px;
    border-top: 1px solid rgba(131, 100, 255, 0.15);
}
 
.textos {
    font-size: 0.8rem;
    padding-left: 20px;
    padding-top: 20px;
    padding-bottom: 10px;
 
    color: #8a6eff;
    text-transform: uppercase;
    letter-spacing: 3px;
    text-shadow: 0 0 10px rgba(131, 100, 255, 0.3);
}
 
.nav-item {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 10px 20px;
    cursor: pointer;
    transition: all 0.3s ease;
    margin: 4px 10px;
    border-radius: 8px;
    position: relative;
    border: 1px solid transparent;
}
 
.nav-item:hover {
    transform: translateX(6px);
    background: rgba(131, 100, 255, 0.08);
 
    box-shadow: 0 0 20px rgba(131, 100, 255, 0.05);
}
 
.nav-item:hover .nav-icon {
    color: #a88bff;
    text-shadow: 0 0 20px rgba(131, 100, 255, 0.5);
}
 
.nav-item.active {
    background: rgba(131, 100, 255, 0.15);
    border: 1px solid rgba(131, 100, 255, 0.4);
    box-shadow:
        0 0 20px rgba(131, 100, 255, 0.2),
        inset 0 0 30px rgba(131, 100, 255, 0.05);
    transform: translateX(6px);
}
 
.nav-item.active::after {
    content: '';
    position: absolute;
    left: 0;
    top: 20%;
    height: 60%;
    width: 3px;
    background: linear-gradient(180deg,
        transparent,
        #8364ff,
        #b388ff,
        #8364ff,
        transparent
    );
    border-radius: 2px;
    box-shadow:
        0 0 15px #8364ff,
        0 0 30px rgba(131, 100, 255, 0.3);
    animation: pulseNeon 2s ease-in-out infinite;
}
 
.nav-item.active .nav-icon {
    color: #b388ff;
    text-shadow:
        0 0 20px rgba(131, 100, 255, 0.6),
        0 0 40px rgba(131, 100, 255, 0.3),
        0 0 60px rgba(131, 100, 255, 0.1);
}
 
.nav-item.active span {
    color: #d4c4ff;
    text-shadow: 0 0 20px rgba(131, 100, 255, 0.3);
}
 
.nav-icon {
    width: 18px;
    color: #6a4e9a;
    font-size: 16px;
    transition: all 0.3s ease;
}
 
.nav-item span {
    font-size: 0.9rem;
    color: #8a7aaa;
    transition: all 0.3s ease;
    letter-spacing: 0.5px;
}
 
.nav-item:not(.active):hover .nav-icon {
    color: #a88bff;
    text-shadow: 0 0 15px rgba(131, 100, 255, 0.3);
}
 
.nav-item:not(.active):hover span {
    color: #b8a8e0;
}
 
@keyframes pulseNeon {
    0%, 100% {
        opacity: 1;
        box-shadow: 0 0 15px #8364ff, 0 0 30px rgba(131, 100, 255, 0.3);
    }
    50% {
        opacity: 0.7;
        box-shadow: 0 0 25px #8364ff, 0 0 50px rgba(131, 100, 255, 0.5);
    }
}
 
@media (prefers-reduced-motion: reduce) {
    .nav-item.active::after,
    .icono-biblioteca { animation: none; }
}
 
/* Scrollbar: fondo transparente y solo se ve la barra azul */
.lista-bibliotecas::-webkit-scrollbar,
.dispositivos::-webkit-scrollbar {
    width: 6px;
    background: transparent;
}
 
.lista-bibliotecas::-webkit-scrollbar-track,
.dispositivos::-webkit-scrollbar-track,
.lista-bibliotecas::-webkit-scrollbar-corner,
.dispositivos::-webkit-scrollbar-corner {
    background: transparent;
}
 
.lista-bibliotecas::-webkit-scrollbar-thumb,
.dispositivos::-webkit-scrollbar-thumb {
    background: linear-gradient(180deg, #5b8cff, #8364ff);
    border-radius: 3px;
    box-shadow: 0 0 10px rgba(91, 140, 255, 0.45);
}
 
.lista-bibliotecas::-webkit-scrollbar-thumb:hover,
.dispositivos::-webkit-scrollbar-thumb:hover {
    background: linear-gradient(180deg, #7aa2ff, #a88bff);
}
 
/* Quita las flechas de arriba y abajo */
.lista-bibliotecas::-webkit-scrollbar-button,
.dispositivos::-webkit-scrollbar-button {
    display: none;
    width: 0;
    height: 0;
}
 
.lista-bibliotecas {
    flex: 1 1 0;
    min-height: 0;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    gap: 2px;
    padding: 4px 0;
}
 
.item-biblioteca {
    --glow: rgba(131, 100, 255, 0.7);
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 9px 20px;
    margin: 2px 10px;
    border-radius: 8px;
    border: 1px solid transparent;
    cursor: pointer;
    transition: all 0.3s ease;
}
 
.item-biblioteca:hover {
    transform: translateX(6px);
    background: rgba(131, 100, 255, 0.08);
    box-shadow: 0 0 20px rgba(131, 100, 255, 0.05);
}
 
.item-biblioteca.activa {
    background: rgba(131, 100, 255, 0.15);
    border: 1px solid rgba(131, 100, 255, 0.4);
    box-shadow: 0 0 20px rgba(131, 100, 255, 0.2), inset 0 0 30px rgba(131, 100, 255, 0.05);
}
 
.item-biblioteca.activa .icono-biblioteca { color: #b388ff; }
.item-biblioteca.activa .nombre-biblioteca { color: #d4c4ff; }
 
.icono-biblioteca {
    width: 18px;
    height: 18px;
    flex-shrink: 0;
    color: #6a4e9a;
    filter: drop-shadow(0 0 4px var(--glow));
    animation: pulso-biblioteca 3s ease-in-out infinite;
    transition: color 0.3s ease, filter 0.3s ease, transform 0.3s ease;
}
 
.item-biblioteca:hover .icono-biblioteca {
    color: #a88bff;
    --glow: rgba(179, 136, 255, 0.9);
    transform: scale(1.1);
    animation-play-state: paused;
}
 
@keyframes pulso-biblioteca {
    50% { filter: drop-shadow(0 0 8px var(--glow)); }
}
 
.nombre-biblioteca {
    font-size: 12.5px;
    letter-spacing: 0.6px;
    color: #e8e8ff;
}
 
/* ---------- Dispositivos ---------- */
.lista-dispositivos {
    display: flex;
    flex-direction: column;
    gap: 2px;
    padding: 4px 0;
}
 
.item-dispositivo {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 9px 14px 9px 20px;
    margin: 2px 10px;
    border-radius: 8px;
    border: 1px solid transparent;
    cursor: pointer;
    transition: all 0.3s ease;
}
 
.item-dispositivo:hover {
    transform: translateX(6px);
    background: rgba(131, 100, 255, 0.08);
}
 
.item-dispositivo:focus-visible {
    outline: 2px solid #b388ff;
}
 
.item-dispositivo.activa {
    background: rgba(131, 100, 255, 0.15);
    border-color: rgba(131, 100, 255, 0.4);
    box-shadow: 0 0 20px rgba(131, 100, 255, 0.2), inset 0 0 30px rgba(131, 100, 255, 0.05);
}
 
.icono-dispositivo {
    width: 22px;
    height: 22px;
    flex-shrink: 0;
    color: #8a6eff;
}
 
.item-dispositivo.activa .icono-dispositivo { color: #b388ff; }
 
.info-dispositivo {
    display: flex;
    flex-direction: column;
    gap: 4px;
    flex: 1;
    min-width: 0;
}
 
.nombre-dispositivo {
    font-size: 12.5px;
    letter-spacing: 0.6px;
    color: #e8e8ff;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}
 
.barra-uso {
    height: 5px;
    border-radius: 3px;
    background: rgba(131, 100, 255, 0.15);
    overflow: hidden;
}
 
.barra-uso-relleno {
    height: 100%;
    border-radius: 3px;
    background: linear-gradient(90deg, #8364ff, #b388ff);
    box-shadow: 0 0 8px rgba(131, 100, 255, 0.5);
}
 
/* Disco casi lleno: se avisa en rojo, como el Explorador de Windows */
.barra-uso-relleno.lleno {
    background: linear-gradient(90deg, #ff6e6e, #ff9a9a);
    box-shadow: 0 0 8px rgba(255, 110, 110, 0.5);
}
 
.detalle-dispositivo {
    font-size: 10px;
    color: #8a7aaa;
    letter-spacing: 0.3px;
}
 
.boton-expulsar {
    width: 28px;
    height: 28px;
    flex-shrink: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    border: 1px solid transparent;
    border-radius: 6px;
    background: transparent;
    cursor: pointer;
    transition: all 0.3s ease;
}
 
.boton-expulsar svg {
    width: 16px;
    height: 16px;
    color: #8a7aaa;
}
 
.boton-expulsar:hover {
    background: rgba(131, 100, 255, 0.15);
    border-color: rgba(131, 100, 255, 0.3);
}
 
.boton-expulsar:hover svg { color: #d4c4ff; }
.boton-expulsar:focus-visible { outline: 2px solid #b388ff; }
</style>
 