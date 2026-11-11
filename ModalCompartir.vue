<template>
  <Teleport to="body">
    <div v-if="visible" class="fondo" @click.self="cerrar">
      <div ref="ventana" class="ventana" role="dialog" aria-modal="true" aria-labelledby="titulo-compartir">
        <header class="cabecera">
          <h2 id="titulo-compartir" class="titulo">Compartir</h2>
          <button class="boton-cerrar" aria-label="Cerrar" @click="cerrar">
            <svg viewBox="0 0 24 24" fill="none">
              <path d="M6 6l12 12M18 6L6 18" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" />
            </svg>
          </button>
        </header>
 
        <p class="resumen">{{ resumen }}</p>
        <ul class="lista-rutas">
          <li v-for="r in rutas.slice(0, 3)" :key="r">{{ r }}</li>
          <li v-if="rutas.length > 3" class="mas">y {{ rutas.length - 3 }} más</li>
        </ul>
 
        <div class="opciones">
          <button
            v-for="o in opciones"
            :key="o.id"
            class="opcion"
            :disabled="!o.activa"
            @click="o.accion && o.accion()"
          >
            <svg class="icono-opcion" viewBox="0 0 24 24" fill="none">
              <path :d="o.icono" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" />
            </svg>
            <span class="texto-opcion">
              <span class="nombre-opcion">{{ o.titulo }}</span>
              <span class="detalle-opcion">{{ o.detalle }}</span>
            </span>
            <span v-if="!o.activa" class="etiqueta">Próximamente</span>
          </button>
        </div>
      </div>
    </div>
  </Teleport>
</template>
 
<script setup>
import { ref, computed, watch, nextTick, onMounted, onBeforeUnmount } from 'vue'
 
const props = defineProps({
  visible: { type: Boolean, default: false },
  rutas: { type: Array, default: () => [] },
})
const emit = defineEmits(['cerrar', 'aviso'])
 
const ventana = ref(null)
const cerrar = () => emit('cerrar')
 
const resumen = computed(() =>
  props.rutas.length === 1 ? '1 elemento seleccionado' : `${props.rutas.length} elementos seleccionados`
)
 
async function copiarRutas() {
  try {
    await navigator.clipboard.writeText(props.rutas.join('\n'))
    emit('aviso', props.rutas.length > 1 ? 'Rutas copiadas' : 'Ruta copiada')
  } catch {
    emit('aviso', 'No se pudo copiar al portapapeles')
  }
  cerrar()
}
 
function enviarCorreo() {
  const asunto = encodeURIComponent('Compartido desde EstudiOS')
  const cuerpo = encodeURIComponent('Te comparto:\n' + props.rutas.join('\n'))
  window.location.href = `mailto:?subject=${asunto}&body=${cuerpo}`
  emit('aviso', 'Abriendo tu programa de correo')
  cerrar()
  async function abrirExterno(url) {
  try {
    const { openUrl } = await import('@tauri-apps/plugin-opener')
    await openUrl(url)
  } catch {
    window.open(url, '_blank', 'noopener')
  }
}

const textoCompartir = () => 'Te comparto:\n' + props.rutas.join('\n')

function enviarWhatsapp() {
  abrirExterno(`https://wa.me/?text=${encodeURIComponent(textoCompartir())}`)
  emit('aviso', 'Abriendo WhatsApp')
  cerrar()
}

function enviarGmail() {
  const asunto = encodeURIComponent('Compartido desde EstudiOS')
  const cuerpo = encodeURIComponent(textoCompartir())
  abrirExterno(`https://mail.google.com/mail/?view=cm&fs=1&su=${asunto}&body=${cuerpo}`)
  emit('aviso', 'Abriendo Gmail')
  cerrar()
}

// Discord y GitHub no permiten prellenar texto: copiamos y abrimos el sitio
async function copiarYAbrir(url, nombre) {
  try {
    await navigator.clipboard.writeText(props.rutas.join('\n'))
    emit('aviso', `Rutas copiadas, pégalas en ${nombre}`)
  } catch {
    emit('aviso', 'No se pudo copiar al portapapeles')
  }
  abrirExterno(url)
  cerrar()
}
const enviarDiscord = () => copiarYAbrir('https://discord.com/app', 'Discord')
const enviarGithub = () => copiarYAbrir('https://gist.github.com/', 'GitHub')
}
 
// Las dos últimas necesitan el backend de Tauri, por eso están desactivadas por ahora
const opciones = computed(() => [
  {
    id: 'ruta', activa: true, accion: copiarRutas,
    titulo: 'Copiar ruta',
    detalle: props.rutas.length > 1 ? 'Copia las rutas, una por línea' : 'Copia la ruta al portapapeles',
    icono: 'M9 9h11v11H9zM5 15V5a1 1 0 0 1 1-1h10',
  },
  {
    id: 'correo', activa: true, accion: enviarCorreo,
    titulo: 'Enviar por correo',
    detalle: 'Abre tu programa de correo con las rutas en el mensaje',
    icono: 'M3 6h18v12H3zM3 7l9 6 9-6',
  },
  {
    id: 'red', activa: false,
    titulo: 'Compartir en la red local',
    detalle: 'Carpeta compartida con Samba',
    icono: 'M9 3h6v5H9zM12 8v4M5 12h14M5 12v3M19 12v3M3 15h4v5H3zM17 15h4v5h-4z',
  },
  {
    id: 'zip', activa: false,
    titulo: 'Comprimir en ZIP',
    detalle: 'Crea un archivo .zip para enviar',
    icono: 'M4 4h16v4H4zM5 8v12h14V8M10 12h4',
  },
])
 
