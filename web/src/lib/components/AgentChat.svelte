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

	function cleanAgentReply(text: string) {
		return text
			.replace(
				'Gemini is temporarily rate-limited. Wait a moment and try again, or check the Gemini API quota for this key.',
				'AI assistant is temporarily rate-limited. Please wait and try again'
			)
			.replaceAll('*', '');
	}

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
				text: cleanAgentReply(result.status === 'ok' ? result.data.reply : result.message)
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
					<div class:agent-message-row--user={message.role === 'user'} class="agent-message-row">
						{#if message.role === 'agent'}
							<img class="agent-message-avatar" src="/brand/ginseng-avatar-reversed.svg" alt="Ginseng agent" />
						{/if}
						<div class="agent-message-content">
							<span>{message.role === 'user' ? 'You' : 'Agent'}</span>
							<p class="agent-message">{message.text}</p>
						</div>
					</div>
				{/each}
				{#if sending}<p class="agent-thinking" aria-label="Agent is thinking">...</p>{/if}
			</div>
			<form class="agent-form" onsubmit={sendMessage}>
				<label class="sr-only" for="agent-message">Ask Ginseng</label>
				<input id="agent-message" bind:value={draft} placeholder="Asking Ginseng AI Assistant" autocomplete="off" />
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
		min-height: 15 rem;
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
	.agent-toggle img { width: 2.6rem; height: 2.6rem; object-fit: contain; }
	.agent-panel {
		position: fixed;
		z-index: 20;
		bottom: 1rem;
		left: calc(4rem + 0.65rem);
		display: grid;
		grid-template-rows: auto minmax(8rem, 1fr) auto;
		width: min(21rem, calc(100vw - 5.3rem));
		max-width: calc(100vw - 5.3rem);
		height: 25rem;
		overflow: hidden;
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
	.agent-message-row { align-self: flex-start; display: flex; align-items: flex-start; gap: 0.4rem; max-width: 88%; }
	.agent-message-row--user { align-self: flex-end; justify-items: end; }
	.agent-message-content { display: grid; gap: 0.22rem; min-width: 0; }
	.agent-message-row--user .agent-message-content { justify-items: end; }
	.agent-message-content > span { color: var(--ink-muted); font: 700 0.58rem var(--font-mono); letter-spacing: 0.06em; text-transform: uppercase; }
	.agent-message-row--user .agent-message-content > span { color: var(--cobalt); }
	.agent-message-avatar { width: 1.35rem; height: 1.35rem; flex: 0 0 1.35rem; margin-top: 0.95rem; padding: 0.16rem; background: var(--cobalt); border-radius: 50%; object-fit: contain; }
	.agent-message { margin: 0; padding: 0.6rem 0.7rem; background: var(--paper-soft); border-radius: 0.7rem 0.7rem 0.7rem 0.15rem; font-size: 0.82rem; line-height: 1.35; }
	.agent-message-row--user .agent-message { background: var(--cobalt); border-radius: 0.7rem 0.7rem 0.15rem 0.7rem; color: var(--paper); }
	.agent-thinking {
		display: inline-block;
		margin: 0;
		color: var(--ink-muted);
		font: 700 0.8rem var(--font-mono);
		letter-spacing: 0.18em;
		animation: agent-thinking-bounce 900ms ease-in-out infinite;
	}
	@keyframes agent-thinking-bounce {
		0%, 100% { opacity: 0.35; transform: translateY(0); }
		50% { opacity: 1; transform: translateY(-0.15rem); }
	}
	@media (prefers-reduced-motion: reduce) {
		.agent-thinking { animation: none; }
	}
	.agent-form { display: flex; gap: 0.4rem; padding: 0.65rem; border-top: 1px solid var(--rule); }
	.agent-form input { min-width: 0; flex: 1; padding: 0.55rem 0.8rem; background: var(--paper-soft); border: 0; border-radius: 999px; outline: 0; color: var(--ink); font-size: 0.8rem; }
	.agent-form input:focus { outline: 0; box-shadow: none; }
	.agent-form button { display: grid; place-items: center; flex: 0 0 2.15rem; width: 2.15rem; height: 2.15rem; padding: 0; background: var(--cobalt); border: 0; border-radius: 50%; color: var(--paper); font-size: 1.1rem; cursor: pointer; }
	.agent-form button:disabled { opacity: 0.45; cursor: not-allowed; }
	@media (max-width: 48rem) {
		.agent-panel { position: fixed; bottom: 4.25rem; left: 0.65rem; width: min(21rem, calc(100vw - 1.3rem)); }
		.agent-toggle { position: fixed; z-index: 11; bottom: 4.45rem; left: 0.65rem; width: 4.8rem; min-height: 2.8rem; }
		.agent-toggle--open { display: none; }
	}
</style>