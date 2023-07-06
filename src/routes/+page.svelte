<script lang="ts">
	import { useChat } from 'ai/svelte';
	import { load } from 'js-yaml';
	import { Chord } from 'tonal';
	import * as Tone from 'tone';

	import Button from '@specialdoom/proi-ui/Button.svelte';

	import Summary from '../components/Summary.svelte';
	import TextInput from '../components/TextInput.svelte';

	const { input, handleSubmit, messages } = useChat();

	let genre = 'City Pop';
	let mood = '80s Japan';

	let result = `---
genre: City Pop
mood: 80s Japan
chordProgression: 
  - chordSymbol: Dmaj7
    scaleDegree: I
  - chordSymbol: F#m7
    scaleDegree: iii
  - chordSymbol: Bm7
    scaleDegree: vi
  - chordSymbol: Emaj7
    scaleDegree: II
  - chordSymbol: G#m7
    scaleDegree: IV
  - chordSymbol: C#m7
    scaleDegree: vii
  - chordSymbol: F#7
    scaleDegree: iii7
  - chordSymbol: Bmaj7
    scaleDegree: VI
title: Neon Lights
description: A nostalgic and vibrant City Pop song inspired by the 80s Japan music scene. The catchy melodies and energetic rhythms will transport you to the neon-lit streets of Tokyo, evoking a sense of longing and excitement.
analysis: The chord progression starts with a bright and uplifting Dmaj7 (I) chord, setting the nostalgic tone for the song. The progression then moves to F#m7 (iii) and Bm7 (vi), adding a touch of melancholy. The Emaj7 (II) chord brings a sense of anticipation and leads smoothly to G#m7 (IV) and C#m7 (vii), creating tension and depth. The progression resolves with F#7 (iii7) and finally lands on Bmaj7 (VI), providing a satisfying conclusion. Overall, the chord progression captures the essence of City Pop with its blend of major and minor chords, creating a nostalgic yet vibrant atmosphere.
`;

	let parsed = {
		chordProgression: new Array<{ chordSymbol: string; scaleDegree: string }>(),
		title: '',
		description: '',
		analysis: ''
	};

	let chordProgression = new Array<{ chordSymbol: string; scaleDegree: string }>();
	let title = '';
	let description = '';
	let analysis = '';

	$: {
		input.update(
			() => `You are a music composing assistant.
Generate a chord progression in yaml that contains "chordProgression" list.
Each chord has properties for chordSymbol (e.g. A7) and scaleDegree (e.g. iii).
The length of bars should be 8.
Finally, describe the song in "title" and "description".
Analyze the chord progression in "analysis", too.
Here are my requests:
${genre ? 'genre: ' + genre : ''}
${mood ? 'mood: ' + mood : ''}
`
		);
	}

	$: {
		try {
			if ($messages.length % 2 === 0 && $messages.slice(-1)[0]) {
				result = $messages.slice(-1)[0].content;
			}
		} catch (err) {}
	}

	$: {
		try {
			parsed = load(result) as {
				chordProgression: any[];
				title: string;
				description: string;
				analysis: string;
			};
		} catch (err) {}
	}

	$: {
		try {
			if ('chordProgression' in parsed && parsed.chordProgression.length) {
				chordProgression = parsed.chordProgression;
			}
			if ('title' in parsed && parsed.title) {
				title = parsed.title;
			}
			if ('description' in parsed && parsed.description) {
				description = parsed.description;
			}
			if ('analysis' in parsed && parsed.analysis) {
				analysis = parsed.analysis;
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
			const notes = Chord.get(chord.chordSymbol).notes;
			const synth = new Tone.PolySynth(Tone.Synth).toDestination();

			const now = Tone.now();

			notes.forEach((note) => {
				synth.triggerAttackRelease(note + '4', 1.5, now + barIndex * 2);
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
	<Summary {title} {description} {analysis} />
	<div class="chord-group">
		{#each chordProgression as chord}
			<button class="chord-box" on:click={() => chord && playChord(chord.chordSymbol)}>
				<span class="chord-name">{chord && chord.chordSymbol ? chord.chordSymbol : '...'}</span>
			</button>
		{/each}
	</div>
	<div class="degree-group">
		{#each chordProgression as chord}
			<div class="degree-box">
				<span class="degree-name">{chord && chord.scaleDegree ? chord.scaleDegree : '...'}</span>
			</div>
		{/each}
	</div>
	<Button on:click={() => playChordProgression(chordProgression)}>Play Chords</Button>
	<form on:submit={handleSubmit}>
		<div class="input-group">
			<TextInput bind:value={genre} label="Genre" placeholder="City Pop" />
			<TextInput bind:value={mood} label="Mood" placeholder="80s Japan" />
		</div>
		<div class="submit-box">
			<Button>Generate Chords!</Button>
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

	.chord-group,
	.degree-group {
		display: flex;
		width: 100%;
		overflow-x: auto;
	}

	.chord-box {
		flex: 1;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		border: solid 0.5rem gray;
		border-radius: 1rem;
		box-sizing: border-box;
		background-color: white;
	}

	.chord-name {
		display: flex;
		justify-content: center;
		font-size: 2rem;
		width: 100%;
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

	.submit-box {
		display: flex;
		justify-content: center;
	}
</style>
