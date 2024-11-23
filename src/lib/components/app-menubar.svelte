<script lang="ts">
	import '../../app.css';
	import * as Menubar from '$lib/components/ui/menubar/index.js';
</script>

<Menubar.Root>
	<Menubar.Menu>
		<a href="/">
			<Menubar.Trigger>LLami</Menubar.Trigger>
		</a>
	</Menubar.Menu>

	<Menubar.Menu>
		<Menubar.Trigger>Features</Menubar.Trigger>
		<Menubar.Content>
			<a href="/llama">
				<Menubar.Item>llama</Menubar.Item>
			</a>
			<a href="/whisper">
				<Menubar.Item>whisper</Menubar.Item>
			</a>
		</Menubar.Content>
	</Menubar.Menu>

	<Menubar.Menu>
		<Menubar.Trigger>The Team</Menubar.Trigger>
		<Menubar.Content>
			<Menubar.Item>Baguette 🥖</Menubar.Item>
			<Menubar.Item>Pizza 🍕</Menubar.Item>
		</Menubar.Content>
	</Menubar.Menu>
</Menubar.Root>
