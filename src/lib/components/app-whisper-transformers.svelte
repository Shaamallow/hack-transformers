<script>
	import { Input } from '$lib/components/ui/input/index.js';
	import { Button } from '$lib/components/ui/button/index.js';
	import * as Card from '$lib/components/ui/card/index.js';
	import { Textarea } from '$lib/components/ui/textarea/index.js';
	import { pipeline, env } from '@huggingface/transformers';

	// Create a text generation pipeline
	// Generate a response

	let prompt = $state('');
	let promptAnswer = $state('');
	let status = $state('idle');

	env.backends.onnx.wasm.proxy = true;
	let transcriber = $state();

	let device = $state('wasm');

	let url = 'https://huggingface.co/datasets/Xenova/transformers.js-docs/resolve/main/jfk.wav';

	async function isWebGPUSupported() {
		try {
			if (!navigator.gpu) {
				device = 'wasm';
				throw Error('WebGPU not supported.');
			}

			const adapter = await navigator.gpu.requestAdapter();
			device = 'webgpu';
			if (!adapter) {
				device = 'wasm';
				throw Error("Couldn't request WebGPU adapter.");
			}
			return true;
		} catch (e) {
			console.error(e);
			return false;
		}
	}

	async function loadModel() {
		try {
			status = 'Loading model...';

			await isWebGPUSupported();

			// Create a text generation pipeline
			transcriber = await pipeline('automatic-speech-recognition', 'Xenova/whisper-tiny.en', {
				device: device
			});

			status = 'Model Loaded';
		} catch (error) {
			status = 'Error: ' + error;
			console.log(error);
		}
	}

	loadModel();

	async function submitPrompt(event) {
		event.preventDefault();
		if (status !== 'Model Loaded' && status !== 'Idle') {
			console.log('status is not correct');
			promptAnswer = 'Status is not correct, please wait';
			return;
		}
		try {
			// Define the list of messages

			status = 'Generating...';

			// Generate a response
			const output = await transcriber(url);
			console.log(output);
			promptAnswer = output['text'];

			status = 'Idle';
		} catch (error) {
			console.log(error);
			promptAnswer = 'Error: ' + error;
		}
	}
	function submitForm() {
		// Programmatically submit the form
		const form = document.querySelector('form');
		if (form) {
			form.dispatchEvent(new Event('submit', { bubbles: true, cancelable: true }));
		}
	}
</script>

<Card.Root class="w-[350px]">
	<Card.Header>
		<Card.Title>Call Whisper using Transformers.js</Card.Title>
		<Card.Description>Thank you Huggingface for the lib 🤗</Card.Description>
	</Card.Header>
	<Card.Content>
		<div class="pb-2 text-sm text-muted-foreground">Status: {status}</div>
		<form onsubmit={submitPrompt}>
			<div class="flex flex-col space-y-1.5">
				<Input id="greet-input" placeholder="Here is your name" bind:value={prompt} />
			</div>
		</form>
	</Card.Content>
	<Card.Footer class="flex justify-between">
		<Button variant="outline">Cancel</Button>
		<Button onclick={submitForm}>Submit</Button>
	</Card.Footer>
	<Card.Footer class="flex justify-center">
		<Textarea
			id="result"
			disabled
			class="max-w-xs"
			placeholder="Llama answer"
			bind:value={promptAnswer}
		/>
	</Card.Footer>
</Card.Root>
