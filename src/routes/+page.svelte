<script lang="ts">
	import { useChat } from 'ai/svelte';
	import { load } from 'js-yaml';
	import { Chord } from 'tonal';
	import * as Tone from 'tone';

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
				const res = load($messages.slice(-1)[0].content) as { chordProgression: any[] };
				console.log(res);
				if ('chordProgression' in res && res.chordProgression.length) {
					chordProgression = res.chordProgression;
				}
			}
		} catch (err) {}
	}

	const playChord = (chordSymbol: string) => {
		const notes = Chord.get(chordSymbol).notes;
		const synth = new Tone.PolySynth(Tone.Synth).toDestination();

		notes.forEach((note) => {
			synth.triggerAttackRelease(note + '4', 1);
		});
	};

	const playChordProgression = (chordProgression: any[]) => {
		chordProgression.forEach((chord, barIndex) => {
			const notes = Chord.get(chord.chord).notes;
			const synth = new Tone.PolySynth(Tone.Synth).toDestination();

			const now = Tone.now();

			notes.forEach((note) => {
				synth.triggerAttackRelease(note + '4', 2, now + barIndex * 2);
			});
		});
	};
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Svelte demo app" />
	<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Tangerine" />
</svelte:head>

<section>
	<h1>ChordPrompter</h1>
	<div style="padding: 12px;">
		<h2>{theme}</h2>
	</div>
	<div class="chord-group">
		{#each chordProgression as chord}
			<button class="chord-box" on:click={() => chord && playChord(chord.chord)}>
				<div class="chord-name">{chord && chord.chord ? chord.chord : '...'}</div>
			</button>
		{/each}
	</div>
	<div class="degree-group">
		{#each chordProgression as chord}
			<div class="degree-box">
				<span>{chord && chord.tonality ? chord.tonality : '...'}</span>
				<span>{chord && chord.chordType ? chord.chordType : '...'}</span>
				<span class="degree-name">{chord && chord.scaleDegree ? chord.scaleDegree : '...'}</span>
			</div>
		{/each}
	</div>
	<button on:click={() => playChordProgression(chordProgression)}>Play Chords</button>
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
		height: 1.5rem;
		font-size: 1.5rem;
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
