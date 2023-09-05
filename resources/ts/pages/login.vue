<script setup lang="ts">
import AuthProvider from '@/views/pages/authentication/AuthProvider.vue'
import { useTheme } from 'vuetify'

import logo from '@images/logo.svg?raw'
import authV1MaskDark from '@images/pages/auth-v1-mask-dark.png'
import authV1MaskLight from '@images/pages/auth-v1-mask-light.png'
import authV1Tree2 from '@images/pages/auth-v1-tree-2.png'
import authV1Tree from '@images/pages/auth-v1-tree.png'

import axios from 'axios'
import { useRouter } from 'vue-router'; // Asegúrate de que la ruta sea correcta según tu configuración

const form = ref({
  email: '',
  password: '',
  remember: false,
})

const vuetifyTheme = useTheme()
const authThemeMask = computed(() => {
  return vuetifyTheme.global.name.value === 'light'
    ? authV1MaskLight
    : authV1MaskDark
})

const isPasswordVisible = ref(false)

const serverErrors = ref(null);

const router = useRouter();

const submit = async () => {
  console.log('Iniciando solicitud...');

  try {
    // Obtener la cookie CSRF primero
    await axios.get('/sanctum/csrf-cookie');

    // Ahora, realizar la solicitud POST de inicio de sesión
    const response = await axios.post('http://127.0.0.1:8000/api/login', {
      email: form._value.email,
      password: form._value.password,
    });

    console.log('Respuesta del servidor:', response);
    
    // Verificar si la respuesta contiene un token
    if (response.data.access_token) {
      // Guardar el token en el localStorage
      localStorage.setItem('access_token', response.data.access_token);

      // Redirigir al usuario utilizando el enrutador interno de Vue.js
      router.push('/dashboard'); // Utiliza la instancia del enrutador
    } else {
      serverErrors.value = 'Error en la solicitud. Verifica tus credenciales e inténtalo de nuevo.';
      console.log('serverErrors', serverErrors.value);
    }

    // Resto del código de manejo de éxito aquí
  } catch (err) {
    serverErrors.value = err.response.data.message;
    console.error('Error en la solicitud:', err);
    // Resto del código de manejo de errores aquí
    console.log('serverErrors', serverErrors.value);
  }
};

</script>

<template>
  <div class="auth-wrapper d-flex align-center justify-center pa-4">
    <VCard
      class="auth-card pa-4 pt-7"
      max-width="448"
    >
      <VCardItem class="justify-center">
        <template #prepend>
          <div class="d-flex">
            <div v-html="logo" />
          </div>
        </template>

        <VCardTitle class="font-weight-semibold text-2xl text-uppercase">
          Materio
        </VCardTitle>
      </VCardItem>

      <VCardText class="pt-2">
        <h5 class="text-h5 font-weight-semibold mb-1">
          Welcome to Materio! 👋🏻
        </h5>
        <p class="mb-0">
          Please sign-in to your account and start the adventure
        </p>
      </VCardText>

      <VCardText>
        <VForm @submit.prevent="(submit)">
          <VRow>
            <!-- email -->
            <VCol cols="12">
              <VTextField
                v-model="form.email"
                label="Email"
                type="email"
              />
            </VCol>

            <!-- password -->
            <VCol cols="12">
              <VTextField
                v-model="form.password"
                label="Password"
                :type="isPasswordVisible ? 'text' : 'password'"
                :append-inner-icon="isPasswordVisible ? 'mdi-eye-off-outline' : 'mdi-eye-outline'"
                @click:append-inner="isPasswordVisible = !isPasswordVisible"
              />

              <!-- remember me checkbox -->
              <div class="d-flex align-center justify-space-between flex-wrap mt-1 mb-4">
                <VCheckbox
                  v-model="form.remember"
                  label="Remember me"
                />

                <a
                  class="ms-2 mb-1"
                  href="javascript:void(0)"
                >
                  Forgot Password?
                </a>
              </div>

              <!-- login button -->
              <VBtn
                block
                type="submit"

              >
                Login
              </VBtn>
            </VCol>
            

            <div
             v-if="serverErrors" class="error-message text-center text-base" cols="12">
              {{ serverErrors }}
            </div>
            
            <!-- create account -->
            <VCol
              cols="12"
              class="text-center text-base"
            >
              <span>New on our platform?</span>
              <RouterLink
                class="text-primary ms-2"
                to="/register"
              >
                Create an account
              </RouterLink>
            </VCol>

            <VCol
              cols="12"
              class="d-flex align-center"
            >
              <VDivider />
              <span class="mx-4">or</span>
              <VDivider />
            </VCol>

            <!-- auth providers -->
            <VCol
              cols="12"
              class="text-center"
            >
              <AuthProvider />
            </VCol>
          </VRow>
        </VForm>
      </VCardText>
    </VCard>

    <VImg
      class="auth-footer-start-tree d-none d-md-block"
      :src="authV1Tree"
      :width="250"
    />

    <VImg
      :src="authV1Tree2"
      class="auth-footer-end-tree d-none d-md-block"
      :width="350"
    />

    <!-- bg img -->
    <VImg
      class="auth-footer-mask d-none d-md-block"
      :src="authThemeMask"
    />
  </div>
</template>

<style lang="scss">
@use "@core-scss/pages/page-auth.scss";
</style>
