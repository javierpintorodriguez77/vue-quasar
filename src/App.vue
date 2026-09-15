<template>
  <q-layout view="lHh Lpr lFf" class="bg-grey-2 text-body">
    <q-header elevated class="bg-primary text-white">
      <q-toolbar class="q-py-xs">
        <q-icon name="build_circle" size="md" class="q-mr-sm" />
        <q-toolbar-title class="text-weight-bold">
          Taller Don Efraín
          <div class="text-caption text-weight-regular text-grey-4">Servicio Técnico Móvil</div>
        </q-toolbar-title>

   
        <q-btn
          color="negative"
          icon="cleaning_services"
          label="Borrar Entregados"
          @click="confirmarBorrarEntregados"
          unelevated
          rounded
          class="text-weight-bold q-mr-sm"
          :disable="!hayEntregados"
        >
          <q-tooltip v-if="!hayEntregados">No hay servicios entregados para borrar</q-tooltip>
        </q-btn>

        <q-btn color="secondary" icon="add" label="Nuevo Servicio" @click="abrirModalCrear" unelevated rounded class="text-weight-bold q-px-md" />
      </q-toolbar>
    </q-header>

    <q-page-container class="q-pa-md">
      <q-page>
        <div v-if="servicios.length === 0" class="text-center q-pa-xl bg-white rounded-borders shadow-1 q-my-lg">
          <q-icon name="assignment_late" size="5rem" color="grey-5" />
          <div class="text-h6 text-grey-8 q-mt-md text-weight-medium">No hay servicios registrados aún</div>
          <div class="text-subtitle1 text-grey-6 q-mb-md">Haz clic en el botón para registrar un nuevo equipo.</div>
          <q-btn color="primary" icon="add" label="Registrar Primer Servicio" @click="abrirModalCrear" unelevated />
        </div>

        <div v-else class="row q-col-gutter-md">
          <div v-for="(servicio, index) in servicios" :key="servicio.id" class="col-12 col-sm-6 col-md-4">
            <q-card flat bordered :class="{
              'bg-red-1 border-red': servicio.estadoPago === 'pendiente',
              'bg-orange-1 border-orange': servicio.estadoPago === 'abono',
              'bg-white': servicio.estadoPago === 'pagado'
            }" class="servicio-card shadow-2 transition-generic">
              <q-card-section class="q-pb-sm">
                <div class="row items-center no-wrap justify-between">
                  <div class="col ellipsis q-pr-sm">
                    <div class="text-h6 text-bold text-grey-9 ellipsis">
                      {{ servicio.cliente || 'Sin nombre' }}
                    </div>
                    <div class="text-subtitle1 text-primary text-bold row items-center">
                      <q-icon name="smartphone" class="q-mr-xs" />
                      <span>{{ servicio.marca }} {{ servicio.modelo }}</span>
                    </div>
                  </div>

                  <div class="col-auto">
                    <q-chip :color="obtenerColorEstadoEquipo(servicio.estadoEquipo)" text-color="white" size="md" class="text-weight-bold shadow-1">
                      <q-icon :name="obtenerIconoEstadoEquipo(servicio.estadoEquipo)" class="q-mr-xs" />
                      {{ servicio.estadoEquipo }}
                    </q-chip>
                  </div>
                </div>
              </q-card-section>

              <q-separator />

              <q-card-section class="q-py-sm text-body1 q-gutter-xs">
              
                <div>
                  <span class="text-weight-bold block q-mb-xs">Reparaciones:</span>
                  <template v-if="Array.isArray(servicio.tipoReparacion) && servicio.tipoReparacion.length">
                    <q-chip v-for="(rep, rIdx) in servicio.tipoReparacion" :key="rIdx" dense color="primary" text-color="white" icon="build" size="sm" class="q-mr-xs q-mb-xs">
                      {{ rep }}
                    </q-chip>
                  </template>
                  <template v-else-if="typeof servicio.tipoReparacion === 'string' && servicio.tipoReparacion">
                    <q-chip dense color="primary" text-color="white" icon="build" size="sm">
                      {{ servicio.tipoReparacion }}
                    </q-chip>
                  </template>
                  <template v-else>
                    <span class="text-grey-7 text-italic">No especificadas</span>
                  </template>
                </div>

                <div><span class="text-weight-bold">Técnico:</span> {{ servicio.tecnico }}</div>
                
                <div><span class="text-weight-bold">Fecha:</span> {{ servicio.fecha }}</div>
                <div><span class="text-weight-bold">Hora:</span> {{ servicio.hora }}</div>

                <div><span class="text-weight-bold">Precio Total:</span> <span class="text-weight-bolder">${{ formatearMoneda(servicio.precio) }}</span></div>

                <div v-if="servicio.estadoPago === 'abono'" class="bg-orange-2 q-pa-xs rounded-borders">
                  <span class="text-weight-bold">Abonado:</span> ${{ formatearMoneda(servicio.valorAbono) }}
                  <span class="text-negative text-bold"> (Resta: ${{ formatearMoneda(servicio.precio - servicio.valorAbono) }})</span>
                </div>

                <div class="row items-center q-mt-xs">
                  <span class="text-weight-bold q-mr-xs">Estado Pago:</span>
                  <q-badge :color="servicio.estadoPago === 'pagado' ? 'positive' : (servicio.estadoPago === 'abono' ? 'warning' : 'negative')" class="text-caption text-bold q-px-sm q-py-xs">
                    {{ servicio.estadoPago.toUpperCase() }} ({{ servicio.metodoPago }})
                  </q-badge>
                </div>

                <div v-if="servicio.estadoEquipo === 'entregado'" class="q-mt-sm bg-grey-3 q-pa-sm rounded-borders text-center shadow-1">
                  <div class="text-weight-bold text-grey-9 text-caption">Calificación del cliente</div>
                  <q-rating v-model="servicio.calificacion" max="5" size="1.8em" color="amber-9" icon="star_border" icon-selected="star" />
                </div>

                <div v-if="servicio.observaciones" class="q-mt-xs text-italic text-grey-8 bg-grey-2 q-pa-xs rounded-borders">
                  <span class="text-weight-bold">Obs:</span> "{{ servicio.observaciones }}"
                </div>
              </q-card-section>

              <q-separator />

              <q-card-actions align="right" class="bg-grey-1">
                <q-btn flat round color="primary" icon="edit" :disable="servicio.estadoEquipo === 'entregado'" @click="abrirModalEditar(servicio, index)">
                  <q-tooltip>{{ servicio.estadoEquipo === 'entregado' ? 'Un servicio entregado no se puede editar' : 'Editar' }}</q-tooltip>
                </q-btn>
                <q-btn flat round color="negative" icon="delete" :disable="servicio.estadoEquipo === 'entregado'" @click="confirmarEliminacion(index)">
                  <q-tooltip>{{ servicio.estadoEquipo === 'entregado' ? 'Un servicio entregado no se puede eliminar' : 'Eliminar' }}</q-tooltip>
                </q-btn>
              </q-card-actions>
            </q-card>
          </div>
        </div>

        <q-page-sticky position="bottom-right" :offset="[20, 20]">
          <q-btn fab icon="add" color="secondary" @click="abrirModalCrear" class="shadow-4" />
        </q-page-sticky>

    
        <q-dialog v-model="modalAbierto" persistent transition-show="scale" transition-hide="scale">
          <q-card style="width: 550px; max-width: 95vw;" class="rounded-borders">
            <q-card-section class="row items-center bg-primary text-white q-py-sm">
              <q-icon name="edit_note" size="sm" class="q-mr-sm" />
              <div class="text-h6 text-weight-bold">{{ editandoIndex === null ? 'Registrar Servicio' : 'Editar Servicio' }}</div>
              <q-space />
              <q-btn icon="close" flat round dense v-close-popup />
            </q-card-section>

            <q-form ref="formRef" @submit.prevent="guardarServicio" class="q-pa-md q-gutter-sm text-body1">
              <q-input 
                v-model="form.cliente" 
                label="Nombre del cliente *" 
                outlined 
                dense 
                :rules="[val => (val && val.trim().length > 0) || 'Ingrese un nombre válido (no solo espacios)']" 
              >
                <template #prepend><q-icon name="person" /></template>
              </q-input>

              <div class="row q-col-gutter-sm">
                <div class="col-6">
                  <q-select v-model="form.marca" :options="opcionesMarcas" label="Marca *" outlined dense :rules="[val => !!val || 'Seleccione una marca']">
                    <template #prepend><q-icon name="branding_watermark" /></template>
                  </q-select>
                </div>
                <div class="col-6">
                  <q-input 
                    v-model="form.modelo" 
                    label="Modelo *" 
                    hint="Ej: Galaxy A15, iPhone 13" 
                    outlined 
                    dense 
                    :rules="[val => (val && val.trim().length > 0) || 'Escriba un modelo válido']"
                  >
                    <template #prepend><q-icon name="smartphone" /></template>
                  </q-input>
                </div>
              </div>

              
              <q-select 
                v-model="form.tipoReparacion" 
                :options="opcionesReparacion" 
                label="Tipo(s) de reparación *" 
                placeholder="Seleccione una o varias..."
                multiple 
                use-chips 
                outlined 
                dense 
                :rules="[val => (Array.isArray(val) && val.length > 0) || 'Seleccione al menos una reparación']"
              >
                <template #prepend><q-icon name="build" /></template>
              </q-select>

              <q-select v-model="form.tecnico" :options="opcionesTecnicos" label="Técnico que atendió *" outlined dense :rules="[val => !!val || 'Seleccione un técnico']">
                <template #prepend><q-icon name="badge" /></template>
              </q-select>

              <q-input 
                :model-value="formatearMoneda(form.precio)" 
                @update:model-value="val => form.precio = desformatearNumero(val)" 
                label="Precio cobrado *" 
                prefix="$" 
                outlined 
                dense 
                :rules="[val => desformatearNumero(val) > 0 || 'Ingrese un precio válido mayor a 0']"
              >
                <template #prepend><q-icon name="payments" /></template>
              </q-input>

              <div class="row q-col-gutter-sm">
                <div class="col-6">
                  <q-select v-model="form.metodoPago" :options="opcionesMetodoPago" label="Método de pago" outlined dense>
                    <template #prepend><q-icon name="account_balance_wallet" /></template>
                  </q-select>
                </div>
                <div class="col-6">
                  <q-select v-model="form.estadoPago" :options="opcionesEstadoPago" label="Estado del pago" outlined dense @update:model-value="alCambiarEstadoPago">
                    <template #prepend><q-icon name="pending_actions" /></template>
                  </q-select>
                </div>
              </div>

  
              <q-input 
                v-if="form.estadoPago === 'abono'" 
                :model-value="formatearMoneda(form.valorAbono)" 
                @update:model-value="val => form.valorAbono = desformatearNumero(val)" 
                label="Monto abonado *" 
                prefix="$" 
                outlined 
                dense 
                class="bg-orange-1 rounded-borders" 
                :rules="[val => {
                  const num = desformatearNumero(val);
                  return (num > 0 && num < form.precio) || 'El abono debe ser mayor a 0 y menor al precio total';
                }]"
              >
                <template #prepend><q-icon name="price_check" color="warning" /></template>
              </q-input>

       
              <q-select 
                v-if="editandoIndex !== null" 
                v-model="form.estadoEquipo" 
                :options="opcionesEstadoEquipoFiltradas" 
                label="Estado del equipo" 
                outlined 
                dense
              >
                <template #prepend><q-icon name="sync" /></template>
              </q-select>

              <q-input 
                v-model="form.observaciones" 
                type="textarea" 
                label="Observaciones (opcional)" 
                hint="Ej: Pantalla partida, viene sin bandeja SIM..." 
                outlined 
                dense 
                rows="2"
              >
                <template #prepend><q-icon name="notes" /></template>
              </q-input>

              <q-card-actions align="right" class="q-mt-md q-px-none">
                <q-btn label="Cancelar" color="grey-7" flat v-close-popup />
                <q-btn :label="editandoIndex === null ? 'Guardar Servicio' : 'Actualizar Servicio'" type="submit" color="primary" unelevated class="q-px-md" />
              </q-card-actions>
            </q-form>
          </q-card>
        </q-dialog>

        <q-dialog v-model="modalEliminarAbierto">
          <q-card class="rounded-borders">
            <q-card-section class="row items-center q-pb-none">
              <q-avatar icon="warning" color="negative" text-color="white" class="shadow-1" />
              <span class="q-ml-md text-subtitle1 text-weight-medium">¿Está seguro de que desea eliminar este registro?</span>
            </q-card-section>

            <q-card-actions align="right" class="q-pa-md">
              <q-btn flat label="Cancelar" color="grey-7" v-close-popup />
              <q-btn unelevated label="Eliminar" color="negative" @click="eliminarServicio" v-close-popup />
            </q-card-actions>
          </q-card>
        </q-dialog>

        <q-dialog v-model="modalBorrarEntregadosAbierto">
          <q-card class="rounded-borders">
            <q-card-section class="row items-center q-pb-none">
              <q-avatar icon="delete_sweep" color="negative" text-color="white" class="shadow-1" />
              <span class="q-ml-md text-subtitle1 text-weight-medium">¿Eliminar todos los servicios entregados?</span>
            </q-card-section>
            <q-card-section class="text-body2 text-grey-8 q-pt-sm">
              Esta acción eliminará de forma permanente todos los registros que ya hayan sido marcados como **entregados**.
            </q-card-section>

            <q-card-actions align="right" class="q-pa-md">
              <q-btn flat label="Cancelar" color="grey-7" v-close-popup />
              <q-btn unelevated label="Sí, eliminar todos" color="negative" @click="borrarServiciosEntregados" v-close-popup />
            </q-card-actions>
          </q-card>
        </q-dialog>

      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const servicios = useLocalStorage('don_efrain_servicios', [])

