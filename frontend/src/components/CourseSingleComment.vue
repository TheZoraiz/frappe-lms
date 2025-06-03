<template>
  <div class="w-80">
    <div class="flex items-center gap-2">
      <UserAvatar :user="comment.owner_details" :size="'xl'" />

      <div>
        <span class="text-ink-gray-9 font-bold">{{ comment.owner_details?.full_name ?? 'Anonymous' }}</span>
        <br />
        <span
          v-if="Boolean(comment.creation)"
          class="text-ink-gray-5 text-sm"
        >
          {{ dayjs(comment.creation).format('h:mm A - Do MMM, YYYY') }}
        </span>
      </div>
    </div>
    <p class="text-ink-gray-9 my-2 w-full" v-html="comment.comment_text.replace(/\n/g, '<br>')"></p>
  </div>
</template>

<script setup>
import { ref, inject, onMounted, nextTick } from 'vue'
import { useRouter } from 'vue-router'
import { call, toast } from 'frappe-ui'
import UserAvatar from './UserAvatar.vue'

const router = useRouter()
const user = inject('$user')
const dayjs = inject('$dayjs')
const readOnlyMode = window.read_only_mode

const props = defineProps({
	comment: Object,
})

</script>
