<script>
    import Chart from "chart.js/auto";
    import { onMount, onDestroy, afterUpdate } from "svelte";

    let chart;
    let parsedData = [];
    let maxReps;
    let minReps;

    export let exerciseData;
    export let exerciseName;

    function scaleRadius(reps) {
        const minRadius = 5;
        const maxRadius = 15;
        const maxRepsRange = 15; // Adjust this value as needed

        // Ensure that reps are within a reasonable range for scaling
        reps = Math.max(minReps, Math.min(maxRepsRange, reps));

        // Scale reps to a value between minRadius and maxRadius
        return (
            minRadius +
            ((reps - minReps) / (maxRepsRange - minReps)) *
                (maxRadius - minRadius)
        );
    }

    function createChart() {
        const ctx = document
            .getElementById(`chart-${exerciseName}`)
            .getContext("2d");

        chart = new Chart(ctx, {
            type: "scatter",
            data: {
                datasets: [
                    {
                        label: exerciseName,
                        data: parsedData,
                        borderColor: "rgb(2, 48, 71)",
                        backgroundColor: "rgba(33, 158, 188, 0.5)",
                        pointRadius: parsedData.map((dataPoint) =>
                            scaleRadius(dataPoint.r)
                        ),
                    },
                ],
            },
            options: {
                scales: {
                    x: {
                        type: "category",
                        position: "bottom",
                    },
                    y: {
                        title: {
                            display: true,
                            text: "Weight",
                        },
                    },
                },
                maintainAspectRatio: false,
                // responsive: false,
            },
        });
    }

    function recreateChart() {
        if (chart) {
            chart.destroy();
        }

        parsedData = exerciseData.map((item) => ({
            x: item.Date,
            y: item.Weight,
            r: item.Reps,
        }));

        maxReps = Math.max(...exerciseData.map((item) => item.Reps));
        minReps = Math.min(...exerciseData.map((item) => item.Reps));

        parsedData.forEach((dataPoint) => {
            dataPoint.r = scaleRadius(dataPoint.r);
        });

        createChart();
    }

    onMount(() => {
        createChart();
    });

    afterUpdate(() => {
        recreateChart();
    });

    onDestroy(() => {
        if (chart) {
            chart.destroy();
        }
    });
</script>

<div class="table-responsive">
    <canvas
        style="min-height: fill-available;"
        id={`chart-${exerciseName}`}
        class="table"
    />
</div>
