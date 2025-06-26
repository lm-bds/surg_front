<script>
    import Welcome from '$lib/Welcome.svelte';
    import SurgeryForm from '$lib/SurgeryForm.svelte';
	import { onMount } from 'svelte';
    import 'bootstrap/dist/css/bootstrap.css';
    import 'bootstrap-icons/font/bootstrap-icons.css'; // needs additional webpack config!
    import bootstrap5Plugin from '@fullcalendar/bootstrap5';

	let calendar;
	let calendarEl;
	let surgeries = [
		{ id: crypto.randomUUID?.() || Math.random().toString(36), startTime: '', surgeon: '', specialty: '' }
	];

	function addSurgeryForm() {
		surgeries = [...surgeries, { id: crypto.randomUUID?.() || Math.random().toString(36), startTime: '', surgeon: '', specialty: '' }];
	}

	function removeSurgeryForm(id) {
		surgeries = surgeries.filter((s) => s.id !== id);
	}

	function postSurgeries() {
		const events = surgeries
			.filter((s) => s.startTime && s.surgeon && s.specialty)
			.map((s) => ({
				title: `${s.specialty} (${s.surgeon})`,
				start: s.startTime
			}));

		if (calendar) {
			calendar.removeAllEvents();
			events.forEach((event) => calendar.addEvent(event));
		}

		alert('Mock POST done!');
	}

	onMount(async () => {
        // await import('@fullcalendar/core/main.css');
		// await import('@fullcalendar/daygrid/main.css');
		// await import('@fullcalendar/timegrid/main.css');

		const { Calendar } = await import('@fullcalendar/core');
		const dayGridPlugin = (await import('@fullcalendar/daygrid')).default;
		const timeGridPlugin = (await import('@fullcalendar/timegrid')).default;


		calendar = new Calendar(calendarEl, {
			plugins: [dayGridPlugin, timeGridPlugin, bootstrap5Plugin],
			initialView: 'timeGridDay',
			height: 800,
            allDaySlot: false,
            dayHeaders: false,

            themeSystem: 'bootstrap5',
            headerToolbar: {
                left: '',
                right: '',
            },

		});

		calendar.render();
	});
</script>

<style>
	form {
		margin-bottom: 2rem;
	}
	input {
		margin-right: 0.5rem;
	}
	.calendar-container {
		border: 1px solid #ccc;
		padding: 1rem;
	}
    	/* --- MODERN H1 STYLING --- */
	h1 {
		font-family: 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif; /* A more modern sans-serif stack */
		font-size: 2.8em; /* Make it larger and more impactful */
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
<Welcome>

<main>
    <h1>Surgery Scheduler</h1>

    <SurgeryForm />
</main>

</Welcome>
<!-- <form on:submit|preventDefault={postSurgeries}>
	{#each surgeries as s (s.id)}
		<div>
			<input type="datetime-local" bind:value={s.startTime} />
			<input type="text" placeholder="Surgeon" bind:value={s.surgeon} />
			<input type="text" placeholder="Specialty" bind:value={s.specialty} />
			<button type="button" on:click={() => removeSurgeryForm(s.id)}>Remove</button>
		</div>
	{/each}
	<button type="button" on:click={addSurgeryForm}>Add Surgery</button>
	<button type="submit">Post</button>
</form> -->

<div bind:this={calendarEl} class="calendar-container"></div>
