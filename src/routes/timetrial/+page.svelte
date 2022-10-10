<script>
	import PocketBase from 'pocketbase';
	const client = new PocketBase('http://127.0.0.1:8090');
	let racers = [];
	let times = [];
	let submitTime = 0;
	let submitRacer;
	async function getRacers() {
		racers = await client.records.getFullList('racers', 100, {
			sort: '-created'
		});
		submitRacer = racers[0].id;
	}

	async function getTimes() {
		times = await client.records.getFullList('times', 500, {
			sort: '-created',
			expand: 'racer'
		});
	}

	async function sendTimeToServer() {
		const data = {
			racer: submitRacer,
			time: submitTime
		};
		const record = await client.records.create('times', data);

		if (record) {
			alert('Added time!');
		}

		getTimes();
	}

	getRacers();
	getTimes();
</script>

<a href="/">Register</a>
<a href="/timetrial">Time Trials</a>
<a href="/bracket">Bracket</a>

<h1>Time Trials</h1>
<label for="racers">Choose a car:</label>
<select name="racers" id="racers" bind:value={submitRacer}>
	{#each racers as racer}
		<option value={racer.id}>{racer.name} - {racer.builder}</option>
	{/each}
</select>
<label for="name">Time:</label>
<input id="name" bind:value={submitTime} />

<button on:click={sendTimeToServer}>Add time</button>

<table>
	<tr>
		<th>Racer</th>
		<th>Time</th>
	</tr>
	{#each times as time}
		<tr>
			<td>{time['@expand'].racer.name}</td>
			<td>{time.time}</td>
		</tr>
	{/each}
</table>
