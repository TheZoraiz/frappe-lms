<template>
	<div class="border-2 rounded-md min-w-80 mb-4 p-5 sticky">
		<span class="font-medium text-ink-gray-9">Comments</span>

		<div ref="commentsContainer" class="mb-4 flex flex-col gap-2 mt-4 max-h-[300px] overflow-y-auto">
			<div v-for="(comment, index) in comments">
				<span class="text-ink-gray-9 font-bold">{{ comment.owner_details?.full_name ?? 'Anonymous' }}</span>
				<br />
				<span
					v-if="Boolean(comment.creation)"
					class="text-ink-gray-5 text-sm"
				>
					{{ dayjs(comment.creation).format('h:mm A - Do MMM, YYYY') }}
				</span>
				<p class="text-ink-gray-9 my-2" v-html="comment.comment_text.replace(/\n/g, '<br>')"></p>

				<hr v-if="index < (comments.length - 1)" class="border-outline-gray-2" />
			</div>
		</div>

		<textarea 
			v-model="newComment" 
			:disabled="readOnlyMode" 
			placeholder="Write a comment..." 
			class="w-full p-2 m-2 border rounded-md bg-transparent text-ink-gray-9"
		></textarea>

		<div class="flex justify-end mt-2">
			<button 
				@click="saveComment" 
				:disabled="readOnlyMode || newComment.trim() === ''" 
				:class="{'opacity-50': loading}"
				class="rounded text-ink-white bg-surface-gray-7 hover:bg-surface-gray-6 p-2"
			>
				{{ loading ? 'Saving...' : 'Save' }}
			</button>
		</div>
	</div>
</template>

<script setup>
import { ref, inject, onMounted, nextTick } from 'vue'
import { useRouter } from 'vue-router'
import { call, toast } from 'frappe-ui'

const router = useRouter()
const user = inject('$user')
const dayjs = inject('$dayjs')
const readOnlyMode = window.read_only_mode

const props = defineProps({
	course: {
		type: Object,
		default: null,
	},
})

const newComment = ref('')
const comments = ref([])
const commentsContainer = ref(null)
const loading = ref(false)

const fetchComments = () => {
	call('lms.lms.api.get_course_comments', {
		course: props.course.data?.name,
	}).then((r) => {
		comments.value = r;
		scrollToBottom()
	})
	.catch((err) => {
		console.error(err)
		toast.error(err?.data?.message ?? err?.message ?? 'Error getting comments')
	})
}

onMounted(() => {
	if (props.course) {
		fetchComments();
	}
})

function saveComment() {
	if (newComment.value.trim()) {
		loading.value = true
		call('lms.lms.api.create_course_comment', {
			course: props.course.data?.name,
			comment: newComment.value.trim(),
		}).then((r) => {
			console.log(r)
			toast.success(r.message)
			newComment.value = ''
			fetchComments();
			loading.value = false
		})
		.catch((err) => {
			console.error(err)
			toast.error(err?.data?.message ?? err?.message ?? 'Error saving comment')
			loading.value = false
		})
	}
}

function scrollToBottom() {
	nextTick(() => {
		if (commentsContainer.value) {
			commentsContainer.value.scrollTop = commentsContainer.value.scrollHeight
		}
	})
}
</script>