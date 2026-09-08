<template>
  <q-layout view="lHh Lpr lFf" class="bg-grey-2 text-body">
    <q-header elevated class="bg-primary text-white">
      <q-toolbar>
        <q-icon name="build" size="md" class="q-mr-sm" />
        <q-toolbar-title class="text-weight-bold">
          Taller Don Efraín
          <div class="text-caption text-weight-regular">Servicio Técnico Móvil</div>
        </q-toolbar-title>
        <q-btn
          color="secondary"
          icon="add"
          label="Nuevo Servicio"
          @click="abrirModalCrear"
          unelevated
          rounded
          class="text-weight-bold"
        />
      </q-toolbar>
    </q-header>

    <q-page-container class="q-pa-md">
      <q-page>
        <!-- Estado Vacío -->
        <div v-if="servicios.length === 0" class="text-center q-pa-xl">
          <q-icon name="assignment_late" size="4.5rem" color="grey-6" />
          <div class="text-h6 text-grey-7 q-mt-md">No hay servicios registrados aún.</div>
          <div class="text-subtitle1 text-grey-6">Haz clic en "Nuevo Servicio" para registrar un equipo.</div>
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
              class="servicio-card"
            >
              <q-card-section class="q-pb-xs">
                <div class="row items-center no-wrap">
                  <div class="col">
                    <div class="text-h6 text-bold text-grey-9">{{ servicio.cliente || 'Sin nombre' }}</div>
                    <div class="text-subtitle1 text-primary text-bold">
                      <q-icon name="smartphone" /> {{ servicio.marca }} {{ servicio.modelo }}
                    </div>
                  </div>

                  <div class="col-auto">
                    <q-chip
                      :color="obtenerColorEstadoEquipo(servicio.estadoEquipo)"
                      text-color="white"
                      size="md"
                      class="text-weight-bold"
                    >
                      <q-icon :name="obtenerIconoEstadoEquipo(servicio.estadoEquipo)" class="q-mr-xs" />
                      {{ servicio.estadoEquipo }}
                    </q-chip>
                  </div>
                </div>
              </q-card-section>

              <q-separator />

              <q-card-section class="q-py-sm text-body1">
                <div><strong>Reparación:</strong> {{ servicio.tipoReparacion || 'No especificada' }}</div>
                <div><strong>Técnico:</strong> {{ servicio.tecnico }}</div>
                <div><strong>Fecha/Hora:</strong> {{ servicio.fechaHora }}</div>
                <div><strong>Precio Total:</strong> ${{ servicio.precio }}</div>
                
                <div v-if="servicio.estadoPago === 'abono'">
                  <strong>Monto Abonado:</strong> ${{ servicio.valorAbono }} 
                  <span class="text-negative text-bold"> (Resta: ${{ servicio.precio - servicio.valorAbono }})</span>
                </div>

                <div>
                  <strong>Pago:</strong> 
                  <q-badge
                    :color="servicio.estadoPago === 'pagado' ? 'positive' : (servicio.estadoPago === 'abono' ? 'warning' : 'negative')"
                    class="q-ml-xs text-caption text-bold"
                  >
                    {{ servicio.estadoPago.toUpperCase() }} ({{ servicio.metodoPago }})
                  </q-badge>
                </div>

                <!-- Calificación directa en la tarjeta si está entregado -->
                <div v-if="servicio.estadoEquipo === 'entregado'" class="q-mt-sm bg-grey-3 q-pa-sm rounded-borders text-center">
                  <div class="text-weight-bold text-grey-9">Calificación del cliente:</div>
                  <q-rating
                    v-model="servicio.calificacion"
                    max="5"
                    size="2em"
                    color="orange-9"
                    icon="star_border"
                    icon-selected="star"
                  />
                </div>

                <div v-if="servicio.observaciones" class="q-mt-xs text-italic text-grey-8">
                  <strong>Obs:</strong> "{{ servicio.observaciones }}"
                </div>
              </q-card-section>

              <q-separator />

              <!-- Acciones: Deshabilitadas si el estado es 'entregado' -->
              <q-card-actions align="right">
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

        <q-page-sticky position="bottom-right" :offset="[18, 18]">
          <q-btn fab icon="add" color="secondary" @click="abrirModalCrear" />
        </q-page-sticky>

        <!-- Modal de Crear / Editar -->
        <q-dialog v-model="modalAbierto" persistent>
          <q-card style="width: 550px; max-width: 95vw;">
            <q-card-section class="row items-center bg-primary text-white">
              <div class="text-h6">{{ editandoIndex === null ? 'Registrar Servicio' : 'Editar Servicio' }}</div>
              <q-space />
              <q-btn icon="close" flat round dense v-close-popup />
            </q-card-section>

            <q-form @submit.prevent="guardarServicio" class="q-pa-md q-gutter-md text-body1">
              <q-input
                v-model="form.cliente"
                label="Nombre del cliente"
                outlined
                dense
              />

              <div class="row q-col-gutter-sm">
                <div class="col-6">
                  <q-select
                    v-model="form.marca"
                    :options="opcionesMarcas"
                    label="Marca del equipo"
                    outlined
                    dense
                  />
                </div>
                <div class="col-6">
                  <q-input
                    v-model="form.modelo"
                    label="Modelo"
                    hint="Ej: Galaxy A15, iPhone 13"
                    outlined
                    dense
                  />
                </div>
              </div>

              <q-select
                v-model="form.tipoReparacion"
                :options="opcionesReparacion"
                label="Tipo de reparación"
                outlined
                dense
              />

              <q-select
                v-model="form.tecnico"
                :options="opcionesTecnicos"
                label="Técnico que atendió"
                outlined
                dense
              />

              <q-input
                v-model.number="form.precio"
                type="number"
                label="Precio cobrado"
                prefix="$"
                outlined
                dense
              />

              <div class="row q-col-gutter-sm">
                <div class="col-6">
                  <q-select
                    v-model="form.metodoPago"
                    :options="opcionesMetodoPago"
                    label="Método de pago"
                    outlined
                    dense
                  />
                </div>
                <div class="col-6">
                  <q-select
                    v-model="form.estadoPago"
                    :options="opcionesEstadoPago"
                    label="Estado del pago"
                    outlined
                    dense
                  />
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
                class="bg-orange-1"
              />

              <!-- Selector de estado del equipo (Solo en Edición) -->
              <q-select
                v-if="editandoIndex !== null"
                v-model="form.estadoEquipo"
                :options="opcionesEstadoEquipo"
                label="Estado del equipo"
                outlined
                dense
              />

              <q-input
                v-model="form.observaciones"
                type="textarea"
                label="Observaciones (opcional)"
                hint="Ej: Pantalla partida, viene sin bandeja SIM..."
                outlined
                dense
                rows="2"
              />

              <q-card-actions align="right" class="q-mt-md">
                <q-btn label="Cancelar" color="grey" flat v-close-popup />
                <q-btn
                  :label="editandoIndex === null ? 'Guardar' : 'Actualizar'"
                  type="submit"
                  color="primary"
                  unelevated
                />
              </q-card-actions>
            </q-form>
          </q-card>
        </q-dialog>

        <!-- Modal de Confirmación de Eliminación -->
        <q-dialog v-model="modalEliminarAbierto">
          <q-card>
            <q-card-section class="row items-center">
              <q-avatar icon="warning" color="negative" text-color="white" />
              <span class="q-ml-sm text-body1">¿Está seguro de que desea eliminar este registro?</span>
            </q-card-section>

            <q-card-actions align="right">
              <q-btn flat label="Cancelar" color="grey" v-close-popup />
              <q-btn flat label="Eliminar" color="negative" @click="eliminarServicio" v-close-popup />
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
  font-size: 15px;
}
.servicio-card {
  font-size: 15px;
  border-radius: 8px;
}
.border-red {
  border: 2px solid #e53935 !important;
}
.border-orange {
  border: 2px solid #fb8c00 !important;
}
</style>
