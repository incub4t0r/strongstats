<script>
    import Chart from "chart.js/auto";
    import { onMount, onDestroy, afterUpdate } from "svelte";
    import "chartjs-adapter-moment"; // Import the Moment.js adapter for Chart.js
    import moment from "moment"; // Import the Moment.js library

    let chart;
    let parsedData = [];
    let maxReps;
    let minReps;
    let startDate;
    let endDate;

    export let exerciseData;
    export let exerciseName;

    function scaleRadius(reps) {
        const minRadius = 5;
        const maxRadius = 15;
        const maxRepsRange = 10; // Adjust this value as needed

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
                        pointHitRadius: 0, // Turn off tooltips
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
                    legend: false, // Hide legend
                    tooltip: {
                        enabled: true,
                        // callbacks: {
                        //     label: function(context) {
                        //         var label = context.dataset.label || '';

                        //         if (label) {
                        //             label += ': ';
                        //         }
                        //         if (context.parsed.y !== null) {
                        //             label += new Intl.NumberFormat('en-US', { maximumFractionDigits: 2 }).format(context.parsed.y);
                        //         }

                        //         // Add reps here
                        //         if (context.parsed.r !== null) {
                        //             label += ' reps: ' + context.parsed.r;
                        //         }

                        //         return label;
                        //     }
                        // },
                        filter: function (tooltipItem) {
                            return tooltipItem.datasetIndex === 0; // Only show tooltips for the first dataset
                        },
                    },
                    hover: {
                        mode: "index",
                        intersect: false,
                        onHover: function (event, chartElement) {
                            if (chartElement[0]) {
                                const index = chartElement[0].datasetIndex;
                                return index === 0; // Only allow hover for the first dataset
                            }
                            return false;
                        },
                    },
                },
            },
        });
    }

    let formattedStartDate;
    let formattedEndDate;

    function recreateChart() {
        if (chart) {
            chart.destroy();
        }

        maxReps = Math.max(...parsedData.map((dataPoint) => dataPoint.r));
        minReps = Math.min(...parsedData.map((dataPoint) => dataPoint.r));

        parsedData.forEach((dataPoint) => {
            dataPoint.r = scaleRadius(dataPoint.r);
        });

        startDate = moment(formattedStartDate, "YYYY-MM-DD");
        endDate = moment(formattedEndDate, "YYYY-MM-DD");

        createChart();
    }

    // Function to filter data based on the selected date range
    function filterData() {
        if (startDate && endDate) {
            parsedData = exerciseData
                .map((item) => ({
                    x: moment(item.Date), // Convert Date to a Moment.js object
                    y: item.Weight,
                    r: item.Reps,
                }))
                .filter((dataPoint) => {
                    const pointDate = dataPoint.x;
                    return (
                        pointDate.isSameOrAfter(startDate, "day") &&
                        pointDate.isSameOrBefore(endDate, "day")
                    );
                });

            maxReps = Math.max(...parsedData.map((item) => item.r));
            minReps = Math.min(...parsedData.map((item) => item.r));

            parsedData.forEach((dataPoint) => {
                dataPoint.r = scaleRadius(dataPoint.r);
            });

            recreateChart();
        }
    }

    function reset_data() {
        parsedData = exerciseData.map((item) => ({
            x: moment(item.Date),
            y: item.Weight,
            r: item.Reps,
        }));
        startDate = moment(parsedData[0].x);
        endDate = moment(parsedData[parsedData.length - 1].x);
        formattedStartDate = moment(startDate).format("YYYY-MM-DD");
        formattedEndDate = moment(endDate).format("YYYY-MM-DD");
    }

    let previousExerciseName;

    onMount(() => {
        previousExerciseName = exerciseName;
        reset_data();
        createChart();
    });

    afterUpdate(() => {
        if (previousExerciseName === exerciseName) {
            recreateChart();
        } else {
            previousExerciseName = exerciseName;
            reset_data();
            recreateChart();
        }
    });

    onDestroy(() => {
        if (chart) {
            chart.destroy();
        }
    });
</script>

<div class="row my-4 d-flex">
    <div class="col flex-shrink-1">
        <label for="start-date">Start Date:</label>
        <input
            type="date"
            id="start-date"
            bind:value={formattedStartDate}
            on:input={filterData}
        />
    </div>
    <div class="col w-100">
        <label for="end-date">End Date:</label>
        <input
            type="date"
            id="end-date"
            bind:value={formattedEndDate}
            on:input={filterData}
        />
    </div>
</div>

<canvas id={`chart-${exerciseName}`} />
