<template>
  <div class="form-elements">
    <va-button class="mr-2 mb-2" color="info" @click="showMediumModal = true"> Guardar Cambios </va-button>
    <div class="row">
      <div class="flex xs12">
        <va-card>
          <va-card-title>DATOS PERSONALES</va-card-title>
          <va-card-content>
            <form>
              <div class="row">
                <div class="flex md4 sm6 xs12">
                  <va-input v-model="nombreUsuario" placeholder="Nombre">
                    <template #prepend>
                      <va-icon color="gray" name="face" />
                    </template>
                  </va-input>
                </div>
                <div class="flex md4 sm6 xs12">
                  <va-input v-model="telefonoMed" placeholder="Teléfono">
                    <template #prepend>
                      <va-icon color="gray" name="phone" />
                    </template>
                  </va-input>
                </div>
                <div class="flex md4 sm6 xs12">
                  <va-input v-model="correoMed" placeholder="Correo">
                    <template #prepend>
                      <va-icon color="gray" name="envelope" />
                    </template>
                  </va-input>
                </div>
                <div class="flex md4 sm6 xs12">
                  <va-input v-model="cedulaMed" placeholder="Cédula">
                    <template #prepend>
                      <va-icon color="gray" name="credit_card" />
                    </template>
                  </va-input>
                </div>
                <div class="flex md4 sm6 xs12" v-if="tipoPerfilUser">
                  <va-input v-model="numeroTarjetaMed" placeholder="Tarjeta ">
                    <template #prepend>
                      <va-icon color="gray" name="newspaper" />
                    </template>
                  </va-input>
                </div>
              </div>
            </form>
          </va-card-content>
        </va-card>
      </div>

      <div class="flex xs12" v-if="tipoPerfilUser">
        <va-card>
          <va-card-title>FECHAS DE DISPONIBILIDAD</va-card-title>
          <va-card-content>
            <form>
              <div class="row">
                <!-- <div class="flex md4 sm6 xs12">
                  <va-date-input v-model="dateInput.simple" :label="t('forms.dateTimePicker.basic')" />
                </div> -->
                <!-- <div class="flex md4 sm6 xs12">
                  <va-date-input
                    v-model="dateInput.simple"
                    :label="t('forms.dateTimePicker.manualInput')"
                    manual-input/>
                </div> -->
                <!-- <div class="flex md4 sm6 xs12">
                  <fieldset>
                    <va-switch v-model="toggles.unselected" label="Unselected toggle" class="mr-4 mb-2" />
                  </fieldset>
                </div> -->
                <div class="flex md4 sm6 xs12">
                  <va-date-input
                    v-model="dateInput.multiple"
                    :label="t('forms.dateTimePicker.multiple')"
                    mode="multiple"
                    clearable
                  />
                </div>
                <!-- <div class="flex md4 sm6 xs12">
                  <va-date-input
                    v-model="dateInput.range"
                    :label="t('forms.dateTimePicker.range')"
                    mode="range"
                    clearable
                  /> -->
                <!-- </div> -->
                <!-- <div class="flex md4 sm6 xs12">
                  <va-date-input
                    v-model="dateInput.simple"
                    :label="t('forms.dateTimePicker.customFirstDay')"
                    first-weekday="Monday"
                    highlight-weekend
                  />
                </div> -->
              </div>
            </form>
          </va-card-content>
          <va-divider />
          <va-card-title>HORAS DE DISPONIBILIDAD</va-card-title>
          <va-card-content>
            <form>
              <div class="row">
                <div class="flex md4 sm6 xs12">
                  <va-time-input
                    v-model="dateInput.simple"
                    :label="t('forms.dateTimePicker.manualInput')"
                    manual-input
                  />
                </div>
                <div class="flex md4 sm6 xs12">
                  <va-time-input
                    v-model="dateInput.simple2"
                    :label="t('forms.dateTimePicker.manualInput')"
                    manual-input
                  />
                </div>
              </div>
            </form>
          </va-card-content>
        </va-card>
      </div>

      <div class="flex xs12" v-if="tipoPerfilUser">
        <va-card>
          <va-card-title>Para cambiar estos datos por favor contactar a soporte</va-card-title>
          <va-card-content>
            <form>
              <div class="row">
                <div class="flex md6 xs12">
                  <va-select
                    v-model="ciudadSeleccionada"
                    :label="t('forms.selects.ciudades')"
                    text-by="description"
                    searchable
                    multiple
                    :options="ciudades"
                  />
                </div>
                <div class="flex md6 xs12">
                  <va-select
                    v-model="especialidadSeleccionada"
                    :label="t('forms.selects.especialidades')"
                    text-by="description"
                    searchable
                    multiple
                    :options="especialidades"
                  />
                </div>
              </div>
            </form>
          </va-card-content>
        </va-card>
      </div>

      <div class="flex xs12 md12" v-if="tipoPerfilUser">
        <va-card>
          <va-card-title>Perfil Profesional</va-card-title>
          <va-card-content class="d-flex justify-center">
            <va-medium-editor @initialized="handleEditorInitialization">
              <h1>Edite su perfil</h1>

              <p id="textoPerfilProfesional">
                {{ perfilProfesional }}
              </p>
            </va-medium-editor>
          </va-card-content>
        </va-card>
      </div>
    </div>
    <va-modal
      v-model="showMediumModal"
      :title="confirmSubmit.title"
      :ok-text="confirmSubmit.confirm"
      :cancel-text="confirmSubmit.cancel"
      :message="confirmSubmit.message"
    />
  </div>
