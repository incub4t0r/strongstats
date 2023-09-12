<script>
    import ExerciseChart from "$lib/ExerciseChart.svelte";
    import papaparse from "papaparse";
    import { onMount } from "svelte";

    let files;
    let exerciseData = {};
    let selectedExercise = Object.keys(exerciseData)[0]; // Initialize with the first exercise

    function parseFile(event) {
        files = event.target.files;
        console.log(files[0]);
        papaparse.parse(files[0], {
            header: true,
            dynamicTyping: true,
            complete: function (result) {
                if (result.data) {
                    groupExerciseData(result.data);
                }
            },
        });
    }

    function groupExerciseData(data) {
        exerciseData = {};
        for (var i = 0; i < data.length; i++) {
            const exerciseName = data[i]["Exercise Name"];
            if (exerciseName) {
                if (!exerciseData[exerciseName]) {
                    exerciseData[exerciseName] = [];
                }
                data[i].Date = data[i].Date.split(" ")[0];
                data[i].Weight = parseInt(data[i].Weight);
                exerciseData[exerciseName].push(data[i]);
            }
        }
        console.log(exerciseData);
    }

    onMount(() => {
        // Initialize the selectedExercise with the first exercise name (if available)
        if (Object.keys(exerciseData).length > 0) {
            selectedExercise = Object.keys(exerciseData)[0];
        }
    });
</script>

<div class="container-lg my-4">
    <div class="card my-4">
        <div class="card-body">
            <p class="card-text">
                Upload a CSV file containing your exercise data exported from
                Strong.
            </p>
            <div class="input-group">
                <input
                    type="file"
                    bind:files
                    on:change={(event) => parseFile(event)}
                    accept=".csv"
                    class="form-control"
                    id="inputGroupFile02"
                />
            </div>
        </div>
    </div>

    {#if Object.keys(exerciseData).length > 0}
        <div class="card" id="chart-parent-card">
            <div class="card-body">
                <!-- Dropdown menu to select exercise -->
                <div class="dropdown">
                    <button
                        class="btn btn-secondary dropdown-toggle"
                        type="button"
                        id="exerciseDropdown"
                        data-bs-toggle="dropdown"
                        aria-expanded="false"
                    >
                        {selectedExercise || "Select progression graph to view"}
                    </button>
                    <ul
                        class="dropdown-menu"
                        aria-labelledby="exerciseDropdown"
                    >
                        {#each Object.keys(exerciseData).sort() as exerciseName (exerciseName)}
                            <li>
                                <a
                                    class="dropdown-item"
                                    href="#"
                                    on:click={() =>
                                        (selectedExercise = exerciseName)}
                                >
                                    {exerciseName}
                                </a>
                            </li>
                        {/each}
                    </ul>
                </div>
                <!-- <select bind:value={selectedExercise}>
                    {#each Object.keys(exerciseData).sort() as exerciseName}
                        <option value={exerciseName}>{exerciseName}</option>
                    {/each}
                </select> -->

                {#if exerciseData[selectedExercise]}
                    <div id="chart">
                        <ExerciseChart
                            exerciseData={exerciseData[selectedExercise]}
                            exerciseName={selectedExercise}
                        />
                    </div>
                {/if}
            </div>
        </div>
    {/if}
</div>
