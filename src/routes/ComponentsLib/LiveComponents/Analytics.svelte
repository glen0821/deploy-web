<script>
  //database callse and hooks
  import { db } from "../../db/firebase";
  import { onSnapshot, collection, query, orderBy } from "firebase/firestore";
  import { onSnaps } from "../BoundComponents/clickOutside";
  import { Line } from "svelte-chartjs";
  import { dataset } from "./daily_data.js";
  import {
    Chart as ChartJS,
    Tooltip,
    LineElement,
    LinearScale,
    PointElement,
    CategoryScale,
    Filler,
  } from "chart.js";
  ChartJS.register(
    Tooltip,
    LineElement,
    LinearScale,
    PointElement,
    CategoryScale,
    Filler,
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
    // { id: 0, label: "Overview" },
    { id: 1, label: "Daily" },
    { id: 2, label: "Monthly" },
    { id: 3, label: "Yearly" },
  ];

  let activeTab = tabs[2];

  const getValue = (myWritable) => {
    let currentValue;
    myWritable.subscribe((value) => {
      currentValue = value;
    })();
    return currentValue;
  };

  const fetchAnalyticsData = async () => {
    const analyticsCollectionRef = collection(db, "analytics");
    let analyticsQuery = query(
      analyticsCollectionRef,
      orderBy("timestamp", "desc"),
    );
    onSnapshot(analyticsQuery, (snapshot) => {
      let analyticsData = snapshot.docs.map((doc) => ({
        ...doc.data(),
        id: doc.id,
      }));
      onSnaps.set(analyticsData);
      const dateParts = selectedDate.split("-");
      const categorizedData = getCategorizedAnalytics(dateParts, analyticsData);
      dataset.set(categorizedData);
      updateChart();
    });
  };

  fetchAnalyticsData();

  const getYearAnalytics = (year, analytics) => {
    const months = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
    const dataset = {
      certificate: [...months],
      clearance: [...months],
      indigency: [...months],
      complaint: [...months],
      voter: [...months],
      id: [...months],
    };
    for (let j = 0; j < analytics.length; j++) {
      let data = analytics[j];
      if (data.year == year) {
        dataset[data.type][data.month - 1]++;
      }
    }
    return dataset;
  };

  const getMonthAnalytics = (year, month, analytics) => {
    // Get the number of days in the specified month
    const daysInMonth = new Date(year, month, 0).getDate();

    // Initialize an array to store the count of occurrences for each date
    const dates = Array(daysInMonth).fill(0);
    const dataset = {
      certificate: [...dates],
      clearance: [...dates],
      indigency: [...dates],
      complaint: [...dates],
      voter: [...dates],
      id: [...dates],
    };
    for (let j = 0; j < analytics.length; j++) {
      let data = analytics[j];
      if (data.year == year && data.month == parseInt(month)) {
        dataset[data.type][data.day - 1]++;
      }
    }

    return dataset;
  };

  const getDayAnalytics = (year, month, day, analytics) => {
    const hours = Array(24).fill(0);
    const dataset = {
      certificate: [...hours],
      clearance: [...hours],
      indigency: [...hours],
      complaint: [...hours],
      voter: [...hours],
      id: [...hours],
    };
    for (let j = 0; j < analytics.length; j++) {
      let data = analytics[j];
      if (
        data.year == year &&
        data.month == parseInt(month) &&
        data.day == day
      ) {
        dataset[data.type][data.hour - 1]++;
      }
    }
    return dataset;
  };

  function handleDateChange(event) {
    selectedDate = event.target.value;
    const date = selectedDate.split("-");
    let analytics;
    onSnaps.subscribe((data) => {
      analytics = data;
    });
    const dtst = getCategorizedAnalytics(date, analytics);
    dataset.set(dtst);
    updateChart();
  }

  function handleTabChange() {
    const currentDateParts = getCurrentDate().split("-");
    let newDate = currentDateParts[0]; // Default to year

    if (activeTab.id == 1) {
      newDate = currentDateParts.join("-");
    } else if (activeTab.id == 2) {
      newDate = `${currentDateParts[0]}-${currentDateParts[1]}`;
    }

    selectedDate = newDate;
    const date = selectedDate.split("-");
    let analytics;
    onSnaps.subscribe((data) => {
      analytics = data;
    });
    const dtst = getCategorizedAnalytics(date, analytics);
    dataset.set(dtst);
    updateChart();
  }

  const getCategorizedAnalytics = (date, analytics) => {
    switch (activeTab.id) {
      case 1:
        return getDayAnalytics(date[0], date[1], date[2], analytics);
      case 2:
        return getMonthAnalytics(date[0], date[1], analytics);
      default:
        return getYearAnalytics(date[0], analytics);
    }
  };

  let chartDataset = {};

  function updateChart() {
    const categories = [
      "certificate",
      "voter",
      "clearance",
      "indigency",
      "complaint",
      "id",
    ];

    const getLabels = () => {
      switch (activeTab.id) {
        case 1:
          return [
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
          ];
        case 2:
          const date = selectedDate.split("-");
          const daysInMonth = new Date(date[0], date[1], 0).getDate();
          return Array.from({ length: daysInMonth }, (_, i) =>
            (i + 1).toString(),
          );
        case 3:
          return [
            "January",
            "February",
            "March",
            "April",
            "May",
            "June",
            "July",
            "August",
            "September",
            "October",
            "November",
            "December",
          ];
        default:
          return [];
      }
    };

    const labels = getLabels();

    categories.forEach((category) => {
      chartDataset[category] = {
        labels,
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
            pointBorderColor: "rgb(205, 130, 158)",
            pointBackgroundColor: "rgb(255, 255, 255)",
            pointBorderWidth: 2,
            pointHoverRadius: 5,
            pointHoverBackgroundColor: "rgb(0, 0, 0)",
            pointHoverBorderColor: "rgba(220, 220, 220,1)",
            pointHoverBorderWidth: 2,
            pointRadius: 4,
            pointHitRadius: 1,
            data: getValue(dataset)[category],
          },
        ],
      };
    });
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
          handleTabChange();
        }}
        on:keydown={(event) => {
          if (event.key === "Enter") {
            activeTab = tab;
            handleTabChange();
          }
        }}
      >
        {tab.label}
      </div>
    {/each}
  </div>

  <div class="mt-4">
    {#if activeTab.id === 1}
      <div class="flex justify-center items-center">
        <p>Date:</p>
        <input
          type="date"
          bind:value={selectedDate}
          on:change={handleDateChange}
          class="h-10 bg-slate-100 p-2 focus:outline-none"
        />
      </div>
      <div class="w-full p-4 justify-center gap-5 grid grid-cols-2">
        {#each ["voter", "id", "certificate", "clearance", "indigency", "complaint"] as category}
          <div class="p-10 border border-solid w-full border-blue-500 rounded-lg">
            <h2 class="font-bold text-xl text-red-700">{category.charAt(0).toUpperCase() + category.slice(1)}</h2>
            {#if chartDataset[category] != undefined}
              <Line class="ml-40 mr-10" data={chartDataset[category]} />
            {/if}
          </div>
        {/each}
      </div>
    {/if}
    {#if activeTab.id === 2}
      <div class="flex justify-center items-center">
        <p>Date:</p>
        <input
          type="month"
          bind:value={selectedDate}
          on:change={handleDateChange}
          class="h-10 bg-slate-100 p-2 focus:outline-none"
        />
      </div>
      <div class="w-full p-4 justify-center gap-5 grid grid-cols-2">
        {#each ["voter", "id", "certificate", "clearance", "indigency", "complaint"] as category}
          <div class="p-10 border border-solid w-full border-blue-500 rounded-lg">
            <h2 class="font-bold text-xl text-red-700">{category.charAt(0).toUpperCase() + category.slice(1)}</h2>
            {#if chartDataset[category] != undefined}
              <Line class="ml-40 mr-10" data={chartDataset[category]} />
            {/if}
          </div>
        {/each}
      </div>
    {/if}
    {#if activeTab.id === 3}
      <div class="flex justify-center items-center">
        <p>Date:</p>
        <input
          type="number"
          id="year"
          name="year"
          min="2023"
          max="2036"
          placeholder="YYYY"
          bind:value={selectedDate}
          on:change={handleDateChange}
          class="h-10 bg-slate-100 p-2 focus:outline-none"
        />
      </div>
      <div class="w-full p-4 justify-center gap-5 grid grid-cols-2">
        {#each ["voter", "id", "certificate", "clearance", "indigency", "complaint"] as category}
          <div class="p-10 border border-solid w-full border-blue-500 rounded-lg">
            <h2 class="font-bold text-xl text-red-700">{category.charAt(0).toUpperCase() + category.slice(1).replace(/_/g, ' ')}</h2>
            {#if chartDataset[category] != undefined}
              <Line class="ml-40 mr-10" data={chartDataset[category]} />
            {/if}
          </div>
        {/each}
      </div>
    {/if}
  </div>
</div>
