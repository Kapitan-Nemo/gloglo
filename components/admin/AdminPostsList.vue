<script setup lang="ts">
import { getFirestore } from '@firebase/firestore'
import { collection, deleteDoc, doc, onSnapshot } from 'firebase/firestore'
import { useEditID } from '~/composables/states'
import { convertTimestamp } from '@/composables/helpers'

const allPosts = useAllPosts()
const editID = useEditID()
const show = useShowPost()

async function fetchPosts() {
  onSnapshot(collection(getFirestore(), 'posts/'), (snap) => {
    allPosts.value = []
    snap.forEach((doc) => {
      allPosts.value?.push(doc.data() as IPost)
    })
  })
}

onMounted(async () => {
  try {
    await fetchPosts()
  }
  catch (error) {
    useToast('Błąd pobierania wpisów', 'error')
  }
})

function editOrder(id: string) {
  editID.value = id
}

async function deleteOrder(id: string) {
  await deleteDoc(doc(getFirestore(), 'posts', id)).then(() => {
    useToast('Wpis blogowy usunięty', 'success')
  }).catch((error) => {
    useToast(error, 'error')
  })
}
</script>

<template>
  <!-- <div class="flex justify-between">
    <input id="search" v-model="search" placeholder="Search product" class="w-1/2 px-3 py-3 mb-3 border-b border-white bg-dark-200 text-white caret-white placeholder-gray-500 focus:outline-none" type="search">
  </div> -->

  <div class="relative overflow-x-auto shadow-md">
    <table class="w-full text-sm text-left text-white">
      <thead class="text-xs uppercase bg-dark-200 text-white">
        <tr>
          <th scope="col" class="table-cell px-6 py-3">
            Tytuł
          </th>
          <th scope="col" class="table-cell px-6 py-3">
            Slug
          </th>
          <th scope="col" class="table-cell px-6 py-3">
            Meta tytuł
          </th>
          <th scope="col" class="table-cell px-6 py-3">
            Meta Opis
          </th>
          <th scope="col" class="table-cell px-6 py-3">
            Data
          </th>
          <th scope="col" class="table-cell px-6 py-3">
            Akcja
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="p in allPosts" :key="p.created_at">
          <th scope="row" class="table-cell px-6 py-4 font-medium whitespace-nowrap">
            {{ p.title }}
          </th>
          <td class="table-cell px-6 py-4">
            {{ p.slug }}
          </td>
          <td class="table-cell px-6 py-4">
            {{ p.meta_title }}
          </td>
          <td class="table-cell px-6 py-4">
            {{ p.meta_description }}
          </td>
          <td class="table-cell px-6 py-4">
            {{ convertTimestamp(p.created_at) }}
          </td>
          <td class="table-cell px-6 py-4">
            <button class="mr-4" @click="editOrder(p.id), (show.drawer = true, show.edit = true)">
              <Icon class="text-red-300" size="30" name="ion:pencil-outline" />
            </button>
            <button @click="deleteOrder(p.id)">
              <Icon class="text-red-300" size="30" name="ion:trash-outline" />
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped>
.table-cell {
  width: 150px; /* Adjust this value to your liking */
}
</style>
