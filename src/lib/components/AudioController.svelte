<script lang="ts">
  /**
   * AudioController Component
   * User-controlled ambient audio with mute/unmute button
   * SAFE: Completely isolated, can be deleted without affecting anything
   */
  import { onMount } from 'svelte';

  let isMuted = true; // Start muted for safety and browser autoplay policy
  let audioElement: HTMLAudioElement | null = null;
  let isPlaying = false;
  let audioError = false;

  /**
   * Toggle audio playback
   * User must click to enable (browser security requirement)
   */
  function toggleAudio() {
    console.log('🔧 toggleAudio called');
    if (!audioElement || audioError) {
      console.log('⚠️ No audio element or error state');
      return;
    }

    if (isPlaying) {
      audioElement.pause();
      isPlaying = false;
      isMuted = true;
      console.log('🔇 Audio paused');
    } else {
      audioElement
        .play()
        .then(() => {
          isPlaying = true;
          isMuted = false;
          console.log('🔊 Audio playing');
        })
        .catch((err) => {
          console.log('⚠️ Audio play prevented by browser:', err.message);
          audioError = true;
        });
    }
  }

  onMount(() => {
    console.log('🎵 AudioController onMount called');
    try {
      // Create audio element only when component mounts
      audioElement = new Audio('/audios/ambient-background.ogg');
      audioElement.loop = true;
      audioElement.volume = 0.15; // Very subtle background volume
      console.log('🎵 Audio element created:', audioElement);

      // Cleanup on unmount
      return () => {
        if (audioElement) {
          audioElement.pause();
          audioElement.src = ''; // Release resources
          console.log('🧹 Audio cleaned up');
        }
      };
    } catch (error) {
      console.error('⚠️ Audio initialization failed:', error);
      audioError = true;
    }
  });
</script>

<!-- Floating mute/unmute button -->
<button
  class="audio-control"
  class:error={audioError}
  on:click={toggleAudio}
  aria-label={isMuted ? 'Enable ambient audio' : 'Disable ambient audio'}
  title={audioError ? 'Audio unavailable'
  : isMuted ? 'Click to enable ambient audio'
  : 'Click to mute audio'}
>
  {#if audioError}
    <span class="icon">⚠️</span>
  {:else if isMuted}
    <span class="icon">🔇</span>
  {:else}
    <span class="icon">🔊</span>
  {/if}
</button>

<style>
  .audio-control {
    position: fixed;
    bottom: 2rem;
    right: 2rem;
    width: 3.5rem;
    height: 3.5rem;
    border-radius: 50%;
    background: rgba(0, 0, 0, 0.7);
    border: 2px solid rgba(255, 255, 255, 0.3);
    color: white;
    font-size: 1.25rem;
    cursor: pointer;
    z-index: 1000;
    transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
    backdrop-filter: blur(10px);
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 0;
  }

  .audio-control:hover {
    transform: scale(1.1);
    background: rgba(0, 0, 0, 0.85);
    border-color: rgba(255, 255, 255, 0.5);
  }

  .audio-control:active {
    transform: scale(0.95);
  }

  .audio-control.error {
    opacity: 0.5;
    cursor: not-allowed;
  }

  .audio-control.error:hover {
    transform: none;
  }

  .icon {
    display: block;
    line-height: 1;
  }

  /* Mobile responsive */
  @media (max-width: 768px) {
    .audio-control {
      bottom: 1rem;
      right: 1rem;
      width: 3rem;
      height: 3rem;
      font-size: 1rem;
    }
  }
</style>
