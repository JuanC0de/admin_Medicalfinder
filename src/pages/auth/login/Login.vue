<template>
  <form @submit.prevent="onsubmit">
    <va-input
      v-model="email"
      class="mb-3"
      type="email"
      :label="t('auth.email')"
      :error="!!emailErrors.length"
      :error-messages="emailErrors"
    />

    <va-input
      v-model="password"
      class="mb-3"
      type="password"
      :label="t('auth.password')"
      :error="!!passwordErrors.length"
      :error-messages="passwordErrors"
    />
    <p v-if="errores" color="danger">{{ errores }}</p>
    <!-- <div class="auth-layout__options d-flex align-center justify-space-between">
      <va-checkbox v-model="keepLoggedIn" class="mb-0" :label="t('auth.keep_logged_in')" />
      <router-link class="ml-1 va-link" :to="{ name: 'recover-password' }">{{
        t('auth.recover_password')
      }}</router-link>
    </div> -->

    <div class="d-flex justify-center mt-3">
      <va-button class="my-0" @click="onsubmit">{{ t('auth.login') }}</va-button>
    </div>
  </form>
</template>

<script setup lang="ts">
  import { computed, ref } from 'vue'
  import { useRouter } from 'vue-router'
  import { useI18n } from 'vue-i18n'
  import ServiceLogin from './ServiceLogin'
  import ServiceProfile from '../../../services/ServiceProfile'
  import store from '../../../stores/vuexStore'

  const { t } = useI18n()

  const email = ref('')
  const password = ref('')
  const keepLoggedIn = ref(false)
  const emailErrors = ref<string[]>([])
  const passwordErrors = ref<string[]>([])
  const router = useRouter()
  let errores = ref('')
  const formReady = computed(() => !emailErrors.value.length && !passwordErrors.value.length)

  async function onsubmit() {
    if (!formReady.value) return

    emailErrors.value = email.value ? [] : ['Email is required']
    passwordErrors.value = password.value ? [] : ['Password is required']
    let response = await ServiceLogin.login(email.value, password.value)
    if (response.token != undefined) {
      console.log('Nos hemos logueado')
      let responsePacientes = await ServiceProfile.consultarListaPacientes()
      let responseMedicos = await ServiceProfile.consultarListaEspecialistas()
      console.log('Este es el valor del store:', store.state.Token, '-', store.state.User)
      responsePacientes = responsePacientes.filter((paciente) => {
        return paciente.correoPac == email.value
      })
      responseMedicos = responseMedicos.filter((medico) => {
        return medico.correoMed == email.value
      })
      let medicoQuemado = {
        IdMedico: 3,
        NombreCompletoMed: 'Cristian Gutierrez Lopez',
        TelefonoMed: '3225896547',
        NumeroTarjetaMed: '2354878896',
        CedulaMed: '110271486',
        CorreoMed: 'cristian_1980@gmail.com',
        CiudadMed: 'Quibdó',
        Especialidad: 'Oftalmología',
        DireccionConsultorio: 'Carrera 15 #34-8',
        HoraInicio: '20:58',
        HoraFinal: '20',
        FechasDisponibilidad: null,
        PerfilProfesional: 'Este es un ejemploooo',
        Activo: true,
      }
      console.log('Pacientes', responsePacientes)
      console.log('responseMedicos', responseMedicos)
      if (responsePacientes.length > 0) {
        store.commit('updateTokenAndUser', { token: response.token, user: responsePacientes })
      } else {
        store.commit('updateTokenAndUser', { token: response.token, user: medicoQuemado })
      }
      console.log('Este es el valor del store 2:', store.state.Token, '-', store.state.User)
      router.push({ name: 'dashboard' })
    } else {
      errores.value = 'Credenciales incorrectas'
      console.log('Ha ocurrido un error')
    }
  }
</script>
