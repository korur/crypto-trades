<script>
	import { onMount } from "svelte";
	import { scaleBand, scaleTime, scaleSqrt } from "d3-scale";
	import { axisBottom, axisLeft, axisTop, axisRight } from "d3-axis";
	import { interval } from "d3-timer";
	import { select } from "d3-selection";
	import Axis from "./components/Axis.svelte";
	import { tooltip } from "./js/tooltip";

	// Define crypto product Ids
	let product_ids = [
		"BTC-USD",
		"ETH-USD",
		"XRP-USD",
		"XLM-USD",
		"LTC-USD",
		"BCH-USD",
		"ZRX-USD",
		"ALGO-USD",
		"EOS-USD",
	];

	let trades;
	// Init trade data
	$: trades = [];

	// Init time offset
	let offset = 0;

	onMount(() => {
		// CB Pro websocket url
		let url = "wss://ws-feed.pro.coinbase.com";
		let webSocket = new WebSocket(url);

		// Define websocket data to subscribe to
		let subscription = {
			type: "subscribe",
			channels: [
				{
					name: "ticker",
					product_ids: product_ids,
				},
			],
		};

		// Send subscription data
		webSocket.onopen = function (event) {
			webSocket.send(JSON.stringify(subscription));
		};

		// Handle incoming messages

		webSocket.onmessage = function (event) {
			// Parse message JSON
			let data = JSON.parse(event.data);

			if (data.type === "ticker") {
				// Add message data to trades array
				data.dateObj = new Date(data.time);
				// data.jitter = Math.random() * 20;
				trades.push(data);
				// Define offset
				if (offset == 0) {
					let now = new Date();
					offset = data.dateObj.getTime() - now.getTime();
				}
			}
		};
	});
	// Set interval callback
	interval(updateChart, 20);

	// SVG size params
	var margin = { top: 40, right: 1, bottom: 50, left: 65 };
	let width = 750;
	let height = product_ids.length * 40;
	let x;

	// Set scaling functions
	$: x = scaleTime()
		.range([margin.left - 5, width - margin.right])
		.domain(timeExtent);

	$: y = scaleBand()
		.range([margin.top, height - margin.bottom])
		.domain(product_ids)
		.paddingOuter(0.1);

	let r = scaleSqrt().domain([0.0, 1.0]).range([0, height]).exponent(0.4);

	// Return time extent to display
	function getTimeExtent() {
		let now = new Date();
		let nowOffset = new Date(now.getTime() + offset);
		let dateStart = new Date(nowOffset.getTime() - 60 * 1000);
		return [dateStart, nowOffset];
	}

	// Update chart axis and data positions
	$: timeExtent = getTimeExtent;

	function updateChart() {
		// Update x axis with new times
		timeExtent = getTimeExtent();
		x.domain(timeExtent);
		// Filter out trades that are outside timeline
		trades = trades.filter(function (trade) {
			return trade.dateObj > new Date(timeExtent[0].getTime() - 5000);
		});
	}

	// Get TimeString for tooltip
	let ms, timeString;

	function getTimeString(d) {
		ms = d.dateObj.getMilliseconds();
		timeString = d.dateObj
			.toLocaleTimeString()
			.replace(" ", "." + ms + " ");
		return timeString;
	}
</script>

<main id="main">
	<h2>CRYPTO TRACKER!</h2>
	<div class="cont">
		<p>
			Here, I converted the Crypto Trades D3 visualization from <a
				href="https://github.com/rhammell/crypto-trades">Bob Hammell</a
			>
			into Svelte. The
			<a href="https://github.com/korur/crypto-trades"
				>code in Svelte
			</a>is more modular and declarative.
		</p>
		The app uses Coinbase Pro websocket to retrieve the real-time data and the
		Trades for various Crypto/USD pairs are displayed on a scrolling timeline.
		The size of the circles are in relation to the trade sizes. You can mouse
		over for the details.

		<h2 class="subtitle">
			number of trades: <span> {trades.length}</span>
		</h2>
	</div>
	<div class="wrapper">
		<svg {width} {height}>
			<g transform={"translate(" + margin.left + "," + 0 + ")"}>
				<g class="grid-lines">
					{#each product_ids as d}
						<line
							stroke-width="0.5px"
							stroke="#14131A"
							class="grid-line"
							x1={0}
							x2={width - margin.left}
							y1={y(d) + y.bandwidth() / 2}
							y2={y(d) + y.bandwidth() / 2}
						/>
					{/each}
				</g>
				<svg id="test" {width} {height}>
					{#each trades as trade}
						<circle
							use:tooltip
							title=""
							class={trade.side}
							cx={x(trade.dateObj)}
							cy={y(trade.product_id) + y.bandwidth() / 2}
							r={r(trade.last_size / trade.volume_24h)}
							product={trade.product_id}
							lastsize={trade.last_size}
							timestring={getTimeString(trade)}
						/>
					{/each}
				</svg>
			</g>
			<!-- // Axes -->
			<Axis {width} {height} {margin} scale={y} position="left" />
			<Axis {width} {height} {margin} scale={x} position="bottom" />
		</svg>
	</div>
</main>

<style>
	#wrapper {
		margin-left: -65px;
	}
	svg {
		background-color: var(--brand-primary);
	}
	.subtitle {
		text-align: center;
		font-size: 1em;
	}

	.subtitle span {
		font-size: 1.4em;
		color: var(--brand_text);
		padding-left: 0.5rem;
	}

	a {
		color: var(--graph_dark2_orange);
	}

	main {
		text-align: center;
		padding: 0em;
		max-width: 240px;
		margin: 20px auto;
	}

	div.cont {
		max-width: 1024px;
		text-align: left;
		width: 100%;
		margin-left: auto;
		margin-right: auto;
		margin-top: 1rem;
		margin-bottom: 0;
		box-sizing: border-box;
	}

	h2 {
		/* color: var(--brand_heading); */
		margin: 0 auto;
		font-size: 2rem;
		line-height: 1em;
		padding-top: 2rem;
		line-height: 1em;
		fill: var(--brand_text);
		text-align: left;
		text-transform: uppercase;
	}

	p {
		color: white;
		display: inline-block;
		margin-right: 0.2em;
	}

	circle {
		fill: #c5d7ea;
		stroke: #14131a;
		stroke-width: 0.5px;
	}

	.buy {
		fill: var(--graph_dark2_green);
	}

	.sell {
		fill: var(--graph_beeswarm_circle);
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
