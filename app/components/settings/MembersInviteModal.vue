<script setup lang="ts">
import * as z from 'zod'
import type { FormSubmitEvent } from '@nuxt/ui'
import type { Member } from '~/types'

const emit = defineEmits<{
  invite: [member: Member]
}>()

const schema = z.object({
  email: z.email('Invalid email'),
  role: z.enum(['member', 'owner'])
})

const open = ref(false)

type Schema = z.output<typeof schema>

const state = reactive<Partial<Schema>>({
  email: '',
  role: 'member'
})

const toast = useToast()

async function onSubmit(event: FormSubmitEvent<Schema>) {
  const { email, role } = event.data
  const name = email.split('@')[0]!.split(/[._-]+/).filter(Boolean).map(part => part.charAt(0).toUpperCase() + part.slice(1)).join(' ')

  emit('invite', { name, username: email, role, avatar: { alt: name } })
  toast.add({ title: 'Success', description: `${email} invited as ${role}`, color: 'success' })
  open.value = false
  state.email = ''
  state.role = 'member'
}
</script>

<template>
  <UModal v-model:open="open" title="Invite people" description="Invite a new member by email address">
    <UButton
      label="Invite people"
      color="neutral"
      class="w-fit lg:ms-auto"
    />

    <template #body>
      <UForm
        :schema="schema"
        :state="state"
        class="space-y-4"
        @submit="onSubmit"
      >
        <UFormField label="Email" name="email">
          <UInput v-model="state.email" placeholder="jane@example.com" class="w-full" />
        </UFormField>
        <UFormField label="Role" name="role">
          <USelect
            v-model="state.role"
            :items="['member', 'owner']"
            color="neutral"
            class="w-full"
            :ui="{ value: 'capitalize', item: 'capitalize' }"
          />
        </UFormField>
        <div class="flex justify-end gap-2">
          <UButton
            label="Cancel"
            color="neutral"
            variant="subtle"
            @click="open = false"
          />
          <UButton
            label="Invite"
            color="primary"
            variant="solid"
            type="submit"
          />
        </div>
      </UForm>
    </template>
  </UModal>
</template>
