<script lang="ts">
  import type { PayjpCheckoutPayload, PayjpCheckoutErrorPayload } from '~/types'

  interface PayjpCheckoutResponse {
    // card: any
    // created: number
    id: string
    // livemode: boolean
    // object: string
    // used: boolean
  }

  interface PayjpCheckoutErrorResponse {
    // code: string
    message: string
    // status: number // http (response) status code
    // type: string
  }

  interface DataProperties {
    key: string,
    partial?: string,
    text?: string,
    submitText?: string,
    tokenName?: string,
    previousToken?: string,
    lang?: string,
    namePlaceholder?: string,
    tenant?: string,
  }

  declare global {
    interface Window {
      payjpCheckoutOnCreated: ((response: PayjpCheckoutResponse) => void) | null
      payjpCheckoutOnFailed: ((statusCode: number, errorResponse: PayjpCheckoutErrorResponse) => void) | null
      PayjpCheckout: unknown | null
    }
  }
</script>

<script setup lang="ts">
  import { onMounted, onBeforeUnmount, useId } from 'vue'

  interface PayjpCheckoutProps {
    data: DataProperties,
    onCreatedHandler: (payload: PayjpCheckoutPayload) => void,
    onFailedHandler: (payload: PayjpCheckoutErrorPayload) => void
  }

  let script: HTMLScriptElement | null | undefined = null
  let element: HTMLElement | null  = null

  const props = defineProps<PayjpCheckoutProps>()

  const payjpCheckoutId = useId()

  const handleCheckoutCreated = (response: PayjpCheckoutResponse) => {
    const payload: PayjpCheckoutPayload = { token: response.id }
    props.onCreatedHandler(payload)
  }

  const handleCheckoutFailed = (statusCode: number, errorResponse: PayjpCheckoutErrorResponse ) => {
    const payload: PayjpCheckoutErrorPayload = { statusCode, message: errorResponse.message }
    props.onFailedHandler(payload)
  }

  onMounted( () => {
    window.payjpCheckoutOnCreated = handleCheckoutCreated
    window.payjpCheckoutOnFailed = handleCheckoutFailed
    /*
    PAY.JP の checkout から呼ばれる window.alert を無効化
    // カード情報が不正のときに window.alert が payjp の checkout から呼ばれるため
    window.alert = () => {}
    */

    script = document.createElement('script')
    script.src = 'https://checkout.pay.jp/'
    script.classList.add('payjp-button')
    script.dataset.onCreated = 'payjpCheckoutOnCreated'
    script.dataset.onFailed = 'payjpCheckoutOnFailed'
    script.dataset.key = props.data.key
    script.dataset.partial = props.data.partial ?? 'false'
    if (props.data.text) script.dataset.text = props.data.text
    if (props.data.submitText) script.dataset.submitText = props.data.submitText
    if (props.data.tokenName) script.dataset.tokenName = props.data.tokenName
    if (props.data.previousToken) script.dataset.previousToken = props.data.previousToken
    if (props.data.lang) script.dataset.lang = props.data.lang
    if (props.data.namePlaceholder) script.dataset.namePlaceholder = props.data.namePlaceholder
    if (props.data.tenant) script.dataset.tenant = props.data.tenant

    element = document.querySelector(`#${payjpCheckoutId}`)
    element?.appendChild(script)
  })

  onBeforeUnmount( () => {
    element?.removeChild(script as Node)
    window.payjpCheckoutOnCreated = null
    window.payjpCheckoutOnFailed = null
    window.PayjpCheckout = null
  })
</script>

<template>
  <div :id="payjpCheckoutId"></div>
</template>
