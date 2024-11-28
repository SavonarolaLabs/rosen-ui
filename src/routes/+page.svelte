<script lang="ts">
	import { chains } from '$lib/chains';

	import { onMount, afterUpdate, tick } from 'svelte';

	let fromChain = 'Cardano';
	let toChain = 'Ethereum';
	let token = 'ADA';
	let amount = '';
	let address = '';
	const fee = 2.5;

	let fromElements = new Map<string, HTMLElement>();
	let toElements = new Map<string, HTMLElement>();
	let svg: SVGSVGElement;
	let arrowPath: SVGPathElement;
	async function updateArrow() {
		await tick();
		const fromElement = fromElements.get(fromChain);
		const toElement = toElements.get(toChain);
		if (!fromElement || !toElement || !svg) return;

		const fromRect = fromElement.getBoundingClientRect();
		const toRect = toElement.getBoundingClientRect();
		const svgRect = svg.getBoundingClientRect();

		const fromX = fromRect.left + fromRect.width / 2 - svgRect.left;
		const fromY = fromRect.bottom - svgRect.top;
		const toX = toRect.left + toRect.width / 2 - svgRect.left;
		const toY = toRect.top - svgRect.top;

		const threshold = 5;
		const isVerticallyAligned = Math.abs(fromX - toX) <= threshold;

		let path = '';

		if (isVerticallyAligned) {
			path = `
            M ${fromX},${fromY}
            L ${fromX},${toY}
        `;
		} else {
			const horizontalY = (fromY + toY) / 2;
			const cornerRadius = 10;
			const arrowOffset = 10;

			const isLeftToRight = fromX <= toX;
			const isTopToBottom = fromY <= toY;

			const signX = isLeftToRight ? 1 : -1;
			const signY = isTopToBottom ? 1 : -1;

			const firstArcSweep = signX === signY ? 0 : 1;
			const secondArcSweep = signX === signY ? 1 : 0;

			path = `
            M ${fromX},${fromY}
            L ${fromX},${horizontalY - signY * cornerRadius}
            A ${cornerRadius},${cornerRadius} 0 0 ${firstArcSweep} ${fromX + signX * cornerRadius},${horizontalY}
            L ${toX - signX * (cornerRadius + arrowOffset)},${horizontalY}
            A ${cornerRadius},${cornerRadius} 0 0 ${secondArcSweep} ${toX - signX * arrowOffset},${horizontalY + signY * cornerRadius}
            L ${toX - signX * 11},${horizontalY + signY * cornerRadius}
            L ${toX - signX * 11},${toY}
        `;
		}

		arrowPath.setAttribute('d', path);
	}

	function storeFromElement(node: HTMLElement, chainName: string) {
		fromElements.set(chainName, node);
		return {
			destroy() {
				fromElements.delete(chainName);
			}
		};
	}

	function storeToElement(node: HTMLElement, chainName: string) {
		toElements.set(chainName, node);
		return {
			destroy() {
				toElements.delete(chainName);
			}
		};
	}

	onMount(() => {
		window.addEventListener('resize', updateArrow);
	});

	afterUpdate(() => {
		updateArrow();
	});

	let selectedToken = null;

	let tokens = [
		{ symbol: 'ERG', name: 'Ergo', selected: false },
		{ symbol: 'rsVYFI', name: 'VyFi', selected: false },
		{ symbol: 'rsSUNDAE', name: 'SundaeSwap', selected: false },
		{ symbol: 'rsLQ', name: 'Liqwid', selected: false },
		{ symbol: 'rsBTN', name: 'Button', selected: false },
		{ symbol: 'rsSNEK', name: 'Snek', selected: false },
		{ symbol: 'RSN', name: 'RSN Token', selected: false },
		{ symbol: 'rsMIN', name: 'MinSwap', selected: false }
	];

	function selectToken(token) {
		tokens.forEach((t) => (t.selected = false)); // Reset all tokens
		token.selected = true; // Mark the clicked token as selected
		selectedToken = token;
		tokens = tokens;
	}
	function focusInput(token) {
		selectToken(token);
		// Use `tick` to ensure DOM is updated before focusing
		tick().then(() => {
			const input = document.querySelector(`#amount-${token.symbol}`);
			if (input) input.focus();
		});
	}
	function targetChainName(name) {
		if (name.startsWith('rs')) {
			return name.split('rs')[0];
		} else {
			return 'rs' + name;
		}
	}
