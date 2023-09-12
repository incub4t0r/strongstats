<script>
    import Chart from "chart.js/auto";
    import { onMount, onDestroy, afterUpdate } from "svelte";
    import 'chartjs-adapter-moment'; // Import the Moment.js adapter for Chart.js
    import moment from "moment"; // Import the Moment.js library

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

    function bestFitLineFunction(x) {
        // Calculate the best-fit line (linear regression)
        const n = parsedData.length;
        let sumX = 0;
        let sumY = 0;
        let sumXY = 0;
        let sumX2 = 0;

        for (let i = 0; i < n; i++) {
            const dataPoint = parsedData[i];
            const dataX = dataPoint.x;
            const dataY = dataPoint.y;

            sumX += dataX;
            sumY += dataY;
            sumXY += dataX * dataY;
            sumX2 += dataX * dataX;
        }

        const meanX = sumX / n;
        const meanY = sumY / n;

        const slope = (sumXY - n * meanX * meanY) / (sumX2 - n * meanX * meanX);
        const intercept = meanY - slope * meanX;

        return slope * x + intercept;
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
                    {
                        label: "Best Fit Line",
                        data: parsedData.map((dataPoint) => ({
                            x: dataPoint.x,
                            y: bestFitLineFunction(dataPoint.x),
                        })),
                        borderColor: "rgba(255, 0, 0, 0.7)",
                        fill: false,
                    },
                    {
                        label: "Best Fit Line (Connect)",
                        data: parsedData.map((dataPoint) => ({
                            x: dataPoint.x,
                            y: bestFitLineFunction(dataPoint.x),
                        })),
                        type: "line",
                        borderColor: "rgba(255, 0, 0, 0.7)", // Adjust the line color
                        fill: false,
                        borderWidth: 1, // Adjust line width
                        pointHoverRadius: 0, // Turn off hover interactions
                        pointHitRadius: 0,   // Turn off tooltips
                    },
                ],
            },
            options: {
                scales: {
                    x: {
                        type: "time",
                        time: {
                            unit: "day",
                        },
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
                plugins: {
                    tooltip: {
                        enabled: true,
                        filter: function (tooltipItem) {
                            return tooltipItem.datasetIndex === 0; // Only show tooltips for the first dataset
                        }
                    },
                    hover: {
                        mode: 'index',
                        intersect: false,
                        onHover: function(event, chartElement) {
                            if (chartElement[0]) {
                                const index = chartElement[0].datasetIndex;
                                return index === 0; // Only allow hover for the first dataset
                            }
                            return false;
                        }
                    }
                }
            },
        });
    }

    function recreateChart() {
        if (chart) {
            chart.destroy();
        }

        parsedData = exerciseData.map((item) => ({
            x: moment(item.Date), // Convert Date to a Moment.js object
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

<canvas
    id={`chart-${exerciseName}`}
/>
