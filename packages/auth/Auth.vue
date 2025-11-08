<template>
  <SocialAuthContainer
    direction="horizontal"
    gap="small"
    v-if="isSignView"
    :theme="props.theme"
    :appearance="appearance"
  >
    <SocialAuth
      v-if="authView === VIEWS.SIGN_IN || authView === VIEWS.SIGN_UP"
      :supabaseClient="supabaseClient"
      :appearance="appearance"
      :providers="providers"
      :providerScopes="providerScopes"
      :queryParams="queryParams"
      :socialLayout="socialLayout"
      :redirectTo="redirectTo"
      :onlyThirdPartyProviders="onlyThirdPartyProviders"
      :i18n="i18n"
    />
    <template v-if="!onlyThirdPartyProviders">
      <EmailAuth
        v-if="authView === VIEWS.SIGN_IN || authView === VIEWS.SIGN_UP"
        :supabaseClient="supabaseClient"
        :appearance="appearance"
        :redirectTo="redirectTo"
        :magicLink="magicLink"
        :showLinks="showLinks"
        :i18n="i18n"
        :additionalData="additionalData"
        :before-submit="props.beforeSubmit"
        @on-submit="handleSubmit"
      />
      <MagicLink
        v-if="authView === VIEWS.MAGIC_LINK"
        :supabaseClient="supabaseClient"
        :appearance="appearance"
        :redirectTo="redirectTo"
        :showLinks="showLinks"
        :i18n="i18n"
        :before-submit="props.beforeSubmit"
        @on-submit="handleSubmit"
      />
    </template>
  </SocialAuthContainer>
  <template v-else>
    <ForgottenPassword
      v-if="authView === VIEWS.FORGOTTEN_PASSWORD"
      :supabaseClient="supabaseClient"
      :appearance="appearance"
      :redirectTo="redirectTo"
      :showLinks="showLinks"
      :i18n="i18n"
      :before-submit="props.beforeSubmit"
      @on-submit="handleSubmit"
    />
    <UpdatePassword
      v-if="authView === VIEWS.UPDATE_PASSWORD"
      :supabaseClient="supabaseClient"
      :appearance="appearance"
      :i18n="i18n"
    />
    <VerifyOtp
      v-if="authView === VIEWS.VERIFY_OTP"
      :supabaseClient="supabaseClient"
      :appearance="appearance"
      :otpType="otpType"
      :i18n="i18n"
    />
    <AnonymousAuth
      v-if="authView === 'anonymous_sign_in'"
      :supabaseClient="supabaseClient"
      :appearance="appearance"
      :i18n="i18n"
      :anonymouslyCredentials="anonymouslyCredentials"
    />
  </template>
</template>

<script lang="ts" setup>
import { provide, ref, watch, computed } from 'vue'
import { I18nVariables, en, merge, VIEWS } from '@supabase/auth-ui-shared'
import { createStitches } from '@stitches/core'
import cloneDeep from 'lodash.clonedeep'

import '../../src/global.css'

import { AuthProps, AuthViewKey, AuthViewType } from '../types'
import SocialAuthContainer from './SocialAuthContainer.vue'
import EmailAuth from './EmailAuth.vue'
import SocialAuth from './SocialAuth.vue'
import MagicLink from './MagicLink.vue'
import ForgottenPassword from './ForgottenPassword.vue'
import UpdatePassword from './UpdatePassword.vue'
import VerifyOtp from './VerifyOtp.vue'
import AnonymousAuth from './AnonymousAuth.vue'

const props = withDefaults(defineProps<AuthProps>(), {
  view: VIEWS.SIGN_IN,
  socialLayout: 'vertical',
  onlyThirdPartyProviders: false,
  magicLink: false,
  showLinks: true,
  dark: false,
  theme: 'default',
  otpType: 'email',
  anonymouslyCredentials: undefined
})

const emit = defineEmits(['update:view', 'on-submit'])

const authView = ref<AuthViewType>(props.view)
const setAuthView = (newView: AuthViewType) => {
  emit('update:view', newView)
  authView.value = newView
}

const handleSubmit = (e: Event) => {
  emit('on-submit', e)
}

provide(AuthViewKey, {
  authView,
  setAuthView
})
/**
 * Localization support
 */
const i18n = computed<I18nVariables>(() => {
  const defaultLanguage = cloneDeep(en)
  const newlanguage = cloneDeep(props?.localization?.variables)
  return merge(defaultLanguage, newlanguage ?? {})
})

/**
 * Simple boolean to detect if authView 'sign_in' or 'sign_up' or 'magic_link' is used
 *
 * @returns boolean
 */
const isSignView = computed(() => {
  return (
    authView.value === VIEWS.SIGN_IN ||
    authView.value === VIEWS.SIGN_UP ||
    authView.value === VIEWS.MAGIC_LINK
  )
})

const theme = computed(() => {
  const appearanceTheme = cloneDeep(props.appearance?.theme)
  const appearanceVariables = cloneDeep(props.appearance?.variables)
  return merge(
    appearanceTheme?.default ?? {},
    appearanceTheme?.[props.theme] ?? {},
    appearanceVariables?.default ?? {},
    appearanceVariables?.[props?.theme] ?? {}
  )
})
watch(
  () => [props.appearance, props.theme],
  () => {
    createStitches({ theme: theme.value })
  },
  { deep: true, immediate: true }
)

watch(
  () => props.view,
  (newView) => {
    /**
     * Overrides the authview if it is changed externally
     */
    const { data: authListener } = props.supabaseClient.auth.onAuthStateChange(
      (event) => {
        if (event === 'PASSWORD_RECOVERY') {
          setAuthView(VIEWS.UPDATE_PASSWORD)
        } else if (event === 'USER_UPDATED') {
          setAuthView(VIEWS.SIGN_IN)
        }
      }
    )
    setAuthView(newView)

    return () => authListener.subscription.unsubscribe()
  },
  { deep: true }
)
</script>
