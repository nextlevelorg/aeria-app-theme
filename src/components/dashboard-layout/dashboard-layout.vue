<script setup lang="ts">
import { type MenuSchema, useStore, useNavbar, t } from 'waltz-ui'
import { inject, ref, computed, onMounted } from 'vue'
import { AeriaIcon, AeriaContextMenu, AeriaPicture, AeriaBadge } from '@waltz-ui/ui'

import {
  breakpoints,
  navbarRefs,
  routes,
  pushRoute,
  routeClick,
} from '../utils'

import NavbarEntry from '../navbar-entry/navbar-entry.vue'
import NavbarEntries from '../navbar-entries/navbar-entries.vue'

const menuSchema = inject<MenuSchema>('menuSchema', [])

const routerReady = ref(false)
const router = ROUTER

const metaStore = useStore('meta')
const userStore = useStore('user')

onMounted(async () => {
  const navbar = await useNavbar({ schema: menuSchema })

  Object.assign(navbarRefs, navbar)
  routerReady.value = true
})

const logoUrl = new URL('/logo.png', import.meta.url).href

const topLevelRoutes = computed(() => {
  return routes.value.filter((route) => route.children.length === 0)
})

const parentRoutes = computed(() => {
  return routes.value.filter((route) => route.children.length > 0)
})
</script>


<template>
  <div
    v-if="routerReady"
    class="
      dashboard
      aeria-body
  ">
    <div :class="`
      no-print
      dashboard__sidebar
      ${ metaStore.menu.visible && 'dashboard__sidebar--visible' }
    `">
      <div class="dashboard__navbar-top">
        <aeria-icon
          v-if="!breakpoints.md"
          v-clickable
          icon="x"
          @click="metaStore.menu.visible = false"
        ></aeria-icon>

        <img
          v-clickable
          :src="logoUrl"
          class="dashboard__navbar-logo"
          @click="pushRoute('/dashboard')"
        />
      </div>

      <nav class="dashboard__navbar">
        <div
          v-if="topLevelRoutes.length > 0"
          class="dashboard__route-group"
        >
          <navbar-entry
            v-for="(item, index) in topLevelRoutes"
            :key="`item-${index}`"
            :item="item"
            :memo-key="`entry-${index}`"
            @click="routeClick(item)"
          ></navbar-entry>
        </div>

        <div
          v-for="(item, index) in parentRoutes"
          :key="`item-${index}`"
          class="dashboard__route-group"
        >
          <navbar-entries
            :item="item"
            :memo-key="`parent-${index}`"
          ></navbar-entries>
        </div>

      </nav>
    </div>

    <div class="dashboard__content">
      <div class="dashboard__topbar">
        <aeria-icon
          v-if="!breakpoints.md"
          v-clickable
          icon="list"
          @click="metaStore.menu.visible = true"
        ></aeria-icon>

        <img
          v-if="!breakpoints.md"
          v-clickable
          :src="logoUrl"
          class="dashboard__topbar-logo"
          @click="pushRoute('/dashboard')"
        />

        <div class="dashboard__topbar-separator"></div>

        <slot
          v-if="$slots.super"
          name="super"
        ></slot>

        <aeria-context-menu>
          <div
            v-clickable
            class="dashboard__user"
          >
            <aeria-picture
              :file-id="currentUser.picture_file?._id || currentUser.picture_file._id"
              class="dashboard__user-picture"
            ></aeria-picture>

            <div>Olá, {{ currentUser.name.split(' ')[0] }}</div>
          </div>

          <template #header>
            <div class="dashboard__user-context-header">
              <div class="tx-text-[12pt]">
                {{ currentUser.name }}
              </div>

              <div class="dashboard__user-context-roles">
                <aeria-badge
                  v-for="role in currentUser.roles"
                  :key="`role-${role}`"
                  large
                >
                  {{ t(role) }}
                </aeria-badge>
              </div>
            </div>
          </template>

          <template #profile>
            <aeria-icon
              icon="user-square"
              @click="router.push('/dashboard/user/profile')"
            >
              Perfil
            </aeria-icon>
          </template>

          <template #logout>
            <aeria-icon
              icon="sign-out"
              @click="userStore.$actions.signout(); router.push('/user/signin')"
            >
              Sair
            </aeria-icon>
          </template>

          <template #extra>
            <slot name="user-profile"></slot>
          </template>
        </aeria-context-menu>

      </div>

      <div :class="{
        'dashboard__view': true,
        'dashboard__view--padded': !$route.meta.fill
      }">

        <div class="dashboard__view-top">
          <slot
            v-if="$slots['view-title']"
            name="view-title"
          ></slot>

          <div
            v-else
            class="dashboard__view-title"
          >
            {{ t(viewTitle, { capitalize: true }) }}
          </div>
          <router-view name="topbar"></router-view>
        </div>


        <router-view v-slot="{ Component }">
          <div class="dashboard__view-content">
            <component :is="Component"></component>
          </div>
        </router-view>
      </div>

    </div>
  </div>

</template>

<style scoped src="./dashboard-layout.less"></style>

