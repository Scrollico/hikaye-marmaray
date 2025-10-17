<!--
  SolutionDetailPanel.svelte
  Detailed information panel for metro suicide prevention solutions
  Shows comprehensive data about specific city implementations
-->

<script lang="ts">
  import PreventionTypeIcons from './PreventionTypeIcons.svelte';

  export let city: string;
  export let station: string;
  export let preventionType: 'door' | 'pit' | 'led' | 'training' | 'helpline';
  export let year: number;
  export let effectiveness: number;
  export let stationCount: number;
  export let implementationStatus: 'Complete' | 'Partial' | 'Pilot';
  export let costEstimate: 'Low' | 'Medium' | 'High';
  export let description: string;
  export let researchSource: string;
  export let additionalMeasures: string;
  export let isVisible: boolean = false;

  // City-specific information
  const cityInfo: Record<
    string,
    { flag: string; population: string; metroSystem: string }
  > = {
    Singapore: { flag: '🇸🇬', population: '5.9M', metroSystem: 'SMRT' },
    Tokyo: { flag: '🇯🇵', population: '37.4M', metroSystem: 'Tokyo Metro' },
    London: {
      flag: '🇬🇧',
      population: '9.5M',
      metroSystem: 'London Underground',
    },
    Seoul: { flag: '🇰🇷', population: '9.7M', metroSystem: 'Seoul Metro' },
    Paris: { flag: '🇫🇷', population: '11.1M', metroSystem: 'RATP' },
    Toronto: { flag: '🇨🇦', population: '6.2M', metroSystem: 'TTC' },
    Berlin: { flag: '🇩🇪', population: '3.7M', metroSystem: 'BVG' },
    Amsterdam: { flag: '🇳🇱', population: '1.2M', metroSystem: 'GVB' },
  };

  const preventionTypeNames = {
    door: 'Platform Kapısı',
    pit: 'Ray Altı Çukur',
    led: 'Mavi LED Işıklar',
    training: 'Personel Eğitimi & Yapay Zeka',
    helpline: 'Yardım Hattı & Tabelalar',
  };

  const statusColors = {
    Complete: '#27ae60',
    Partial: '#f39c12',
    Pilot: '#e74c3c',
  };

  const costColors = {
    Low: '#27ae60',
    Medium: '#f39c12',
    High: '#e74c3c',
  };

  const effectivenessColor =
    effectiveness >= 80 ? '#27ae60'
    : effectiveness >= 40 ? '#f39c12'
    : '#e74c3c';
</script>