const formRef = ref(null)
const modalAbierto = ref(false)
const modalEliminarAbierto = ref(false)
const modalBorrarEntregadosAbierto = ref(false)
const editandoIndex = ref(null)
const eliminarIndex = ref(null)

const form = ref({
  cliente: '',
  marca: 'Samsung',
  modelo: '',
  tipoReparacion: [], 
  tecnico: 'Don Efraín',
  fecha: '',
  hora: '',
  precio: 0,
  valorAbono: 0,
  metodoPago: 'efectivo',
  estadoPago: 'pendiente',
  estadoEquipo: 'recibido',
  calificacion: 5,
  observaciones: ''
})

const opcionesMarcas = ['Samsung', 'Apple', 'Xiaomi', 'Motorola', 'Huawei', 'Oppo', 'Realme', 'ZTE', 'Otra']
const opcionesReparacion = [
  'Cambio de pantalla',
  'Cambio de batería',
  'Cambio de pin de carga',
  'Liberación',
  'Mantenimiento de software',
  'Cambio de flex',
  'Otros'
]
const opcionesTecnicos = ['Don Efraín', 'Técnico 1', 'Técnico 2']
const opcionesMetodoPago = ['efectivo', 'transferencia', 'tarjeta']
const opcionesEstadoPago = ['pagado', 'pendiente', 'abono']

