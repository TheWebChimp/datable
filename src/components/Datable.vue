<template>
	<div class="data-table d-flex flex-column flex-grow-1">
		<div class="data-table-actions mb-3">
			<div class="col-search">
				<datable-search v-model="searchQuery" @search="search" @clear="clearSearch" />
			</div>
			<div class="col-other-actions">
				<slot name="meta-actions" />
			</div>
		</div>

		<div class="data-table-scroll position-relative flex-grow-1 mb-3" ref="dataTableWrapper">
			<loading
				:active="!!isLoading"
				:is-full-page="false"
			/>

			<div
				class="data-table-wrapper pretty-scrolls flex-grow-1"
				:class="{ 'd-flex': !items.length }"
			>
				<table
					class="table mb-0"
					ref="dataTable"
					:class="{ 'd-flex flex-column flex-grow-1': !items.length, 'no-data': !items.length }"
				>
					<thead :class="{ 'd-inline-table': !items.length }">
						<tr>
							<th class="checkbox-header text-center" v-if="props.checkboxes">
								<input
									ref="checkAllCheckbox"
									type="checkbox"
									@input="checkAll"
								/>
							</th>
							<th
								ref="cols"
								v-for="field in props.fields"
								:key="field.value"
								:style="field.width ? { width: field.width } : {}"
							>
								<div
									class="th-wrapper"
									:class="[
										field.sort === false || typeof field.sort === 'undefined' ? 'no-sort' : '',
										field.class === 'text-center' && !field.sort ? 'justify-content-center' : '',
									]"
									@click="sort(field)"
								>
									<span>{{ field.text }}</span>
									<a
										href="#"
										class="sort-icon"
										v-if="!!field.sort"
									>
										<svg
											v-if="!props.sort.field || props.sort.field !== field.value"
											xmlns="http://www.w3.org/2000/svg"
											height="10"
											width="10"
											viewBox="0 0 320 512"
										>
											<path d="M137.4 41.4c12.5-12.5 32.8-12.5 45.3 0l128 128c9.2 9.2 11.9 22.9 6.9 34.9s-16.6 19.8-29.6 19.8H32c-12.9 0-24.6-7.8-29.6-19.8s-2.2-25.7 6.9-34.9l128-128zm0 429.3l-128-128c-9.2-9.2-11.9-22.9-6.9-34.9s16.6-19.8 29.6-19.8H288c12.9 0 24.6 7.8 29.6 19.8s2.2 25.7-6.9 34.9l-128 128c-12.5 12.5-32.8 12.5-45.3 0z" />
										</svg>

										<svg
											v-if="(props.sort.field === field.value || props.sort.field === field.sortValue) && props.sort.order === 'asc'"
											xmlns="http://www.w3.org/2000/svg"
											height="10"
											width="10"
											viewBox="0 0 320 512"
										>
											<path d="M182.6 41.4c-12.5-12.5-32.8-12.5-45.3 0l-128 128c-9.2 9.2-11.9 22.9-6.9 34.9s16.6 19.8 29.6 19.8H288c12.9 0 24.6-7.8 29.6-19.8s2.2-25.7-6.9-34.9l-128-128z" />
										</svg>

										<svg
											v-if="(props.sort.field === field.value || props.sort.field === field.sortValue) && props.sort.order === 'desc'"
											xmlns="http://www.w3.org/2000/svg"
											height="10"
											width="10"
											viewBox="0 0 320 512"
										>
											<path d="M182.6 470.6c-12.5 12.5-32.8 12.5-45.3 0l-128-128c-9.2-9.2-11.9-22.9-6.9-34.9s16.6-19.8 29.6-19.8H288c12.9 0 24.6 7.8 29.6 19.8s2.2 25.7-6.9 34.9l-128 128z" />
										</svg>
									</a>
								</div>
							</th>
						</tr>
					</thead>

					<tbody
						v-if="!items.length"
						class="d-flex flex-grow-1 align-items-center justify-content-center flex-column"
					>
						<tr>
							<td class="border-0">
								<div class="empty-wrapper">
									<img
										src="../assets/tumbleweed-icon.gif"
										:alt="getDictionary('notFound')"
										class="mb-2"
									>
									<p>{{ getDictionary('notFound') }}</p>
								</div>
							</td>
						</tr>
					</tbody>

					<tbody v-else>
						<tr v-for="item in items" :key="item.id">
							<td class="text-center" v-if="props.checkboxes">
								<input
									type="checkbox"
									@input="check(item.id)"
									:checked="checkedRows.includes(item.id)"
								/>
							</td>
							<!-- get the fields from the props -->
							<td
								v-for="field in props.fields"
								:key="field.value"
								:class="field.class ?? ''"
							>
								<!-- dynamic slot -->
								<slot :name="field.value" :item="item">
									{{ resolveProp(item, field.value) }}
								</slot>
							</td>
						</tr>
					</tbody>
				</table>
			</div>
		</div>

		<div class="d-flex align-items-center justify-content-between">
			<!-- showing item from 1 to x of y items -->
			<p class="mb-0" v-if="totalItems">
				{{ getDictionary('showing') }}
				{{ (currentPage - 1) * perPage + 1 }}
				{{ getDictionary('to') }}
				{{ (currentPage - 1) * perPage + items.length }}
				{{ getDictionary('of') }}
				{{ totalItems }}
				{{ getDictionary('items') }}
			</p>

			<div
				class="data-table-controls d-flex align-items-center gap-2 ms-auto"
				:class="{ 'is-disabled': isLoading }"
			>
				<nav class="ms-auto">
					<vue-awesome-paginate
						v-if="totalItems > perPage"
						class="mb-0"
						pagination-container-class="pag pagination"
						paginate-buttons-class="page-link"
						active-page-class="active"
						:total-items="totalItems"
						:items-per-page="parseInt(perPage)"
						:max-pages-shown="5"
						v-model="currentPage"
						@click="changePage"
					>
						<template #prev-button>
							{{ getDictionary('previous') }}
						</template>
						<template #next-button>
							{{ getDictionary('next') }}
						</template>
					</vue-awesome-paginate>
				</nav>

				<!-- select number of elements -->
				<select class="form-select flex-grow-0 w-auto" v-model="perPage" v-if="items.length">
					<option value="5">5</option>
					<option value="10">10</option>
					<option value="25">25</option>
				</select>
			</div>
		</div>
	</div>
