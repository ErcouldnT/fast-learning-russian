<script lang="ts">
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';
	import words from '$lib/kelimeler.json';
	import { shuffle, sleep } from '$lib/modules';
	import correct_audio from '$lib/audio/correct.wav';
	import wrong_audio from '$lib/audio/wrong.wav';

	interface verb {
		fiil: string;
		anlam: string;
	}

	let count = 0;
	let all_words = words.fiiller;
	let verbs = shuffle(all_words);
	let random_verb: verb;
	let correct_answer: string;
	let wrong_answer: string;
	let choices: string[] = [];
	let first_is_correct = false;
	let second_is_correct = false;

	const createWrongAnswer = () => {
		wrong_answer = all_words[Math.floor(Math.random() * all_words.length)].anlam;

		if (wrong_answer === correct_answer) {
			createWrongAnswer();
		}
	};

	const createQuestion = () => {
		if (verbs.length === 0) {
			return alert('Tebrikler, tüm soruları ' + count + ' adımda cevapladın.');
		}

		count++;
		random_verb = verbs.pop();
		correct_answer = random_verb.anlam;

		createWrongAnswer();
		resetChoices();
	};

	const resetChoices = () => {
		first_is_correct = false;
		second_is_correct = false;

		choices = [];
		choices.push(correct_answer);
		choices.push(wrong_answer);
		shuffle(choices);
	};

	const whichAnswerIsCorrect = (choice: string) => {
		if (choice === choices[0]) {
			first_is_correct = true;
		} else {
			second_is_correct = true;
		}
	};

	const checkAnswer = async (choice: string) => {
		if (choice === correct_answer) {
			whichAnswerIsCorrect(choice);
			const correct_beep = new Audio(correct_audio);
			correct_beep.play();
			await sleep(1000);
			createQuestion();
		} else {
			const wrong_beep = new Audio(wrong_audio);
			wrong_beep.play();
			verbs.push(random_verb);
			shuffle(verbs); // same question maybe next question
			// await sleep(1000);
			createQuestion();
		}
	};

	createQuestion();
</script>

<div class="flex flex-col justify-center items-center p-16 gap-6">
	<p class="code">{count}</p>
	<div class="text-6xl">{random_verb.fiil}</div>
	<div class="flex gap-5">
		<button
			on:click={() => {
				checkAnswer(choices[0]);
			}}
			type="button"
			class="btn uppercase {first_is_correct ? 'variant-filled-success' : 'variant-filled-primary'}"
			>{choices[0]}</button
		>
		<button
			on:click={() => {
				checkAnswer(choices[1]);
			}}
			type="button"
			class="btn uppercase {second_is_correct
				? 'variant-filled-success'
				: 'variant-filled-primary'} ">{choices[1]}</button
		>
	</div>
</div>