const hayEntregados = computed(() => {
  return servicios.value.some(s => s.estadoEquipo === 'entregado')
})

const opcionesEstadoEquipoFiltradas = computed(() => {
  const opciones = ['recibido', 'en reparación', 'listo para entregar']
  if (form.value.estadoPago === 'pagado') {
    opciones.push('entregado')
  }
  return opciones
})

function formatearMoneda(val) {
  if (val === null || val === undefined || val === '') return ''
  const numero = typeof val === 'string' ? desformatearNumero(val) : val
  return numero.toLocaleString('es-CO')
}

function desformatearNumero(val) {
  if (!val) return 0
  const soloNumeros = val.toString().replace(/\D/g, '')
  return soloNumeros ? parseInt(soloNumeros, 10) : 0
}

function alCambiarEstadoPago(nuevoEstado) {
  if (nuevoEstado !== 'pagado' && form.value.estadoEquipo === 'entregado') {
    form.value.estadoEquipo = 'listo para entregar'
  }
}

function reiniciarFormulario() {
  form.value = {
    cliente: '',
    marca: 'Samsung',
    modelo: '',
    tipoReparacion: [], // Garantizar array vacío para nuevos registros
    tecnico: 'Don Efraín',
    fecha: '',
    hora: '',
    precio: 0,
    valorAbono: 0,
    metodoPago: 'efectivo',
    estadoPago: 'pendiente',
    estadoEquipo: 'recibido',
    calificacion: 5,
    observaciones: ''
  }
}

