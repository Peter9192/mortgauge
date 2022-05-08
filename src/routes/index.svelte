<script>
  import Chart from 'chart.js/auto/auto.esm';
	import { onMount } from 'svelte';
import { xlink_attr } from 'svelte/internal';

  // Bereken hypotheek
  let T = 300000;
  let r = 2.78;
  let N = 360;

  $: rr = (1 + r / 100) ** (1 / 12) - 1;
  $: A = T * rr / (1 - (1 + rr) ** -N);

  $: a = A - rr * T  // Aflossingsvoet
  $: n = Array.from(Array(N).keys());  // Termijnen
  $: an = n.map(x => a * (1 + rr) ** x);  // Aflossingsdeel
  $: tn = n.map((x, i) => A - an[i]);  // Rentedeel
  $: hra = tn.map(x => 0.37 * x);  // Hypotheekrenteaftrek
  $: nr = tn.map(x => x * 0.63)  // Netto rentelast
  $: netto = hra.map(x => A - x); // Netto maandlast

  let chartCanvas;
  let chart = null;

	onMount(() => {
    chart = new Chart(chartCanvas, {
      type: 'bar',
      data: {
        labels: n,
        datasets: [
          { label: 'Aflossingsdeel', data: an, backgroundColor: 'rgba(255, 99, 132, 0.2)', fill: true, categoryPercentage: 1.0, barPercentage: 1.0 },
          { label: 'Rente - niet aftrekbaar', data: nr, backgroundColor: 'rgba(54, 162, 235, 0.2)', fill: true, categoryPercentage: 1.0, barPercentage: 1.0 },
          { label: 'Rente - aftrekbaar', data: hra, backgroundColor: 'rgba(255, 206, 86, 0.2)', fill: true, categoryPercentage: 1.0, barPercentage: 1.0 }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          y: {
            stacked: true
          },
          x: {
            grid: {display: false},
            stacked: true
          }
        }
      }
    });
  });

</script>

<h1>Bereken je hypotheek</h1>
<ul>
  <li>Totale lening: €<input bind:value={T}></li>
  <li>Rente op jaarbasis: <input bind:value={r}>%</li>
  <li>Termijnen: <input bind:value={N}> termijnen</li>
  <li>Annuiteit: €{A}</li>
  <!-- <li>Netto maandlast: {netto}</li> -->
</ul>

<div>
  <canvas bind:this={chartCanvas} width="400" height="400"></canvas>
</div>
