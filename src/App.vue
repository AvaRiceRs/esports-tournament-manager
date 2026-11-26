
<script setup>
import { ref, computed } from 'vue'

import genGLogo from './assets/logos/gen-g.jpg'
import nrgLogo from './assets/logos/nrg.jpg'
import sentinelsLogo from './assets/logos/sentinels.png'
import fnaticLogo from './assets/logos/fnatic.png'
import liquidLogo from './assets/logos/team-liquid.png'
import paperRexLogo from './assets/logos/paper-rex.png'
import t1Logo from './assets/logos/t1.png'
import g2Logo from './assets/logos/g2.png'

const equiposIniciales = [
  { id: 1, nombre: 'Gen.G', juego: 'VALORANT', region: 'Pacific', victorias: 8, derrotas: 2, logo: genGLogo },
  { id: 2, nombre: 'Gen.G', juego: 'League of Legends', region: 'LCK', victorias: 6, derrotas: 3, logo: genGLogo },
  { id: 3, nombre: 'NRG', juego: 'VALORANT', region: 'Americas', victorias: 7, derrotas: 3, logo: nrgLogo },
  { id: 4, nombre: 'NRG', juego: 'Rocket League', region: 'North America', victorias: 5, derrotas: 2, logo: nrgLogo },
  { id: 5, nombre: 'Sentinels', juego: 'VALORANT', region: 'Americas', victorias: 6, derrotas: 4, logo: sentinelsLogo },
  { id: 6, nombre: 'Fnatic', juego: 'VALORANT', region: 'EMEA', victorias: 9, derrotas: 1, logo: fnaticLogo },
  { id: 7, nombre: 'Team Liquid', juego: 'Counter-Strike 2', region: 'Europe', victorias: 5, derrotas: 5, logo: liquidLogo },
  { id: 8, nombre: 'Paper Rex', juego: 'VALORANT', region: 'Pacific', victorias: 7, derrotas: 3, logo: paperRexLogo },
  { id: 9, nombre: 'T1', juego: 'League of Legends', region: 'LCK', victorias: 8, derrotas: 2, logo: t1Logo },
  { id: 10, nombre: 'G2 Esports', juego: 'Counter-Strike 2', region: 'Europe', victorias: 6, derrotas: 4, logo: g2Logo }
]

const equipos = ref([...equiposIniciales])
const busqueda = ref('')
const filtroJuego = ref('Todos')
const filtroRegion = ref('Todas')
const equipoSeleccionado = ref(equipos.value[0])
const contadorEventos = ref(0)
const contadorBusquedas = ref(0)

const nuevoEquipo = ref({
  nombre: '',
  juego: 'VALORANT',
  region: 'Americas',
  victorias: 0,
  derrotas: 0
})

const nuevaImagen = ref('')
const vistaPreviaLogo = ref('')

const historial = ref([
  { id: 1, mensaje: 'Dashboard iniciado', hora: 'Sistema' },
  { id: 2, mensaje: 'Equipos de esports cargados', hora: 'Sistema' }
])

function guardarEquipos() {
  try {
    localStorage.setItem('equiposEsports', JSON.stringify(equipos.value))
    return true
  } catch (error) {
    console.error('Error al guardar los equipos:', error)
    alert('No se pudieron guardar los equipos. El almacenamiento del navegador podría estar lleno.')
    return false
  }
}

function cargarEquipos() {
  try {
    const equiposGuardados = localStorage.getItem('equiposEsports')

    if (equiposGuardados) {
      const datos = JSON.parse(equiposGuardados)

      if (Array.isArray(datos)) {
        equipos.value = datos
      }
    }
  } catch (error) {
    console.error('Error al cargar los equipos:', error)
  }

  equipoSeleccionado.value = equipos.value[0] || null
}

cargarEquipos()

function seleccionarImagen(event) {
  const archivo = event.target.files[0]

  if (!archivo) return

  const formatosPermitidos = ['image/jpeg', 'image/png', 'image/webp']

  if (!formatosPermitidos.includes(archivo.type)) {
    alert('Selecciona una imagen JPG, PNG o WEBP.')
    event.target.value = ''
    return
  }

  if (archivo.size > 1024 * 1024) {
    alert('La imagen no debe superar 1 MB.')
    event.target.value = ''
    return
  }

  const lector = new FileReader()

  lector.onload = () => {
    nuevaImagen.value = lector.result
    vistaPreviaLogo.value = lector.result
  }

  lector.onerror = () => {
    alert('No se pudo leer la imagen seleccionada.')
    event.target.value = ''
  }

  lector.readAsDataURL(archivo)
}

