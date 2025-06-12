<template>
	<div
		v-if="Boolean(user?.data) && !user.data.is_student"
		class="absolute not-prose"
		:style="{ top: blockTop, right: blockRight }"
	>
		<div
			class="z-10 sticky cursor-pointer rounded bg-surface-white text-sm text-ink-gray-7 hover:text-ink-gray-9 hover:border-ink-gray-9 border-2 p-1"
			@click="toggleComments"
		>
			{{ unresolvedCount > 0 ? `Comments (${unresolvedCount})` : 'Comments' }}
		</div>

		<div class="relative bg-surface-gray-1" v-if="showComments">
			<div class="absolute top-0 right-0">
				<CourseComments
					v-if="Boolean(user?.data) && !user.data.is_student"
					:block="block"
					class="w-96 z-20 sticky bg-surface-white"
				/>
			</div>
		</div>
	</div>
</template>

<script setup>
import { ref, inject, onMounted, watch } from 'vue'
import { call } from 'frappe-ui'
import CourseComments from './CourseComments.vue'

const user = inject('$user')

const blockEl = ref(null)
const blockTop = ref('auto')
const blockRight = ref('auto')
const showComments = ref(false)
const unresolvedCount = ref(0)

const props = defineProps({
	block: {
		type: String,
		default: '',
	},
})

const sleep = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

const alignWithBlock = async () => {
	while (!blockEl.value) {
		await sleep(100)
		blockEl.value = document.querySelector(`[data-id="${props.block}"]`)
	}

	while (true) {
		await sleep(30)
		const rect = blockEl.value.getBoundingClientRect()
		const docTop = window.pageYOffset || document.documentElement.scrollTop
		blockTop.value = `${rect.top + docTop}px`
		blockRight.value = `calc(100% - ${rect.right}px)`
	}
}

const fetchUnresolvedCount = async () => {
	const res = await call('lms.lms.api.get_unresolved_block_comments_count', { block: props.block })
	unresolvedCount.value = res || 0
}

const toggleComments = () => {
	showComments.value = !showComments.value
	if (!showComments.value) {
		fetchUnresolvedCount()
	}
}

onMounted(() => {
	alignWithBlock()
	fetchUnresolvedCount()
	window.addEventListener('resize', alignWithBlock)
})

watch(
	() => blockEl.value,
	() => {
		alignWithBlock()
		fetchUnresolvedCount()
	}
)

</script>

