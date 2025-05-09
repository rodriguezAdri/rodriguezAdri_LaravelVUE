<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useLinks } from '~~/composables/useLinks'
import SearchInput from '@/components/SearchInput.vue'
import TableTh from '@/components/TableTh.vue'
// @ts-expect-error
import { TailwindPagination } from 'laravel-vue-pagination'

definePageMeta({
  middleware: ['auth'],
})

const route = useRoute()
const router = useRouter()

const queries = ref({
  page: Number(route.query.page) || 1,
  sort: route.query.sort?.toString() || '',
  "filter[full_link]": route.query["filter[full_link]"]?.toString() || '',
  ...route.query
})

const { data, index: getLinks, destroy } = useLinks({ queries })

async function handleDelete(id: number) {
  await destroy(id)
  if (data.value) {
    data.value.data = data.value.data.filter(link => link.id !== id)
  }
}

watch(queries, () => {
  router.push({ query: queries.value })
  getLinks()
}, { deep: true })

watch(() => route.query, (newQuery) => {
  queries.value = {
    page: Number(newQuery.page) || 1,
    sort: newQuery.sort?.toString() || '',
    "filter[full_link]": newQuery["filter[full_link]"]?.toString() || '',
    ...newQuery
  }
})

onMounted(() => {
  getLinks()

  document.addEventListener('visibilitychange', () => {
    if (document.visibilityState === 'visible') {
      getLinks()
    }
  })
})
</script>

<template>
  <div>
    <nav class="flex justify-between mb-4 items-center">
      <h1 class="text-xl font-bold mb-0">My Links</h1>
      <div class="flex items-center">
        <SearchInput
          v-model="queries['filter[full_link]']"
          placeholder="Search my URL..."
        />
        <NuxtLink to="/links/create" class="ml-4 text-green-600">
          <IconPlusCircle class="inline" /> Create New
        </NuxtLink>
      </div>
    </nav>

    <div v-if="data?.data?.length! > 0">
      <table class="table-fixed w-full">
        <thead>
          <tr>
            <TableTh class="w-[35%]" name="full_link" :model-value="queries.sort">Full Link</TableTh>
            <TableTh class="w-[35%]" name="short_link" :model-value="queries.sort">Short Link</TableTh>
            <TableTh class="w-[10%]" name="views" :model-value="queries.sort">Views</TableTh>
            <TableTh class="w-[10%]">Edit</TableTh>
            <TableTh class="w-[10%]">Trash</TableTh>
            <TableTh class="w-[6%] text-center">
              <button @click="getLinks()">
                <IconRefresh class="w-[15px] relative top-[2px]" />
              </button>
            </TableTh>
          </tr>
        </thead>
        <tbody>
          <tr v-for="link in data?.data" :key="link.short_link">
            <td :title="`created ${useTimeAgo(link.created_at).value}`">
              <a :href="link.full_link" target="_blank">
                {{ link.full_link.replace(/^http(s?):\/\//, '') }}
              </a>
            </td>
            <td>
              <a
                :href="`${useRuntimeConfig().public.appURL}/${link.short_link}`"
                target="_blank"
              >
                {{
                  useRuntimeConfig().public.appURL.replace(/^http(s?):\/\//, '')
                }}/{{ link.short_link }}
              </a>
            </td>
            <td>{{ link.views }}</td>
            <td>
              <NuxtLink class="no-underline" :to="`/links/${link.short_link}`">
                <IconEdit />
              </NuxtLink>
            </td>
            <td>
              <button @click="handleDelete(link.id)">
                <IconTrash />
              </button>
            </td>
            <td></td>
          </tr>
        </tbody>
      </table>

      <div class="mt-5 flex justify-center">
        <TailwindPagination
          :data="data"
          @pagination-change-page="(value) => (queries.page = value)"
        />
      </div>
    </div>

    <div v-else class="border-dashed border-gray-500 p-3 border-[1px] text-center mt-6">
      <div class="flex justify-center mb-2">
        <IconLink />
      </div>
      <p>
        <span>No links created yet.</span>
        <NuxtLink to="/links/create" class="text-green-600 ml-1">Create your first link</NuxtLink>
      </p>
    </div>
  </div>
</template>
