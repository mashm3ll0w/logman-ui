<script setup>
import { onMounted, reactive, ref } from 'vue'
import { useRouter } from 'vue-router'
import {
  mdiMonitor,
  mdiPlus,
  mdiPencil,
  mdiDelete,
  mdiTextBoxOutline,
  mdiEye,
  mdiEyeOff
} from '@mdi/js'
import SectionMain from '@/components/SectionMain.vue'
import LayoutAuthenticated from '@/layouts/LayoutAuthenticated.vue'
import SectionTitleLineWithButton from '@/components/SectionTitleLineWithButton.vue'
import CardBox from '@/components/CardBox.vue'
import CardBoxModal from '@/components/CardBoxModal.vue'
import BaseButton from '@/components/BaseButton.vue'
import BaseButtons from '@/components/BaseButtons.vue'
import FormField from '@/components/FormField.vue'
import FormControl from '@/components/FormControl.vue'
import NotificationBar from '@/components/NotificationBar.vue'
import { apiClient, errorMessage } from '@/services/api'

const router = useRouter()

const sources = ref([])
const connections = ref([])
const loading = ref(false)
const error = ref('')
const notice = ref('')

const modalOpen = ref(false)
const editing = ref(null)
const saving = ref(false)
const deleteTarget = ref(null)

const form = reactive({ title: '', file_path: '', connection: null, is_active: true })

const connectionOptions = () =>
  connections.value.map((c) => ({ id: c.id, label: `${c.ssh_user}@${c.ssh_host}` }))

const load = async () => {
  loading.value = true
  error.value = ''
  try {
    const [s, c] = await Promise.all([apiClient.get('sources'), apiClient.get('connections')])
    sources.value = s.data
    connections.value = c.data
  } catch (e) {
    error.value = errorMessage(e, 'Failed to load sources')
  } finally {
    loading.value = false
  }
}

const openCreate = () => {
  editing.value = null
  form.title = ''
  form.file_path = ''
  form.connection = connectionOptions()[0] || null
  form.is_active = true
  modalOpen.value = true
}

const openEdit = (src) => {
  editing.value = src
  form.title = src.title
  form.file_path = src.file_path
  form.connection =
    connectionOptions().find((o) => o.id === src.connection?.id) || connectionOptions()[0] || null
  form.is_active = src.is_active
  modalOpen.value = true
}

const save = async () => {
  if (saving.value) return
  error.value = ''
  if (!form.title || !form.file_path || !form.connection) {
    error.value = 'Title, file path and connection are required.'
    modalOpen.value = true
    return
  }
  saving.value = true
  const payload = {
    title: form.title,
    file_path: form.file_path,
    connection: form.connection.id,
    is_active: form.is_active
  }
  try {
    if (editing.value) {
      await apiClient.put(`sources/${editing.value.id}`, payload)
      notice.value = 'Source updated.'
    } else {
      await apiClient.post('sources', payload)
      notice.value = 'Source created.'
    }
    await load()
  } catch (e) {
    error.value = errorMessage(e, 'Failed to save source')
    modalOpen.value = true
  } finally {
    saving.value = false
  }
}

const toggleActive = async (src) => {
  error.value = ''
  try {
    await apiClient.patch(`sources/${src.id}`, { is_active: !src.is_active })
    await load()
  } catch (e) {
    error.value = errorMessage(e, 'Failed to update source')
  }
}

const confirmDelete = async () => {
  if (!deleteTarget.value) return
  try {
    await apiClient.delete(`sources/${deleteTarget.value.id}`)
    notice.value = 'Source deleted.'
    await load()
  } catch (e) {
    error.value = errorMessage(e, 'Failed to delete source')
  } finally {
    deleteTarget.value = null
  }
}

const openLogs = (src) => router.push(`/sources/${src.id}/logs`)

onMounted(load)
</script>

<template>
  <LayoutAuthenticated>
    <SectionMain>
      <SectionTitleLineWithButton :icon="mdiMonitor" title="Log Sources" main>
        <BaseButton :icon="mdiPlus" label="Add Source" color="info" @click="openCreate" />
      </SectionTitleLineWithButton>

      <NotificationBar v-if="error" color="danger" class="mb-4">{{ error }}</NotificationBar>
      <NotificationBar v-if="notice" color="success" class="mb-4">{{ notice }}</NotificationBar>

      <CardBox has-table>
        <table>
          <thead>
            <tr>
              <th>Title</th>
              <th>File path</th>
              <th>Connection</th>
              <th>Status</th>
              <th class="text-right">Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="src in sources" :key="src.id">
              <td data-label="Title">{{ src.title }}</td>
              <td data-label="File path" class="font-mono text-sm">{{ src.file_path }}</td>
              <td data-label="Connection">
                {{ src.connection ? src.connection.ssh_user + '@' + src.connection.ssh_host : '—' }}
              </td>
              <td data-label="Status">
                <span
                  class="text-xs px-2 py-0.5 rounded-full"
                  :class="src.is_active ? 'bg-emerald-100 text-emerald-700' : 'bg-gray-200 text-gray-600'"
                >
                  {{ src.is_active ? 'enabled' : 'disabled' }}
                </span>
              </td>
              <td class="text-right whitespace-nowrap">
                <BaseButtons type="justify-end" no-wrap>
                  <BaseButton
                    :icon="mdiTextBoxOutline"
                    color="success"
                    small
                    :disabled="!src.is_active"
                    title="View logs"
                    @click="openLogs(src)"
                  />
                  <BaseButton :icon="mdiPencil" color="info" small title="Edit" @click="openEdit(src)" />
                  <BaseButton
                    :icon="src.is_active ? mdiEyeOff : mdiEye"
                    color="warning"
                    small
                    :title="src.is_active ? 'Disable' : 'Enable'"
                    @click="toggleActive(src)"
                  />
                  <BaseButton
                    :icon="mdiDelete"
                    color="danger"
                    small
                    title="Delete"
                    @click="deleteTarget = src"
                  />
                </BaseButtons>
              </td>
            </tr>
            <tr v-if="!loading && sources.length === 0">
              <td colspan="5" class="text-center text-slate-500 py-6">
                No sources yet. Click “Add Source” to create one.
              </td>
            </tr>
          </tbody>
        </table>
      </CardBox>
    </SectionMain>

    <CardBoxModal
      v-model="modalOpen"
      :title="editing ? 'Edit Source' : 'Add Source'"
      button-label="Save"
      has-cancel
      @confirm="save"
    >
      <FormField label="Title" help="Shown on the source card">
        <FormControl v-model="form.title" placeholder="e.g. Payments API" />
      </FormField>
      <FormField label="File path" help="Absolute path of the log file on the server">
        <FormControl v-model="form.file_path" placeholder="/var/log/app/app.log" />
      </FormField>
      <FormField label="Connection" help="SSH connection used to tail this source">
        <FormControl v-model="form.connection" :options="connectionOptions()" />
      </FormField>
      <p v-if="editing" class="text-sm text-slate-500">
        Use the enable/disable button on the list to change availability.
      </p>
    </CardBoxModal>

    <CardBoxModal
      :model-value="!!deleteTarget"
      title="Delete source"
      button="danger"
      button-label="Delete"
      has-cancel
      @confirm="confirmDelete"
      @cancel="deleteTarget = null"
      @update:model-value="(v) => { if (!v) deleteTarget = null }"
    >
      <p>
        Delete source <b>{{ deleteTarget?.title }}</b>? This cannot be undone.
      </p>
    </CardBoxModal>
  </LayoutAuthenticated>
</template>
