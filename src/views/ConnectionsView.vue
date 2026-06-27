<script setup>
import { onMounted, reactive, ref } from 'vue'
import { mdiServerNetwork, mdiPlus, mdiPencil, mdiDelete, mdiEye, mdiEyeOff } from '@mdi/js'
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
import { useAuthStore } from '@/stores/auth'

const auth = useAuthStore()

const connections = ref([])
const loading = ref(false)
const error = ref('')
const notice = ref('')

const modalOpen = ref(false)
const editing = ref(null)
const saving = ref(false)
const deleteTarget = ref(null)

const form = reactive({ ssh_user: '', ssh_host: '', ssh_port: 22, ssh_pass: '' })

const load = async () => {
  loading.value = true
  error.value = ''
  try {
    const { data } = await apiClient.get('connections')
    connections.value = data
  } catch (e) {
    error.value = errorMessage(e, 'Failed to load connections')
  } finally {
    loading.value = false
  }
}

const openCreate = () => {
  editing.value = null
  form.ssh_user = ''
  form.ssh_host = ''
  form.ssh_port = 22
  form.ssh_pass = ''
  modalOpen.value = true
}

const openEdit = (conn) => {
  editing.value = conn
  form.ssh_user = conn.ssh_user
  form.ssh_host = conn.ssh_host
  form.ssh_port = conn.ssh_port
  form.ssh_pass = ''
  modalOpen.value = true
}

const save = async () => {
  if (saving.value) return
  error.value = ''
  if (!form.ssh_user || !form.ssh_host || !form.ssh_port) {
    error.value = 'User, host and port are required.'
    modalOpen.value = true
    return
  }
  if (!editing.value && !form.ssh_pass) {
    error.value = 'A password is required for a new connection.'
    modalOpen.value = true
    return
  }
  saving.value = true
  const payload = {
    ssh_user: form.ssh_user,
    ssh_host: form.ssh_host,
    ssh_port: Number(form.ssh_port)
  }
  if (form.ssh_pass) payload.ssh_pass = form.ssh_pass
  try {
    if (editing.value) {
      await apiClient.put(`connections/${editing.value.id}`, payload)
      notice.value = 'Connection updated.'
    } else {
      await apiClient.post('connections', payload)
      notice.value = 'Connection created.'
    }
    await load()
  } catch (e) {
    error.value = errorMessage(e, 'Failed to save connection')
    modalOpen.value = true
  } finally {
    saving.value = false
  }
}

const toggleActive = async (conn) => {
  error.value = ''
  try {
    await apiClient.patch(`connections/${conn.id}`, { is_active: !conn.is_active })
    await load()
  } catch (e) {
    error.value = errorMessage(e, 'Failed to update connection')
  }
}

const confirmDelete = async () => {
  if (!deleteTarget.value) return
  try {
    await apiClient.delete(`connections/${deleteTarget.value.id}`)
    notice.value = 'Connection deleted.'
    await load()
  } catch (e) {
    error.value = errorMessage(e, 'Failed to delete connection (it may still be used by a source)')
  } finally {
    deleteTarget.value = null
  }
}

onMounted(load)
</script>

<template>
  <LayoutAuthenticated>
    <SectionMain>
      <SectionTitleLineWithButton :icon="mdiServerNetwork" title="SSH Connections" main>
        <BaseButton
          v-if="auth.isSuperAdmin"
          :icon="mdiPlus"
          label="Add Connection"
          color="info"
          @click="openCreate"
        />
      </SectionTitleLineWithButton>

      <NotificationBar v-if="error" color="danger" class="mb-4">{{ error }}</NotificationBar>
      <NotificationBar v-if="notice" color="success" class="mb-4">{{ notice }}</NotificationBar>

      <CardBox has-table>
        <table>
          <thead>
            <tr>
              <th>User</th>
              <th>Host</th>
              <th>Port</th>
              <th>Status</th>
              <th v-if="auth.isSuperAdmin" class="text-right">Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="conn in connections" :key="conn.id">
              <td data-label="User">{{ conn.ssh_user }}</td>
              <td data-label="Host" class="font-mono text-sm">{{ conn.ssh_host }}</td>
              <td data-label="Port">{{ conn.ssh_port }}</td>
              <td data-label="Status">
                <span
                  class="text-xs px-2 py-0.5 rounded-full"
                  :class="conn.is_active ? 'bg-emerald-100 text-emerald-700' : 'bg-gray-200 text-gray-600'"
                >
                  {{ conn.is_active ? 'enabled' : 'disabled' }}
                </span>
              </td>
              <td v-if="auth.isSuperAdmin" class="text-right whitespace-nowrap">
                <BaseButtons type="justify-end" no-wrap>
                  <BaseButton :icon="mdiPencil" color="info" small title="Edit" @click="openEdit(conn)" />
                  <BaseButton
                    :icon="conn.is_active ? mdiEyeOff : mdiEye"
                    color="warning"
                    small
                    :title="conn.is_active ? 'Disable' : 'Enable'"
                    @click="toggleActive(conn)"
                  />
                  <BaseButton
                    :icon="mdiDelete"
                    color="danger"
                    small
                    title="Delete"
                    @click="deleteTarget = conn"
                  />
                </BaseButtons>
              </td>
            </tr>
            <tr v-if="!loading && connections.length === 0">
              <td colspan="5" class="text-center text-slate-500 py-6">
                No connections yet. Click “Add Connection” to create one.
              </td>
            </tr>
          </tbody>
        </table>
      </CardBox>
    </SectionMain>

    <CardBoxModal
      v-model="modalOpen"
      :title="editing ? 'Edit Connection' : 'Add Connection'"
      button-label="Save"
      has-cancel
      @confirm="save"
    >
      <FormField label="SSH User">
        <FormControl v-model="form.ssh_user" placeholder="ubuntu" />
      </FormField>
      <FormField label="SSH Host">
        <FormControl v-model="form.ssh_host" placeholder="10.0.0.5 or host.example.com" />
      </FormField>
      <FormField label="SSH Port">
        <FormControl v-model="form.ssh_port" type="number" placeholder="22" />
      </FormField>
      <FormField
        label="SSH Password"
        :help="editing ? 'Leave blank to keep the current password' : 'Stored encrypted'"
      >
        <FormControl v-model="form.ssh_pass" type="password" placeholder="••••••••" />
      </FormField>
    </CardBoxModal>

    <CardBoxModal
      :model-value="!!deleteTarget"
      title="Delete connection"
      button="danger"
      button-label="Delete"
      has-cancel
      @confirm="confirmDelete"
      @cancel="deleteTarget = null"
      @update:model-value="(v) => { if (!v) deleteTarget = null }"
    >
      <p>
        Delete connection <b>{{ deleteTarget?.ssh_user }}@{{ deleteTarget?.ssh_host }}</b>? Sources
        using it will be removed too.
      </p>
    </CardBoxModal>
  </LayoutAuthenticated>
</template>
