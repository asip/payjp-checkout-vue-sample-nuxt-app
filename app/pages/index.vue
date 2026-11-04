<script setup lang="ts">
import type { PayjpCheckoutErrorPayload, PayjpCheckoutPayload } from '~/types'

const runtimeConfig = useRuntimeConfig()

const data = {key: runtimeConfig.public.payjpDataKey, partial: 'true' }

const onCreated = (payload: PayjpCheckoutPayload) => {
  console.log(payload.token)
}

const onFailed = (payload: PayjpCheckoutErrorPayload) => {
  console.log(payload.message)
}
</script>

<template>
  <PayjpCheckout
    :data
    :onCreatedHandler="onCreated"
    :onFailedHandler="onFailed" />
</template>
