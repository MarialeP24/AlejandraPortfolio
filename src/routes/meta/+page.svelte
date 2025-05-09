<script>
	import * as d3 from 'd3';
	import { onMount } from "svelte";
	import { computePosition, autoPlacement, offset } from '@floating-ui/dom';
	import Bar from '$lib/Bar.svelte';


	let data = [];
	let commits = [];
	let averageFileLength = 0;
	let files = [];
	let commitTooltip;
	let tooltipPosition = { x: 0, y: 0 };
	let hoveredIndex = -1;
	$: hoveredCommit = commits[hoveredIndex] ?? {};
	let clickedCommits = [];

	$: allTypes = Array.from(new Set(data.map(d => d.type)));

	$: selectedLines = (clickedCommits.length > 0 ? clickedCommits : commits).flatMap(d => d.lines);

	$: selectedCounts = d3.rollup(
    selectedLines,
    v => v.length,
    d => d.type
	);
	$: languageBreakdown = allTypes.map(type => [type, selectedCounts.get(type) || 0]);

  
	onMount(async () => {
	  data = await d3.csv("/loc.csv", row => ({
		...row,
		line: +row.line,
		depth: +row.depth,
		length: +row.length,
		date: new Date(row.date + "T00:00" + row.timezone),
		datetime: new Date(row.datetime),
	  }));
  
	  commits = d3.groups(data, d => d.commit).map(([commit, lines]) => {
		const { author, date, time, timezone, datetime } = lines[0];
		const ret = {
		  id: commit,
		  url: `https://github.com/vis-society/lab-7/commit/${commit}`,
		  author, date, time, timezone, datetime,
		  hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
		  totalLines: lines.length,
		};
		Object.defineProperty(ret, "lines", {
		  value: lines,
		  configurable: true,
		  writable: true,
		  enumerable: false,
		});
		return ret;
	  });
  
	  files = d3.groups(data, d => d.file);
	  averageFileLength = d3.mean(files.map(([, lines]) => lines.length));
	});
  
	let width = 1000, height = 600;
	let margin = { top: 10, right: 10, bottom: 30, left: 20 };
	let usableArea = {
	  top: margin.top,
	  right: width - margin.right,
	  bottom: height - margin.bottom,
	  left: margin.left
	};
	usableArea.width = usableArea.right - usableArea.left;
	usableArea.height = usableArea.bottom - usableArea.top;
  
	$: minDate = d3.min(commits.map(d => d.date));
	$: maxDate = d3.max(commits.map(d => d.date));
	$: maxDatePlusOne = (() => {
	  const d = new Date(maxDate);
	  d.setDate(d.getDate() + 1);
	  return d;
	})();
  
	$: xScale = d3.scaleTime()
	  .domain([minDate, maxDatePlusOne])
	  .range([usableArea.left, usableArea.right])
	  .nice();
  
	$: yScale = d3.scaleLinear()
	  .domain([24, 0])
	  .range([usableArea.bottom, usableArea.top]);
  
	let xAxis, yAxis, yAxisGridlines;
	$: if (xAxis && yAxis) {
	  d3.select(xAxis).call(d3.axisBottom(xScale));
	  d3.select(yAxis).call(
		d3.axisLeft(yScale)
		  .tickFormat(d => String(d % 24).padStart(2, "0") + ":00")
	  );
	}
	$: if (yAxisGridlines) {
	  d3.select(yAxisGridlines).call(
		d3.axisLeft(yScale)
		  .tickFormat("")
		  .tickSize(-usableArea.width)
	  );
	}
  
	async function dotInteraction (index, evt) {
	let hoveredDot = evt.target;
	if (evt.type === "mouseenter") {
		hoveredIndex = index;
		tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
			strategy: "fixed", // because we use position: fixed
			middleware: [
				offset(5), // spacing from tooltip to dot
				autoPlacement() // see https://floating-ui.com/docs/autoplacement
			],
		});        }
	else if (evt.type === "mouseleave") {
		hoveredIndex = -1
	}
	
	else if (evt.type === "click") {
	let commit = commits[index]
	if (!clickedCommits.includes(commit)) {
		// Add the commit to the clickedCommits array
		clickedCommits = [...clickedCommits, commit];
	}
	else {
			// Remove the commit from the array
			clickedCommits = clickedCommits.filter(c => c !== commit);
	}
	console.log(clickedCommits);
}

}

  </script>
  
  <svelte:head>
	<title>Code Data</title>
  </svelte:head>
  
  <h1>Meta</h1>
  <p>This page includes stats about the code of this website</p>
  
  <dl class="stats">
	<dt>Total <abbr title="Lines of code">LOC</abbr></dt>
	<dd>{data.length}</dd>
	<dt>Total Commits</dt>
	<dd>{commits.length}</dd>
	<dt>Average File Length</dt>
	<dd>{averageFileLength ? averageFileLength.toFixed(2) : 0} lines</dd>
	<dt>Number of Files</dt>
	<dd>{files.length}</dd>
  </dl>
  
  <h3>Commits by Time of Day</h3>
  <svg viewBox="0 0 {width} {height}">
	<g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
	<g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
	<g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
  
	<g class="dots">
	  {#each commits as commit, index}
		<circle
		class:selected={ clickedCommits.includes(commit) }
		  on:click={ evt => dotInteraction(index, evt) }
		  cx={xScale(commit.datetime)}
		  cy={yScale(commit.hourFrac)}
		  r="5"
		  fill="steelblue"
		  on:mouseenter={evt => dotInteraction(index, evt)}
		  on:mouseleave={evt => dotInteraction(index, evt)}
		/>
	  {/each}
	</g>
  </svg>
  
  <Bar data={languageBreakdown} width={width} />

  <dl
	class="info tooltip"
	bind:this={commitTooltip}
	hidden={hoveredIndex === -1}
	style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px;"
  >
	<dt>Commit</dt>
	<dd><a href={hoveredCommit.url} target="_blank">{hoveredCommit.id}</a></dd>
	<dt>Date</dt>
	<dd>{hoveredCommit.datetime?.toLocaleDateString("en", { dateStyle: "full" })}</dd>
	<dt>Time</dt>
	<dd>{hoveredCommit.datetime?.toLocaleTimeString("en", { timeStyle: "medium" })}</dd>
	<dt>Author</dt>
	<dd>{hoveredCommit.author}</dd>
	<dt>Lines edited</dt>
	<dd>{hoveredCommit.totalLines}</dd>
  </dl>
  
  <style>
	.stats {
	  display: grid;
	  grid-template-columns: repeat(4, 1fr);
	  gap: 1rem;
	  margin: 2rem 0;
	}
	.stats dt { font-weight: bold; }
	.stats dd {
	  margin: 0;
	  font-variant-numeric: tabular-nums;
	}
  
	h1 { font-size: 3rem; margin-bottom: 1rem; }
	p { line-height: 1.6; margin-bottom: 1rem; }
  
	svg { overflow: visible; }
	.gridlines { stroke-opacity: 0.2; }
  
	dl.info {
	  display: grid !important;  /* override UA hidden display */
	  grid-template-columns: auto 1fr;
	  gap: 0.5rem 1rem;
	  padding: 1rem;
	  background-color: white;
	  border-radius: 4px;
	  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
	  transition: opacity 500ms, visibility 500ms;
	}
	dl.info[hidden]:not(:hover):not(:focus-within) {
	  opacity: 0;
	  visibility: hidden;
	}
	dl.info dt { color: #555; }
	dl.info dd { font-weight: bold; color: #555; margin: 0; }
  
	.tooltip {
	  position: fixed;
	  z-index: 100;
	  pointer-events: none;
	  max-width: 300px;
	  background-color: oklch(100% 0% 0 / 80%);
	  padding: 12px;
	  border-radius: 6px;
	  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
	  backdrop-filter: blur(4px);
	}
  
	circle {
	transition: 200ms;
	transform-origin: center;
transform-box: fill-box;

	&:hover {
		transform: scale(1.5);
	}
	
}
.selected {
    fill: var(--color-accent);
}
  </style>
  