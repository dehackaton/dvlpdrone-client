<template>
  <header class="w-full border-b bg-white shadow-sm">
    <div class="mx-auto max-w-[900px] flex items-center justify-between px-4 py-3">
      <div class="flex items-center gap-6">
        <NuxtLink to="/" class="text-xl font-bold text-primary">
          LOGO
        </NuxtLink>

        <nav class="hidden md:flex gap-4 text-sm font-medium text-gray-700">
          <NuxtLink
            to="/"
            :class="[
              isActive('/') ? 'text-primary font-semibold' : 'hover:text-primary',
              'transition'
            ]"
          >
            Мониторинг
          </NuxtLink>
          <NuxtLink
            to="/requests"
            :class="[
              isActive('/requests') ? 'text-primary font-semibold' : 'hover:text-primary',
              'transition'
            ]"
          >
            Заявки
          </NuxtLink>
          <NuxtLink
            to="/create-request"
            :class="[
              isActive('/create-request') ? 'text-primary font-semibold' : 'hover:text-primary',
              'transition'
            ]"
          >
            Создать заявку
          </NuxtLink>
        </nav>
      </div>

      <div class="flex items-center gap-3">
        <UAvatar
          alt="Hрофиль"
          size="md"
        />
        <span class="font-semibold">Hasan</span>
      </div>
    </div>
  </header>
</template>

<script setup>
import { useRoute } from 'vue-router'

const route = useRoute()

const isActive = (path) => {
  if (path === '/') return route.path === '/'
  return route.path.startsWith(path)
}
</script>
