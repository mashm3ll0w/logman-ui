<script setup>
import { reactive, ref } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { mdiAccount, mdiAsterisk } from '@mdi/js'
import SectionFullScreen from '@/components/SectionFullScreen.vue'
import CardBox from '@/components/CardBox.vue'
import FormField from '@/components/FormField.vue'
import FormControl from '@/components/FormControl.vue'
import BaseButton from '@/components/BaseButton.vue'
import BaseButtons from '@/components/BaseButtons.vue'
import LayoutGuest from '@/layouts/LayoutGuest.vue'
import NotificationBar from '@/components/NotificationBar.vue'
import { useAuthStore } from '@/stores/auth'
import { useMainStore } from '@/stores/main'
import { errorMessage } from '@/services/api'

const form = reactive({
  email: '',
  pass: ''
})

const router = useRouter()
const route = useRoute()
const auth = useAuthStore()
const mainStore = useMainStore()

const error = ref('')
const loading = ref(false)

const submit = async () => {
  error.value = ''
  loading.value = true
  try {
    const data = await auth.login(form.email, form.pass)
    if (data.user) mainStore.setUser(data.user)
    const redirect = route.query.redirect || '/sources'
    router.push(redirect)
  } catch (e) {
    error.value = errorMessage(e, 'Invalid email or password')
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <LayoutGuest>
    <SectionFullScreen v-slot="{ cardClass }" bg="purplePink">
      <CardBox :class="cardClass" is-form @submit.prevent="submit">
        <div class="flex flex-col items-center text-center mb-6">
          <img src="/imgs/logman_login.png" alt="LogMan" class="h-14 mb-2 object-contain" />
          <p class="text-gray-500 text-sm">Sign in to stream your logs in real time</p>
        </div>

        <NotificationBar v-if="error" color="danger" class="mb-4">
          {{ error }}
        </NotificationBar>

        <FormField label="Email" help="Please enter your email">
          <FormControl
            v-model="form.email"
            :icon="mdiAccount"
            name="email"
            type="email"
            autocomplete="username"
            required
          />
        </FormField>

        <FormField label="Password" help="Please enter your password">
          <FormControl
            v-model="form.pass"
            :icon="mdiAsterisk"
            type="password"
            name="password"
            autocomplete="current-password"
            required
          />
        </FormField>

        <template #footer>
          <BaseButtons>
            <BaseButton
              type="submit"
              color="info"
              :label="loading ? 'Signing in…' : 'Login'"
              :disabled="loading"
            />
          </BaseButtons>
        </template>
      </CardBox>
    </SectionFullScreen>
  </LayoutGuest>
</template>