</template>

<script setup lang="ts">
  import { ref } from 'vue'
  import CountriesList from '../forms/data/CountriesList'
  import { useI18n } from 'vue-i18n'
  import { nextTick } from 'vue'
  import store from '../../../stores/vuexStore'
  import ServiceData from '../../../services/ServiceData'
  import ServiceProfile from '../../../services/ServiceProfile'
  import { onMounted } from 'vue'

  import type MediumEditor from 'medium-editor'
  import VaMediumEditor from '../../../components/va-medium-editor/VaMediumEditor.vue'
  const { t } = useI18n()
  const showMediumModal = ref(false)
  let token = {
    tipoPerfil: 'medico',
  }
  let perfil = store.state.User
  let nombrePerfil = ref('')
  const nombreUsuario = ref('')
  let telefonoMed = ref('')
  let numeroTarjetaMed = ref('')
  let cedulaMed = ref('')
  let correoMed = ref('')
  let tipoPerfilUser = ref(false)
  let horaInicio = new Date()
  let horaFinal = perfil.HoraFinal
  let fechasDisponibilidad = perfil.FechasDisponibilidad
  let confirmSubmit = {
    message:'¿Desea guardar cambios?',
    confirm:'Confirmar',
    cancel:'Cancelar',
    title:'Actualizar Perfil'
  }
  let perfilProfesional = ref('')
  let ciudadSeleccionada = perfil.CiudadMed
  let especialidadSeleccionada = perfil.Especialidad
  let especialidades = [{}]
  let ciudades = [{}]

  const datePlusDay = (date: Date, days: number) => {
    const d = new Date(date)
    d.setDate(d.getDate() + days)
    return d
  }

  const dateInput = ref({
    simple: new Date(),
    simple2: new Date(),
    disabled: '2018-05-09',
    range: { start: new Date(), end: datePlusDay(new Date(), 7) },
    multiple: ['2023-07-01', '2023-06-29', '2023-06-30'],
  })

  function handleEditorInitialization(editor: typeof MediumEditor) {
    nextTick(() => highlightSampleText(editor))
  }

  function highlightSampleText(editor: typeof MediumEditor) {
    const sampleText = document.getElementsByClassName('default-selection')[0] as HTMLElement
    editor.selectElement(sampleText)
  }
  async function dataPerfil() {
    tipoPerfilUser.value = perfil.NumeroTarjetaMed != undefined ? true : false
    if (tipoPerfilUser.value == true) {
      nombreUsuario.value = perfil.NombreCompletoMed
      telefonoMed.value = perfil.telefonoMed
      correoMed.value = perfil.CorreoMed
      cedulaMed.value = perfil.CedulaMed
      numeroTarjetaMed.value = perfil.NumeroTarjetaMed
      perfilProfesional.value = perfil.PerfilProfesional
    } else {
      nombreUsuario.value = perfil[0].NombreCompletoPac
      console.log('Nombre:',nombreUsuario.value);
      telefonoMed.value = perfil[0].telefonoPac
      console.log('Telefono:',telefonoMed.value);
      correoMed.value = perfil[0].correoPac
      console.log('Correo:',correoMed.value);
      cedulaMed.value = perfil[0].cedulaPac
      console.log('Cedula:',cedulaMed.value);

    }
    console.log('Terminado la data', perfil)
  }
  async function guardarCambios() {
    if (token.tipoPerfil == 'medico') {
      // nombrePerfil
      // telefonoMed
      // numeroTarjetaMed
      // cedulaMed
      // correoMed
      // dateInput.multiple
      // dateInput.simple
      // dateInput.simple2
      // ciudadSeleccionada
      // especialidadSeleccionada
      let descripcionProfesional = document.getElementById('textoPerfilProfesional')
      console.log('ya entre a editar la data')
      let actualizarPerfilMedico = await ServiceProfile.editarMedico(
        nombrePerfil,
        telefonoMed,
        numeroTarjetaMed,
        cedulaMed,
        correoMed,
        dateInput.value.multiple,
        dateInput.value.simple,
        dateInput.value.simple2,
        descripcionProfesional.text,
      )
      console.log('Resultado del edit', actualizarPerfilMedico.json())
    } else {
      console.log('Hay un error mi estimado')
    }
  }
  onMounted(dataPerfil)
</script>

<style lang="scss" scoped>
  fieldset {
    margin-bottom: 0.5rem;
  }
</style>
