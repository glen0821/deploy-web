<script>
  import { db } from "../../db/firebase";
  import { onSnapshot, collection, query, orderBy } from "firebase/firestore";
  import { onMount } from "svelte";
  import { writable } from "svelte/store";
  import { fly } from "svelte/transition";
  import { Bar } from "svelte-chartjs";
  import {
    Chart as ChartJS,
    Tooltip,
    LineElement,
    LinearScale,
    PointElement,
    CategoryScale,
    Filler,
    BarElement,
  } from "chart.js";

  const counters = writable({
    counter1: 0,
    counter2: 0,
    counter3: 0,
    counter4: 0,
    counter5: 0,
    counter6: 0
  });

  const collections = [
    { ref: "votersList", field: null },
    { ref: "barangayID", field: "createdAt" },
    { ref: "barangayCertificate", field: "createdAt" },
    { ref: "barangayClearance", field: null },
    { ref: "Complaints", field: null },
    { ref: "barangayIndigency", field: null }
  ];

  collections.forEach((collectionItem, index) => {
    const colRef = collection(db, collectionItem.ref);
    const queryRef = collectionItem.field ? query(colRef, orderBy(collectionItem.field, "desc")) : colRef;
    onSnapshot(queryRef, (snapshots) => {
      counters.update(c => {
        c[`counter${index + 1}`] = snapshots.size;
        return c;
      });
    });
  });

  ChartJS.register(
    Tooltip,
    LineElement,
    LinearScale,
    PointElement,
    CategoryScale,
    BarElement,
    Filler,
  );
  
  const overViewData = writable({
    labels: [],
    datasets: [
      {
        data: [],
        backgroundColor: [],
        borderColor: [],
        borderWidth: 1,
      },
    ],
  });

  function updateOverviewData(snapshots, field = "purpose") {
    let purposesCount = {};

    snapshots.forEach((doc) => {
      const purpose = doc.data()[field];
      if (purpose) {
        purposesCount[purpose] = (purposesCount[purpose] || 0) + 1;
      }
    });

    const labels = Object.keys(purposesCount);
    const data = Object.values(purposesCount);
    const backgroundColor = [];
    const borderColor = [];

    const maxDataValue = Math.max(...data);
    labels.forEach((label, index) => {
      const intensity = data[index] / maxDataValue;
      const hue = Math.floor(Math.random() * 360); // Random hue for rainbow colors
      const vibrantColor = `hsla(${hue}, 100%, 50%, ${intensity})`;
      backgroundColor.push(vibrantColor);
      borderColor.push(vibrantColor.replace(`${intensity}`, "1"));
    });

    const overviewData = {
      labels,
      datasets: [
        {
          label: "Number of Requests",
          data,
          backgroundColor,
          borderColor,
          borderWidth: 1,
        },
      ],
    };

    overViewData.set(overviewData);
  }

  function getAnalytics(colRef, field) {
    onSnapshot(colRef, (snapshot) => {
      updateOverviewData(snapshot, field);
    });
  }
  
  let chartData = {
    labels: [],
    datasets: [
      {
        data: [],
        backgroundColor: [],
        borderColor: [],
        borderWidth: 1,
      },
    ],
  };

  const unsubscribe = overViewData.subscribe((value) => {
    chartData = value;
  });

  onMount(() => {
    return () => {
      unsubscribe();
    };
  });
  getAnalytics(collection(db, "barangayCertificate"), "purpose");
</script>

<div
  class="flex w-full bg-gray-100 min-h-screen content-center flex-col flex-wrap"
>
  <div class="w-5/6 gap-10 mt-1 flex p-4 overflow-y-auto">
    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{$counters.counter1}</span>
            <span class="text-sm">Total app users</span>
          </div>
          <span class="text-6xl">
            <i class="fi fi-rr-vote-yea"></i>
          </span>
        </div>
      </div>
    </div>

    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm relative">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{$counters.counter2}</span>
            <span class="text-sm">Total I.D </span>
          </div>
          <span class="text-6xl">
            <i class="fi fi-rr-id-badge"></i>
          </span>
        </div>
        <button
          on:click={() => {
            getAnalytics(collection(db, "barangayID"), "gender");
          }}
          class="cursor-pointer absolute bg-yellow-400 w-full bottom-0 rounded-bl-lg rounded-br-lg p-1 flex justify-center content-center gap-2 flex-wrap"
        >
          More info
          <i class="fi fi-rr-arrow-circle-right"></i>
        </button>
      </div>
    </div>

    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{$counters.counter3}</span>
            <span class="text-sm">Total certificate </span>
          </div>
          <span class="text-6xl">
            <i class="fi fi-rr-memo"></i>
          </span>
        </div>
        <div
          on:click={() => {
            getAnalytics(collection(db, "barangayCertificate"), "purpose");
          }}
          on:keydown={(event) => {
            if (event.key === "Enter") {
              getAnalytics(collection(db, "barangayCertificate"), "purpose");
            }
          }}
          class="cursor-pointer absolute bg-yellow-400 w-full bottom-0 rounded-bl-lg rounded-br-lg p-1 flex justify-center content-center gap-2 flex-wrap"
        >
          More info
          <i class="fi fi-rr-arrow-circle-right"></i>
        </div>
      </div>
    </div>

    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{$counters.counter4}</span>
            <span class="text-sm">Total clearance </span>
          </div>
          <span class="text-6xl">
            <i class="fi fi-rr-memo"></i>
          </span>
        </div>
        <div
          on:click={() => {
            getAnalytics(collection(db, "barangayClearance"), "purpose");
          }}
          on:keydown={(event) => {
            if (event.key === "Enter") {
              getAnalytics(collection(db, "barangayClearance"), "purpose");
            }
          }}
          class="cursor-pointer absolute bg-yellow-400 w-full bottom-0 rounded-bl-lg rounded-br-lg p-1 flex justify-center content-center gap-2 flex-wrap"
        >
          More info
          <i class="fi fi-rr-arrow-circle-right"></i>
        </div>
      </div>
    </div>

    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{$counters.counter6}</span>
            <span class="text-sm">total Indigency </span>
          </div>
          <span class="text-6xl">
            <i class="fi fi-rr-memo"></i>
          </span>
        </div>
        <div
          on:click={() => {
            getAnalytics(collection(db, "barangayIndigency"), "purpose");
          }}
          on:keydown={(event) => {
            if (event.key === "Enter") {
              getAnalytics(collection(db, "barangayIndigency"), "purpose");
            }
          }}
          class="cursor-pointer absolute bg-yellow-400 w-full bottom-0 rounded-bl-lg rounded-br-lg p-1 flex justify-center content-center gap-2 flex-wrap"
        >
          More info
          <i class="fi fi-rr-arrow-circle-right"></i>
        </div>
      </div>
    </div>

    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{$counters.counter5}</span>
            <span class="text-sm">total complaint </span>
          </div>
          <span class="text-6xl">
            <i class="fi fi-rr-memo-pad"></i>
          </span>
        </div>
      </div>
    </div>
  </div>
  <div class="mt-4 mb-10">
    <div class="w-full rounded-lg flex justify-center" style="height: 450px;">
      <Bar class="" data={chartData} />
    </div>
  </div>
</div>
