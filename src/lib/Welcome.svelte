<script>
  import 'bootstrap/dist/css/bootstrap.css';
    import 'bootstrap-icons/font/bootstrap-icons.css'; // needs additional webpack config!
	let expandBar = false;
  import { onMount } from 'svelte';

  let phase = 0;

  onMount(async () => {
    await delay(50);
    phase = 1; 
    expandBar = true; // fade in + bar grow
    await delay(2000);
    phase = 2; // split 
    await delay(100);
    phase = 3; // wipe away
    await delay(2600);
    phase = 4; // show content
  });

  function delay(ms) { 
    return new Promise((res) => setTimeout(res, ms));
  }
</script>

<style>
  .welcome {
    position: fixed;
    inset: 0;
    background: white;
    z-index: 999;
    overflow: hidden;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;

  }

.title {
  font-size: 4rem;
  color: black;
  opacity: 0; /* Starts invisible */
  transform: translateY(20px); /* Starts slightly below its final position */
  transition: opacity 2.6s ease, transform 0.6s ease; /* Transitions over 0.6s */
  z-index: 1001; 
}

.title.visible {
  opacity: 1; /* Becomes visible */
  transform: translateY(0); /* Moves to its final position */
}

  .full-bar {
    position: absolute;
    top: 50%;
    left: 50%;
    height: 6px;
    background: black;
    width: 0;
    transform: translate(-50%, -50%);
    opacity: 1;
    z-index: 1000;
    transition: width 1.6s ease, opacity 0.6s, transform 0.6s; /* Smooth width transition */
  }

  .full-bar.expand {
    width: 100%;
  }

  /* Split overlay stage */
  .bar-half {
    position: absolute; 
    width: 100%;
    height: 50%;
    background: black;
    z-index: 1000;
    transform: translateY(0%); /* Initial state */
    opacity: 0; /* Start transparent */
    transition: transform 2.5s ease, opacity 0.5s ease; /* Added opacity transition */
  }

  .bar-half.visible { /* New class for fading in */
    opacity: 1;
  }

  .top-bar {
    top: 0;
  }

  .bottom-bar {
    bottom: 0;
  }

  .top-bar.wipe {
    transform: translateY(-100%);
  }

  .bottom-bar.wipe {
    transform: translateY(100%);
  }

  .main {
    padding: 2rem;
    opacity: 0;
    transform: translateY(20px);
    transition: opacity 0.8s ease, transform 0.8s ease;
  }

  .main.visible {
    opacity: 1;
    transform: translateY(0);
  }
  h1 {
		font-family: 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif; /* A more modern sans-serif stack */
		font-size: 0.8em; /* Make it larger and more impactful */
		font-weight: 700; /* Bold */
		color: #2c3e50; /* A deep, professional grey/blue */
		text-align: center;
		margin-top: 2rem; /* More space above */
		margin-bottom: 2.5rem; /* More space below */
		letter-spacing: 0.02em; /* Slightly increased letter spacing for refinement */
		text-transform: uppercase; /* Optional: adds a modern, bold feel */
		/* Optional: Subtle text shadow for depth */
		text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.08);

		/* Optional: A subtle underline effect */
		position: relative;
		display: inline-block; /* Needed for text-decoration-thickness or ::after */
		left: 50%;
		transform: translateX(-50%); /* Center the inline-block element */
	}

	/* Optional: More sophisticated underline for h1 */
	h1::after {
		content: '';
		display: block;
		width: 60px; /* Width of the underline */
		height: 4px; /* Thickness of the underline */
		background-color: #007bff; /* Bootstrap primary blue */
		margin: 10px auto 0; /* Center the underline below the text */
		border-radius: 2px; /* Soften the edges */
	}
</style>

{#if phase < 4}
  <div class="welcome">
<div class="title {phase >= 2 ? 'visible' : ''}"><h1>Surgery Scheduler</h1></div>
        {#if phase === 0 || phase === 1} <div class="full-bar {expandBar ? 'expand' : ''}"></div>
{/if} 

    {#if phase >= 1}
        <div class="bar-half top-bar {phase >= 3 ? 'wipe' : ''} {phase >= 2 ? 'visible' : ''}"></div>
  <div class="bar-half bottom-bar {phase >= 3 ? 'wipe' : ''} {phase >= 2 ? 'visible' : ''}"></div>
    {/if}
  </div>
{/if}

<div class="main {phase === 4 ? 'visible' : ''}">
  <slot />
</div>   
