<template>
  <div class="flex justify-between items-center">
    <h1 class="text-4xl font-black text-gray-800">Entre na sua conta</h1>
    <button @click="close" class="text-4xl text-gray-600 focus:outline-none">
      &times;
    </button>
  </div>

  <div class="mt-16">
    <form @submit.prevent="handleSubmit">
      <label for="" class="block">
        <span class="text-lg font-medium text-gray-800">E-mail</span>
        <input
          v-model="state.email.value"
          type="email"
          :class="{ 'border-brand-danger': !!state.email.errorMessage }"
          class="black w-full px-4 py-3 mt-1 text-lg bg-gray-100 border-2 border-transparent rounded"
          placeholder="jane.doe@mail.com"
        />
        <span
          v-if="!!state.email.errorMessage"
          class="block font-medium text-brand-danger"
          >{{ state.email.errorMessage }}</span
        >
      </label>

      <label for="" class="block mt-9">
        <span class="text-lg font-medium text-gray-800">Senha</span>
        <input
          v-model="state.password.value"
          type="password"
          :class="{ 'border-brand-danger': !!state.password.errorMessage }"
          class="black w-full px-4 py-3 mt-1 text-lg bg-gray-100 border-2 border-transparent rounded"
          placeholder="***"
        />
        <span
          v-if="!!state.password.errorMessage"
          class="block font-medium text-brand-danger"
          >{{ state.password.errorMessage }}</span
        >
      </label>

      <button
        :disabled="state.isLoading"
        type="submit"
        :class="{ 'opacity-50': state.isLoading }"
        class="px-8 py-3 mt-10 text-2xl font-bold text-white rounded-full bg-brand-main transition-all duration-150 focus:outline-none"
      >
        <icon v-if="state.isLoading" name="loading" class="animate-spin" />
        <span v-else>Entrar</span>
      </button>
    </form>
  </div>
</template>

<script>
import { reactive } from '@vue/reactivity'
import { useField } from 'vee-validate'
import { useRouter } from 'vue-router'
import { useToast } from 'vue-toastification'
import useModal from '../../hooks/useModal'
import {
  validateEmptyAndLength3,
  validateEmptyAndEmail,
} from '../../utils/validators'
import services from '../../services'
import Icon from '../Icon'

export default {
  name: 'ModalLogin',
  components: { Icon },
  setup() {
    const modal = useModal()
    const router = useRouter()
    const toast = useToast()

    const { value: emailValue, errorMessage: emailErrorMessage } = useField(
      'email',
      validateEmptyAndEmail
    )
    const { value: passwordValue, errorMessage: passwordErrorMessage } =
      useField('password', validateEmptyAndLength3)

    const state = reactive({
      hasErrors: false,
      isLoading: false,
      email: {
        value: emailValue,
        errorMessage: emailErrorMessage,
      },
      password: {
        value: passwordValue,
        errorMessage: passwordErrorMessage,
      },
    })

    async function handleSubmit() {
      try {
        toast.clear()
        state.isLoading = true
        const { data, errors } = await services.auth.login({
          email: state.email.value,
          password: state.password.value,
        })

        if (!errors) {
          window.localStorage.setItem('token', data.token)
          router.push({ name: 'Feedbacks' })
          state.isLoading = false
          modal.close()
          return
        }

        if (errors.status === 400) {
          toast.error('Ocorreu um erro ao fazer login')
        }
        if (errors.status === 401) {
          toast.error('E-mail ou senha inválidos')
        }
        if (errors.status === 404) {
          toast.error('E-mail não encontrado')
        }

        state.isLoading = false
      } catch (error) {
        state.isLoading = false
        state.hasErrors = !!error
        toast.error('Ocorreu um erro ao fazer o login')
      }
    }

    return { state, close: modal.close, handleSubmit }
  },
}
</script>
