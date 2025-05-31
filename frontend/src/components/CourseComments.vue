<template>
	<div class="border-2 rounded-md min-w-80 mb-4 p-5 sticky">
		Comments
		<textarea 
			v-model="newComment" 
			:disabled="readOnlyMode" 
			placeholder="Write a comment..." 
			class="w-full p-2 mt-2 border rounded-md"
		></textarea>

		<div class="flex justify-end mt-2">
			<button 
				@click="saveComment" 
				:disabled="readOnlyMode || newComment.trim() === ''" 
				class="rounded text-white bg-black p-2"
			>
				Save
			</button>
		</div>
	</div>
</template>

<script setup>
import { ref, inject } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const user = inject('$user')
const readOnlyMode = window.read_only_mode

const props = defineProps({
	course: {
		type: Object,
		default: null,
	},
})

const newComment = ref('')

function saveComment() {
	if (newComment.value.trim()) {
		console.log(newComment.value.trim())
		newComment.value = ''
	}
}
</script>

