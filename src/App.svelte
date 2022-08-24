<script>
	import { ethers, ContractFactory } from 'ethers'

	let account = null

	let contract = {
		abi: null,
		bytecode: null
	}

	const idToName = (networkId) => {
		if (networkId == 1) return 'Ethereum'
		if (networkId == 42) return 'Kovan'
		if (networkId == 3) return 'Ropsten'
		if (networkId == 4) return 'Rinkeby'
		if (networkId == 5) return 'Goerli'
		return 'Not found'
	}

	let networkName = idToName(window.ethereum.networkVersion)

	const loginWithMetaMask = async (method) => {
		if (!window.ethereum) return alert('You need to install MetaMask!')
		const accounts = await window.ethereum.request({ method: method })
		account = accounts && accounts[0] ? accounts[0] : null
	}

	const parseInput = (e) => {
		// get file
		const file = e.target.files[0]

		// parse file
		const fileReader = new FileReader()
		fileReader.readAsText(file, 'UTF-8')
		fileReader.onload = (e) => {
			let ret = JSON.parse(e.target.result)
			contract.abi = ret.abi
			contract.bytecode = ret.bytecode
		}
	}

	const deploy = async () => {
		if (!account) return alert('You have to be logged in with MetaMask first')
		if (!contract.abi || !contract.bytecode)
			return alert('You need to upload artifact (.json) file before')

		const provider = new ethers.providers.Web3Provider(window.ethereum)
		const signer = provider.getSigner()
		const factory = new ContractFactory(contract.abi, contract.bytecode, signer)
		const result = await factory.deploy()

		console.log(result)
		result.deployTransaction
		await result.deployTransaction.wait()
		alert(`Contract successfully deployed on ${networkName} network!`)
	}

	const onAccountChange = () =>
		window.ethereum.on('accountsChanged', () => window.location.reload())

	const onNetworkChange = () =>
		window.ethereum.on('networkChanged', () => window.location.reload())

	if (window.ethereum) {
		loginWithMetaMask('eth_accounts')
		onAccountChange()
		onNetworkChange()
	}
</script>

<main>
	<ol>
		<li>
			{#if account}
				<p>Logged with: {account}</p>
			{:else}
				<button on:click={() => loginWithMetaMask('eth_requestAccounts')}>
					login with MetaMask</button
				>
			{/if}
		</li>
		<br />

		<li>
			<p>Select the right network</p>
			<p>↪ current network: <span>{networkName}</span></p>
		</li>
		<br />

		<li>
			<p>Import yout artifact (.json) file here</p>
			<form action="/action_page.php">
				<input
					type="file"
					accept="application/JSON"
					name="filename"
					on:change={parseInput}
				/>
			</form>
		</li>
		<br />

		<li>
			<button on:click={deploy}>Deploy your smart contract </button>
		</li>
	</ol>
</main>

<style>
	p span {
		color: #00b224;
	}
</style>
