<script>
    import { v4 as uuidv4 } from 'uuid'; // For unique IDs for each card
    import { onMount } from 'svelte'; // Import onMount for initial setup

    // --- State Variables ---
    let surgeries = []; // Your array to hold surgery data

    // These will eventually be populated with your actual lists.
    // For now, they include a default "Select" option.
    let surgeons = ['Select Surgeon', 'Dr. Smith', 'Dr. Jones', 'Dr. Williams'];
    let procedures = ['Select Procedure', 'Appendectomy', 'Cholecystectomy', 'Hernia Repair'];
    let diagnoses = ['Select Diagnosis', 'Acute Appendicitis', 'Gallstones', 'Inguinal Hernia'];

    // --- Animation State Variables (from previous interactions) ---
    let phase = 0;
    let expandBar = false;
    let titleVisible = false;

    // --- Life Cycle Hooks ---
    onMount(async () => {
        // Animation sequence
        await delay(50); // Small delay for initial bar render

        phase = 1;
        expandBar = true; // Bar grow
        await delay(1000); // Wait for bar expansion

        phase = 2; // Split (black halves appear)
        titleVisible = true; // Show title
        await delay(100); // Brief pause before wipe starts

        phase = 3; // Wipe away
        titleVisible = false; // Hide title
        await delay(1000); // Wait for wipe animation

        phase = 4; // Show content (the form)

        // Initialize with one empty surgery card after animation completes
        addSurgeryForm();
    });

    // --- Utility Functions ---
    function delay(ms) {
        return new Promise((res) => setTimeout(res, ms));
    }

    // --- Surgery Form Functions ---
    // Function to add a new surgery form card
    function addSurgeryForm() {
        surgeries = [...surgeries, {
            id: uuidv4(), // Generate a unique ID for Svelte's reactivity
            predictedStart: '', // For datetime-local
            surgeon: surgeons[0], // Default to the 'Select Surgeon' option
            procedure: procedures[0], // Default to the 'Select Procedure' option
            diagnosis: diagnoses[0] // Default to the 'Select Diagnosis' option
        }];
    }

    // Function to remove a surgery form card
    function removeSurgeryForm(idToRemove) {
        surgeries = surgeries.filter(s => s.id !== idToRemove);
    }

    // Configurable API base — point this at the Surg_sim backend (or a proxy
    // that wraps it). Override at build time via VITE_API_BASE if needed.
    const API_BASE = import.meta.env.VITE_API_BASE || '';

    // Submission state for UI feedback
    let submitting = false;
    let submitError = null;
    let predictions = null; // array of {procedure, surgeon, predictedMinutes}

    // Function to handle form submission
    async function postSurgeries() {
        // Clean and prepare data for posting
        const dataToPost = surgeries.map(s => ({
            surgeon: s.surgeon === surgeons[0] ? null : s.surgeon,
            procedure: s.procedure === procedures[0] ? null : s.procedure,
            diagnosis: s.diagnosis === diagnoses[0] ? null : s.diagnosis,
            predictedStart: s.predictedStart || null
        })).filter(s => s.surgeon && s.procedure); // only fully-specified cases

        if (dataToPost.length === 0) {
            submitError = "Add at least one surgery with a surgeon and procedure.";
            return;
        }

        submitting = true;
        submitError = null;
        predictions = null;
        try {
            const res = await fetch(`${API_BASE}/api/predict-duration`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ surgeries: dataToPost })
            });
            if (!res.ok) throw new Error(`Server responded ${res.status}`);
            const json = await res.json();
            predictions = json.predictions || json;
        } catch (err) {
            submitError = `Could not reach the scheduling service: ${err.message}`;
        } finally {
            submitting = false;
        }
    }
</script>

