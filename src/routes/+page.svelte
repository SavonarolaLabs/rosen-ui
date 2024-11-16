<script lang="ts">
	let fromChain = 'Cardano';
	let toChain = 'Ethereum';
	let token = 'ADA';
	let amount = '';
	let address = '';
	const fee = 2.5; // Example fee
	const calculateReceiveAmount = () => (amount ? Math.max(Number(amount) - fee, 0) : 0);
</script>

<div class="widget">
	<div class="section">
		<div class="field">
			<label for="from-chain">From</label>
			<select id="from-chain" bind:value={fromChain}>
				<option value="Cardano">Cardano</option>
				<option value="Ergo">Ergo</option>
				<option value="Ethereum">Ethereum</option>
				<option value="Bitcoin">Bitcoin</option>
			</select>
		</div>
		<div class="field">
			<label for="token">Token</label>
			<select id="token" bind:value={token}>
				<option value="ADA">ADA</option>
				<option value="ERG">ERG</option>
				<option value="ETH">ETH</option>
				<option value="BTC">BTC</option>
			</select>
		</div>
		<div class="field">
			<label for="amount">Amount</label>
			<input type="number" id="amount" bind:value={amount} placeholder="Enter amount" min="0" />
		</div>
	</div>
	<div class="section">
		<div class="field">
			<label for="to-chain">To</label>
			<select id="to-chain" bind:value={toChain}>
				<option value="Cardano">Cardano</option>
				<option value="Ergo">Ergo</option>
				<option value="Ethereum">Ethereum</option>
				<option value="Bitcoin">Bitcoin</option>
			</select>
		</div>
		<div class="field">
			<label for="address">Address</label>
			<input type="text" id="address" bind:value={address} placeholder="Enter recipient address" />
		</div>
		<div class="field">
			<label for="fee">Fee</label>
			<input type="number" id="fee" value={fee} readonly />
		</div>
		<div class="field">
			<label for="receive">Receive</label>
			<input type="number" id="receive" value={calculateReceiveAmount()} readonly />
		</div>
	</div>
	<button class="button" on:click={() => alert('Transfer initiated!')}> Transfer </button>
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
		border-radius: 12px;
		box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
	}
	.section {
		display: flex;
		flex-direction: column;
		gap: 1rem;
	}
	.field {
		display: flex;
		flex-direction: column;
	}
	.field label {
		font-size: 0.9rem;
		font-weight: bold;
		margin-bottom: 0.5rem;
	}
	.field select,
	.field input {
		padding: 0.75rem;
		font-size: 1rem;
		border: 1px solid #ddd;
		border-radius: 8px;
		outline: none;
	}
	.field input[readonly] {
		background: #f9f9f9;
	}
	.button {
		padding: 1rem;
		font-size: 1.2rem;
		font-weight: bold;
		text-align: center;
		color: white;
		background: #4a7dfb;
		border: none;
		border-radius: 8px;
		cursor: pointer;
		transition: background 0.3s ease;
	}
	.button:hover {
		background: #345ecb;
	}
</style>
