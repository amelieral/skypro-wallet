<template>
  <header class="header">
    <div class="logo">
      <img src="@/assets/icon/wallet-logo.svg" alt="Логотип" />
    </div>
    <nav class="desktop-nav">
      <router-link to="/expenses" class="nav-link" :exact-active-class="'active'">
        Мои расходы
      </router-link>
      <router-link to="/analysis" class="nav-link" :exact-active-class="'active'">
        Анализ расходов
      </router-link>
    </nav>
    <button class="logout-btn" @click="handleLogout" :disabled="isLoggingOut">
      {{ isLoggingOut ? 'Выход...' : 'Выйти' }}
    </button>
    <transition name="fade">
      <div v-if="logoutError" class="error-message">
        {{ logoutError }}
      </div>
    </transition>
  </header>
</template>

<script setup>
import { ref } from 'vue'
import { authStore } from '@/store/auth'
import { useRouter } from 'vue-router'

const router = useRouter()
const isLoggingOut = ref(false)
const logoutError = ref(null)

const handleLogout = async () => {
  try {
    isLoggingOut.value = true
    logoutError.value = null

    await authStore.logout()

    if (router.currentRoute.value.path !== '/signin') {
      router.replace('/signin').catch(() => {
        window.location.href = '/signin'
      })
    }
  } catch (error) {
    console.error('Ошибка при выходе:', error)
    logoutError.value = 'Не удалось выйти. Попробуйте ещё раз.'
    setTimeout(() => {
      logoutError.value = null
    }, 5000)
  } finally {
    isLoggingOut.value = false
  }
}
</script>

<style scoped>
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 24px 48px;
  background: #ffffff;
  box-shadow: 0 2px 10px #0000000d;
  position: relative;
}

.desktop-nav {
  display: flex;
  gap: 48px;
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
}

.nav-link {
  color: #000000;
  text-decoration: none;
  font-size: 14px;
  transition: all 0.2s ease;
  position: relative;
}

.nav-link:hover:not(.active) {
  color: #6d28d9;
  font-weight: 600;
}

.nav-link.active {
  color: #6d28d9;
  font-weight: 600;
}

.nav-link.active::after {
  content: '';
  position: absolute;
  left: 0;
  bottom: -4px;
  width: 100%;
  height: 2px;
  background: #6d28d9;
}

.logout-btn {
  background: none;
  border: none;
  cursor: pointer;
  color: #000000;
  font-size: 14px;
  font-weight: 600;
  padding: 0;
  transition: color 0.2s ease;
}

.logout-btn:hover {
  color: #6d28d9;
}
</style>
