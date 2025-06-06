<template>
	<div class="border-2 rounded-md mb-4 p-5 sticky">
		<span class="font-medium text-ink-gray-9">Comments</span>

		<div ref="commentsContainer" class="my-4 flex flex-col gap-2 max-h-[300px] overflow-y-auto">
			<div v-if="comments.length == 0" class="text-sm text-ink-gray-5">
				No comments made.
			</div>

			<div v-for="(comment, index) in comments">
				<CourseSingleComment :comment="comment" />

				<div class="flex items-center flex-wrap gap-2 mb-2">
					<button
						v-if="Boolean(user?.data) && !user.data.is_student && !readOnlyMode"
						@click="replyTo = comment.name" 
						:class="{'opacity-50': readOnlyMode}"
						class="rounded text-ink-gray-7 hover:text-ink-gray-9 hover:border-ink-gray-9 border-2 p-1"
					>
						Reply
					</button>
					<button
						v-if="comment.replies && comment.replies.length > 0"
						@click="comment.showReplies = !comment.showReplies"
						class="rounded text-ink-gray-7 hover:text-ink-gray-9 hover:border-ink-gray-9 border-2 p-1"
					>
						{{ comment.showReplies ? __(`Hide replies (${comment.replies.length})`) : __(`Show replies (${comment.replies.length})`) }}
					</button>
				</div>

				<div v-if="replyTo === comment.name && Boolean(user?.data) && !user.data.is_student && !readOnlyMode" class="pl-4">
					<textarea
						v-model="newReply" 
						:disabled="readOnlyMode" 
						placeholder="Write a reply..." 
						class="p-2 mb-2 border rounded-md bg-transparent text-ink-gray-9 w-full"
						@click="resetComment()"
					></textarea>

					<div class="flex justify-end mb-2">
						<button 
							@click="resetCommentReply()"
							:disabled="readOnlyMode" 
							class="rounded text-ink-gray-7 hover:text-ink-gray-9 hover:border-ink-gray-9 border-2 p-2 mr-2"
						>
							Cancel
						</button>
						<button 
							@click="saveComment" 
							:disabled="readOnlyMode || newReply.trim() === ''" 
							:class="{'opacity-50': loading}"
							class="rounded text-ink-white bg-surface-gray-7 hover:bg-surface-gray-6 p-2"
						>
							{{ loading ? 'Saving...' : 'Save' }}
						</button>
					</div>
				</div>

				<div
					v-if="comment.replies && comment.replies.length > 0"
					class="pl-4 mt-2 transition-all duration-300 ease-in-out"
					:class="{'h-0 overflow-hidden': !comment.showReplies, 'h-auto': comment.showReplies}"
				>
					<div v-for="reply in comment.replies" class="mt-2">
						<CourseSingleComment :comment="reply" />
					</div>
				</div>

				<hr v-if="index < (comments.length - 1)" class="border-outline-gray-2" />
			</div>
		</div>

		<div v-if="Boolean(user?.data) && !user.data.is_student && !readOnlyMode">
			<textarea
				v-model="newComment" 
				:disabled="readOnlyMode" 
				placeholder="Write a comment..." 
				class="w-full p-2 mb-2 border rounded-md bg-transparent text-ink-gray-9"
				@click="resetCommentReply()"
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
	</div>
</template>

<script setup>
import { ref, inject, onMounted, nextTick, watch } from 'vue'
import { useRouter } from 'vue-router'
import { call, toast } from 'frappe-ui'
import CourseSingleComment from './CourseSingleComment.vue'

const router = useRouter()
const user = inject('$user')
const dayjs = inject('$dayjs')
const readOnlyMode = window.read_only_mode

const props = defineProps({
	course: {
		type: Object,
		default: null,
	},
	lesson: {
		type: Object,
		default: null,
	},
})

const newComment = ref('')
const newReply = ref('')
const replyTo = ref(null)
const comments = ref([])
const commentsContainer = ref(null)
const loading = ref(false)

const resetComment = () => {
	newComment.value = ''
}

const resetCommentReply = () => {
	newReply.value = ''
	replyTo.value = null
}

const fetchComments = () => {
	let payload = {};

	if(props.lesson) {
		payload = {
			lesson: props.lesson.data.name,
		}
	} else {
		payload = {
			course: props.course.data?.name,
		}
	}
	call('lms.lms.api.get_course_comments', payload).then((r) => {
		comments.value = r.map(comment => {
			if(comment.name !== replyTo.value)
				comment.showReplies = false
			else
				comment.showReplies = true
			return comment
		})

		if(replyTo.value) {
			newReply.value = ''
			replyTo.value = null

		} else {
			scrollToBottom()
		}
	})
	.catch((err) => {
		console.error(err)
		toast.error(err?.data?.message ?? err?.message ?? 'Error getting comments')
	})
}

const expandCommentReplies = (commentName) => {
	comments.value.forEach(comment => {
		if (comment.name === commentName) {
			comment.showReplies = !comment.showReplies
		}
	})
}

onMounted(() => {
	if (props.course || props.lesson) {
		fetchComments();
	}
})

watch(() => [props.course, props.lesson], () => {
	comments.value = []
	fetchComments();
}, { deep: true })

function saveComment() {
	if (newComment.value.trim() || newReply.value.trim()) {
		let payload = {
			comment: newComment.value.trim() || newReply.value.trim(),
			reply_to: replyTo.value,
		}

		if(props.lesson) {
			payload.lesson = props.lesson.data?.name
		} else {
			payload.course = props.course.data?.name
		}

		loading.value = true
		call('lms.lms.api.create_course_comment', payload).then((r) => {
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