const juegos = computed(() => [
  'Todos',
  ...new Set(equipos.value.map(equipo => equipo.juego))
])

const regiones = computed(() => [
  'Todas',
  ...new Set(equipos.value.map(equipo => equipo.region))
])

const equiposFiltrados = computed(() => {
  return equipos.value.filter(equipo => {
    const texto = busqueda.value.toLowerCase()

    const coincideBusqueda =
      equipo.nombre.toLowerCase().includes(texto) ||
      equipo.juego.toLowerCase().includes(texto) ||
      equipo.region.toLowerCase().includes(texto)

    const coincideJuego =
      filtroJuego.value === 'Todos' ||
      equipo.juego === filtroJuego.value

    const coincideRegion =
      filtroRegion.value === 'Todas' ||
      equipo.region === filtroRegion.value

    return coincideBusqueda && coincideJuego && coincideRegion
  })
})

const totalVictorias = computed(() =>
  equipos.value.reduce((total, equipo) => total + Number(equipo.victorias), 0)
)

const totalDerrotas = computed(() =>
  equipos.value.reduce((total, equipo) => total + Number(equipo.derrotas), 0)
)

const totalJuegos = computed(() => juegos.value.length - 1)

function registrarActividad(mensaje) {
  const hora = new Date().toLocaleTimeString('es-MX', {
    hour: '2-digit',
    minute: '2-digit'
  })

  historial.value.unshift({
    id: Date.now() + Math.random(),
    mensaje,
    hora
  })

  historial.value = historial.value.slice(0, 8)
}

function registrarBusqueda() {
  contadorBusquedas.value++
}

function seleccionarEquipo(equipo) {
  equipoSeleccionado.value = equipo
  contadorEventos.value++

  registrarActividad(`Equipo seleccionado: ${equipo.nombre} (${equipo.juego})`)
}

function agregarEquipo() {
  const nombre = nuevoEquipo.value.nombre.trim()

  if (!nombre) {
    alert('Escribe el nombre del equipo.')
    return
  }

  const equipoDuplicado = equipos.value.some(equipo =>
    equipo.nombre.trim().toLowerCase() === nombre.toLowerCase() &&
    equipo.juego === nuevoEquipo.value.juego
  )

  if (equipoDuplicado) {
    alert('Este equipo ya está registrado en este videojuego.')
    return
  }

  if (!nuevaImagen.value) {
    alert('Selecciona el logotipo del equipo.')
    return
  }

  const victorias = Number(nuevoEquipo.value.victorias)
  const derrotas = Number(nuevoEquipo.value.derrotas)

  if (
    !Number.isInteger(victorias) ||
    !Number.isInteger(derrotas) ||
    victorias < 0 ||
    derrotas < 0
  ) {
    alert('Las victorias y derrotas deben ser números enteros positivos.')
    return
  }

  const equipo = {
    id: Date.now(),
    nombre,
    juego: nuevoEquipo.value.juego,
    region: nuevoEquipo.value.region,
    victorias,
    derrotas,
    logo: nuevaImagen.value
  }

  equipos.value.push(equipo)

  if (!guardarEquipos()) {
    equipos.value.pop()
    return
  }

  equipoSeleccionado.value = equipo
  contadorEventos.value++

  registrarActividad(`Nuevo equipo agregado: ${nombre}`)

  nuevoEquipo.value = {
    nombre: '',
    juego: 'VALORANT',
    region: 'Americas',
    victorias: 0,
    derrotas: 0
  }

  nuevaImagen.value = ''
  vistaPreviaLogo.value = ''

  const inputImagen = document.getElementById('logo-input')

  if (inputImagen) {
    inputImagen.value = ''
  }
}

function eliminarEquipo(equipo) {
  const confirmar = confirm(
    `¿Estás seguro de eliminar a ${equipo.nombre} de ${equipo.juego}?`
  )

  if (!confirmar) return

  const equiposAnteriores = [...equipos.value]

  equipos.value = equipos.value.filter(item => item.id !== equipo.id)

  if (!guardarEquipos()) {
    equipos.value = equiposAnteriores
    return
  }

  if (equipoSeleccionado.value?.id === equipo.id) {
    equipoSeleccionado.value = equipos.value[0] || null
  }

  contadorEventos.value++

  registrarActividad(
    `Equipo eliminado: ${equipo.nombre} (${equipo.juego})`
  )
}

