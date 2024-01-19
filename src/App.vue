<template>
	<div class="container">
		<div class="d-flex flex-column datable-wrapper">
			<datable
				:endpoint="'http://localhost:1337/projects?fetch-workspace=1'"
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
		[ 'Authorization', 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkIjp7ImlkIjoxLCJ1aWQiOiJjbHIwNnVpZjEwMDAwMTF3bjVub3dqZGlqIiwibG9naW4iOiJhZG1pbkBhZzEudGVjaCIsImVtYWlsIjoiYWRtaW5AYWcxLnRlY2giLCJuaWNlbmFtZSI6IkFkbWluIEFHMSIsInR5cGUiOiJBZG1pbiIsInN0YXR1cyI6IkFjdGl2ZSIsImxhbmd1YWdlIjoiZW4iLCJtZXRhcyI6eyJpZEF2YXRhciI6MSwibGFzdG5hbWUiOiJBRzEiLCJmaXJzdG5hbWUiOiJBZG1pbiJ9LCJjcmVhdGVkIjoiMjAyNC0wMS0wNVQwNToyMToxMC4zMzNaIiwibW9kaWZpZWQiOiIyMDI0LTAxLTA1VDA1OjIxOjEwLjMzM1oiLCJwZXJtaXNzaW9ucyI6eyJhcmVhIjoiIiwicm9sZSI6ImFkbWluIn19LCJpYXQiOjE3MDU2MDY0Nzl9.sn-61GBzno3iiihh5QI7o2ZciNxzMrHERXD6dJ9w3Dw' ],
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
			value: 'name',
			sort: true,
			sortValue: 'name',
			width: 50,
			minWidth: 100,
		},
	];
</script>

<style scoped>

	.datable-wrapper {
		height: 500px;
	}
</style>