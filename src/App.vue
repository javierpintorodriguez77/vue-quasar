<template>
  <q-layout view="lHh Lpr lFf" class="bg-grey-2 text-body">
    <!-- Header Principal -->
    <q-header elevated class="bg-primary text-white">
      <q-toolbar class="q-py-xs">
        <q-icon name="build_circle" size="md" class="q-mr-sm" />
        <q-toolbar-title class="text-weight-bold">
          Taller Don Efraín
          <div class="text-caption text-weight-regular text-grey-4">Servicio Técnico Móvil</div>
        </q-toolbar-title>
        <q-btn
          color="secondary"
          icon="add"
          label="Nuevo Servicio"
          @click="abrirModalCrear"
          unelevated
          rounded
          class="text-weight-bold q-px-md"
        />
      </q-toolbar>
    </q-header>

    <q-page-container class="q-pa-md">
      <q-page>
        <!-- Estado Vacío -->
        <div v-if="servicios.length === 0" class="text-center q-pa-xl bg-white rounded-borders shadow-1 q-my-lg">
          <q-icon name="assignment_late" size="5rem" color="grey-5" />
          <div class="text-h6 text-grey-8 q-mt-md text-weight-medium">No hay servicios registrados aún</div>
          <div class="text-subtitle1 text-grey-6 q-mb-md">Haz clic en el botón para registrar un nuevo equipo.</div>
          <q-btn color="primary" icon="add" label="Registrar Primer Servicio" @click="abrirModalCrear" unelevated />
        </div>

        <!-- Tarjetas de Servicios -->
        <div v-else class="row q-col-gutter-md">
          <div
            v-for="(servicio, index) in servicios"
            :key="servicio.id"
            class="col-12 col-sm-6 col-md-4"
          >
            <q-card
              flat
              bordered
              :class="{
                'bg-red-1 border-red': servicio.estadoPago === 'pendiente',
                'bg-orange-1 border-orange': servicio.estadoPago === 'abono',
                'bg-white': servicio.estadoPago === 'pagado'
              }"
              class="servicio-card shadow-2 transition-generic"
            >
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
                    <q-chip
                      :color="obtenerColorEstadoEquipo(servicio.estadoEquipo)"
                      text-color="white"
                      size="md"
                      class="text-weight-bold shadow-1"
                    >
                      <q-icon :name="obtenerIconoEstadoEquipo(servicio.estadoEquipo)" class="q-mr-xs" />
                      {{ servicio.estadoEquipo }}
                    </q-chip>
                  </div>
                </div>
              </q-card-section>

              <q-separator />

              <q-card-section class="q-py-sm text-body1 q-gutter-xs">
                <div><span class="text-weight-bold">Reparación:</span> {{ servicio.tipoReparacion || 'No especificada' }}</div>
                <div><span class="text-weight-bold">Técnico:</span> {{ servicio.tecnico }}</div>
                <div><span class="text-weight-bold">Fecha/Hora:</span> {{ servicio.fechaHora }}</div>
                <div><span class="text-weight-bold">Precio Total:</span> <span class="text-weight-bolder">${{ servicio.precio }}</span></div>
                
                <div v-if="servicio.estadoPago === 'abono'" class="bg-orange-2 q-pa-xs rounded-borders">
                  <span class="text-weight-bold">Abonado:</span> ${{ servicio.valorAbono }} 
                  <span class="text-negative text-bold"> (Resta: ${{ servicio.precio - servicio.valorAbono }})</span>
                </div>

                <div class="row items-center q-mt-xs">
                  <span class="text-weight-bold q-mr-xs">Estado Pago:</span>
                  <q-badge
                    :color="servicio.estadoPago === 'pagado' ? 'positive' : (servicio.estadoPago === 'abono' ? 'warning' : 'negative')"
                    class="text-caption text-bold q-px-sm q-py-xs"
                  >
                    {{ servicio.estadoPago.toUpperCase() }} ({{ servicio.metodoPago }})
                  </q-badge>
                </div>

                <!-- Calificación directa en la tarjeta si está entregado -->
                <div v-if="servicio.estadoEquipo === 'entregado'" class="q-mt-sm bg-grey-3 q-pa-sm rounded-borders text-center shadow-1">
                  <div class="text-weight-bold text-grey-9 text-caption">Calificación del cliente</div>
                  <q-rating
                    v-model="servicio.calificacion"
                    max="5"
                    size="1.8em"
                    color="amber-9"
                    icon="star_border"
                    icon-selected="star"
                  />
                </div>

                <div v-if="servicio.observaciones" class="q-mt-xs text-italic text-grey-8 bg-grey-2 q-pa-xs rounded-borders">
                  <span class="text-weight-bold">Obs:</span> "{{ servicio.observaciones }}"
                </div>
              </q-card-section>

              <q-separator />

              <!-- Acciones: Deshabilitadas si el estado es 'entregado' -->
              <q-card-actions align="right" class="bg-grey-1">
                <q-btn
                  flat
                  round
                  color="primary"
                  icon="edit"
                  :disable="servicio.estadoEquipo === 'entregado'"
                  @click="abrirModalEditar(servicio, index)"
                >
                  <q-tooltip>{{ servicio.estadoEquipo === 'entregado' ? 'Un servicio entregado no se puede editar' : 'Editar' }}</q-tooltip>
                </q-btn>
                <q-btn
                  flat
                  round
                  color="negative"
                  icon="delete"
                  :disable="servicio.estadoEquipo === 'entregado'"
                  @click="confirmarEliminacion(index)"
                >
                  <q-tooltip>{{ servicio.estadoEquipo === 'entregado' ? 'Un servicio entregado no se puede eliminar' : 'Eliminar' }}</q-tooltip>
                </q-btn>
              </q-card-actions>
            </q-card>
          </div>
        </div>

        <!-- Botón flotante -->
        <q-page-sticky position="bottom-right" :offset="[20, 20]">
          <q-btn fab icon="add" color="secondary" @click="abrirModalCrear" class="shadow-4" />
        </q-page-sticky>

        <!-- Modal de Crear / Editar -->
        <q-dialog v-model="modalAbierto" persistent transition-show="scale" transition-hide="scale">
          <q-card style="width: 550px; max-width: 95vw;" class="rounded-borders">
            <q-card-section class="row items-center bg-primary text-white q-py-sm">
              <q-icon name="edit_note" size="sm" class="q-mr-sm" />
              <div class="text-h6 text-weight-bold">{{ editandoIndex === null ? 'Registrar Servicio' : 'Editar Servicio' }}</div>
              <q-space />
              <q-btn icon="close" flat round dense v-close-popup />
            </q-card-section>

            <q-form @submit.prevent="guardarServicio" class="q-pa-md q-gutter-sm text-body1">
              <q-input
                v-model="form.cliente"
                label="Nombre del cliente"
                outlined
                dense
              >
                <template #prepend><q-icon name="person" /></template>
              </q-input>

              <div class="row q-col-gutter-sm">
                <div class="col-6">
                  <q-select
                    v-model="form.marca"
                    :options="opcionesMarcas"
                    label="Marca"
                    outlined
                    dense
                  >
                    <template #prepend><q-icon name="branding_watermark" /></template>
                  </q-select>
                </div>
                <div class="col-6">
                  <q-input
                    v-model="form.modelo"
                    label="Modelo"
                    hint="Ej: Galaxy A15, iPhone 13"
                    outlined
                    dense
                  >
                    <template #prepend><q-icon name="smartphone" /></template>
                  </q-input>
                </div>
              </div>

              <q-select
                v-model="form.tipoReparacion"
                :options="opcionesReparacion"
                label="Tipo de reparación"
                outlined
                dense
              >
                <template #prepend><q-icon name="build" /></template>
              </q-select>

              <q-select
                v-model="form.tecnico"
                :options="opcionesTecnicos"
                label="Técnico que atendió"
                outlined
                dense
              >
                <template #prepend><q-icon name="badge" /></template>
              </q-select>

              <q-input
                v-model.number="form.precio"
                type="number"
                label="Precio cobrado"
                prefix="$"
                outlined
                dense
              >
                <template #prepend><q-icon name="payments" /></template>
              </q-input>

              <div class="row q-col-gutter-sm">
                <div class="col-6">
                  <q-select
                    v-model="form.metodoPago"
                    :options="opcionesMetodoPago"
                    label="Método de pago"
                    outlined
                    dense
                  >
                    <template #prepend><q-icon name="account_balance_wallet" /></template>
                  </q-select>
                </div>
                <div class="col-6">
                  <q-select
                    v-model="form.estadoPago"
                    :options="opcionesEstadoPago"
                    label="Estado del pago"
                    outlined
                    dense
                  >
                    <template #prepend><q-icon name="pending_actions" /></template>
                  </q-select>
                </div>
              </div>

              <!-- Input Dinámico para Valor Abonado -->
              <q-input
                v-if="form.estadoPago === 'abono'"
                v-model.number="form.valorAbono"
                type="number"
                label="Monto abonado"
                prefix="$"
                outlined
                dense
                class="bg-orange-1 rounded-borders"
              >
                <template #prepend><q-icon name="price_check" color="warning" /></template>
              </q-input>

              <!-- Selector de estado del equipo (Solo en Edición) -->
              <q-select
                v-if="editandoIndex !== null"
                v-model="form.estadoEquipo"
                :options="opcionesEstadoEquipo"
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
                <q-btn
                  :label="editandoIndex === null ? 'Guardar Servicio' : 'Actualizar Servicio'"
                  type="submit"
                  color="primary"
                  unelevated
                  class="q-px-md"
                />
              </q-card-actions>
            </q-form>
          </q-card>
        </q-dialog>

        <!-- Modal Confirmación de Eliminación -->
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

      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const servicios = useLocalStorage('don_efrain_servicios', [])

const modalAbierto = ref(false)
const modalEliminarAbierto = ref(false)
const editandoIndex = ref(null)
const eliminarIndex = ref(null)

const form = ref({
  cliente: '',
  marca: 'Samsung',
  modelo: '',
  tipoReparacion: 'Cambio de pantalla',
  tecnico: 'Don Efraín',
  fechaHora: '',
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
const opcionesEstadoEquipo = ['recibido', 'en reparación', 'listo para entregar', 'entregado']

function reiniciarFormulario() {
  form.value = {
    cliente: '',
    marca: 'Samsung',
    modelo: '',
    tipoReparacion: 'Cambio de pantalla',
    tecnico: 'Don Efraín',
    fechaHora: '',
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
  form.value = { ...servicio }
  modalAbierto.value = true
}

function guardarServicio() {
  if (editandoIndex.value === null) {
    const nuevoServicio = {
      ...form.value,
      id: Date.now(),
      estadoEquipo: 'recibido',
      fechaHora: new Date().toLocaleString('es-CO')
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
</style>
