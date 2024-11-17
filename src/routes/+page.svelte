<script lang="ts">
	import { chains } from './chains';
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

		const horizontalY = (fromY + toY) / 2; // Horizontal center
		const cornerRadius = 10; // Radius for smooth corners
		const arrowOffset = 10; // Horizontal offset for line before the arrow tip
		const path = `
	M ${fromX},${fromY} 
	L ${fromX},${horizontalY - cornerRadius} 
	A ${cornerRadius},${cornerRadius} 0 0 0 ${fromX + cornerRadius},${horizontalY} 
	L ${toX - cornerRadius - arrowOffset},${horizontalY} 
	A ${cornerRadius},${cornerRadius} 0 0 1 ${toX - arrowOffset},${horizontalY + cornerRadius} 
	L ${toX - 11},${horizontalY + cornerRadius} 
	L ${toX - 11},${toY}
`;

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
				fill="#3b82f6"
			>
				<polygon points="0 0, 10 3.5, 0 7" />
			</marker>
		</defs>
		<path
			id="connector-path"
			bind:this={arrowPath}
			fill="none"
			stroke="#3b82f6"
			stroke-width="2"
			marker-end="url(#arrowhead)"
		/>
	</svg>

	<div class="widget">
		<div class="chain-selector">
			<div class="chain-list from">
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
							on:click={() => (toChain = chainName)}
							use:storeToElement={chainName}
						>
							{@html chain.logo}
							<span>{chainName}</span>
						</div>
					{/each}
				</div>
			</div>

			<div class="field w-full">
				<input type="text" id="address" bind:value={address} placeholder="Target Address" />
			</div>
		</div>

		<div class="token-selector">
			<div class="token-list">
				{#each Object.entries(chains) as [_, chain]}
					<div
						class="token-item"
						class:selected={token === chain.symbol}
						on:click={() => (token = chain.symbol)}
					>
						<div class="flex w-20 gap-4">
							{@html chain.logo}
							<div class="token-info">
								<span class="token-symbol">{chain.symbol}</span>
								<span class="token-name">{chain.name}</span>
							</div>
						</div>
						{#if token === chain.symbol}
							<input id="amount" bind:value={amount} placeholder="Enter amount" min="0" />
						{/if}
					</div>
				{/each}
			</div>
		</div>

		<button class="transfer-button" on:click={() => alert('Transfer initiated!')}>
			Transfer {token}
		</button>
	</div>
</div>

<style>
	.widget {
		display: flex;
		flex-direction: column;
		gap: 2rem;
		max-width: 600px;
		margin: 0 auto;
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
		background: #e0f2fe;
		border-color: #3b82f6;
	}

	.chain-logo {
		width: 32px;
		height: 32px;
		border-radius: 50%;
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
		border: 1px solid #e2e8f0;
		border-radius: 8px;
		outline: none;
		transition: border-color 0.2s;
	}

	input:focus {
		border-color: #3b82f6;
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
