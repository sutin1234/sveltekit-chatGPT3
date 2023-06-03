<script lang="ts">
	import type { CreateCompletionResponse } from 'openai';
	import { SSE } from 'sse.js';

	let context = '';
	let loading = false;
	let error = false;
	let answer = '';

	const handleSubmit = async () => {
		loading = true;
		error = false;
		answer = '';

		const eventSource = new SSE('/api/explain', {
			headers: {
				'Content-Type': 'application/json'
			},
			payload: JSON.stringify({ context })
		});

		context = '';

		eventSource.addEventListener('error', (e: Record<string, any>) => {
			error = true;
			loading = false;
			console.log('Something went wrong!', e.data);
		});

		eventSource.addEventListener('message', (e) => {
			try {
				loading = false;

				if (e.data === '[DONE]') {
					return;
				}

				const completionResponse: CreateCompletionResponse = JSON.parse(e.data);

				const [{ text }] = completionResponse.choices;

				answer = (answer ?? '') + text;
			} catch (err) {
				error = true;
				loading = false;
				console.error(err);
				alert('Something went wrong!');
			}
		});

		eventSource.stream();
	};
</script>

<h1 class="text-sxl font-semibold">Explain It Like I'm Five</h1>
<form on:submit|preventDefault={() => handleSubmit()} class="flex flex-col pt-8 max-w-lg w-full">
	<label for="context" class="pb-2 text-lg">Enter the text you want summarized/explained</label>
	<textarea name="context" rows="5" bind:value={context} class="rounded-md p-3 text-zinc-900" />
	<button
		class="px-2.5 py-3 font-medium text-lg bg-emerald-400 text-zinc-900 rounded-md mt-4 hover:bg-emerald-500 transition-all duration-200 ease-in-out"
	>
		Explain it
	</button>
	<div class="pt-4">
		<h2>Explanation:</h2>
		{#if answer}
			<p>{answer}</p>
		{/if}
	</div>
</form>
