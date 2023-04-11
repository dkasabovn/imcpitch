<script>
import { onMount, onDestroy } from "svelte";
import { io } from "socket.io-client";

let socket;
let bet_input = 0;
let bet_size = 0;
let loading = false;
let pnl = null;
let ctx;
let chartCanvas;
let chart;

const socketInit = () => {
	socket = io("https://54.197.24.74", {
		withCredentials: false,
	});

	socket.on("connect", () => {
		console.log("client connected");
	});

	socket.on("results", (res) => {
		pnl = res;	
	});
}

const submitBet = () => {
	loading = true;
	if (!socket) {
		return;
	}

	socket.emit("bet_input", bet_input, (res) => {
		loading = false;
	});
}


onMount(async () => {
	socketInit();
})

onDestroy(() => {
	if (socket) {
		socket.disconnect();
	}
})

</script>

<div class="w-full min-h-screen flex-col flex items-center bg-slate-900 pt-20">
	<div>
		<input type="number" min="9000" max="11000" bind:value={bet_input} class="bg-blue-700 py-2 px-4 text-white rounded-lg mr-4"/>	
		<button on:click={submitBet} class="bg-blue-500 px-4 py-2 rounded-lg text-white ring-2">Submit Bet</button>
	</div>
	{#if loading}
		<div id="asdf">
			<div id="loading-bar-spinner" class="spinner"><div class="spinner-icon"></div></div>
		</div>
	{/if}
	<p class="mt-4 text-white">{ pnl ? "Profit from last trade: " + pnl.toFixed(2) : "No trade"}</p>
</div>

<style>
#loading-bar-spinner.spinner {
    position: absolute;
    animation: loading-bar-spinner 400ms linear infinite;
}

#loading-bar-spinner.spinner .spinner-icon {
    width: 10px;
    height: 10px;
    border:  solid 4px transparent;
    border-top-color:  #00C8B1 !important;
    border-left-color: #00C8B1 !important;
    border-radius: 50%;
}

@keyframes loading-bar-spinner {
  0%   { transform: rotate(0deg);   transform: rotate(0deg); }
  100% { transform: rotate(360deg); transform: rotate(360deg); }
}
</style>
