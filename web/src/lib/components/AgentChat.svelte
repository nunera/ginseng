<script lang="ts">
	import { postChat, type ChatContext } from '$lib/api';

	interface Message {
		role: 'user' | 'agent';
		text: string;
	}

	interface Props {
		context: ChatContext;
	}

	let { context }: Props = $props();
	let open = $state(false);
	let draft = $state('');
	let sending = $state(false);
	let messages = $state<Message[]>([
		{ role: 'agent', text: 'Ask me about this liquidity scenario.' }
	]);

	async function sendMessage(event: SubmitEvent) {
		event.preventDefault();
		const text = draft.trim();
		if (!text || sending) return;

		messages = [...messages, { role: 'user', text }];
		draft = '';
		sending = true;
		const result = await postChat(text, context);
		messages = [
			...messages,
			{
				role: 'agent',
				text: result.status === 'ok' ? result.data.reply : result.message
			}
		];
		sending = false;
	}
</script>

<div class="agent-anchor">
	{#if open}
		<section class="agent-panel" aria-label="Ginseng AI agent">
			<header class="agent-panel-header">
				<div>
					<p class="agent-kicker">Ginseng agent</p>
					<h2>Ask the model</h2>
				</div>
				<button class="agent-close" type="button" aria-label="Close agent" onclick={() => (open = false)}>×</button>
			</header>
			<div class="agent-messages" aria-live="polite">
				{#each messages as message}
					<p class:agent-message--user={message.role === 'user'} class="agent-message">
						<span>{message.role === 'user' ? 'You' : 'Agent'}</span>{message.text}
					</p>
				{/each}
				{#if sending}<p class="agent-thinking">Thinking…</p>{/if}
			</div>
			<form class="agent-form" onsubmit={sendMessage}>
				<label class="sr-only" for="agent-message">Ask Ginseng</label>
				<input id="agent-message" bind:value={draft} placeholder="Ask about the forecast" autocomplete="off" />
				<button type="submit" aria-label="Send message" disabled={!draft.trim() || sending}>↑</button>
			</form>
		</section>
	{/if}
	<button class:agent-toggle--open={open} class="agent-toggle" type="button" aria-label="Open Ginseng AI agent" aria-expanded={open} onclick={() => (open = !open)}>
		<img src="/brand/ginseng-avatar-reversed.svg" alt="" />
	</button>
</div>

<style>
	.agent-anchor { position: relative; }
	.agent-toggle {
		display: grid;
		place-items: center;
		gap: 0.15rem;
		width: 100%;
		min-height: 3.2rem;
		padding: 0.35rem;
		background: var(--cobalt);
		border: 0;
		color: rgb(255 255 255 / 82%);
		font-family: var(--font-mono);
		font-size: 0.52rem;
		font-weight: 700;
		letter-spacing: 0.04em;
		text-transform: uppercase;
		cursor: pointer;
	}
	.agent-toggle:hover, .agent-toggle--open { background: var(--paper); color: var(--cobalt); }
	.agent-toggle img { width: 2.1rem; height: 2.1rem; object-fit: contain; }
	.agent-panel {
		position: absolute;
		z-index: 20;
		bottom: 0;
		left: calc(100% + 0.65rem);
		display: grid;
		grid-template-rows: auto minmax(8rem, 1fr) auto;
		width: min(21rem, calc(100vw - 5rem));
		height: 25rem;
		background: var(--paper);
		border: 1px solid var(--rule-strong);
		box-shadow: 0.35rem 0.35rem 0 rgb(0 0 79 / 20%);
		color: var(--ink);
	}
	.agent-panel-header { display: flex; align-items: start; justify-content: space-between; padding: 0.9rem; background: var(--cobalt); color: var(--paper); }
	.agent-kicker { margin: 0 0 0.25rem; color: rgb(255 255 255 / 68%); font: 700 0.58rem var(--font-mono); letter-spacing: 0.08em; text-transform: uppercase; }
	.agent-panel h2 { margin: 0; font-size: 1.25rem; letter-spacing: -0.02em; }
	.agent-close { padding: 0; background: none; border: 0; color: var(--paper); font-size: 1.35rem; line-height: 1; cursor: pointer; }
	.agent-messages { display: flex; flex-direction: column; gap: 0.65rem; overflow-y: auto; padding: 0.8rem; }
	.agent-message { margin: 0; padding: 0.6rem; background: var(--paper-soft); font-size: 0.82rem; line-height: 1.35; }
	.agent-message span { display: block; margin-bottom: 0.25rem; color: var(--ink-muted); font: 700 0.58rem var(--font-mono); letter-spacing: 0.06em; text-transform: uppercase; }
	.agent-message--user { background: var(--cobalt); color: var(--paper); }
	.agent-message--user span { color: rgb(255 255 255 / 72%); }
	.agent-thinking { margin: 0; color: var(--ink-muted); font: 0.68rem var(--font-mono); }
	.agent-form { display: flex; gap: 0.4rem; padding: 0.65rem; border-top: 1px solid var(--rule); }
	.agent-form input { min-width: 0; flex: 1; padding: 0.55rem; background: var(--paper); border: 1px solid var(--control-border); color: var(--ink); font-size: 0.8rem; }
	.agent-form button { width: 2.15rem; background: var(--cobalt); border: 0; color: var(--paper); font-size: 1.1rem; cursor: pointer; }
	.agent-form button:disabled { opacity: 0.45; cursor: not-allowed; }
	@media (max-width: 48rem) {
		.agent-panel { position: fixed; bottom: 4.25rem; left: 0.65rem; width: min(21rem, calc(100vw - 1.3rem)); }
		.agent-toggle { position: fixed; z-index: 11; bottom: 4.45rem; left: 0.65rem; width: 4.8rem; min-height: 2.8rem; }
		.agent-toggle--open { display: none; }
	}
</style>