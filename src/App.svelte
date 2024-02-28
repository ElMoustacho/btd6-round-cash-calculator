<script>
	// Data from https://bloons.fandom.com/wiki/Rounds_(BTD6)
	// JSON.stringify([...document.querySelectorAll("table.article-table.sortable.jquery-tablesorter tr")].filter((_, index) => index > 1 && index < 102).map(line => +line.querySelector("td:nth-child(5)").innerText.substring(1)))

	import './app.css';
	import roundsData from './lib/rounds-data.json';

	const MIN_ROUND = 1;
	const MAX_ROUND = roundsData.length;
	const MIN_CASH = 0;

	const numberFormat = getNumberFormat();
	let currentRound = MIN_ROUND;
	let targetRound = MAX_ROUND;
	let currentCash = 650;
	$: calculatedCash = numberFormat.format(
		roundsData
			.slice(Math.max(currentRound - 2, currentRound), targetRound)
			.reduce((prevCash, currCash) => prevCash + currCash, currentCash)
	);

	// Values used for array indexing
	$: _currentRound = currentRound - 1;
	$: _targetRound = targetRound - 1;

	function validateInputs() {
		targetRound = minMax(targetRound, currentRound, MAX_ROUND);
		currentRound = minMax(currentRound, MIN_ROUND, targetRound);
		currentCash = Math.max(currentCash, MIN_CASH);
	}

	/**
	 * @param {number} value
	 * @param {number} min
	 * @param {number} max
	 */
	function minMax(value, min, max) {
		return Math.min(Math.max(value, min), max);
	}

	function getLang() {
		if (navigator.languages != undefined) return navigator.languages[0];
		return navigator.language;
	}

	function getNumberFormat() {
		try {
			return Intl.NumberFormat(getLang());
		} catch {
			Intl.NumberFormat('en-en');
		}
	}
</script>

<svelte:head>
	<title>BTD6 End Of Round Cash Calculator</title>
</svelte:head>

<header>
	<h1>BTD6 End Of Round Cash Calculator</h1>
</header>

<main>
	<form on:submit|preventDefault={() => {}}>
		<p>
			<label for="currentRound">About to start round: </label>
			<br />
			<input id="currentRound" type="number" bind:value={currentRound} on:change={validateInputs} />
		</p>
		<p>
			<label for="targetRound">Target round: </label>
			<br />
			<input id="targetRound" type="number" bind:value={targetRound} on:change={validateInputs} />
		</p>
		<p>
			<label for="currentCash">Current cash: </label>
			<br />
			<input id="currentCash" type="number" bind:value={currentCash} on:change={validateInputs} />
		</p>
		<p>
			<label for="calculatedCash">
				Cash for round {targetRound}:
			</label>
			<br />
			<input
				id="calculatedCash"
				type="text"
				bind:value={calculatedCash}
				on:change={validateInputs}
				readonly
				disabled />
		</p>
	</form>

	<aside>
		<details>
			<summary>Rounds details</summary>
			{#each roundsData.slice(_currentRound, _targetRound) as cash, index}
				{@const currentRoundIter = index + _currentRound + 1}
				{@const cumulativeCashIter = roundsData
					.slice(_currentRound, currentRoundIter)
					.reduce((prev, curr) => prev + curr, currentCash)}

				<p>
					<span>Round <b>{currentRoundIter}</b>: </span>
					<span style="color: limegreen;">${numberFormat.format(cash)}</span>
					<span>|</span>
					<span>Cumulative: </span>
					<span style="color: limegreen;">
						${numberFormat.format(cumulativeCashIter)}
					</span>
				</p>
			{/each}
		</details>
	</aside>
</main>
