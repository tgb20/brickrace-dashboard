<script>
	import PocketBase from 'pocketbase';
	const client = new PocketBase('http://127.0.0.1:8090');

	let name, builder;
	let racers = [];

	async function createRacer() {
		const data = {
			name: name,
			builder: builder
		};
		const record = await client.records.create('racers', data);

		if (record) {
			alert('Added racer!');
		}

		getRacers();
	}

	async function getRacers() {
		racers = await client.records.getFullList('racers', 100, {
			sort: '-created'
		});
	}

	getRacers();
</script>

<a href="/">Register</a>
<a href="/timetrial">Time Trials</a>
<a href="/bracket">Bracket</a>

<h1>Registration</h1>
<label for="name">Brick Name:</label>
<input id="name" bind:value={name} />

<label for="builder">Builder Name:</label>
<input id="builder" bind:value={builder} />

<button on:click={createRacer}>Add Racer</button>

<table>
	<tr>
		<th>Brick Name</th>
		<th>Builder Name</th>
		<th>Photo</th>
	</tr>
	{#each racers as racer}
		<tr>
			<td>{racer.name}</td>
			<td>{racer.builder}</td>
			<td>null</td>
		</tr>
	{/each}
</table>
