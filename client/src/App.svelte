<script>
	import { onDestroy, onMount} from 'svelte';
	import { 
		transactions,
		sortTransaction,
		balance,
		income, 
		expense
	} from './stores';

	import Loading from './components/Loading.svelte';
	import Transaction from './components/Transaction.svelte';
	import Summary from './components/Summary.svelte';
	
	let input;
	let typeOfTranasaction = '+';
	let loading = false;
	$: disable = !input;

	async function passQuery(query, variables) {
		try {
			const response = await fetch(`/api/transaction`, {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json',
					'Accept': 'application/json',
				},
				body: JSON.stringify({
					query,
					variables: variables || 0,
				})
			});
			const json = await response.json();
			const { data } = await json;
			return data;
		}catch(err) {
			throw Error('404 Bad request');
		};
	};

	async function addTransaction() {
		const value = typeOfTranasaction === '+' ? input : input * -1;
		const query = `mutation CreateTransaction($input: sendTransaction!){
			createTransaction(input: $input) {
				_id
				date
				value
			}
		}`;
		const variables = {
			input: {
				value,
				date: Date.now(),
			}
		}
		const data = await passQuery(query, variables);
		$transactions = [data.createTransaction, ... $transactions];
		input = null;
	};

	async function deleteTransaction(id) {
		const query = `mutation DeleteTransaction($_id: ID!){
			deleteTransaction(_id: $_id) {
				_id
				value
				date
			}
		}`;
		const variables = {
			_id: id,
		}
		const data = await passQuery(query, variables);
		const { _id } = data.deleteTransaction;
		if (_id === id) {
			$transactions = $transactions.filter(t => t._id != id);
		};
	};

	onMount( async () => {
		loading = true;
		const query = `query {
			transactions {
				_id
				value
				date
			}
		}`;
		const data = await passQuery(query);
		$transactions = data.transactions;
		loading = false;
	});
	
</script>
<div calss="main">
	<div class="app container">
		<p>
			<span>
				<select bind:value={typeOfTranasaction}>
					<option value="+">+</option>
					<option value="-">-</option>
				</select>
			</span>
			<input type="number" placeholder="Amount" bind:value={input}>
			<button disabled={disable} on:click={addTransaction}>Post</button>	
		</p>
	</div>

	{#if loading}
		<Loading />
	{/if}

	{#if $transactions.length > 0}
		<div class="blance">
			<Summary value={$balance} />
		</div>
		<div calsl="income">
			<Summary mode="income" value={$income} />
		</div>
		<div calsl="income">
			<Summary mode="expense" value={$expense} /> 
		</div>
		{:else if !loading}
		<div class="">Add your first transaction</div>
	{/if}

	<div class="app">
		{#each $sortTransaction as transaction (transaction._id)}
			<Transaction {transaction} {deleteTransaction} />
		{/each}
	</div>
</div>
<style>
	
</style>
