<script lang="ts">
	import SurgeryForm from '$lib/SurgeryForm.svelte';
</script>

<svelte:head>
	<title>Surg_sim Scheduler</title>
	<meta
		name="description"
		content="Request reproducible illustrative surgery-duration predictions."
	/>
</svelte:head>

<main>
	<p class="eyebrow">Surg_sim</p>
	<h1>Surgery duration planner</h1>
	<p class="intro">
		Build a case list and request illustrative duration predictions from the companion Rust service.
	</p>
	<SurgeryForm />
	<p class="notice">
		These predictions use demonstration coefficients and are not clinically validated.
	</p>
</main>

<style>
	:global(body) {
		margin: 0;
		background: #f7f8fa;
		color: #172033;
		font-family: Inter, ui-sans-serif, system-ui, sans-serif;
	}
	main {
		width: min(70rem, calc(100% - 2rem));
		margin: 0 auto;
		padding: 4rem 0;
	}
	.eyebrow {
		color: #1f5fae;
		font-weight: 800;
		letter-spacing: 0.12em;
		text-transform: uppercase;
	}
	h1 {
		margin: 0.25rem 0;
		font-size: clamp(2.2rem, 6vw, 4.5rem);
		line-height: 1;
	}
	.intro {
		max-width: 42rem;
		color: #526070;
		font-size: 1.15rem;
		margin-bottom: 2rem;
	}
	.notice {
		color: #687384;
		margin-top: 2rem;
		font-size: 0.9rem;
	}
</style>
