<script>
import { onMount, onDestroy } from "svelte";
import { io } from "socket.io-client";
import { Chart } from "chart.js/auto";

let socket;
let token = "cb60b7d2-276b-4040-afec-97f80d09232d";
let loading = false;
let userData = {};
let ctx;
let chartCanvas;
let chart;
let avgPrice = 0;

const labels = (() => {
	const labels = [];
	for (let i = 9050; i < 11051; i += 100) {
		labels.push(i);
	}
	return labels;
})();


const socketInit = () => {
	socket = io("https://54.197.24.74", {
		withCredentials: false,
	});


	socket.on("connect", () => {
		console.log("connected-client");
	});

	socket.on("admin_data", (utils, data) => {
		userData = data;
		avgPrice = utils.avg;
		updateChart(utils);
	});
}

const submitToken = () => {
	loading = true;
	if (!socket) {
		return;
	}

	socket.emit("login", token, (res) => {
		loading = !res.status;
	});
}

const clearData = () => {
	loading = true;
	if (!socket) {
		return;
	}

	socket.emit("clear", (res) => {
		loading = false;
	});
}

const sendResults = () => {
	loading = true;
	if (!socket) {
		return;
	}

	socket.emit("send_results", (res) => {
		loading = !res.status;
	});
}

const updateChart = (utils) => {
	if (!chart) {
		return;
	}
	const losing = utils.bottom_50.reduce((a, v) => {
		const avg_max = Math.ceil(utils.avg / 100) * 100;
		const v_max = Math.ceil(v / 100 + 0.00001) * 100;
		if (v_max > avg_max) {
			return a;
		}

		const v_floor = Math.floor(v / 100 + 0.00001) * 100;
		const v_bin = (v_floor + v_max) / 2;
		a[v_bin] = (a[v_bin] ?? 0) + 1;
		return a;
	}, {});

	const winning = utils.bottom_50.reduce((a, v) => {
		const avg_max = Math.ceil(utils.avg / 100) * 100;
		const v_floor = Math.floor(v / 100 + 0.00001) * 100;
		if (v_floor < avg_max) {
			return a;
		}

		const v_max = Math.ceil(v / 100 + 0.00001) * 100;
		const v_bin = (v_floor + v_max) / 2;
		a[v_bin] = (a[v_bin] ?? 0) + 1;
		return a;
	}, {});

	const no_trade = utils.top_50.reduce((a, v) => {
		const v_max = Math.ceil(v/ 100 + 0.00001) * 100;
		const v_floor = Math.floor(v/ 100 + 0.00001) * 100;
		const v_bin = (v_floor + v_max) / 2;
		a[v_bin] = (a[v_bin] ?? 0) + 1;
		return a;
	}, {});

	const final_losing = labels.reduce((a, v) => {
		a.push((losing[v] ?? 0));
		return a;
	}, []);
	const final_winning = labels.reduce((a, v) => {
		a.push((winning[v] ?? 0));
		return a;
	}, []);

	const final_no_trade = labels.reduce((a, v) => {
		a.push((no_trade[v] ?? 0));
		return a;
	}, []);

	chart.data = {
		labels: labels,
		datasets: [
			{
				label: "Loss",
				data: final_losing,
			},
			{
				label: "Winning",
				data: final_winning,
			},
			{
				label: "No Trade",
				data: final_no_trade,
			}
		],
	};
	chart.update();
}

const renderChart = () => {
	chart = new Chart(ctx, {
		type: 'bar',
		data: {},
		options: {
			plugins: {
				legend: {
					display: false,
				},
			},
			responsive: true,
			scales: {
				y: {
					grid: {
						display: false,
					},
					ticks: {
						maxTicksLimit: 4,
						autoSkip: true,
						color: "#ffffff",
						crossAlign: "center",
						font: {
							size: 14,
							weight: "bold",
						}
					},
				},
				x: {
					grid: {
						display: false,
					},
					ticks: {
						maxRotation: 0,
						autoSkip: true,
						color: "#ffffff",
						crossAlign: "center",
						font: {
							size: 14,
							weight: "bold",
						}
					},
				}
			}
		}
	});
}



onMount(async () => {
	ctx = chartCanvas.getContext('2d');
	renderChart();
	socketInit();
})


onDestroy(async () => {
	if (socket) {
		socket.disconnect();
	}
})

</script>

<div class="w-full min-h-screen flex-col flex items-center bg-slate-900 pt-20">
	<div style="width: 70vw;">
		<canvas bind:this={chartCanvas}></canvas>
	</div>
	<div>
		<input type="text" bind:value={token} class="bg-blue-700 py-2 px-4 text-white rounded-lg mr-4" />
		<button on:click={submitToken} class="bg-blue-500 px-4 py-2 rounded-lg text-white ring-2">login</button>
		<button on:click={clearData} class="bg-blue-500 px-4 py-2 rounded-lg text-white ring-2">Clear Data</button>
		<button on:click={sendResults} class="bg-blue-500 px-4 py-2 rounded-lg text-white ring-2">Send Results</button>
	</div>
	{#if loading}
		<div id="asdf">
			<div id="loading-bar-spinner" class="spinner"><div class="spinner-icon"></div></div>
		</div>
	{/if}
	<div class="my-4 bg-blue-700 px-3 py-2 text-white">{avgPrice}</div>
	<div class="grid grid-cols-6 gap-2 mt-10 max-w-7xl">
		{#each Object.keys(userData) as userKey}
			<div class="bg-purp text-white px-3 py-1 rounded-lg">{userData[userKey].toFixed(2)}</div>
		{/each}
	</div>
</div>

<style>
#chart {
	width: 1000px;
}
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
