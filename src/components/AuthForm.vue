<template>
  <div class="login-container">
    <form @submit.prevent="handleSubmit">
      <h2>{{ isSignUp ? 'Регистрация' : 'Вход' }}</h2>

      <input
        v-if="isSignUp"
        v-model="name"
        placeholder="Имя"
        required
        autocomplete="name"
        :class="{
          invalid: errors.name,
          filled: name.trim() && !errors.name,
        }"
        @blur="validateField('name')"
      />

      <input
        v-model="login"
        placeholder="Логин"
        required
        autocomplete="username"
        :class="{
          invalid: errors.login,
          filled: login.trim() && !errors.login,
        }"
        @blur="validateField('login')"
      />

      <input
        type="password"
        v-model="password"
        :placeholder="isSignUp ? 'Придумайте пароль' : 'Пароль'"
        required
        :autocomplete="isSignUp ? 'new-password' : 'current-password'"
        :class="{
          invalid: errors.password,
          filled: password.trim() && !errors.password,
        }"
        @blur="validateField('password')"
      />

      <button type="submit" :disabled="isDisabled">
        {{ isSignUp ? 'Зарегистрироваться' : 'Войти' }}
        <span v-if="authStore.state.value.isLoading">...</span>
      </button>

      <p v-if="formattedError" class="error" v-html="formattedError"></p>

      <div v-show="!isSignUp" class="modal__form-group">
        <p>Нужно зарегистрироваться?</p>
        <RouterLink to="/signup" class="modal__link" @click="resetForm">
          Регистрируйтесь здесь
        </RouterLink>
      </div>

      <div v-show="isSignUp" class="modal__form-group">
        <p>Уже есть аккаунт?</p>
        <RouterLink to="/signin" class="modal__link" @click="resetForm"> Войдите здесь </RouterLink>
      </div>
    </form>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { authStore } from '@/store/auth'
import { useRouter } from 'vue-router'

const router = useRouter()
const props = defineProps({
  isSignUp: Boolean,
})

const name = ref('')
const login = ref('')
const password = ref('')

const errors = ref({
  name: false,
  login: false,
  password: false,
})

const validateField = (field) => {
  switch (field) {
    case 'name':
      errors.value.name = props.isSignUp && !name.value.trim()
      break
    case 'login':
      errors.value.login = !login.value.trim()
      break
    case 'password':
      errors.value.password = !password.value || password.value.length < 4
      break
  }
}

const validateForm = () => {
  validateField('name')
  validateField('login')
  validateField('password')
  return !Object.values(errors.value).some(Boolean)
}

const handleSubmit = async () => {
  if (!validateForm()) return

  try {
    const credentials = {
      login: login.value.trim(),
      password: password.value,
      ...(props.isSignUp && { name: name.value.trim() }),
    }

    const { success } = props.isSignUp
      ? await authStore.register(credentials)
      : await authStore.login(credentials)

    if (success) {
      router.push('/expenses')
      resetForm()
    }
  } catch (error) {
    authStore.state.value.error = { messages: [error.message || 'Произошла ошибка'] }
    password.value = ''
  }
}

const resetForm = () => {
  name.value = ''
  login.value = ''
  password.value = ''
  errors.value = { name: false, login: false, password: false }
}

const formattedError = computed(() => {
  if (!authStore.state.value.error?.messages) return ''
  return authStore.state.value.error.messages.join('<br>')
})

const isDisabled = computed(() => {
  return (
    Object.values(errors.value).some(Boolean) ||
    !login.value.trim() ||
    !password.value.trim() ||
    (props.isSignUp && !name.value.trim())
  )
})
</script>

<style scoped>
.login-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  background: #f0f0f0;
  padding: 20px;
}

form {
  width: 380px;
  border: 1px solid #e0e0e0;
  padding: 30px;
  border-radius: 15px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  background: white;
  box-sizing: border-box;
}

form h2 {
  text-align: center;
  margin-bottom: 30px;
  font-size: 24px;
  color: #333333;
}

input,
button {
  width: 100%;
  padding: 12px 10px;
  margin-bottom: 18px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 15px;
  transition: all 0.3s ease;
  box-sizing: border-box;
}

input.invalid {
  border-color: #dc2626;
  background: #fef2f2;
}

button:disabled {
  background: #999;
  cursor: not-allowed;
  opacity: 0.8;
}

input:focus {
  outline: 2px solid #6a11cb;
  border-color: transparent;
}

input.filled {
  background-color: #f3ebff;
}

button {
  background-color: #6a11cb;
  color: white;
  border: none;
  margin-bottom: 20px;
  cursor: pointer;
}

button:hover {
  background-color: #4a008f;
}

button[type='button'] {
  background-color: #6c757d;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 3px;
}

button[type='button']:hover {
  background-color: #495057;
}

.error {
  color: red;
  margin: 10px 0;
  text-align: center;
}

.logo img {
  width: 100px;
  height: auto;
  margin-bottom: 20px;
}

p {
  margin: 10px 0;
}

.toggle-btn {
  display: inline-block;
  background: none;
  border: none;
  color: #6a11cb;
  padding: 0;
  margin-left: 5px;
  cursor: pointer;
  font-weight: 500;
  text-decoration: none;
  transition: color 0.3s;
}

.toggle-btn:hover {
  text-decoration: underline;
  color: #4a008f;
}

.toggle-text {
  color: #666666;
  font-size: 14px;
  margin: 15px 0 0;
}

.form-footer {
  text-align: center;
  margin-top: 20px;
}

.modal__form-group {
  text-align: center;
  margin-top: 15px;
}

.modal__form-group p,
.modal__form-group a {
  color: rgba(148, 166, 190, 0.4);
  font-size: 14px;
  font-weight: 400;
  line-height: 150%;
  letter-spacing: -0.14px;
}

.modal__form-group a {
  text-decoration: underline;
}

.modal__link {
  color: #6a11cb;
  text-decoration: none;
  transition: color 0.3s;
}

.modal__link:hover {
  text-decoration: underline;
  color: #4a008f;
}
</style>
