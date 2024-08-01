<script setup lang="ts">
import type { Address } from '~/types'

const isOpen = ref(false)
const isLoading = ref(false)
const hasFailed = ref(false)

const form = reactive({
  type: 'home',
  zipCode: '',
  street: '',
  number: '',
  neighborhood: '',
  city: '',
  state: '',
})

const onChangeZipCode = useDebounceFn(async () => {
  if (form.zipCode.length === 9 && isLoading.value === false) {
    isLoading.value = true

    const address = await $fetch<Address>(
      `https://viacep.com.br/ws/${form.zipCode}/json`
    )

    if ('erro' in address) {
      hasFailed.value = true
      isLoading.value = false
      throw new Error('Could not fetch address via zip code')
    }

    form.street = address.logradouro
    form.neighborhood = address.bairro
    form.city = address.localidade
    form.state = address.uf
    isLoading.value = false
  }

  if (hasFailed.value) {
    hasFailed.value = false
  }
}, 1000)

watch(form, onChangeZipCode, { deep: true })
</script>

<template>
  <div>
    <UButton
      label="Cadastrar Endereço"
      icon="i-heroicons-plus"
      size="lg"
      @click="isOpen = true"
    />

    <UModal v-model="isOpen">
      <UCard
        :ui="{
          ring: '',
          divide: 'divide-y divide-gray-100 dark:divide-gray-800',
        }"
      >
        <template #header>
          <h2 class="text-2xl">
            <UIcon name="i-ph-address-book" class="w-5 h-5 mr-1" />
            Cadastrar Endereço
          </h2>
        </template>

        <form @submit.prevent class="space-y-3">
          <div class="flex gap-2">
            <UFormGroup
              label="Tipo de Endereço"
              class="flex-1"
              size="lg"
              required
            >
              <USelect
                v-model="form.type"
                :options="[
                  {
                    name: 'Residencial',
                    value: 'home',
                  },
                  {
                    name: 'Comercial',
                    value: 'business',
                  },
                ]"
                option-attribute="name"
              />
            </UFormGroup>

            <UFormGroup
              label="CEP"
              class="flex-1"
              size="lg"
              required
              :error="hasFailed"
            >
              <UInput
                placeholder="CEP"
                v-model="form.zipCode"
                v-mask="['#####-###']"
                :loading="isLoading"
                :disabled="isLoading"
                :trailing-icon="
                  hasFailed
                    ? 'i-heroicons-exclamation-triangle-20-solid'
                    : undefined
                "
                autofocus
              />
            </UFormGroup>
          </div>

          <div class="flex gap-2">
            <UFormGroup label="Logradouro" size="lg" class="flex-1" required>
              <UInput placeholder="Logradouro" v-model="form.street" disabled />
            </UFormGroup>

            <UFormGroup label="Número" size="lg" class="w-4/12" required>
              <UInput placeholder="Número" v-model="form.number" />
            </UFormGroup>
          </div>

          <UFormGroup label="Bairro" size="lg" required>
            <UInput placeholder="Bairro" v-model="form.neighborhood" disabled />
          </UFormGroup>

          <UFormGroup label="Cidade" size="lg" required>
            <UInput placeholder="Cidade" v-model="form.city" disabled />
          </UFormGroup>

          <UFormGroup label="Estado" size="lg" required>
            <UInput placeholder="Estado" v-model="form.state" disabled />
          </UFormGroup>
        </form>

        <template #footer>
          <UButtonGroup class="flex justify-end" size="lg">
            <UButton
              size="lg"
              color="white"
              label="Fechar"
              @click="isOpen = false"
            />
            <UButton
              size="lg"
              color="primary"
              icon="i-heroicons-plus"
              label="Cadastrar Endereço"
              type="submit"
            />
          </UButtonGroup>
        </template>
      </UCard>
    </UModal>
  </div>
</template>