{#if isVisible}
  <div class="detail-panel" class:visible={isVisible}>
    <div class="panel-header">
      <div class="city-info">
        <span class="flag">{cityInfo[city]?.flag || '🌍'}</span>
        <div class="city-details">
          <h3>{city}</h3>
          <p class="metro-system">
            {cityInfo[city]?.metroSystem || 'Metro Sistemi'}
          </p>
        </div>
      </div>
      <div class="effectiveness-indicator">
        <span class="effectiveness-value" style="color: {effectivenessColor}">
          %{effectiveness}
        </span>
        <span class="effectiveness-label">azalma</span>
      </div>
    </div>

    <div class="panel-content">
      <div class="implementation-details">
        <div class="detail-item">
          <span class="detail-label">İstasyon:</span>
          <span>{station} ({year})</span>
        </div>

        <div class="detail-item">
          <span class="detail-label">Önlem Türü:</span>
          <div class="prevention-type-display">
            <PreventionTypeIcons type={preventionType} size={16} />
            <span>{preventionTypeNames[preventionType]}</span>
          </div>
        </div>

        <div class="detail-item">
          <span class="detail-label">Kapsam:</span>
          <span class="scope-info">
            {stationCount} istasyon -
            <span
              class="status-badge"
              style="background-color: {statusColors[implementationStatus]}"
            >
              {implementationStatus === 'Complete' ? 'Tam'
              : implementationStatus === 'Partial' ? 'Kısmi'
              : 'Pilot'}
            </span>
          </span>
        </div>

        <div class="detail-item">
          <span class="detail-label">Maliyet:</span>
          <span
            class="cost-badge"
            style="background-color: {costColors[costEstimate]}"
          >
            {costEstimate === 'Low' ? 'Düşük'
            : costEstimate === 'Medium' ? 'Orta'
            : 'Yüksek'}
          </span>
        </div>
      </div>

      <div class="description-section">
        <h4>Detaylar</h4>
        <p>{description}</p>
      </div>

      <div class="research-section">
        <h4>Kaynak</h4>
        <p class="research-source">{researchSource}</p>
      </div>

      {#if additionalMeasures}
        <div class="additional-measures">
          <h4>Ek Önlemler</h4>
          <p>{additionalMeasures}</p>
        </div>
      {/if}

      <div class="city-context">
        <h4>Şehir Bilgileri</h4>
        <div class="context-grid">
          <div class="context-item">
            <span class="context-label">Nüfus:</span>
            <span class="context-value"
              >{cityInfo[city]?.population || 'N/A'}</span
            >
          </div>
          <div class="context-item">
            <span class="context-label">Metro Sistemi:</span>
            <span class="context-value"
              >{cityInfo[city]?.metroSystem || 'N/A'}</span
            >
          </div>
        </div>
      </div>
    </div>
  </div>
{/if}

<style>
  .detail-panel {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%) scale(0.9);
    background: white;
    border-radius: 12px;
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
    max-width: 500px;
    max-height: 80vh;
    overflow-y: auto;
    opacity: 0;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    z-index: 2000;
  }

  .detail-panel.visible {
    opacity: 1;
    transform: translate(-50%, -50%) scale(1);
  }

  .panel-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 20px 24px 16px;
    border-bottom: 1px solid #eee;
    background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
    border-radius: 12px 12px 0 0;
  }

  .city-info {
    display: flex;
    align-items: center;
    gap: 12px;
  }

  .flag {
    font-size: 24px;
  }

  .city-details h3 {
    margin: 0;
    font-size: 18px;
    font-weight: 700;
    color: #2c3e50;
  }

  .metro-system {
    margin: 0;
    font-size: 12px;
    color: #7f8c8d;
    font-weight: 500;
  }

  .effectiveness-indicator {
    text-align: center;
  }

  .effectiveness-value {
    display: block;
    font-size: 24px;
    font-weight: 700;
    line-height: 1;
  }

  .effectiveness-label {
    font-size: 11px;
    color: #7f8c8d;
    font-weight: 500;
  }

  .panel-content {
    padding: 20px 24px 24px;
  }

  .implementation-details {
    margin-bottom: 20px;
  }

  .detail-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 8px 0;
    border-bottom: 1px solid #f1f2f6;
  }

  .detail-item:last-child {
    border-bottom: none;
  }

  .detail-label {
    font-weight: 600;
    color: #2c3e50;
    font-size: 13px;
  }

  .prevention-type-display {
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .status-badge,
  .cost-badge {
    padding: 3px 8px;
    border-radius: 12px;
    color: white;
    font-size: 11px;
    font-weight: 600;
  }

  .scope-info {
    font-size: 13px;
    color: #2c3e50;
  }

  .description-section,
  .research-section,
  .additional-measures,
  .city-context {
    margin-bottom: 16px;
  }

  .description-section h4,
  .research-section h4,
  .additional-measures h4,
  .city-context h4 {
    margin: 0 0 8px 0;
    font-size: 14px;
    font-weight: 600;
    color: #2c3e50;
  }

  .description-section p,
  .research-section p,
  .additional-measures p {
    margin: 0;
    font-size: 13px;
    line-height: 1.5;
    color: #555;
  }

  .research-source {
    font-style: italic;
    color: #7f8c8d;
  }

  .context-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 12px;
  }

  .context-item {
    display: flex;
    flex-direction: column;
    gap: 2px;
  }

  .context-label {
    font-size: 11px;
    color: #7f8c8d;
    font-weight: 500;
  }

  .context-value {
    font-size: 13px;
    color: #2c3e50;
    font-weight: 600;
  }

  /* Mobile responsiveness */
  @media (max-width: 768px) {
    .detail-panel {
      max-width: 90vw;
      max-height: 90vh;
    }

    .panel-header {
      flex-direction: column;
      gap: 12px;
      text-align: center;
    }

    .context-grid {
      grid-template-columns: 1fr;
    }
  }
</style>