</script>

<div class="relative flex flex-col justify-center" style="height:100vh;">
	<svg
		bind:this={svg}
		class="pointer-events-none absolute left-0 top-0 h-full w-full"
		style="z-index: 10;"
	>
		<defs>
			<marker
				id="arrowhead"
				markerWidth="10"
				markerHeight="7"
				refX="9"
				refY="3.5"
				orient="auto"
				fill="#999"
			>
				<polygon points="0 0, 10 3.5, 0 7" />
			</marker>
		</defs>
		<path
			id="connector-path"
			bind:this={arrowPath}
			fill="none"
			stroke="#999"
			stroke-width="2"
			marker-end="url(#arrowhead)"
		/>
	</svg>

	<div class="vertical-text ml-10 text-7xl text-white" style="opacity:0.75;">ROSEN</div>
	<div class="secured">Secured by Ergo.</div>

	<div class="widget self-end" style="height:fit-content">
		<div class="chain-selector">
			<div class="chain-list from" style="margin-top:0;">
				<div class="chain-options">
					{#each Object.entries(chains) as [chainName, chain]}
						<div
							class="chain-option"
							class:selected={fromChain === chainName}
							on:click={() => (fromChain = chainName)}
							use:storeFromElement={chainName}
						>
							{@html chain.logo}
							<span>{chainName}</span>
						</div>
					{/each}
				</div>
			</div>

			<div class="chain-list to">
				<div class="chain-options">
					{#each Object.entries(chains) as [chainName, chain]}
						<div
							class="chain-option"
							class:selected={toChain === chainName}
							class:selected-tab={toChain === chainName}
							on:click={() => (toChain = chainName)}
							use:storeToElement={chainName}
						>
							{@html chain.logo}
							<span>{chainName}</span>
						</div>
					{/each}
				</div>
				<div class="tab-content" style="margin-top:-2px; z-index:2; position:relative;">
					<input type="text" id="address" bind:value={address} placeholder="{toChain} Address" />
				</div>
			</div>
		</div>

		<div class="token-selector">
			<div class="token-list">
				{#each tokens as token}
					<div
						class="token-item"
						class:selected={token.selected}
						on:click={() => focusInput(token)}
					>
						<div class="flex gap-4" style="width:240px;">
							<div class="token-info">
								<span class="token-symbol">{token.symbol}</span>
								<span class="token-name">{fromChain}</span>
							</div>
						</div>
						<input
							id="amount-{token.symbol}"
							bind:value={amount}
							placeholder="Amount"
							min="0"
							style="visibility: {token.selected ? 'visible' : 'hidden'};"
						/>
						<span style="visibility: {token.selected ? 'visible' : 'hidden'};">→</span>
						<div
							class="w-full text-end"
							style="visibility: {token.selected ? 'visible' : 'hidden'};"
						>
							<div class="token-info">
								<span>{'489.08 '}{targetChainName(token.symbol)}</span>
								<span class="token-name">{toChain}</span>
							</div>
						</div>
					</div>
				{/each}
			</div>
		</div>

		<button class="transfer-button" on:click={() => alert('Transfer initiated!')}>
			Transfer {tokens.find((t) => t.selected)?.symbol}
		</button>
	</div>
</div>

<style>
	.secured {
		color: rgba(255, 255, 255, 0.724);
		font-family: monospace, monospace;
		position: absolute;
		bottom: 0;
		right: 0;
		margin-bottom: 20px;
		margin-right: 20px;
	}
	.vertical-text {
		position: absolute;
		font-size: 160px;
		writing-mode: vertical-rl; /* Makes the text vertical */
		text-orientation: upright; /* Ensures letters are upright */
	}
	.invert {
		filter: invert(1) !important;
	}

	.widget {
		display: flex;
		flex-direction: column;
		margin-right: 6em;
		gap: 2rem;
		max-width: 610px;
		padding: 2rem;
		background: #ffffff;
		border-radius: 16px;
		box-shadow: 0 4px 24px rgba(0, 0, 0, 0.1);
		position: relative;
		z-index: 2;
	}

	.chain-selector {
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		align-items: flex-start;
		gap: 2rem;
		position: relative;
	}

	.chain-list {
		flex: 1;
		width: 100%;
	}

	.chain-options {
		display: flex;
		flex-direction: row;
		gap: 0.5rem;
		margin-top: 1rem;
	}

	.chain-option {
		display: flex;
		align-items: center;
		gap: 1rem;
		padding: 1rem;
		background: #f8fafc;
		border: 2px solid transparent;
		border-radius: 12px;
		cursor: pointer;
		transition: all 0.2s;
	}

	.chain-option:hover {
		background: #f1f5f9;
	}

	.chain-option.selected {
		background: #e2e8f0;
		border-color: #e2e8f0;
	}

	.selected-tab {
		border-bottom-left-radius: 0;
		border-bottom-right-radius: 0;
		border-bottom-color: #e2e8f0 !important;
		z-index: 3;
	}

	.tab-content {
		margin-top: 1rem;
		padding: 1rem;
		border: 2px solid #e2e8f0;
		border-radius: 12px;
	}

	.tab-content input {
		width: 100%;
		padding: 0.75rem;
		font-size: 1rem;
		border-radius: 8px;
		outline: none;
	}

	.token-selector {
		margin-top: 1rem;
	}

	.token-list {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
		max-height: 200px;
		overflow-y: auto;
		padding: 0.5rem;
		background: #f8fafc;
		border-radius: 12px;
	}

	.token-item {
		display: flex;
		align-items: center;
		gap: 1rem;
		padding: 0.75rem;
		cursor: pointer;
		border-radius: 8px;
		transition: background 0.2s;
	}

	.token-item:hover {
		background: #e2e8f0;
	}

	.token-item.selected {
		background: #e2e8f0;
	}

	.token-info {
		display: flex;
		flex-direction: column;
	}

	.token-symbol {
		font-weight: bold;
	}

	.token-name {
		font-size: 0.875rem;
		color: #64748b;
	}

	.field {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
	}

	label {
		font-size: 0.875rem;
		font-weight: 600;
		color: #475569;
	}

	input {
		padding: 0.75rem;
		font-size: 1rem;
		border-radius: 8px;
		outline: none;
		transition: border-color 0.2s;

		border: none !important;
		outline: none !important;
		box-shadow: none !important;
	}
	input:-webkit-autofill {
		-webkit-box-shadow: 0 0 0 1000px white inset !important;
		box-shadow: 0 0 0 1000px white inset !important;
		-webkit-text-fill-color: inherit !important;
	}

	input:focus {
		border: none !important;
		outline: none !important;
		box-shadow: none !important;
	}

	input[readonly] {
		background: #f8fafc;
		cursor: not-allowed;
	}

	.fee-info {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 1rem;
		margin-top: 1rem;
	}

	.transfer-button {
		padding: 1rem;
		font-size: 1.125rem;
		font-weight: 600;
		color: white;
		background: #3b82f6;
		border: none;
		border-radius: 12px;
		cursor: pointer;
		transition: background 0.2s;
	}

	.transfer-button:hover {
		background: #2563eb;
	}
</style>
