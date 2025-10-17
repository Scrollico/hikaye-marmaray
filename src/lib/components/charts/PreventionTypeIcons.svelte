<!--
  PreventionTypeIcons.svelte
  SVG icons for different metro suicide prevention types
  Used in GlobalSolutionsSwarm and detail panels
-->

<script lang="ts">
  export let type: 'door' | 'pit' | 'led' | 'training' | 'helpline';
  export let size: number = 16;
  export let color: string = '#2c3e50';
  export let showLabel: boolean = false;

  const iconPaths = {
    door: 'M4 6h16v2H4V6zm0 4h16v2H4v-2zm0 4h16v2H4v-2zm0 4h16v2H4v-2z',
    pit: 'M2 12h20v8H2v-8zm2 2v4h16v-4H4zm8-2v2h2v-2h-2z',
    led: 'M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z',
    training:
      'M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z',
    helpline:
      'M6.62 10.79c1.44 2.83 3.76 5.14 6.59 6.59l2.2-2.2c.27-.27.67-.36 1.02-.24 1.12.37 2.33.57 3.57.57.55 0 1 .45 1 1V20c0 .55-.45 1-1 1-9.39 0-17-7.61-17-17 0-.55.45-1 1-1h3.5c.55 0 1 .45 1 1 0 1.25.2 2.45.57 3.57.11.35.03.74-.25 1.02l-2.2 2.2z',
  };

  const iconLabels = {
    door: 'Platform Kapısı',
    pit: 'Ray Altı Çukur',
    led: 'Mavi LED',
    training: 'Eğitim & AI',
    helpline: 'Yardım Hattı',
  };

  const iconDescriptions = {
    door: 'Fiziksel engel oluşturur',
    pit: 'Güvenli alan sağlar',
    led: 'Sakinleştirici etki',
    training: 'Erken müdahale',
    helpline: 'Psikolojik destek',
  };
</script>

<div class="prevention-icon" class:with-label={showLabel}>
  <svg width={size} height={size} viewBox="0 0 24 24" fill="none">
    <path d={iconPaths[type]} fill={color} />
  </svg>
  {#if showLabel}
    <div class="icon-label">
      <div class="label-text">{iconLabels[type]}</div>
      <div class="label-desc">{iconDescriptions[type]}</div>
    </div>
  {/if}
</div>

<style>
  .prevention-icon {
    display: inline-flex;
    align-items: center;
    gap: 8px;
  }

  .with-label {
    flex-direction: column;
    align-items: flex-start;
    gap: 4px;
  }

  .icon-label {
    display: flex;
    flex-direction: column;
    gap: 2px;
  }

  .label-text {
    font-size: 12px;
    font-weight: 600;
    color: var(--text-primary, #2c3e50);
  }

  .label-desc {
    font-size: 10px;
    color: var(--text-secondary, #7f8c8d);
  }

  svg {
    transition: transform 0.2s ease;
  }

  .prevention-icon:hover svg {
    transform: scale(1.1);
  }
</style>