function abrirModalCrear() {
  editandoIndex.value = null
  reiniciarFormulario()
  modalAbierto.value = true
}

function abrirModalEditar(servicio, index) {
  editandoIndex.value = index
  
  let reparaciones = servicio.tipoReparacion
  if (typeof reparaciones === 'string') {
    reparaciones = reparaciones ? [reparaciones] : []
  } else if (!Array.isArray(reparaciones)) {
    reparaciones = []
  }

  form.value = {
    ...servicio,
    tipoReparacion: reparaciones
  }
  modalAbierto.value = true
}

function guardarServicio() {
  form.value.cliente = form.value.cliente.trim()
  form.value.modelo = form.value.modelo.trim()
  if (form.value.observaciones) {
    form.value.observaciones = form.value.observaciones.trim()
  }

  const ahora = new Date()
  const fechaActual = ahora.toLocaleDateString('es-CO')
  const horaActual = ahora.toLocaleTimeString('es-CO', { hour: '2-digit', minute: '2-digit', hour12: true })

  if (editandoIndex.value === null) {
    const nuevoServicio = {
      ...form.value,
      id: Date.now(),
      estadoEquipo: 'recibido',
      fecha: fechaActual,
      hora: horaActual
    }
    servicios.value.push(nuevoServicio)
  } else {
    servicios.value[editandoIndex.value] = { ...form.value }
  }

  modalAbierto.value = false
  reiniciarFormulario()
}

