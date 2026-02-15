<script lang="ts">
	import { onMount } from 'svelte';

	const DURATIONS_MS = {
		easy: 20000,
		medium: 15000,
		hard: 5000
	} as const;
	const QUESTIONS = [
		'What are we building, who is it for, and how does it work?',
		'Who are our competitors?',
		'What are we REALLY building?',
		'What has surprised you about user behavior?',
		'What makes new users try us?',
		'How did we meet = why are we working together? Why are we the perfect team to run Bedrock?',
		'Why are we the best team?',
		'What have we done since we applied?',
		'Why is Bedrock the best product?',
		'Why is now the best time to build Bedrock?',
		'How are we going to make money with $50 outbound calls?',
		"Why isn't someone already doing this?",
		'What is your user growth rate?',
		'What is our growth like?',
		'Why did we pick this idea to work on?',
		"What do we know about this space/product others don't know?",
		'What are the key things about our field that outsiders don’t understand?',
		'What problems/hurdles are we anticipating?',
		'How do we know people want this?',
		'What is the next step with the product evolution?',
		"What's new about what we make?",
		"Who needs what we're making?",
		'What resistance will they have to trying our product and how will we overcome it?',
		'What have we learned so far from working on our product?',
		'How much money could we make per year?',
		'What are we going to do next?',
		'Who would we hire or how would we add to your team?',
		'How will we get users?',
		'How will customers and/or users find out about us?',
		"What's the worst thing that has happened?",
		"What's the funniest thing that has happened to us?",
		'How big an opportunity is there?',
		'Who in our team does what?',
		'How long can we go before funding?',
		'How are we understanding customer needs?',
		'How many users do we have?',
		'Why will WE succeed?',
		'Where do new users come from?',
		'How is our product different?',
		'How are we meeting new customers?',
		'What is our distribution strategy?',
		"What's an impressive we've done?",
		"What is something surprising we've done?",
		'Will we stick at this?',
		'How will we make money?',
		'What obstacles will we face and how will we overcome then?',
		"What will we do if YC doesn't fund us?",
		'Are we open to changing our idea?',
		'If our startup succeeds, what additional areas might we be able to expand into?',
		'Who might become competitors?',
		'Who are our competitors and what is wrong with them?',
		'Who would be our next hire?',
		"What's the conversion rate?",
		"What's a tough problem we've solved?",
		"Six months from now, what's going to be our biggest problem?",
		"How do we know customers need what we're making?",
		'How do we know our team will stick together?',
		'What competition do we fear the most?',
		'What systems have you hacked?',
		'What is our burn rate?',
		'What domain expertise do we have?',
		'In what ways are we resourceful?',
		'What are the top things users want?',
		'What do we understand about our users?',
		"What's the biggest mistake we've made?",
		"They don't like our idea. What else do we have?",
		'Why are we the best team?',
		'Where is the rocket science here?',
		'What else have you created together?',
		"What do we understand that others don't?",
		'What part of our project are we going to build first?',
		'Do we have a demo?',
		'Why do the reluctant users hold back?'
	];

	let questions = $state<string[]>([]);
	let currentIndex = $state(0);
	let startedAt = $state(0);
	let elapsed = $state(0);
	let rafId: number | null = null;
	let difficulty = $state<'easy' | 'medium' | 'hard'>('medium');
	let phase = $state<'start' | 'running' | 'summary'>('start');
	let tapCount = $state(0);
	let timeoutCount = $state(0);

	const durationMs = $derived(DURATIONS_MS[difficulty]);
	const progress = $derived(Math.max(0, 1 - elapsed / durationMs));

	function formatQuestion(text: string) {
		return text
			.replace(/\bwe're\b/gi, (match) => (match[0] === 'W' ? "You're" : "you're"))
			.replace(/\bwe've\b/gi, (match) => (match[0] === 'W' ? "You've" : "you've"))
			.replace(/\bwe'll\b/gi, (match) => (match[0] === 'W' ? "You'll" : "you'll"))
			.replace(/\bwe'd\b/gi, (match) => (match[0] === 'W' ? "You'd" : "you'd"))
			.replace(/\bwe\b/gi, (match) => (match[0] === 'W' ? 'You' : 'you'))
			.replace(/\bour\b/gi, (match) => (match[0] === 'O' ? 'Your' : 'your'))
			.replace(/\bus\b/gi, (match) => (match[0] === 'U' ? 'You' : 'you'));
	}

	function shuffle(items: string[]) {
		for (let i = items.length - 1; i > 0; i -= 1) {
			const j = Math.floor(Math.random() * (i + 1));
			[items[i], items[j]] = [items[j], items[i]];
		}
		return items;
	}

	function startTimer() {
		startedAt = performance.now();
		elapsed = 0;
		tick();
	}

	function tick() {
		elapsed = performance.now() - startedAt;
		if (elapsed >= durationMs) {
			advance('timeout');
			return;
		}
		rafId = requestAnimationFrame(tick);
	}

	function advance(reason: 'tap' | 'timeout') {
		if (rafId !== null) {
			cancelAnimationFrame(rafId);
		}
		if (reason === 'tap') {
			tapCount += 1;
		} else {
			timeoutCount += 1;
		}
		const nextIndex = currentIndex + 1;
		if (nextIndex >= questions.length) {
			phase = 'summary';
			return;
		}
		currentIndex = nextIndex;
		startTimer();
	}

	function startSession(next: 'easy' | 'medium' | 'hard') {
		difficulty = next;
		questions = shuffle([...QUESTIONS]);
		currentIndex = 0;
		tapCount = 0;
		timeoutCount = 0;
		phase = 'running';
		startTimer();
	}

	function handleTap() {
		if (phase !== 'running') return;
		advance('tap');
	}

	onMount(() => {
		return () => {
			if (rafId !== null) {
				cancelAnimationFrame(rafId);
			}
		};
	});
</script>

<svelte:head>
	<link
		rel="stylesheet"
		href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;600&display=swap"
	/>
</svelte:head>

<main
	class="relative grid min-h-[100dvh] touch-manipulation place-items-center overflow-hidden bg-white pt-[calc(env(safe-area-inset-top)+28px)] pr-[calc(env(safe-area-inset-right)+22px)] pb-[calc(env(safe-area-inset-bottom)+28px)] pl-[calc(env(safe-area-inset-left)+22px)] font-['Space_Grotesk'] text-neutral-900 [-webkit-tap-highlight-color:transparent]"
	onpointerdown={handleTap}
>
	{#if phase === 'running'}
		<div
			class="pointer-events-none absolute inset-0 origin-bottom bg-neutral-100 [will-change:transform]"
			style={`transform: scaleY(${progress});`}
			aria-hidden="true"
		></div>
	{/if}

	{#if phase === 'start'}
		<div class="relative z-10 grid max-w-[32rem] gap-6 text-center">
			<p
				class="m-0 font-['Garamond'] text-[clamp(24px,6vw,40px)] leading-[1.15] tracking-[-0.01em]"
			>
				YC interview prep
			</p>
			<p class="m-0 text-[14px] leading-[1.6] text-neutral-600">
				Choose a difficulty to begin. You have one pass through all questions.
			</p>
			<div class="flex flex-col items-center gap-3">
				<button
					type="button"
					class="w-52 rounded-full border border-neutral-200 bg-white px-5 py-3 text-[13px] tracking-[0.18em] text-neutral-700 uppercase"
					onclick={() => startSession('easy')}
				>
					Easy · 20s
				</button>
				<button
					type="button"
					class="w-52 rounded-full border border-neutral-900 bg-neutral-900 px-5 py-3 text-[13px] tracking-[0.18em] text-white uppercase"
					onclick={() => startSession('medium')}
				>
					Medium · 15s
				</button>
				<button
					type="button"
					class="w-52 rounded-full border border-neutral-200 bg-white px-5 py-3 text-[13px] tracking-[0.18em] text-neutral-700 uppercase"
					onclick={() => startSession('hard')}
				>
					Hard · 5s
				</button>
			</div>
		</div>
	{:else if phase === 'running'}
		<div class="relative z-10 grid max-w-[36rem] gap-3.5 text-center">
			<p
				class="m-0 font-['Garamond'] text-[clamp(22px,5.4vw,36px)] leading-[1.2] tracking-[-0.01em]"
			>
				{formatQuestion(questions[currentIndex])}
			</p>
		</div>
		<p
			class="absolute bottom-7 left-1/2 m-0 -translate-x-1/2 text-[12px] tracking-[0.18em] text-neutral-500 uppercase"
		>
			Tap to continue
		</p>
	{:else}
		<div class="relative z-10 grid max-w-[28rem] gap-6 text-center">
			<p
				class="m-0 font-['Garamond'] text-[clamp(24px,6vw,40px)] leading-[1.15] tracking-[-0.01em]"
			>
				Session summary
			</p>
			<div class="grid gap-2 text-[14px] text-neutral-600">
				<p class="m-0">Timed out: {timeoutCount}</p>
				<p class="m-0">Tapped to continue: {tapCount}</p>
			</div>
			<button
				type="button"
				class="mx-auto w-52 rounded-full border border-neutral-900 bg-neutral-900 px-5 py-3 text-[13px] tracking-[0.18em] text-white uppercase"
				onclick={() => startSession(difficulty)}
			>
				Run again
			</button>
		</div>
	{/if}
</main>
