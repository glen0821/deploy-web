<script>
  //database callse and hooks
  import { auth, db } from "../../db/firebase";
  import Inputs from "../GeneralComponents/Inputs.svelte";
  import {
    onSnapshot,
    collection,
    doc,
    getDoc,
    getDocs,
    query,
    orderBy,
  } from "firebase/firestore";
  import {
    showAdd,
    onSnaps,
    compareValue,
    printing,
  } from "../BoundComponents/clickOutside";
  import { onMount } from "svelte";
  import { fly } from "svelte/transition";
  import { Line } from "svelte-chartjs";
  import { data as daily_data, points as daily_points } from "./daily_data.js";
  import {
    Chart as ChartJS,
    Tooltip,
    LineElement,
    LinearScale,
    PointElement,
    CategoryScale,
    Filler
  } from "chart.js";
  ChartJS.register(
    Tooltip,
    LineElement,
    LinearScale,
    PointElement,
    CategoryScale,
    Filler
  );
  let selectedDate = getCurrentDate();
  function getCurrentDate() {
    const today = new Date();
    const year = today.getFullYear();
    const month = String(today.getMonth() + 1).padStart(2, "0"); // Months are zero-based
    const day = String(today.getDate()).padStart(2, "0");
    return `${year}-${month}-${day}`;
  }
  let tabs = [
    { id: 1, label: "Daily" },
    { id: 2, label: "Monthly" },
    { id: 3, label: "Yearly" },
  ];
  let activeTab = tabs[0];
  const getValue = (myWritable) => {
    let currentValue;
    const unsubscribe = myWritable.subscribe((value) => {
      currentValue = value;
    });
    return currentValue;
  };
  const getAnalytics = async () => {
    const colRef = collection(db, "analytics");
    let q = query(colRef, orderBy("timestamp", "desc"));
    onSnapshot(q, (snapshots) => {
      let fbData = [];
      snapshots.docs.forEach((doc) => {
        let data = { ...doc.data(), id: doc.id };
        fbData = [data, ...fbData];
      });
      onSnaps.set(fbData);
      console.log(fbData.length);
      const date = selectedDate.split("-");
      const points = getDayAnalytics(date[0], date[1], date[2], fbData);
      console.log(points);
      daily_points.set(points);
      updateChart();
    });
  };

  getAnalytics();

  const getYearAnalytics = (year, analytics) => {
    const months = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
    for (let j = 0; j < analytics.length; j++) {
      let data = analytics[j];
      console.log(data.year == year);
      if (data.year == year) {
        months[data.month - 1]++;
      }
    }
    return months;
  };

  const getMonthAnalytics = (year, month, analytics) => {
    // Get the number of days in the specified month
    const daysInMonth = new Date(year, month, 0).getDate();

    // Initialize an array to store the count of occurrences for each date
    const dates = Array(daysInMonth).fill(0);
    for (let j = 0; j < analytics.length; j++) {
      let data = analytics[j];
      if (data.year === year && data.month === parseInt(month)) {
        dates[data.day]++;
      }
    }

    return dates;
  };

  const getDayAnalytics = (year, month, day, analytics) => {
    const hours = [
      0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
    ];
    for (let j = 0; j < analytics.length; j++) {
      let data = analytics[j];
      
      if (
        data.year == year &&
        data.month == parseInt(month) &&
        data.day == day
      ) {
        hours[data.hour - 1]++;
      }
    }
    return hours;
  };

  function handleDateChange(event) {
    selectedDate = event.target.value;
    if (activeTab.id === 1) {
      const date = selectedDate.split("-");
      let analytics;
      onSnaps.subscribe(data => {
        analytics = data
    })
      const points = getDayAnalytics(date[0], date[1], date[2], analytics);
      daily_points.set(points);
      updateChart();
    }
  }

  let chartData = {};

  function updateChart() {
    // Update the data prop with the new daily_points value
    chartData = {
      labels: [
        "1:00",
        "02:00",
        "03:00",
        "04:00",
        "05:00",
        "06:00",
        "07:00",
        "08:00",
        "09:00",
        "10:00",
        "11:00",
        "12:00",
        "13:00",
        "14:00",
        "15:00",
        "16:00",
        "17:00",
        "18:00",
        "19:00",
        "20:00",
        "21:00",
        "22:00",
        "23:00",
        "24:00",
      ],
      datasets: [
        {
          label: "",
          fill: true,
          lineTension: 0.3,
          backgroundColor: "rgba(225, 204,230, .3)",
          borderColor: "rgb(205, 130, 158)",
          borderCapStyle: "butt",
          borderDash: [],
          borderDashOffset: 0.0,
          borderJoinStyle: "miter",
          pointBorderColor: "rgb(205, 130,1 58)",
          pointBackgroundColor: "rgb(255, 255, 255)",
          pointBorderWidth: 10,
          pointHoverRadius: 5,
          pointHoverBackgroundColor: "rgb(0, 0, 0)",
          pointHoverBorderColor: "rgba(220, 220, 220,1)",
          pointHoverBorderWidth: 2,
          pointRadius: 1,
          pointHitRadius: 10,
          data: getValue(daily_points),
        },
      ],
    };
  }