function limpiarFiltros() {
  busqueda.value = ''
  filtroJuego.value = 'Todos'
  filtroRegion.value = 'Todas'

  registrarActividad('Filtros restablecidos')
}
</script>

<template>
  <div class="dashboard">
    <aside class="sidebar">
      <div class="logo">
        <h2>ESPORTS<span>.</span></h2>
        <small>TOURNAMENT MANAGER</small>
      </div>

      <nav>
        <a href="#inicio" class="active">Dashboard</a>
        <a href="#equipos">Equipos</a>
        <a href="#galeria">Galería</a>
        <a href="#actividad">Actividad</a>
        <a href="#agregar">Agregar equipo</a>
      </nav>

      <div class="sidebar-footer">
        <span class="status-dot"></span>
        Sistema operativo
      </div>
    </aside>

    <main class="main-content">
      <header id="inicio" class="topbar">
        <div>
          <p class="eyebrow">ESPORTS MANAGEMENT SYSTEM</p>
          <h1>Dashboard</h1>
        </div>

        <div class="topbar-info">
          <span class="status-dot"></span>
          <span>Panel de administración</span>
        </div>
      </header>

      <section class="welcome-banner">
        <div>
          <p class="eyebrow">CENTRO DE COMPETICIÓN</p>
          <h2>Gestiona tus equipos de esports</h2>
          <p>Administra equipos, videojuegos y resultados desde un solo lugar.</p>
        </div>

        <div class="banner-decoration">ESPORTS</div>
      </section>

      <section class="stats-grid">
        <div class="stat-card">
          <span>Equipos registrados</span>
          <h2>{{ equipos.length }}</h2>
          <small>Divisiones competitivas</small>
        </div>

        <div class="stat-card">
          <span>Videojuegos</span>
          <h2>{{ totalJuegos }}</h2>
          <small>Títulos diferentes</small>
        </div>

        <div class="stat-card">
          <span>Victorias</span>
          <h2>{{ totalVictorias }}</h2>
          <small>Resultados registrados</small>
        </div>

        <div class="stat-card">
          <span>Eventos</span>
          <h2>{{ contadorEventos }}</h2>
          <small>Interacciones realizadas</small>
        </div>
      </section>

      <section id="equipos" class="panel">
        <div class="section-heading">
          <div>
            <p class="eyebrow">GESTIÓN</p>
            <h2>Equipos registrados</h2>
          </div>

          <span class="record-count">
            {{ equiposFiltrados.length }} resultados
          </span>
        </div>

        <div class="filters">
          <input
            v-model="busqueda"
            @input="registrarBusqueda"
            type="text"
            placeholder="Buscar equipo o videojuego..."
          />

          <select v-model="filtroJuego">
            <option value="Todos">Todos los videojuegos</option>
            <option
              v-for="juego in juegos.filter(j => j !== 'Todos')"
              :key="juego"
              :value="juego"
            >
              {{ juego }}
            </option>
          </select>

          <select v-model="filtroRegion">
            <option value="Todas">Todas las regiones</option>
            <option
              v-for="region in regiones.filter(r => r !== 'Todas')"
              :key="region"
              :value="region"
            >
              {{ region }}
            </option>
          </select>

          <button class="secondary-button" @click="limpiarFiltros">
            Limpiar
          </button>
        </div>

        <div class="table-container">
          <table class="teams-table">
            <thead>
              <tr>
                <th>Equipo</th>
                <th>Videojuego</th>
                <th>Región</th>
                <th>Victorias</th>
                <th>Derrotas</th>
                <th>Acciones</th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="equipo in equiposFiltrados" :key="equipo.id">
                <td class="team-name">
                  <img :src="equipo.logo" :alt="equipo.nombre" />
                  <strong>{{ equipo.nombre }}</strong>
                </td>

                <td>{{ equipo.juego }}</td>
                <td>{{ equipo.region }}</td>
                <td class="wins">{{ equipo.victorias }}</td>
                <td class="losses">{{ equipo.derrotas }}</td>

                <td class="action-buttons">
                  <button
                    class="view-button"
                    @click="seleccionarEquipo(equipo)"
                  >
                    Ver
                  </button>

                  <button
                    class="delete-button"
                    @click="eliminarEquipo(equipo)"
                  >
                    Eliminar
                  </button>
                </td>
              </tr>

              <tr v-if="equiposFiltrados.length === 0">
                <td colspan="6" class="empty-history">
                  No se encontraron equipos.
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <section id="galeria" class="content-grid">
        <div class="panel gallery-panel">
          <div class="section-heading">
            <div>
              <p class="eyebrow">EXPLORAR</p>
              <h2>Galería de equipos</h2>
            </div>
          </div>

          <div class="gallery-grid">
            <button
              v-for="equipo in equipos"
              :key="equipo.id"
              class="team-card"
              @click="seleccionarEquipo(equipo)"
            >
              <img :src="equipo.logo" :alt="equipo.nombre" />
              <strong>{{ equipo.nombre }}</strong>
              <small>{{ equipo.juego }}</small>
            </button>
          </div>
        </div>

        <div class="panel preview-panel">
          <p class="eyebrow">VISTA PREVIA</p>
          <h2>Equipo seleccionado</h2>

          <div v-if="equipoSeleccionado" class="team-preview">
            <img
              :src="equipoSeleccionado.logo"
              :alt="equipoSeleccionado.nombre"
            />

            <h3>{{ equipoSeleccionado.nombre }}</h3>
            <p>{{ equipoSeleccionado.juego }}</p>

            <div class="preview-details">
              <div>
                <span>Región</span>
                <strong>{{ equipoSeleccionado.region }}</strong>
              </div>

              <div>
                <span>Victorias</span>
                <strong class="wins">{{ equipoSeleccionado.victorias }}</strong>
              </div>

              <div>
                <span>Derrotas</span>
                <strong class="losses">{{ equipoSeleccionado.derrotas }}</strong>
              </div>
            </div>
          </div>

          <div v-else class="empty-history">
            Selecciona un equipo para visualizarlo.
          </div>
        </div>
      </section>

      <section id="agregar" class="panel">
        <div class="section-heading">
          <div>
            <p class="eyebrow">REGISTRO</p>
            <h2>Agregar nuevo equipo</h2>
          </div>
        </div>

        <form class="add-team-form" @submit.prevent="agregarEquipo">
          <input
            v-model="nuevoEquipo.nombre"
            type="text"
            placeholder="Nombre del equipo"
            required
          />

          <select v-model="nuevoEquipo.juego">
            <option>VALORANT</option>
            <option>League of Legends</option>
            <option>Counter-Strike 2</option>
            <option>Rocket League</option>
            <option>Apex Legends</option>
            <option>Overwatch 2</option>
            <option>Dota 2</option>
          </select>

          <select v-model="nuevoEquipo.region">
            <option>Americas</option>
            <option>Pacific</option>
            <option>EMEA</option>
            <option>LCK</option>
            <option>Europe</option>
            <option>North America</option>
            <option>Global</option>
          </select>

          <input
            v-model.number="nuevoEquipo.victorias"
            type="number"
            min="0"
            step="1"
            placeholder="Victorias"
          />

          <input
            v-model.number="nuevoEquipo.derrotas"
            type="number"
            min="0"
            step="1"
            placeholder="Derrotas"
          />

          <input
            id="logo-input"
            type="file"
            accept=".jpg,.jpeg,.png,.webp"
            @change="seleccionarImagen"
          />

          <div v-if="vistaPreviaLogo" class="logo-preview">
            <img :src="vistaPreviaLogo" alt="Vista previa del logotipo" />
            <span>Vista previa del logotipo</span>
          </div>

          <button type="submit" class="primary-button">
            Agregar equipo
          </button>
        </form>
      </section>

      <section id="actividad" class="panel activity-panel">
        <div class="section-heading">
          <div>
            <p class="eyebrow">REGISTRO DEL SISTEMA</p>
            <h2>Actividad reciente</h2>
          </div>

          <span class="record-count">
            {{ contadorBusquedas }} búsquedas
          </span>
        </div>

        <div v-if="historial.length">
          <div
            v-for="evento in historial"
            :key="evento.id"
            class="history-item"
          >
            <div>
              <span class="status-dot"></span>
              <strong>{{ evento.mensaje }}</strong>
            </div>

            <small>{{ evento.hora }}</small>
          </div>
        </div>

        <p v-else class="empty-history">
          No hay actividad registrada.
        </p>
      </section>

      <footer class="dashboard-footer">
        ESPORTS TOURNAMENT MANAGER · PROYECTO ACADÉMICO
      </footer>
    </main>
  </div>
</template>
