<script>
	import PocketBase from 'pocketbase';
	const client = new PocketBase('http://127.0.0.1:8090');
	let bracket;
	let match = {
		winner: null,
		slotA: null,
		slotB: null
	};

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

		let match1 = structuredClone(match);
		match1.slotA = seeds.shift();
		match1.slotB = seeds.shift();
		bracket = generateBracket(match1, [match1], seeds);
	}

	function generateBracket(ogHead, head, seeds) {
		if (seeds.length <= 0) {
			return ogHead;
		}

		let newMatches = [];
		for (let i = 0; i < head.length; i++) {
			let newMatch1 = structuredClone(match);
			let newMatch2 = structuredClone(match);

			newMatch1.slotA = head[i].slotA;
			newMatch2.slotA = head[i].slotB;
			head[i].slotA = newMatch1;
			head[i].slotB = newMatch2;
			newMatches.push(newMatch1);
			newMatches.push(newMatch2);
		}

		newMatches.sort((a, b) => {
			return b.slotA - a.slotA;
		});

		for (let i = 0; i < newMatches.length; i++) {
			if (seeds.length > 0) {
				newMatches[i].slotB = seeds.shift();
			}
		}

		for (let i = 0; i < head.length; i++) {
			if (head[i].slotA.slotB == null) {
				head[i].slotA = head[i].slotA.slotA;
			}

			if (head[i].slotB.slotB == null) {
				head[i].slotB = head[i].slotB.slotA;
			}
		}

		return generateBracket(ogHead, newMatches, seeds);
	}

	getTimes();
</script>

<a href="/">Register</a>
<a href="/timetrial">Time Trials</a>
<a href="/bracket">Bracket</a>

<h1>Bracket</h1>
<h2>Seeds:</h2>
{#each times as time, index}
	<tr>
		<td>#{index + 1}</td>
		<td>{time['@expand'].racer.name}</td>
		<td>{time.time}</td>
	</tr>
{/each}

<br />
<p>{JSON.stringify(bracket)}</p>
