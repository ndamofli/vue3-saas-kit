<template>
  <div class="relative shadow-2xl" v-click-outside="closeDropdownTenant">
    <span class="inline-block w-full rounded-sm shadow-sm">
      <button
        @click="toggleDropDown"
        type="button"
        aria-haspopup="listbox"
        aria-expanded="true"
        aria-labelledby="listbox-label"
        class="bg-slate-800 hover:bg-theme-700 hover:text-theme-50 truncate text-slate-300 cursor-pointer w-full pl-3 py-2 text-left focus:outline-none transition ease-in-out duration-150 sm:leading-5 rounded-sm shadow-sm pr-7"
      >
        <span class="font-bold">{{ currentTenant.name }}</span>
        <span class="absolute inset-y-0 right-0 flex items-center pr-2 pointer-events-none">
          <svg
            class="h-5 w-5 text-slate-200"
            viewBox="0 0 20 20"
            fill="none"
            stroke="currentColor"
          >
            <path
              d="M7 7l3-3 3 3m0 6l-3 3-3-3"
              stroke-width="1.5"
              stroke-linecap="round"
              stroke-linejoin="round"
            />
          </svg>
        </span>
      </button>
    </span>

    <!-- Select popover, show/hide based on select state. -->
    <transition
      enter-active-class="transition ease-out duration-100"
      enter-class="transform opacity-0 scale-95"
      enter-to-class="transform opacity-100 scale-100"
      leave-active-class="transition ease-in duration-75"
      leave-class="transform opacity-100 scale-100"
      leave-to-class="transform opacity-0 scale-95"
    >
      <div
        v-show="dropdown"
        class="z-40 absolute object-top mt-1 w-full rounded-sm bg-white shadow-lg"
        :class="dropdown ? 'z-40 absolute object-top mt-1 w-full rounded-sm bg-white shadow-lg' : 'hidden'"
      >
        <div
          class="m-1 border border-gray-100 relative flex items-stretch flex-grow focus-within:z-10"
        >
          <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
            <!-- Heroicon name: solid/users -->
            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="h-5 w-5 text-gray-500"
              viewBox="0 0 20 20"
              fill="currentColor"
            >
              <path
                fill-rule="evenodd"
                d="M8 4a4 4 0 100 8 4 4 0 000-8zM2 8a6 6 0 1110.89 3.476l4.817 4.817a1 1 0 01-1.414 1.414l-4.816-4.816A6 6 0 012 8z"
                clip-rule="evenodd"
              />
            </svg>
          </div>
          <input
            :disabled="loading"
            id="search"
            ref="inputSearch"
            :placeholder="$t('shared.searchDot')"
            v-model="search"
            autocomplete="off"
            @keyup.enter="enter"
            class="focus:ring-theme-500 focus:border-theme-500 block w-full rounded-none rounded-l-sm pl-10 sm:text-sm px-3 py-2 bg-gray-100 text-sm focus:outline-none"
          />
        </div>
        <ul
          tabindex="-1"
          role="listbox"
          aria-labelledby="listbox-label"
          class="max-h-60 rounded-sm text-sm leading-5 shadow-xs overflow-auto focus:outline-none sm:text-sm sm:leading-5"
        >
          <li v-for="(tenant, index) in myTenants" :key="index" role="option">
            <button
              type="button"
              @click="changeTenant(tenant)"
              class="w-full text-left text-gray-900 cursor-pointer select-none relative py-2 pl-3 pr-9 hover:bg-gray-100 border-b border-gray-200 focus:outline-none"
            >
              <div>
                <img
                  alt="Tenant icon"
                  v-if="tenant.icon"
                  class="h-8 w-8 rounded-sm overflow-hidden float-left mr-2 -ml-1 mt-1"
                  :src="tenant.icon"
                />
                <span class="font-medium">{{ tenant.name }}</span>
                <span
                  v-if="tenant.id === currentTenant.id"
                  class="text-slate-500 absolute inset-y-0 right-0 flex items-center pr-4 -mt-4"
                >
                  <svg class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                    <path
                      fill-rule="evenodd"
                      d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z"
                      clip-rule="evenodd"
                    />
                  </svg>
                </span>
                <p class="text-xs">{{ getPlanFromTenant(tenant) }}</p>
              </div>
            </button>
          </li>
          <button
            @click="addTenant"
            role="option"
            class="w-full text-left font-bold text-slate-800 cursor-pointer select-none relative py-2 pl-3 pr-9 hover:bg-gray-100 focus:outline-none"
          >{{ $t("settings.tenant.create") }}</button>

          <!-- More options... -->
        </ul>
      </div>
    </transition>
  </div>
</template>

<script setup lang="ts">
import services from "@/services";
import { TenantDto } from "@/application/dtos/core/tenants/TenantDto";
import store from "@/store";
import { useRouter } from "vue-router";
import i18n from "@/locale/i18n";
import { computed, nextTick, ref } from "vue";

const $t = i18n.global.t;
const router = useRouter();

const $emit = defineEmits(['add'])

const inputSearch = ref<HTMLInputElement>();

const loading = ref(false);
const dropdown = ref(false);
const search = ref("");

function closeDropdownTenant() {
  dropdown.value = false;
}
function toggleDropDown() {
  dropdown.value = !dropdown.value;
  if (dropdown.value) {
    nextTick(() => {
      inputSearch.value?.focus();
      inputSearch.value?.select();
    });
  }
}
function enter() {
  if (myTenants.value.length === 1) {
    changeTenant(myTenants.value[0]);
  }
}
function changeTenant(tenant: TenantDto) {
  search.value = "";
  closeDropdownTenant();
  loading.value = true;
  services.users
    .updateDefaultTenant(tenant.id)
    .then(() => {
      router.go(0);
    })
    .finally(() => {
      loading.value = false;
    });
}
function addTenant() {
  store.commit("layout/sidebarOpen", false);
  closeDropdownTenant();
  $emit("add");
}
function getPlanFromTenant(tenant: TenantDto): string {
  if (tenant.products && tenant.products.length > 0 && tenant.products[0].subscriptionProduct) {
    return $t(tenant.products[0].subscriptionProduct.title).toString();
  } else {
    if (tenant.subdomain === "admin") {
      return "Admin";
    } else {
      return $t("settings.subscription.notSubscribed").toString();
    }
  }
}
const currentTenant = computed((): TenantDto => {
  return store.state.tenant.current ?? ({ name: "Undefinded" } as TenantDto);
})
const myTenants = computed((): TenantDto[] => {
  return store.state.tenant.tenants
    .filter((f) => !search.value || f.name.toLowerCase().includes(search.value.toLowerCase()))
    .sort((x, y) => {
      if (x.name && y.name) {
        return (x.name > y.name ? 1 : -1) ?? 1;
      }
      return 1;
    });
})
</script>
