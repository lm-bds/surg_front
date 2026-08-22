<script lang="ts">
	type Surgery = {
		id: string;
		predictedStart: string;
		surgeon: string;
		procedure: string;
		diagnosis: string;
	};

	type Prediction = {
		procedure: string;
		surgeon: string;
		predictedMinutes: number;
	};

	const surgeons = ['Dr. A', 'Dr. B', 'Dr. C'];
	const procedures = ['ProcA', 'ProcB', 'ProcC', 'ProcD', 'ProcE'];
	const apiBase = import.meta.env.VITE_API_BASE || 'http://127.0.0.1:3001';

	let surgeries: Surgery[] = [newSurgery()];
	let predictions: Prediction[] = [];
	let submitting = false;
	let submitError = '';

	function newSurgery(): Surgery {
		return {
			id: crypto.randomUUID(),
			predictedStart: '',
			surgeon: '',
			procedure: '',
			diagnosis: ''
		};
	}

	function addSurgery(): void {
		surgeries = [...surgeries, newSurgery()];
	}

	function removeSurgery(id: string): void {
		surgeries = surgeries.filter((surgery) => surgery.id !== id);
		if (surgeries.length === 0) addSurgery();
	}

	async function submit(): Promise<void> {
		submitError = '';
		predictions = [];
		if (surgeries.some((surgery) => !surgery.surgeon || !surgery.procedure)) {
			submitError = 'Choose a surgeon and procedure for every case.';
			return;
		}

		submitting = true;
		try {
			const response = await fetch(`${apiBase}/api/predict-duration`, {
				method: 'POST',
				headers: { 'content-type': 'application/json' },
				body: JSON.stringify({
					surgeries: surgeries.map(({ surgeon, procedure, diagnosis, predictedStart }) => ({
						surgeon,
						procedure,
						diagnosis: diagnosis || null,
						predictedStart: predictedStart || null
					}))
				})
			});
			if (!response.ok) throw new Error((await response.text()) || `HTTP ${response.status}`);
			const result = (await response.json()) as { predictions?: Prediction[] };
			if (!Array.isArray(result.predictions))
				throw new Error('Invalid response from prediction service');
			predictions = result.predictions;
		} catch (error) {
			submitError = error instanceof Error ? error.message : 'Prediction request failed';
		} finally {
			submitting = false;
		}
	}
</script>

<form on:submit|preventDefault={submit}>
	{#each surgeries as surgery, index (surgery.id)}
		<fieldset>
			<legend>Case {index + 1}</legend>
			<label
				>Planned start <input type="datetime-local" bind:value={surgery.predictedStart} /></label
			>
			<label
				>Surgeon
				<select bind:value={surgery.surgeon} required
					><option value="">Select</option>{#each surgeons as surgeon (surgeon)}<option
							value={surgeon}>{surgeon}</option
						>{/each}</select
				>
			</label>
			<label
				>Procedure
				<select bind:value={surgery.procedure} required
					><option value="">Select</option>{#each procedures as procedure (procedure)}<option
							value={procedure}>{procedure}</option
						>{/each}</select
				>
			</label>
			<label>Diagnosis <input bind:value={surgery.diagnosis} placeholder="Optional" /></label>
			<button class="remove" type="button" on:click={() => removeSurgery(surgery.id)}>Remove</button
			>
		</fieldset>
	{/each}
	<div class="actions">
		<button type="button" on:click={addSurgery}>Add case</button>
		<button class="primary" type="submit" disabled={submitting}
			>{submitting ? 'Predicting…' : 'Predict durations'}</button
		>
	</div>
</form>

{#if submitError}<p class="error" role="alert">{submitError}</p>{/if}
{#if predictions.length}
	<section aria-live="polite">
		<h2>Predictions</h2>
		<ul>
			{#each predictions as prediction, index (index)}<li>
					<strong>{prediction.procedure}</strong> with {prediction.surgeon}: {prediction.predictedMinutes}
					minutes
				</li>{/each}
		</ul>
	</section>
{/if}

<style>
	form {
		display: grid;
		gap: 1rem;
	}
	fieldset {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(12rem, 1fr));
		gap: 1rem;
		border: 1px solid #d7dce2;
		border-radius: 0.75rem;
		padding: 1rem;
	}
	legend {
		font-weight: 700;
		padding: 0 0.4rem;
	}
	label {
		display: grid;
		gap: 0.35rem;
		font-weight: 600;
	}
	input,
	select,
	button {
		font: inherit;
		padding: 0.65rem 0.75rem;
		border: 1px solid #aab2bd;
		border-radius: 0.45rem;
	}
	button {
		cursor: pointer;
		background: white;
	}
	.primary {
		color: white;
		background: #1f5fae;
		border-color: #1f5fae;
	}
	.remove {
		align-self: end;
		color: #9d1c1c;
	}
	.actions {
		display: flex;
		gap: 0.75rem;
		flex-wrap: wrap;
	}
	.error {
		color: #9d1c1c;
		font-weight: 600;
	}
	section {
		margin-top: 2rem;
		padding: 1rem;
		border-radius: 0.75rem;
		background: #eef5ff;
	}
</style>
