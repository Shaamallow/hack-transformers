<script>
  import { invoke } from "@tauri-apps/api/core";
  import { Input } from "$lib/components/ui/input/index.js";
  import { Button } from "$lib/components/ui/button/index.js";
  import * as Card from "$lib/components/ui/card/index.js";
  import { Textarea } from "$lib/components/ui/textarea/index.js";

  let name = $state("");
  let greetMsg = $state("");

  async function greet(event) {
    event.preventDefault();
    // Learn more about Tauri commands at https://tauri.app/develop/calling-rust/
    greetMsg = await invoke("greet", { name: name });
  }
  function submitForm() {
    // Programmatically submit the form
    const form = document.querySelector("form");
    if (form) {
      form.dispatchEvent(
        new Event("submit", { bubbles: true, cancelable: true }),
      );
    }
  }
</script>

<Card.Root class="w-[350px]">
  <Card.Header>
    <Card.Title>Call greet function from rust</Card.Title>
    <Card.Description>Run printLn.</Card.Description>
  </Card.Header>
  <Card.Content>
    <form onsubmit={greet}>
      <div class="flex flex-col space-y-1.5">
        <Input
          id="greet-input"
          placeholder="Here is your name"
          bind:value={name}
        />
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
      bind:value={greetMsg}
    />
  </Card.Footer>
</Card.Root>
