<!-- ExerciseDataParser.svelte -->
<script>
    import { onMount } from "svelte";
    import papaparse from "papaparse";
    
    export let files;
    export let exerciseData = {}; // Object to store grouped exercise data

    function parseFile() {
        if (files && files[0]) {
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
    }

    /**
     * @param {string | any[]} data
     */
    function groupExerciseData(data) {
        exerciseData = {}; // Clear previous data
        for (var i = 0; i < data.length; i++) {
            const exerciseName = data[i]["Exercise Name"];
            if (exerciseName) {
                // Check if exerciseName is not empty or undefined
                if (!exerciseData[exerciseName]) {
                    exerciseData[exerciseName] = [];
                }
                data[i].Date = data[i].Date.split(" ")[0];
                data[i].Weight = parseInt(data[i].Weight);
                exerciseData[exerciseName].push(data[i]);
            }
        }
    }

    onMount(parseFile);
</script>
