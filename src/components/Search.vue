<template>
	<form @submit.prevent="search">
		<div class="input-group">
			<input
				type="text"
				class="form-control"
				:placeholder="getDictionary('search')"
				v-model="searchQuery"
			>
			<button
				type="reset"
				class="btn btn-clear border-top border-bottom"
				v-if="searchQuery"
				@click="clear"
			>
				<svg
					aria-hidden="true"
					focusable="false"
					role="img"
					xmlns="http://www.w3.org/2000/svg"
					viewBox="0 0 320 512"
				>
					<path
						fill="currentColor"
						d="M207.6 256l107.72-107.72c6.23-6.23 6.23-16.34 0-22.58l-25.03-25.03c-6.23-6.23-16.34-6.23-22.58 0L160 208.4 52.28 100.68c-6.23-6.23-16.34-6.23-22.58 0L4.68 125.7c-6.23 6.23-6.23 16.34 0 22.58L112.4 256 4.68 363.72c-6.23 6.23-6.23 16.34 0 22.58l25.03 25.03c6.23 6.23 16.34 6.23 22.58 0L160 303.6l107.72 107.72c6.23 6.23 16.34 6.23 22.58 0l25.03-25.03c6.23-6.23 6.23-16.34 0-22.58L207.6 256z"
					></path>
				</svg>

			</button>
			<button class="btn btn-primary" type="submit">
				<i class="far fa-fw fa-search" /> {{ getDictionary('searchButton') }}
			</button>
		</div>
	</form>
</template>

<script setup>
	import { computed } from 'vue';

	const props = defineProps({
		modelValue: {
			type: String,
			default: '',
		},
		dictionary: {
			type: Object,
			default: () => ({}),
		},
		fetchOptions: {
			type: Object,
			default: () => ({}),
		},
	});

	// computed searchQuery
	const searchQuery = computed({
		get: () => props.modelValue,
		set: (value) => {
			emit('update:modelValue', value);
		},
	});

	const emit = defineEmits([ 'update:modelValue', 'search' ]);

	const baseDictionary = {
		search: 'Search...',
		searchButton: 'Search',
	};

	// function to get the dictionary word from props, if not found, use the baseDictionary
	const getDictionary = (word) => props.dictionary[word] ?? baseDictionary[word];

	const search = () => {
		emit('search', searchQuery.value);
	};

	const clear = () => {
		searchQuery.value = '';
		emit('update:modelValue', '');
	};
</script>

<style lang="sass" scoped>

	.btn-clear
		background: var(--bs-body-bg)

		svg
			width: 1.25em
			display: inline-block
			font-size: inherit
			height: 1em
			overflow: visible
			vertical-align: -0.125em
</style>