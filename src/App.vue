<template>
	<div class="d-flex flex-column" style="width: 1000px; height: 500px;">
		<datable
			:endpoint="'http://localhost:1337/users'"
			:fields="fields"
			:fetch-options="{ headers: headers }"
		/>
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
			width: '100px',
			minWidth: 50,
			class: 'text-center',
			sort: true,
		},
		{
			text: 'Name',
			value: 'type',
			sort: true,
			sortValue: 'type',
			minWidth: '100px',
		},
	];
</script>

<style scoped>
</style>