<script lang="ts">
	import { useChat } from 'ai/svelte';
	import { load } from 'js-yaml';
	import { Chord } from 'tonal';
	import * as Tone from 'tone';

	import Button from '@specialdoom/proi-ui/Button.svelte';

	import Analysis from '../components/Analysis.svelte';
	import Summary from '../components/Summary.svelte';
	import TextInput from '../components/TextInput.svelte';

	const { input, handleSubmit, messages } = useChat();

	let genre = 'City Pop';
	let mood = '80s Japan';

	const generateThemeString = (genre?: string, mood?: string) => {
		let titleList = new Array<string>();

		if (mood) {
			titleList = [...titleList, mood];
		}

		if (genre) {
			titleList = [...titleList, genre];
		}

		return titleList.join(' ') ?? 'Free Style';
	};

	$: theme = generateThemeString(genre, mood);

	// let chordProgression: any[] = [];
	let chordProgression = [
		{
			chordSymbol: 'Cmaj7',
			scaleDegree: 'I'
		},
		{
			chordSymbol: 'Am7',
			scaleDegree: 'vi'
		},
		{
			chordSymbol: 'Fmaj7',
			scaleDegree: 'IV'
		},
		{
			chordSymbol: 'G7',
			scaleDegree: 'V'
		},
		{
			chordSymbol: 'Cmaj7',
			scaleDegree: 'I'
		},
		{
			chordSymbol: 'Am7',
			scaleDegree: 'vi'
		},
		{
			chordSymbol: 'Fmaj7',
			scaleDegree: 'IV'
		},
		{
			chordSymbol: 'G7',
			scaleDegree: 'V'
		}
	];
	let title = 'Neon Lights';
	let description = 'A nostalgic city pop track inspired by the vibrant nightlife of 80s Japan.';
	let analysis =
		'The chord progression for "Neon Lights" follows a classic city pop structure, reminiscent of the music scene in 80s Japan. The song starts with a dreamy Cmaj7 chord as the tonic (I) and progresses to Am7, Fmaj7, and G7, creating a smooth and nostalgic atmosphere. The repetition of the chord progression provides a sense of familiarity, while the G7 chord leading back to Cmaj7 adds a subtle tension and resolution. Overall, "Neon Lights" captures the essence of 80s city pop, drawing listeners into a nostalgic journey through neon-lit streets.';

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
			if ($messages.slice(-1)[0]) {
				const result = load($messages.slice(-1)[0].content) as {
					chordProgression: any[];
					title: string;
					description: string;
					analysis: string;
				};
				console.log(result);
				if ('chordProgression' in result && result.chordProgression.length) {
					chordProgression = result.chordProgression;
				}
				if ('title' in result && result.title) {
					title = result.title;
				}
				if ('description' in result && result.description) {
					description = result.description;
				}
				if ('analysis' in result && result.analysis) {
					analysis = result.analysis;
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
