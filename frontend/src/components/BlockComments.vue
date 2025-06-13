<template>
	<div
		ref="elBlock"
		class="absolute not-prose"
		v-if="Boolean(user?.data) && !user.data.is_student"
		:class="unresolvedCount > 0 || showComments || elVisible ? 'block' : 'hidden'"
		:style="{ top: blockTop, right: blockRight }"
	>
		<div
			style="background-image: url(https://static-00.iconduck.com/assets.00/comment-icon-1024x964-julk98bl.png);"
			class="flex items-start justify-center z-10 sticky cursor-pointer rounded bg-surface-white text-sm font-bold text-black hover:text-ink-gray-9 hover:border-ink-gray-9 border-0 p-0 pt-[4px] bg-no-repeat bg-contain w-6 h-7"
			@click="toggleComments"
		>
			{{ unresolvedCount > 0 ? `+` : '' }}
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
const elVisible = ref(false)
const showComments = ref(false)
const unresolvedCount = ref(0)
const elBlock = ref(null)

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

	blockEl.value.addEventListener('mouseover', (e) => {
		elVisible.value = blockEl.value.contains(e.target)
	})
	blockEl.value.addEventListener('mouseout', (e) => {
		if(!elBlock.value.contains(e.target))
			elVisible.value = false
	})
	elBlock.value.addEventListener('mouseover', (e) => {
		elVisible.value = elBlock.value.contains(e.target)
	})
	elBlock.value.addEventListener('mouseout', (e) => {
		if(!blockEl.value.contains(e.target))
			elVisible.value = false
	})

	const anchors = blockEl.value.querySelectorAll('a')
	anchors.forEach(anchor => {
		if (anchor.target !== '_blank') {
			anchor.target = '_blank'
		}
	})

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

