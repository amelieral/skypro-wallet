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
  </header>
</template>

<script setup></script>

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
  color: #333;
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
</style>