</template>

<script setup>
	import DatableSearch from './Search.vue';
	import { VueAwesomePaginate } from 'vue-awesome-paginate';
	import Loading from 'vue3-loading-overlay';
	import 'vue3-loading-overlay/dist/vue3-loading-overlay.css';
	import { nextTick, onMounted, ref, watch } from 'vue';

	const props = defineProps({
		endpoint: {
			type: String,
			required: true,
		},
		fields: {
			type: Array,
			default: () => [],
		},
		page: {
			type: Number,
			default: 1,
		},
		checkboxes: {
			type: Boolean,
			default: true,
		},
		sort: {
			type: Object,
			default: () => ({
				field: 'id',
				order: 'desc',
			}),
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

	// function to resolve the element from item
	const resolveProp = function(item, val) {
		// first, we check if we don't have a key of type "foo.bar.baz"
		if(!val.includes('.')) return item[val];

		// if we have a key of type "foo.bar.baz", we need to resolve it, but we can have multiple levels,
		// so we need to split the string by the dot
		const keys = val.split('.');

		// now we iterate the keys and resolve the value
		let value = item;
		keys.forEach((key) => {
			value = value[key];
		});

		return value;
	};

	let isLoading = ref(false);
	let dataTableWrapper = ref(null);
	let checkAllCheckbox = ref(null);

	// Check if we have perPage in localStorage, if not set it to 10
	let perPage = ref(localStorage.getItem(`data-table-${ props.endpoint }-perPage`) ?? 10);

	const items = ref([]);
	const pages = ref(0);
	const totalItems = ref(0);
	const dataTable = ref(null);
	const cols = ref([]);
	const currentPage = ref(1);
	const checkedRows = ref([]);
	const searchQuery = ref('');

	const baseDictionary = {
		search: 'Search...',
		searchButton: 'Search',
		previous: 'Previous',
		next: 'Next',
		notFound: 'Oopsy Daisy, such empty',
		showing: 'Showing',
		to: 'to',
		of: 'of',
		items: 'items',
	};

	// function to get the dictionary word from props, if not found, use the baseDictionary
	const getDictionary = (word) => props.dictionary[word] ?? baseDictionary[word];

	onMounted(() => {

		addResizers();
		currentPage.value = props.page;

		// check if we have data-table-${ props.endpoint }-perPage in localStorage, if so, set it to perPage
		if(localStorage.getItem(`data-table-${ props.endpoint }-perPage`)) {
			perPage.value = localStorage.getItem(`data-table-${ props.endpoint }-perPage`);
		}

		// Get the data from the endpoint
		fetchPage();

		/*
			Update the width of the fields based on local storage considering it was stored this way:
			localStorage.setItem(`data-table-${ props.endpoint }-${ field.value }-width`, field.width);
		*/

		// Set the width of the fields
		cols.value.forEach((col, index) => {
			const field = props.fields[index];
			const width = localStorage.getItem(`data-table-${ props.endpoint }-${ field.value }-width`);

			if(field.width === 'auto') {
				field.width = 'auto';
			} else if(width) {
				field.width = width;
			}
		});
	});

	const check = function(id) {
		console.log('check', id);
		if(checkedRows.value.includes(id)) {
			checkedRows.value = checkedRows.value.filter((row) => row !== id);
		} else {
			checkedRows.value.push(id);
		}

		// Set indeterminate state of the checkbox
		const checkbox = dataTable.value.querySelector('thead input[type="checkbox"]');
		if(checkedRows.value.length === items.value.length) {
			checkbox.indeterminate = false;
			checkbox.checked = true;
		} else if(checkedRows.value.length > 0) {
			checkbox.indeterminate = true;
			checkbox.checked = false;
		} else {
			checkbox.indeterminate = false;
			checkbox.checked = false;
		}
	};

	const checkAll = function() {
		// If the checkbox is checked, add all the rows to the checkedRows array
		if(checkAllCheckbox.value.checked) {

			// Add all the items to the array, considering that it may have other ids
			checkedRows.value = [
				...checkedRows.value,
				...items.value.map((item) => item.id).filter((id) => !checkedRows.value.includes(id)),
			];

		} else {

			// Remove all the items from the array, considering that it may have other ids
			checkedRows.value = checkedRows.value.filter((row) => !items.value.map((item) => item.id).includes(row));
		}
	};

	const changePage = function(page) {
		console.log('changePage', page);
		currentPage.value = page;
	};

	// Search ---------------------------------------------------------------------------------------------------------

	const search = function() {
		currentPage.value = 1;
		fetchPage();
	};

	const clearSearch = function() {
		searchQuery.value = '';
		search();
	};

	// Fetch ----------------------------------------------------------------------------------------------------------

	const fetchPage = async function() {

		// Show the loader
		isLoading.value = true;

		try {
			const params = {
				page: currentPage.value,
				limit: perPage.value,
				by: props.sort.field,
				order: props.sort.order,
				q: searchQuery.value,
				refresh: new Date().getTime(),
			};

			// prepare the params
			const paramsString = Object.keys(params).map((key) => `${ key }=${ params[key] }`).join('&');

			// check if the endpoint already has a query string, if yes, add the params with &
			// if not, add the params with ?
			const url = props.endpoint.includes('?') ? `${ props.endpoint }&${ paramsString }` : `${ props.endpoint }?${ paramsString }`;

			const res = await fetch(url, {
				method: 'GET',
				...props.fetchOptions,
			});

			if(res.ok) {

				// Hide the loader
				isLoading.value = false;

				const data = await res.json();

				// Check if items are empty
				if(!data.data || !data.data.length) {

					items.value = [];
					pages.value = 0;
					totalItems.value = 0;

					// wait for the next tick to update the resizers height
					await nextTick();

					// Update resizers height
					cols.value.forEach((col) => {
						const resizer = col.querySelector('.resizer');
						resizer.style.height = `${ dataTable.value.offsetHeight }px`;
					});

					return;
				}

				items.value = data.data;
				pages.value = data.pages;
				totalItems.value = data.count;

				// Check if the current page is greater than the total pages, if so, set it to the last page
				if(currentPage.value > data.pages) currentPage.value = data.value.pages;

				// wait for the next tick to update the resizes height
				await nextTick();

				// wait 3 seconds to update the resizers height
				await new Promise((resolve) => setTimeout(resolve, 500));

				// Update resizers height
				cols.value.forEach((col) => {
					const resizer = col.querySelector('.resizer');
					resizer.style.height = `${ dataTable.value.clientHeight }px`;
				});

				console.log('resizer', `${ dataTable.value.clientHeight }px`);

			} else {
				console.log('error', res);
				isLoading.value = false;
			}
		} catch(error) {

			console.log(error);
			isLoading.value = false;
		}
	};

	// Watch ----------------------------------------------------------------------------------------------------------

	watch(perPage, async () => {
		await fetchPage();

		currentPage.value = 1;

		// Save selection in local storage
		localStorage.setItem(`data-table-${ props.endpoint }-perPage`, perPage.value);
	});

	watch(currentPage, async () => {
		await fetchPage();

		// Check if the checkedRows array contains any of the new items
		const newCheckedRows = [];
		items.value.forEach((item) => {
			if(checkedRows.value.includes(item.id)) newCheckedRows.push(item.id);
		});

		// If new checked rows have items, this means that the checkAllCheckbox should be indeterminate
		// If new checked rows are empty, this means that the checkAllCheckbox should be unchecked
		// If new checked rows have the same length as the items, this means that the checkAllCheckbox should be checked
		if(newCheckedRows.length === items.value.length) {
			checkAllCheckbox.value.indeterminate = false;
			checkAllCheckbox.value.checked = true;
		} else if(newCheckedRows.length > 0) {
			checkAllCheckbox.value.indeterminate = true;
			checkAllCheckbox.value.checked = false;
		} else if(newCheckedRows.length === 0) {
			checkAllCheckbox.value.indeterminate = false;
			checkAllCheckbox.value.checked = false;
		}
	});

	// Sort -----------------------------------------------------------------------------------------------------------

	const sort = (column) => {

		// If the column is not sortable, return, this means that sort is false or there is no value
		if(
			column.sort === false ||
			typeof column.sort === 'undefined'
		) return;

		console.log('sort', column);

		props.sort.field = column.sortValue || column.value;

		// If the column is already sorted, reverse the order
		if(props.sort.field === column.value || props.sort.field === column.sortValue) {
			props.sort.order = props.sort.order === 'asc' ? 'desc' : 'asc';
		} else {
			props.sort.order = 'asc';
		}

		// Reset the current page
		currentPage.value = 1;

		// Fetch the new page
		fetchPage();
	};

	// Resize ---------------------------------------------------------------------------------------------------------

	const addResizers = function() {
		// Iterate cols and add resizer
		cols.value.forEach((col, i) => {
			// get field
			const field = props.fields[i];

			const resizer = document.createElement('div');

			resizer.classList.add('resizer');

			// If we have field.resizable set to false, add class to disable resizing
			if(field.resizable === false || field.width === 'auto') {
				resizer.classList.add('disabled');
			}

			resizer.style.height = `${ dataTable.value.offsetHeight }px`;
			//resizer.addEventListener('mousedown', initResize, false);
			col.appendChild(resizer);

			createResizableColumn(col, resizer, i);
		});
	};

	const createResizableColumn = function(col, resizer, index) {
		// Track the current position of mouse
		let x = 0;
		let w = 0;

		const resizeMouseDownHandler = function(e) {

			// Get the current mouse position
			x = e.clientX;

			// Calculate the current width of column
			const styles = window.getComputedStyle(col);
			w = parseInt(styles.width, 10);

			// Attach listeners for document's events
			document.addEventListener('mousemove', resizeMouseMoveHandler);
			document.addEventListener('mouseup', resizeMouseUpHandler);

			resizer.classList.add('resizing');
		};

		const resizeMouseMoveHandler = function(e) {
			// Determine how far the mouse has been moved
			const dx = e.clientX - x;

			// Get the field based on the index
			const field = props.fields[index];

			// if there is no minWidth, default to 50px
			if(!field.minWidth) field.minWidth = 50;

			// If we have a min width or a max width from the field, cap the resize
			if(field.minWidth && field.maxWidth) {
				// Update the width of column, minimum 100px
				col.style.width = `${ Math.min(Math.max(w + dx, field.minWidth), field.maxWidth) }px`;
			} else if(field.maxWidth) {
				// Update the width of column, minimum 100px
				col.style.width = `${ Math.min(w + dx, field.maxWidth) }px`;
			} else if(field.minWidth) {
				// Update the width of column, minimum 100px
				col.style.width = `${ Math.max(w + dx, field.minWidth) }px`;
			} else {
				// Update the width of column, minimum 100px
				col.style.width = `${ w + dx }px`;
			}

			// Update the width in the field
			field.width = col.style.width;

			// Save the width in local storage, so we can retrieve it later
			localStorage.setItem(`data-table-${ props.endpoint }-${ field.value }-width`, field.width);
		};

		// When user releases the mouse, remove the existing event listeners
		const resizeMouseUpHandler = function() {
			document.removeEventListener('mousemove', resizeMouseMoveHandler);
			document.removeEventListener('mouseup', resizeMouseUpHandler);

			resizer.classList.remove('resizing');
		};

		resizer.addEventListener('mousedown', resizeMouseDownHandler);
	};

	// Refresh table --------------------------------------------------------------------------------------------------
	const refreshTable = function() {
		currentPage.value = 1;
		fetchPage();
	};

	const setLoading = function() {
		isLoading.value = true;
	};

	defineExpose({
		refreshTable,
		setLoading,
		currentPage,
		perPage,
		searchQuery,
	});

</script>

<style lang="sass">

	.pagination
		li
			&:first-child
				.page-link
					border-top-left-radius: var(--bs-pagination-border-radius)
					border-bottom-left-radius: var(--bs-pagination-border-radius)

			&:not(:first-child)
				.page-link
					margin-left: -1px

			&:last-child
				.page-link
					border-top-right-radius: var(--bs-pagination-border-radius)
					border-bottom-right-radius: var(--bs-pagination-border-radius)
</style>

<style lang="sass" scoped>

	$md: 576px


	.data-table
		container-type: inline-size

		input[type="checkbox"]
			accent-color: var(--bs-primary)

	.data-table-actions
		display: flex
		flex-direction: column
		gap: 1rem
		margin-bottom: 1rem

		@container (min-width: #{$md})
			flex-direction: row
			justify-content: space-between
			align-items: center

	.col-search
		width: 100%

		@container (min-width: #{$md})
			width: 380px

	.col-other-actions
		flex: 1

	.d-inline-table
		display: inline-table

	.checkbox-header
		width: 50px

	:deep(.resizing)
		background: var(--bs-border-color)

	.data-table-wrapper
		position: absolute
		height: 100%
		width: 100%
		overflow: auto

	.table
		width: 100%
		position: relative
		border-collapse: collapse
		background: var(--bs-body-bg)
		table-layout: fixed

		&.no-data
			:deep(.resizer)
				display: none

		:deep(td)
			padding: 0.5rem
			border-bottom: 1px solid var(--bs-border-color)
			white-space: nowrap
			text-overflow: ellipsis
			overflow: hidden
			vertical-align: middle

		:deep(th)
			position: sticky
			top: 0
			z-index: 1000
			background: var(--bs-table-bg)
			font-size: 0.65rem
			white-space: nowrap
			text-overflow: ellipsis
			text-transform: uppercase
			min-width: 100px
			//Disable text selection
			user-select: none

			.th-wrapper
				position: relative
				cursor: pointer
				inset: 0
				display: flex
				align-items: center
				//width: calc(100% - 10px)
				padding: 5px
				justify-content: space-between

				&.no-sort
					cursor: default

			// Add a after with a border bottom
			&::after
				content: ''
				position: absolute
				bottom: 0
				left: 0
				width: 100%
				height: 1px
				background: var(--bs-border-color)

			span
				text-overflow: ellipsis
				overflow: hidden
				display: block

		:deep(.resizer)
			position: absolute
			top: 0
			right: 0
			width: 5px
			cursor: col-resize
			user-select: none

			&.disabled
				pointer-events: none

			&:hover
				background: var(--bs-border-color)

		// Last column resizer cannot be dragged
		:deep(th:last-child .resizer)
			display: none

		.sort-icon-fa
			&:before
				display: inline-block
				font-style: normal
				font-variant: normal
				text-rendering: auto
				-webkit-font-smoothing: antialiased
				font-family: "Font Awesome 5 Pro", serif
				font-weight: 900
				font-size: 0.75rem

			&.fa-sort
				&:before
					content: "\f0dc"

			&.fa-sort-up
				&:before
					content: "\f0de"

			&.fa-sort-down
				&:before
					content: "\f0dd"

	.data-table-controls
		&.is-disabled
			pointer-events: none
			opacity: 0.5

	.btn-clear
		background: white
		border-top: 1px solid #ced4da
		border-bottom: 1px solid #ced4da

	.empty-wrapper
		pointer-events: none
		display: flex
		flex-direction: column
		width: 200px
		aspect-ratio: 1
		justify-content: center
		align-items: center
		background: #F6FCFF
		border-radius: 50%
		margin-bottom: 1rem

		// disable select
		*
			user-select: none

		& > *
			background: transparent

		p
			width: 80%
			margin: 0 auto
			font-size: 1rem
			line-height: 1.1
			text-align: center
			text-wrap: balance

		img
			width: 50%

	.sort-icon
		svg
			path
				fill: var(--bs-primary)

	:deep(.vld-overlay)
		svg
			stroke: var(--bs-primary)

		.vld-background
			opacity: 0.75
			background: var(--bs-body-bg)
</style>