<script lang="ts">
	import { useChat } from 'ai/svelte';
	import { load } from 'js-yaml';

	const { input, handleSubmit, messages } = useChat();

	let genre = 'City Pop';
	let mood = '80s';
	let key = '';

	const generateThemeString = (genre?: string, mood?: string, key?: string) => {
		let titleList = new Array<string>();

		if (mood) {
			titleList = [...titleList, mood];
		}

		if (genre) {
			titleList = [...titleList, genre];
		}

		if (key) {
			titleList = [...titleList, `in ${key}`];
		}

		return titleList.join(' ') ?? 'Free Style';
	};

	$: theme = generateThemeString(genre, mood, key);

	// let chordProgression: any[] = [];
	let chordProgression = [
		{
			chord: 'G',
			tonality: 'major',
			chordType: 'triad',
			scaleDegree: 'I'
		},
		{
			chord: 'Em7',
			tonality: 'minor',
			chordType: 'seventh',
			scaleDegree: 'iii'
		},
		{
			chord: 'Am7',
			tonality: 'minor',
			chordType: 'seventh',
			scaleDegree: 'vi'
		},
		{
			chord: 'D7',
			tonality: 'major',
			chordType: 'seventh',
			scaleDegree: 'IV'
		}
	];

	$: {
		input.update(
			() => `You are a music composing assistant.
Generate a chord progression in yaml that starts with "chordProgression:".
Each chord has properties for chord, tonality, chordType, and scaleDegree.
Here are my requests.
${genre ? 'genre: ' + genre : ''}
${mood ? 'mood: ' + mood : ''}
${key ? 'key: ' + key : ''}`
		);
	}

	$: {
		try {
			if ($messages.slice(-1)[0]) {
				const res = load($messages.slice(-1)[0].content);
				console.log(res);
				if ('chordProgression' in res && res.chordProgression.length) {
					chordProgression = res.chordProgression;
				}
			}
		} catch (err) {}
	}
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Svelte demo app" />
	<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Tangerine" />
</svelte:head>

<section>
	<h1>ChordPrompter</h1>
	<!-- <ul>
		{#each $messages as message}
			<li>{message.role}: {message.content}</li>
		{/each}
	</ul> -->
	<div style="padding: 12px;">
		<h2>{theme}</h2>
	</div>
	<div class="chord-group">
		{#each chordProgression as chord}
			<div class="chord-box">
				<div class="chord-name">{chord && chord.chord ? chord.chord : '...'}</div>
			</div>
		{/each}
	</div>
	<div class="degree-group">
		{#each chordProgression as chord}
			<div class="degree-box">
				<!-- <span>{chord && chord.tonality}</span>
        <span>{chord && chord.chordType}</span> -->
				<span class="degree-name">{chord && chord.scaleDegree ? chord.scaleDegree : '...'}</span>
			</div>
		{/each}
	</div>
	<form on:submit={handleSubmit}>
		<div class="input-group">
			<div class="input-box">
				<label for="genre">Genre</label>
				<input bind:value={genre} name="genre" placeholder="City Pop" />
			</div>

			<div class="input-box">
				<label for="mood">Mood</label>
				<input bind:value={mood} name="mood" placeholder="Urban" />
			</div>

			<div class="input-box">
				<label for="key">Key</label>
				<input bind:value={key} placeholder="G" />
			</div>
		</div>

		<div class="submit-box">
			<button type="submit">Generate Chords!</button>
		</div>
	</form>
</section>

<style>
	section {
		display: flex;
		flex-direction: column;
		align-items: center;
		flex: 0.6;
	}

	h1 {
		font-family: 'Tangerine', serif;
		text-shadow: 4px 4px 4px #aaa;
		font-size: 5rem;
		width: 100%;
	}

	h2 {
		margin-top: 0;
		height: 2rem;
		font-size: 2rem;
		text-decoration: underline;
	}

	.chord-group,
	.degree-group {
		display: flex;
		width: 100%;
	}

	.chord-box {
		flex: 1;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	.chord-name {
		display: flex;
		justify-content: center;
		font-size: 2.5rem;
		border: solid 0.5rem gray;
		border-radius: 1rem;
		box-sizing: border-box;
		width: 100%;
		background-color: white;
	}

	.degree-box {
		flex: 1;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	.degree-name {
		font-size: 1.5rem;
	}

	.input-group {
		width: 300px;
		padding: 24px;
	}

	.input-box {
		display: flex;
		justify-content: space-between;
	}

	.submit-box {
		display: flex;
		justify-content: center;
	}
</style>
