<script>
  //database callse and hooks
  import { auth, db } from "../../db/firebase";
  import {
    onSnapshot,
    collection,
    doc,
    getDoc,
    getDocs,
    query,
    orderBy,
  } from "firebase/firestore";
  import { onMount } from "svelte";
  import { writable } from "svelte/store";
  import { fly } from "svelte/transition";

  //listofvoters count
  let counter = 0;
  const colRef = collection(db, "votersList");

  // $: counterDaTa = getTotalCountDataCollections();

  //   const getTotalCountDataCollections = async () => {
  //     const data =  await getDocs(colRef)

  //     counter = data.docs.length

  //   }
  onSnapshot(colRef, (snapshots) => {
    let counterConvert = 0;
    snapshots.docs.forEach((doc) => {
      // let fbStoreCount = Number(doc.data().voterCounter);
      counterConvert += 1;
    });
    counter = counterConvert;
  });

  //barangay requst id count
  let counter2 = 0;
  const colRef2 = collection(db, "barangayID");
  const q2 = query(colRef2, orderBy("createdAt", "desc"));
  onSnapshot(q2, (snapshots) => {
    let counterConvert = 0;
    snapshots.docs.forEach((doc) => {
      // let fbStoreCount = Number(doc.data().barangayCounter);
      counterConvert += 1;
    });
    counter2 = counterConvert;
  });

  //barangay certificate count
  let counter3 = 0;
  const colRef3 = collection(db, "barangayCertificate");
  const q3 = query(colRef3, orderBy("createdAt", "desc"));
  onSnapshot(q3, (snapshots) => {
    let counterConvert = 0;
    snapshots.docs.forEach((doc) => {
      // let fbStoreCount = Number(doc.data().bgyCertificateCounter);

      const data = { ...doc.data() };

      counterConvert += 1;
    });
    counter3 = counterConvert;
  });

  //barangay clearance count
  let counter4 = 0;
  const colRef4 = collection(db, "barangayClearance");
  onSnapshot(colRef4, (snapshots) => {
    let counterConvert = 0;
    snapshots.docs.forEach((doc) => {
      let fbStoreCount = Number(doc.data().bgyClearanceCounter);
      counterConvert += fbStoreCount;
    });
    counter4 = counterConvert;
  });

  //complaint count
  let counter5 = 0;
  const colRef5 = collection(db, "complaints");
  onSnapshot(colRef5, (snapshots) => {
    let counterConvert = 0;
    snapshots.docs.forEach((doc) => {
      let fbStoreCount = Number(doc.data().complaintCounter);
      counterConvert += fbStoreCount;
    });
    counter5 = counterConvert;
  });

  let counter6 = 0;
  const colRef6 = collection(db, "barangayIndigency");
  onSnapshot(colRef6, (snapshots) => {
    let counterConvert = 0;
    snapshots.docs.forEach((doc) => {
      counterConvert++;
    });
    console.log(counterConvert);
    counter6 = counterConvert;
  });

  const showAnalytics = writable(false);
  const selectedAnalytics = writable("ID");

  import { Line, Bar } from "svelte-chartjs";
  // import Bar from "svelte-chartjs/Bar.svelte";
  // import Bar from "svelte-chartjs";
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
  ChartJS.register(
    Tooltip,
    LineElement,
    LinearScale,
    PointElement,
    CategoryScale,
    BarElement,
    Filler
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
  function updateOverviewData(snapshots, field="purpose") {
    let counterConvert = 0;
  let purposesCount = {};

  const overviewData = {
    labels: [],
    datasets: [{
      label: "Number of Requests",
      data: [],
      backgroundColor: [],
      borderColor: [],
      borderWidth: 1
    }]
  };

  snapshots.forEach((doc) => {
    const purpose = doc.data()[field];
    if (purpose) {
      if (!purposesCount[purpose]) {
        purposesCount[purpose] = 1;
      } else {
        purposesCount[purpose]++;
      }
      counterConvert++;
    }
  });

  // Reset overviewData to empty arrays
  overviewData.labels = [];
  overviewData.datasets[0].data = [];
  overviewData.datasets[0].backgroundColor = [];
  overviewData.datasets[0].borderColor = [];

  // Populate overviewData with counted purposes
  for (const purpose in purposesCount) {
    overviewData.labels.push(purpose);
    overviewData.datasets[0].data.push(purposesCount[purpose]);
    // Random background and border colors for visualization
    const randomColor = `rgba(${Math.floor(Math.random() * 256)}, ${Math.floor(Math.random() * 256)}, ${Math.floor(Math.random() * 256)}, 0.2)`;
    overviewData.datasets[0].backgroundColor.push(randomColor);
    overviewData.datasets[0].borderColor.push(randomColor.replace("0.2", "1"));
  }

  console.log(overviewData);

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

  // Subscribe to changes in overViewData
  const unsubscribe = overViewData.subscribe(value => {
    chartData = value;
    console.log(chartData)
  });

  // Unsubscribe from the store on component destroy to prevent memory leaks
  onMount(() => {
    return () => {
      unsubscribe();
    };
  });
  getAnalytics(colRef3, "purpose");
</script>

<div
  class="flex w-full bg-gray-100 min-h-screen content-center flex-col flex-wrap"
>
  <div class="w-5/6 grid grid-cols-3 grid-flow-row gap-5 mt-10">
    <!--Regitered voters-->
    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{counter}</span>
            <span class="text-sm">Total app users</span>
          </div>
          <span class="text-6xl">
            <i class="fi fi-rr-vote-yea"></i>
          </span>
        </div>
      </div>
    </div>

    <!--TOTAL ID REQUEST-->
    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm relative">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{counter2}</span>
            <span class="text-sm">Total I.D request </span>
          </div>
          <span class="text-6xl">
            <i class="fi fi-rr-id-badge"></i>
          </span>
        </div>
        <div
          on:click={() => {
            getAnalytics(colRef2, "gender");
          }}
          class="cursor-pointer absolute bg-yellow-400 w-full bottom-0 rounded-bl-lg rounded-br-lg p-1 flex justify-center content-center gap-2 flex-wrap"
        >
          More info
          <i class="fi fi-rr-arrow-circle-right"></i>
        </div>
      </div>
    </div>

    <!--TOTAL CERTIFICATE REQUEST-->
    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{counter3}</span>
            <span class="text-sm">Total certificate </span>
          </div>
          <span class="text-6xl">
            <i class="fi fi-rr-memo"></i>
          </span>
        </div>
        <div
        on:click={() => {
          getAnalytics(colRef3, "purpose");
        }}
          class="cursor-pointer absolute bg-yellow-400 w-full bottom-0 rounded-bl-lg rounded-br-lg p-1 flex justify-center content-center gap-2 flex-wrap"
        >
          More info
          <i class="fi fi-rr-arrow-circle-right"></i>
        </div>
      </div>
    </div>

    <!--TOTAL CLEARANCE REQUEST-->
    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{counter4}</span>
            <span class="text-sm">Total clearance </span>
          </div>
          <span class="text-6xl">
            <i class="fi fi-rr-memo"></i>
          </span>
        </div>
        <div
        on:click={() => {
          getAnalytics(colRef4, "purpose");
        }}
          class="cursor-pointer absolute bg-yellow-400 w-full bottom-0 rounded-bl-lg rounded-br-lg p-1 flex justify-center content-center gap-2 flex-wrap"
        >
          More info
          <i class="fi fi-rr-arrow-circle-right"></i>
        </div>
      </div>
    </div>

    <!--TOTAL INDIGENCY REQUEST-->
    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{counter6}</span>
            <span class="text-sm">total Indigency </span>
          </div>
          <span class="text-6xl">
            <i class="fi fi-rr-memo"></i>
          </span>
        </div>
        <div
        on:click={() => {
          getAnalytics(colRef6, "purpose");
        }}
          class="cursor-pointer absolute bg-yellow-400 w-full bottom-0 rounded-bl-lg rounded-br-lg p-1 flex justify-center content-center gap-2 flex-wrap"
        >
          More info
          <i class="fi fi-rr-arrow-circle-right"></i>
        </div>
      </div>
    </div>

    <!--TOTAL OF COMPLAINT-->
    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{counter5}</span>
            <span class="text-sm">total complaint </span>
          </div>
          <span class="text-6xl">
            <i class="fi fi-rr-memo-pad"></i>
          </span>
        </div>
      </div>
    </div>
  </div>
  <div class="mt-10 mb-10">
    <div class="p-10 border border-solid w-full border-blue-500 rounded-lg">
      <Bar class="" data={chartData} />
    </div>
  </div>
</div>
