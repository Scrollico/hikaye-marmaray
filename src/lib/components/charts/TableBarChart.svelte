<script lang="ts">
  import { onMount } from 'svelte';

  interface TableData {
    category: string;
    value: number;
  }

  export let data: TableData[] = [];
  export let title: string = '';
  export let width: number = 600;
  export let height: number = 400;
  export let autoHeight: boolean = false;
  export let barProgress: number = 1;
  export let colorScheme: string[] = [
    '#e74c3c',
    '#c0392b',
    '#a93226',
    '#922b21',
    '#7b241c',
    '#641e16',
    '#1b0f0a',
  ];

  let container: HTMLDivElement;
  let maxValue: number = 0;
  let processedData: Array<TableData & { barWidth: number; color: string }> =
    [];

  // Process data and calculate bar widths
  $: {
    if (data && data.length > 0) {
      maxValue = Math.max(...data.map((d) => d.value));
      processedData = data.map((item, index) => ({
        ...item,
        barWidth: (item.value / maxValue) * 100 * barProgress,
        color: colorScheme[index % colorScheme.length],
      }));
    }
  }

  // Color scale for bars
  function getBarColor(value: number, maxValue: number): string {
    const intensity = value / maxValue;
    if (intensity >= 0.8) return '#e74c3c'; // Red for highest values
    if (intensity >= 0.6) return '#c0392b';
    if (intensity >= 0.4) return '#a93226';
    if (intensity >= 0.3) return '#922b21';
    if (intensity >= 0.2) return '#7b241c';
    return '#641e16'; // Darker red for lower values
  }
</script>

<div
  bind:this={container}
  class="table-bar-chart"
  class:auto-height={autoHeight}
  style="width: {width}px; {autoHeight ? '' : `height: ${height}px;`}"
>
  {#if title}
    <h3 class="chart-title">{title}</h3>
  {/if}

  <div class="table-container">
    <table class="data-table">
      <thead>
        <tr>
          <th class="category-header">Gün</th>
          <th class="bar-header">Vakalar</th>
        </tr>
      </thead>
      <tbody>
        {#each processedData as item, index}
          <tr class="data-row">
            <td class="category-cell">
              <span class="category-label">{item.category}</span>
            </td>
            <td class="bar-cell">
              <div class="bar-container">
                <div
                  class="rectangle-bar"
                  style="width: {item.barWidth}%; background-color: {getBarColor(
                    item.value,
                    maxValue
                  )};"
                  role="img"
                  aria-label="{item.category}: {item.value} vaka"
                ></div>
                <span class="bar-label">{item.value}</span>
              </div>
            </td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
</div>

<style lang="scss">
  .table-bar-chart {
    font-family: 'Helvetica Neue', Arial, sans-serif;
    background: transparent;
    border-radius: 0;
    padding: 0.5rem; /* Much smaller padding */
    box-shadow: none;
    margin: 1rem 0; /* Reduced margin */
    max-width: 100%; /* Ensure it doesn't exceed container */
  }

  .table-bar-chart.auto-height {
    min-height: auto;
    height: auto;
  }

  .chart-title {
    font-size: 1rem; /* Smaller title */
    font-weight: 600;
    color: #1f2937;
    margin: 0 0 0.75rem 0; /* Much smaller margin */
    text-align: center;
  }

  .table-container {
    overflow-x: auto;
  }

  .table-bar-chart.auto-height .table-container {
    overflow-y: visible;
  }

  .data-table {
    width: 100%;
    border-collapse: collapse;
    font-size: 0.75rem; /* Smaller overall font */
  }

  .category-header,
  .bar-header {
    padding: 0.25rem 0.75rem; /* Very compact padding */
    text-align: left;
    font-weight: 600;
    font-size: 0.8rem; /* Smaller font */
    color: #374151;
    border-bottom: 1px solid #e5e7eb;
    background-color: #f9fafb;
  }

  .category-header {
    width: 30%;
  }

  .bar-header {
    width: 70%;
  }

  .data-row {
    border-bottom: 1px solid #e5e7eb;
    transition: background-color 0.2s ease;

    &:hover {
      background-color: #f9fafb;
    }

    &:last-child {
      border-bottom: none;
    }
  }

  .category-cell {
    padding: 0.25rem 0.75rem; /* Very compact padding */
    vertical-align: middle;
    font-weight: 500;
    font-size: 0.8rem; /* Smaller font */
    color: #1f2937;
  }

  .category-label {
    display: block;
    font-size: 0.75rem; /* Smaller label font */
  }

  .bar-cell {
    padding: 0.25rem 0.75rem; /* Very compact padding */
    vertical-align: middle;
  }

  .bar-container {
    position: relative;
    display: flex;
    align-items: center;
    gap: 0.5rem; /* Very small gap */
    min-height: 1rem; /* Very compact min-height */
  }

  .rectangle-bar {
    height: 0.75rem; /* Very compact bar height */
    border-radius: 3px;
    transition: width 0.6s cubic-bezier(0.4, 0, 0.2, 1);
    position: relative;
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);

    &::after {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background: linear-gradient(
        90deg,
        transparent 0%,
        rgba(255, 255, 255, 0.2) 50%,
        transparent 100%
      );
      border-radius: 4px;
    }
  }

  .bar-label {
    font-weight: 600;
    color: #374151;
    font-size: 0.85rem;
    min-width: 2rem;
    text-align: right;
  }

  // Responsive design
  @media (max-width: 768px) {
    .table-bar-chart {
      padding: 1rem;
      margin: 1rem 0; /* Reduced margin on mobile */
    }

    .chart-title {
      font-size: 1.1rem;
      margin-bottom: 1rem;
    }

    .category-header,
    .bar-header {
      padding: 0.5rem 0.75rem;
      font-size: 0.85rem;
    }

    .category-cell,
    .bar-cell {
      padding: 0.75rem 0.5rem;
    }

    .bar-container {
      gap: 0.75rem;
    }

    .rectangle-bar {
      height: 1.25rem;
    }

    .bar-label {
      font-size: 0.8rem;
    }
  }

  // Animation for progressive reveal
  .rectangle-bar {
    animation: revealBar 0.8s ease-out;
  }

  @keyframes revealBar {
    0% {
      width: 0% !important;
      opacity: 0;
    }
    100% {
      opacity: 1;
    }
  }

  // High contrast mode support
  @media (prefers-contrast: high) {
    .rectangle-bar {
      border: 2px solid #000;
    }

    .data-row:hover {
      background-color: #e5e7eb;
    }
  }

  // Reduced motion support
  @media (prefers-reduced-motion: reduce) {
    .rectangle-bar {
      transition: none;
      animation: none;
    }
  }
</style>