<style>

    
    .form-controls-vertical {
    display: flex;
    gap: 12px;
    margin-top: 10px;
    align-items: center; /* Optional: center the buttons horizontally */
}
    .plus-btn {
    background-color: #28a745;
    color: white;
    border-radius: 50%;
    width: 48px;
    height: 48px;
    font-size: 2rem;
    display: flex;
    align-items: center;
    justify-content: center;
    border: none;
    box-shadow: 0 2px 8px rgba(40,167,69,0.08);
    transition: background 0.2s, transform 0.2s;
}
.plus-btn:hover {
    background-color: #218838;
    transform: scale(1.08);
}
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
        opacity: 0;
        transform: translateY(20px);
        transition: opacity 0.8s ease, transform 0.8s ease;
        z-index: 1001;
    }

    .title.visible {
        opacity: 1;
        transform: translateY(0);
    }

    .title.fading-out {
        opacity: 0;
        transform: translateY(-20px);
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
        transition: width 0.6s ease, opacity 0.6s ease, transform 0.6s ease;
    }

    .full-bar.expand {
        width: 100%;
    }

    .bar-half {
        position: absolute;
        width: 100%;
        height: 50%;
        background: black;
        z-index: 1000;
        transform: translateY(0%);
        opacity: 0;
        transition: transform 1.5s ease, opacity 0.5s ease;
    }

    .bar-half.visible {
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
    /* REMOVE these lines if they are here: */
    /* display: flex; */
    /* flex-direction: column; */
    /* justify-content: center; */
    /* align-items: center; */
}

    .main.visible {
        opacity: 1;
        transform: translateY(0);
    }


    form {
        display: flex; /* Keep flex display */
        flex-direction: row; /* Make items arrange in a row */
        flex-wrap: wrap; /* Allow items to wrap to the next line */
        gap: 20px; /* Space between cards */
        padding: 20px;
        border: 1px solid #eee;
        border-radius: 8px;
        background-color: #f9f9f9;
        max-width: 100%; /* Allow form to take full width of its parent */
        margin: 20px auto; /* Center the form if its content is less than 100% */
        box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        justify-content: center; /* Center cards horizontally in the available space */
    }

    .surgery-card {
        border: 1px solid #ddd;
        padding: 15px;
        border-radius: 6px;
        background-color: white;
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); /* Slightly adjusted min-width for more cards per row */
        gap: 10px; /* Reduced gap within cards for tighter layout */
        align-items: center;
        box-shadow: 0 1px 5px rgba(0,0,0,0.05);
        flex: 0 0 calc(33.333% - 20px); /* Allow 3 cards per row, accounting for gap */
        max-width: calc(33.333% - 20px); /* Max width for 3 cards per row */
        min-width: 280px; /* Ensure cards don't get too small on smaller screens */
    }

    .form-group {
        display: flex;
        flex-direction: column;
    }

    label {
        font-size: 0.9em;
        color: #555;
        margin-bottom: 5px;
    }

    input[type="time"],
    select {
        padding: 10px;
        border: 1px solid #ccc;
        border-radius: 4px;
        font-size: 1em;
        width: 100%;
        box-sizing: border-box;
    }

    button {
        padding: 10px 15px;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        font-size: 1em;
        transition: background-color 0.2s ease;
    }

button[type="button"] {
	background-color: #6c757d;
	color: white;
    border-radius: 16px;
    transition: 400ms ease-in-out;
}

button[type="button"]:hover {
	background-color: #5a6268;
    border-radius: 4px;
    transition: 400ms ease-in-out;

}

button[type="submit"] {
	background-color: #007bff;
	color: white;
	margin-top: 1rem;
    border-radius: 40px;
    transition: 400ms ease-in-out;
}

