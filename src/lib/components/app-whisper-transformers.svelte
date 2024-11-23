<script>
	import { onMount } from 'svelte';
	import * as Card from '$lib/components/ui/card/index.js';
	import { Textarea } from '$lib/components/ui/textarea/index.js';
	import { Button } from '$lib/components/ui/button/index.js';
	import { Input } from '$lib/components/ui/input/index.js';
	import { pipeline, env } from '@huggingface/transformers';

	let media = [];
	let mediaRecorder = null;
	let status = $state('idle');
	let transcriber = null;
	let transcription = $state('');
	let isRecorderLoaded = $state(false);
	let device = $state('wasm');

	let offloadIp = $state('');

	env.backends.onnx.wasm.proxy = true;
	env.localModelPath = '/';
	env.allowLocalModels = true;
	env.allowRemoteModels = false;

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

	// Load the Whisper model
	async function loadModel() {
		try {
			status = 'Loading model...';
			await isWebGPUSupported();

			transcriber = await pipeline('automatic-speech-recognition', 'whisper', {
				device: device
			});
			status = 'Model Loaded';

			// Call loadRecorder once the model is loaded
			await loadRecorder();
		} catch (error) {
			console.error('Error loading model:', error);
			status = 'Error loading model';
		}
	}

	// Load the recorder setup
	async function loadRecorder() {
		try {
			const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
			mediaRecorder = new MediaRecorder(stream);

			mediaRecorder.ondataavailable = (e) => media.push(e.data);

			mediaRecorder.onstop = async function () {
				const blob = new Blob(media, { type: 'audio/ogg; codecs=opus' });
				media = [];
				const audio = document.querySelector('audio');
				audio.src = window.URL.createObjectURL(blob);

				// Transcription
				try {
					status = 'Transcribing...';
					const output = await transcriber(audio.src);
					transcription = output['text'];
					status = 'Idle';
				} catch (error) {
					console.error('Error during transcription:', error);
					transcription = 'Error transcribing audio.';
					status = 'Idle';
				}
			};

			isRecorderLoaded = true;
		} catch (error) {
			console.error('Error loading recorder:', error);
			status = 'Error loading recorder';
		}
	}

	// Start recording
	function startRecording() {
		if (mediaRecorder) {
			mediaRecorder.start();
			status = 'Recording...';
		}
	}

	// Stop recording
	function stopRecording() {
		if (mediaRecorder) {
			mediaRecorder.stop();
			status = 'Processing...';
		}
	}

	// Load the model on mount
	onMount(loadModel);

	async function sendOffload() {
		// generate a post request to specific ip address:
		console.log('Sending offload request to', offloadIp);

		try {
			const response = await fetch(offloadIp, {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({
					prompt: transcription
				})
			});

			if (!response.ok) {
				throw new Error('Failed to fetch');
			}
		} catch (error) {
			console.error('Error sending offload request:', error);
		}
	}
</script>

<Card.Root class="w-[350px]">
	<Card.Header>
		<Card.Title>Call Whisper using Transformers.js</Card.Title>
		<Card.Description>Record and transcribe audio with Whisper 🤗</Card.Description>
	</Card.Header>
	<Card.Content>
		<div class="pb-2 text-sm text-muted-foreground">Status: {status}</div>
	</Card.Content>
	<Card.Footer class="flex justify-center">
		<Textarea
			id="result"
			disabled
			class="max-w-xs"
			placeholder="Transcription will appear here"
			bind:value={transcription}
		/>
	</Card.Footer>
</Card.Root>

<section class="mt-8 flex flex-col items-center space-y-4">
	<audio controls class="w-full max-w-xs" />
	<div class="flex space-x-4">
		<button
			onclick={startRecording}
			class="rounded-lg bg-green-500 px-4 py-2 text-white hover:bg-green-600"
			disabled={!isRecorderLoaded}
		>
			Start Recording
		</button>
		<button
			onclick={stopRecording}
			class="rounded-lg bg-red-500 px-4 py-2 text-white hover:bg-red-600"
			disabled={!isRecorderLoaded}
		>
			Stop Recording
		</button>
	</div>
	<p class="text-sm text-gray-500">
		{!isRecorderLoaded ? 'Loading recorder setup...' : 'Recorder ready. Click to record audio.'}
	</p>

	<Button onclick={sendOffload}>Offload to dedicated device</Button>
	<Input bind:value={offloadIp} placeholder="Enter IP address to offload to" />
</section>
