<template>
	<div class="container">
		<div class="d-flex flex-column datable-wrapper">
			<datable
				:endpoint="'http://localhost:1337/users'"
				:fields="fields"
				:fetch-options="{ headers: headers }"
			>
				<template #object="{ item }">
					<pre>{{ item }}</pre>
				</template>
			</datable>
		</div>
	</div>
</template>

<script setup>
	import Datable from './components/Datable.vue';
	import 'bootstrap/dist/css/bootstrap.min.css';
	import 'bootstrap';

	const headers = [
		[ 'Id-Workspace', 1 ],
	];

	const fields = [
		{
			text: 'Id',
			value: 'id',
			width: 100,
			minWidth: 50,
			class: 'text-center',
			sort: true,
		},
		{
			text: 'Name',
			value: 'type',
			sort: true,
			sortValue: 'type',
			width: 50,
			minWidth: 100,
		},
		{
			text: 'Object',
			value: 'object',
			minWidth: '100px',
		},
	];
</script>

<style scoped>

	.datable-wrapper {
		height: 500px;
	}
</style>