button[type="submit"]:hover {
	background-color: #0056b3;
    border-radius: 4px;
    transition: 400ms;

}

    .remove-button-container {
        display: flex;
        justify-content: flex-end;
        align-items: center;
        min-height: 40px;
        grid-column: 1 / -1; /* Make the remove button span all columns within the card's grid */
        margin-top: 5px; /* Add a little space above the remove button */
    }

    .remove-button {
        background-color: #dc3545;
        color: white;
        padding: 8px 12px;
        font-size: 0.9em;
    }

    .remove-button:hover {
        background-color: #c82333;
    }

    .form-controls {
        display: flex;
        gap: 10px;
        margin-top: 10px;
        width: 100%; /* Ensure buttons span full width */
        justify-content: center; /* Center the add/post buttons */
    }
    
</style>

{#if phase < 4}
    <div class="welcome">
        <div class="title {titleVisible ? 'visible' : 'fading-out'}">Surgical Dashboard</div>
        {#if phase === 0 || phase === 1}
            <div class="full-bar {expandBar ? 'expand' : ''}"></div>
        {/if}

        {#if phase >= 2}
            <div class="bar-half top-bar {phase >= 3 ? 'wipe' : ''} {phase >= 2 ? 'visible' : ''}"></div>
            <div class="bar-half bottom-bar {phase >= 3 ? 'wipe' : ''} {phase >= 2 ? 'visible' : ''}"></div>
        {/if}
    </div>
{/if}

<div class="main {phase === 4 ? 'visible' : ''}">
    <form on:submit|preventDefault={postSurgeries}>
        {#each surgeries as s (s.id)}
            <div class="surgery-card">
                <div class="form-group">
                    <label for="predictedStart-{s.id}">Predicted Start</label>
                    <input type="time" id="predictedStart-{s.id}" bind:value={s.predictedStart} />
                </div>

                <div class="form-group">
                    <label for="surgeon-{s.id}">Surgeon</label>
                    <select id="surgeon-{s.id}" bind:value={s.surgeon}>
                        {#each surgeons as surgeonOption}
                            <option value={surgeonOption} disabled={surgeonOption === 'Select Surgeon'}>{surgeonOption}</option>
                        {/each}
                    </select>
                </div>

                <div class="form-group">
                    <label for="procedure-{s.id}">Procedure</label>
                    <select id="procedure-{s.id}" bind:value={s.procedure}>
                        {#each procedures as procedureOption}
                            <option value={procedureOption} disabled={procedureOption === 'Select Procedure'}>{procedureOption}</option>
                        {/each}
                    </select>
                </div>

                <div class="form-group">
                    <label for="diagnosis-{s.id}">Diagnosis</label>
                    <select id="diagnosis-{s.id}" bind:value={s.diagnosis}>
                        {#each diagnoses as diagnosisOption}
                            <option value={diagnosisOption} disabled={diagnosisOption === 'Select Diagnosis'}>{diagnosisOption}</option>
                        {/each}
                    </select>
                </div>

                <div class="remove-button-container">
                    <button type="button" class="remove-button" on:click={() => removeSurgeryForm(s.id)}>Remove</button>
                </div>
            </div>
        {/each}

        <div class="form-controls-vertical">
    <button type="button" class="plus-btn" on:click={addSurgeryForm} aria-label="Add Surgery">
        <span aria-hidden="true">+</span>
    </button>
</div>
    <div style="display: flex; gap: 10px; margin-top: 10px; justify-content: center;">
            <button type="submit" disabled={submitting}>{submitting ? 'Predicting…' : 'Run Simulation'}</button>
        </div>

    {#if submitError}
        <p style="color:#dc3545; margin-top:10px; text-align:center;">{submitError}</p>
    {/if}

    {#if predictions}
        <div style="margin-top:16px;">
            <h3 style="text-align:center;">Predicted Durations</h3>
            <ul style="list-style:none; padding:0; max-width:420px; margin:0 auto;">
                {#each predictions as p}
                    <li style="display:flex; justify-content:space-between; padding:8px 12px; border-bottom:1px solid #eee;">
                        <span>{p.procedure} — {p.surgeon}</span>
                        <strong>{p.predictedMinutes} min</strong>
                    </li>
                {/each}
            </ul>
        </div>
    {/if}
</div>