</script>

<div class="flex w-full bg-gray-100 min-h-screen flex-col">
  <div class="flex h-min space-evenly justify-evenly w-full">
    {#each tabs as tab (tab.id)}
      <div
        class="cursor-pointer px-4 py-2 bg-white border border-gray-300 w-1/3 text-center"
        class:selected={activeTab === tab}
        on:click={() => {
          activeTab = tab;
          console.log(tab);
        }}
      >
        {tab.label}
      </div>
    {/each}
  </div>

  <div class="mt-4">
    {#if activeTab.id === 1}
      <div class="flex justify-center items-center">
        <p>Date</p>
        <input
          type="date"
          bind:value={selectedDate}
          on:change={handleDateChange}
          class="h-10 bg-slate-100 p-2 focus:outline-none"
        />
      </div>
      <div class="w-full p-4">
        <div class="p-10 border border-solid border-blue-500 rounded-lg">
          <h2 class="font-bold text-xl text-red-700">Voters</h2>
          {#if $daily_points.length > 1}
            <Line class="ml-40 mr-10" data={chartData} />
          {/if}
        </div>
      </div>
    {/if}
    {#if activeTab.id === 2}
      <div class="flex justify-center items-center">
        <p>Date</p>
        <input
          type="date"
          bind:value={selectedDate}
          class="h-10 bg-slate-100 p-2 focus:outline-none"
        />
      </div>
      <div class="w-full p-4">
        <div class="p-10 border border-solid border-blue-500 rounded-lg">
          <h2 class="font-bold text-xl text-red-700">Voters</h2>
          {#if $daily_points.length > 1}
            <Line
              class="ml-40 mr-10"
              data={{
                labels: [
                  "1",
                  "2",
                  "3",
                  "4",
                  "5",
                  "6",
                  "7",
                  "8",
                  "9",
                  "10",
                  "11",
                  "12",
                  "13",
                  "14",
                  "15",
                  "16",
                  "17",
                  "18",
                  "19",
                  "20",
                  "21",
                  "22",
                  "23",
                  "24",
                  "25",
                  "26",
                  "27",
                  "28",
                  "29",
                  "30",
                  "31",
                ],
                datasets: [
                  {
                    label: "",
                    fill: true,
                    lineTension: 0.3,
                    backgroundColor: "rgba(225, 204,230, .3)",
                    borderColor: "rgb(205, 130, 158)",
                    borderCapStyle: "butt",
                    borderDash: [],
                    borderDashOffset: 0.0,
                    borderJoinStyle: "miter",
                    pointBorderColor: "rgb(205, 130,1 58)",
                    pointBackgroundColor: "rgb(255, 255, 255)",
                    pointBorderWidth: 10,
                    pointHoverRadius: 5,
                    pointHoverBackgroundColor: "rgb(0, 0, 0)",
                    pointHoverBorderColor: "rgba(220, 220, 220,1)",
                    pointHoverBorderWidth: 2,
                    pointRadius: 1,
                    pointHitRadius: 10,
                    data: getValue(daily_points),
                  },
                ],
              }}
            />
          {/if}
        </div>
      </div>
    {/if}
    {#if activeTab.id === 3}
      <div>Yearly</div>
    {/if}
  </div>
</div>
