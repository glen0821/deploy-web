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
</script>

<div class="flex w-full bg-gray-100 min-h-screen justify-center">
  <div class="w-5/6 h-32 grid grid-cols-3 grid-flow-row gap-5 mt-10">
    <!--Regitered voters-->
    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm ">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{counter}</span>
            <span class="text-sm">Total app users</span>
          </div>
          <span
          class="text-6xl">
            <i class="fi fi-rr-vote-yea"></i>
          </span>
        </div>
      </div>
    </div>

    <!--TOTAL ID REQUEST-->
    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm ">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{counter2}</span>
            <span class="text-sm">Total I.D request </span>
        </div>
          <span
          class="text-6xl">
            <i class="fi fi-rr-id-badge"></i>
          </span>
        </div>
      </div>
    </div>

    

    <!--TOTAL CERTIFICATE REQUEST-->
    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm ">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{counter3}</span>
            <span class="text-sm">Total certificate </span>
        </div>
          <span
          class="text-6xl">
            <i class="fi fi-rr-memo"></i>
          </span>
        </div>
      </div>
    </div>

    <!--TOTAL CLEARANCE REQUEST-->
    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm ">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{counter4}</span>
            <span class="text-sm">Total clearance </span>
        </div>
          <span
          class="text-6xl">
            <i class="fi fi-rr-memo"></i>
          </span>
        </div>
      </div>
    </div>
    

    <!--TOTAL INDIGENCY REQUEST-->
    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm ">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{counter6}</span>
            <span class="text-sm">total Indigency </span>
        </div>
          <span
          class="text-6xl">
            <i class="fi fi-rr-memo"></i>
          </span>
        </div>
      </div>
    </div>
    

    <!--TOTAL OF COMPLAINT-->
    <div in:fly={{ y: 300, duration: 1000 }}>
      <div class="max- bg-white rounded-lg drop-shadow-sm ">
        <div
          class="text-6xl font-bold p-14 text-black capitalize hover:scale-105 duration-700 flex items-center gap-5 flex justify-center"
        >
          <div class="flex flex-col justify-center content-center">
            <span class="text-center">{counter5}</span>
            <span class="text-sm">total complaint </span>
        </div>
          <span
          class="text-6xl">
            <i class="fi fi-rr-memo-pad"></i>
          </span>
        </div>
      </div>
    </div>
  </div>
</div>
