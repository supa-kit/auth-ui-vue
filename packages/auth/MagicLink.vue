<template>
  <form
    :id="`auth-${authView}`"
    @submit.prevent="handleSubmit"
    autoComplete="on"
    :style="{
      width: '100%'
    }"
  >
    <Container direction="vertical" gap="large" :appearance="appearance">
      <div>
        <Label htmlFor="email" :appearance="appearance">
          {{ labels?.email_input_label }}
        </Label>
        <Input
          id="email"
          type="email"
          name="email"
          autoComplete="email"
          autofocus
          :shape="appearance?.shape"
          :placeholder="labels?.email_input_placeholder"
          :appearance="appearance"
          v-model="email"
        />
      </div>

      <BrandButton
        type="submit"
        :loading="isLoading"
        :shape="appearance?.shape"
        :brand="appearance?.brand"
        :appearance="appearance"
      >
        {{ isLoading ? labels?.loading_button_label : labels?.button_label }}
      </BrandButton>

      <template v-if="showLinks">
        <Anchor
          href="#auth-sign-in"
          :appearance="appearance"
          @click.prevent="setAuthView(VIEWS.SIGN_IN)"
        >
          {{ i18n?.sign_in?.link_text }}
        </Anchor>
      </template>
    </Container>

    <Message v-if="message" :appearance="appearance">{{ message }}</Message>

    <Message v-if="error" color="danger" :appearance="appearance">
      {{ error }}
    </Message>
  </form>
</template>

<script lang="ts" setup>
import { ref, computed } from 'vue'
import { SupabaseClient } from '@supabase/supabase-js'
import { RedirectTo, VIEWS } from '@supabase/auth-ui-shared'

import {
  AuthViewKey,
  type Appearance,
  type AuthViewInjection,
  type AuthI18nVariables
} from '../types'
import {
  Anchor,
  Button,
  BrandButton,
  Container,
  Input,
  Label,
  Message
} from '../ui/index'
import { injectStrict } from '../utils'
import { useSupabaseUser } from './UserContextProvider'

export interface MagicLinkProps {
  appearance?: Appearance
  supabaseClient: SupabaseClient
  redirectTo?: RedirectTo
  showLinks?: boolean
  i18n?: AuthI18nVariables
  /**
   * A function that will be called before the magic link is sent.
   * If the function returns `false`, the submission will be aborted.
   */
  beforeSubmit?: (email: string) => Promise<boolean | void> | boolean | void
}

const emit = defineEmits(['on-submit'])

const props = withDefaults(defineProps<MagicLinkProps>(), {})

const { supabaseUser } = useSupabaseUser(props.supabaseClient)

const email = ref('')
const error = ref('')
const message = ref('')
const isLoading = ref(false)

const { authView, setAuthView } = injectStrict<AuthViewInjection>(AuthViewKey)

const labels = computed(
  () => props.i18n?.[authView.value] as AuthI18nVariables['magic_link']
)

const handleSubmit = async (e: Event) => {
  if (props.beforeSubmit) {
    const canSubmit = await props.beforeSubmit(email.value)
    if (canSubmit === false) {
      return
    }
  }

  // console.log(props)
  error.value = ''
  message.value = ''
  isLoading.value = true
  const isAnonymous = supabaseUser.value?.is_anonymous
  let signInError: Error | null = null
  emit('on-submit', email.value)
  if (isAnonymous) {
    const { error: err } = await props.supabaseClient.auth.updateUser(
      {
        email: email.value
      },
      {
        emailRedirectTo: props.redirectTo
      }
    )
    signInError = err
  } else {
    const { error: err } = await props.supabaseClient.auth.signInWithOtp({
      email: email.value,
      options: { emailRedirectTo: props.redirectTo }
    })
    signInError = err
  }

  if (signInError) {
    error.value = signInError.message
  } else {
    message.value = props.i18n?.magic_link?.confirmation_text as string
  }
  isLoading.value = false
}
</script>