/* Esc cierra; Tab se queda dentro de la ventana */
function alTeclado(e) {
  if (!props.visible) return
  if (e.key === 'Escape') return cerrar()
  if (e.key !== 'Tab' || !ventana.value) return
  const enfocables = [...ventana.value.querySelectorAll('button:not(:disabled)')]
  if (!enfocables.length) return
  const primero = enfocables[0], ultimo = enfocables[enfocables.length - 1]
  if (e.shiftKey && document.activeElement === primero) { e.preventDefault(); ultimo.focus() }
  else if (!e.shiftKey && document.activeElement === ultimo) { e.preventDefault(); primero.focus() }
}
onMounted(() => document.addEventListener('keydown', alTeclado))
onBeforeUnmount(() => document.removeEventListener('keydown', alTeclado))
 
// Al abrir, el foco pasa a la primera opción disponible
watch(() => props.visible, async (v) => {
  if (!v) return
  await nextTick()
  ventana.value?.querySelector('.opcion:not(:disabled)')?.focus()
})
</script>
 
<style scoped>
* { font-family: 'Courier New', monospace; }
 
.fondo {
  position: fixed;
  inset: 0;
  z-index: 100;
  display: grid;
  place-items: center;
  padding: 20px;
  background: rgba(5, 0, 12, 0.72);
  backdrop-filter: blur(3px);
}
 
.ventana {
  width: min(460px, 100%);
  max-height: 90vh;
  overflow-y: auto;
  padding: 20px 22px 22px;
  border-radius: 12px;
  border: 1px solid rgba(131, 100, 255, 0.35);
  background: linear-gradient(180deg, #14062a 0%, #0a0015 100%);
  box-shadow: 0 0 40px rgba(131, 100, 255, 0.2), 0 20px 60px rgba(0, 0, 0, 0.6);
}
 
.cabecera {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-bottom: 10px;
  border-bottom: 2px solid rgba(131, 100, 255, 0.3);
}
 
.titulo {
  margin: 0;
  font-size: 0.8rem;
  color: #8a6eff;
  text-transform: uppercase;
  letter-spacing: 3px;
  text-shadow: 0 0 10px rgba(131, 100, 255, 0.3);
}
 
.boton-cerrar {
  width: 28px;
  height: 28px;
  display: grid;
  place-items: center;
  border: 1px solid transparent;
  border-radius: 6px;
  background: transparent;
  color: #8a7aaa;
  cursor: pointer;
  transition: all 0.3s ease;
}
.boton-cerrar svg { width: 16px; height: 16px; }
.boton-cerrar:hover { background: rgba(255, 0, 0, 0.15); border-color: rgba(255, 0, 0, 0.3); color: #ff6b6b; }
.boton-cerrar:focus-visible { outline: 2px solid #b388ff; }
 
.resumen {
  margin: 14px 0 6px;
  font-size: 0.85rem;
  color: #d4c4ff;
}
 
.lista-rutas {
  margin: 0 0 16px;
  padding: 8px 10px;
  list-style: none;
  border-radius: 8px;
  border: 1px solid rgba(131, 100, 255, 0.15);
  background: rgba(131, 100, 255, 0.05);
  font-size: 11px;
  color: #a89bd0;
  word-break: break-all;
}
.lista-rutas li + li { margin-top: 3px; }
.lista-rutas .mas { color: #6a4e9a; font-style: italic; }
 
.opciones { display: flex; flex-direction: column; gap: 6px; }
 
.opcion {
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 10px 12px;
  border: 1px solid transparent;
  border-radius: 8px;
  background: transparent;
  text-align: left;
  cursor: pointer;
  transition: all 0.3s ease;
}
.opcion:hover:not(:disabled) {
  background: rgba(131, 100, 255, 0.12);
  border-color: rgba(131, 100, 255, 0.35);
}
.opcion:focus-visible { outline: 2px solid #b388ff; }
.opcion:disabled { opacity: 0.45; cursor: default; }
 
.icono-opcion { width: 24px; height: 24px; flex-shrink: 0; color: #b388ff; }
 
.texto-opcion { display: flex; flex-direction: column; gap: 3px; flex: 1; min-width: 0; }
.nombre-opcion { font-size: 0.9rem; color: #e8e8ff; }
.detalle-opcion { font-size: 11px; color: #8a7aaa; }
 
.etiqueta {
  flex-shrink: 0;
  padding: 2px 8px;
  border-radius: 10px;
  border: 1px solid rgba(131, 100, 255, 0.3);
  font-size: 10px;
  color: #a89bd0;
}
</style>