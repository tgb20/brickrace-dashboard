<script>
	import { onMount } from 'svelte';
	import PocketBase from 'pocketbase';
	const client = new PocketBase('http://127.0.0.1:8090');

	let times = [];

	async function getTimes() {
		times = await client.records.getFullList('times', 500, {
			sort: '-created',
			expand: 'racer'
		});

		let bestTimes = {};
		for (let i = 0; i < times.length; i++) {
			let time = times[i];
			if (!bestTimes[time.racer]) {
				bestTimes[time.racer] = time;
			} else {
				if (bestTimes[time.racer].time > time.time) {
					bestTimes[time.racer] = time;
				}
			}
		}

		let arrayTimes = Object.values(bestTimes);
		arrayTimes.sort((a, b) => {
			return a.time - b.time;
		});

		times = arrayTimes;
		let seeds = [];
		for (let i = 0; i < times.length; i++) {
			seeds.push(i);
		}
	}

	onMount(async () => {
		client.realtime.subscribe('times', function (e) {
			getTimes();
		});
		getTimes();
	});
</script>

<div class="h-screen flex flex-col items-center">
	{#each times as time, index}
		<div class="bg-slate-200 w-3/5 m-2 rounded-lg p-2 flex">
			<p class="text-4xl text-bold basis-1/3">#{index + 1}</p>
			<p class="text-4xl text-center basis-1/3">{time['@expand'].racer.name}</p>
			<p class="text-4xl basis-1/3 text-right">{time.time}s</p>
		</div>
	{/each}
</div>