function confirmarEliminacion(index) {
  eliminarIndex.value = index
  modalEliminarAbierto.value = true
}

function eliminarServicio() {
  if (eliminarIndex.value !== null) {
    servicios.value.splice(eliminarIndex.value, 1)
    eliminarIndex.value = null
  }
}

function confirmarBorrarEntregados() {
  modalBorrarEntregadosAbierto.value = true
}

function borrarServiciosEntregados() {
  servicios.value = servicios.value.filter(s => s.estadoEquipo !== 'entregado')
}

function obtenerColorEstadoEquipo(estado) {
  if (estado === 'recibido') return 'blue-7'
  if (estado === 'en reparación') return 'orange-8'
  if (estado === 'listo para entregar') return 'teal-7'
  if (estado === 'entregado') return 'grey-7'
  return 'grey'
}

function obtenerIconoEstadoEquipo(estado) {
  if (estado === 'recibido') return 'inbox'
  if (estado === 'en reparación') return 'build'
  if (estado === 'listo para entregar') return 'check_circle'
  if (estado === 'entregado') return 'handshake'
  return 'help'
}
</script>

<style scoped>
.text-body {
  font-size: 14px;
}

.servicio-card {
  font-size: 14px;
  border-radius: 10px;
}

.border-red {
  border: 2px solid #e53935 !important;
}

.border-orange {
  border: 2px solid #fb8c00 !important;
}
</style>
