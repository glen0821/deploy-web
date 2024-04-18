<script>
  import Button from "../GeneralComponents/Button.svelte";
  import Inputs from "../GeneralComponents/Inputs.svelte";
  import { fly } from "svelte/transition";
  import {
    onSnapsBgyID,
    compareIDvalue,
    showAddModal,
    printing,
  } from "../BoundComponents/clickOutside";
  import { showPrintModel, formattedDate } from "./stateStore";
  import bgyClearance from "../Images/bgyClearance.jpg";

  //database calls and hooks
  import { auth, db } from "../../db/firebase";
  import {
    onSnapshot,
    addDoc,
    collection,
    serverTimestamp,
    increment,
    doc,
    deleteDoc,
    query,
    orderBy,
    setDoc,
    where,
  } from "firebase/firestore";

  import IdContent from "./IDContent.svelte";
  import reportHeader from "./images/header_report.png";
  import { onSnaps } from "../BoundComponents/clickOutside";
  import { writable } from "svelte/store";
  //handler to show add modal
  const toShowAddModal = () => {
    showAddModal.set(true);
  };

  //barangayID varStore
  const bgyVarStore = {
    status: "",
    completeName: "",
    firstName: "",
    middleInitial: "",
    lastName: "",
    suffixName: "",
    address: "",
    birthdate: "",
    gender: "",
    height: "",
    weight: "",
    dateOfAppointment: "",
    kwiri: "",
    trigger: false,
  };

  //submit data to database
  const submitData = async () => {
    const colRef = collection(db, "barangayID");
    addDoc(colRef, {
      createdAt: serverTimestamp(),
      completeName: bgyVarStore.completeName.BINDTHIS,
      address: bgyVarStore.address.BINDTHIS,
      birthdate: bgyVarStore.birthdate.BINDTHIS,
      gender: bgyVarStore.gender.BINDTHIS,
      height: bgyVarStore.height.BINDTHIS,
      weight: bgyVarStore.weight.BINDTHIS,
      dateOfAppointment: bgyVarStore.dateOfAppointment.BINDTHIS,
      barangayCounter: increment(1),
    }).then(() => {
      showAddModal.set(false);
    });
  };

  //fetch data from database
  const colRef = collection(db, "barangayID");
  const q = query(colRef, orderBy("createdAt", "desc"));
  onSnapshot(q, (snapshots) => {
    let fbData = [];
    snapshots.docs.forEach((doc) => {
      let data = { ...doc.data(), id: doc.id };
      fbData = [data, ...fbData];
    });
    onSnapsBgyID.set(fbData);
    console.log(fbData);
  });

  //removeData from database
  const removeData = async (data) => {
    const docRef = doc(colRef, data);
    await deleteDoc(docRef);
  };

  const updateStatus = async (userID, selectedStatus, ownerID) => {
    const docRef = doc(colRef, userID);

    const updatedData = {
      status: selectedStatus,
      lastUpdated: serverTimestamp(),
    };

    await setDoc(docRef, updatedData, { merge: true });

    const notifDocRef = doc(collection(db, "notifications"));
    let message;
    if (selectedStatus == "Processing") {
      message = "Your I.D. is being processed";
    } else if (selectedStatus == "Ready for pickup") {
      message =
        "Your I.D. is ready for pickup, get it before or on appointed date";
    } else {
      message = "Your I.D. has been claimed";
    }
    var currentDate = Date.now();
    const notificationData = {
      message: message,
      userID: ownerID,
      timestamp: currentDate,
    };
    console.log(notificationData);
    await setDoc(notifDocRef, notificationData);
  };

  
  import flatpickr from "flatpickr";
  import 'flatpickr/dist/flatpickr.css'
  const disableWeekends = (event) => {
    const element = event.target;
    if(element.getAttribute("picker_set") == "yes"){
      return
    }
    flatpickr(element, {
      "disable": [
        function(date) {
            return (date.getDay() === 0 || date.getDay() === 6);
        }
    ],
    });
    element.setAttribute("picker_set", "yes")
    
    
  };

  //updateData from database
  const updateData = async (data) => {
    const docRef = doc(colRef, data);
    console.log({
      lastUpdated: serverTimestamp(),
      firstName: bgyVarStore.firstName.BINDTHIS,
      middleInitial: bgyVarStore.middleInitial.BINDTHIS,
      lastName: bgyVarStore.lastName.BINDTHIS,
      suffixName: bgyVarStore.suffixName.BINDTHIS,
      address: bgyVarStore.address.BINDTHIS,
      birthdate: bgyVarStore.birthdate.BINDTHIS,
      gender: bgyVarStore.gender.BINDTHIS,
      height: bgyVarStore.height.BINDTHIS,
      weight: bgyVarStore.weight.BINDTHIS,
      dateOfAppointment: bgyVarStore.dateOfAppointment.BINDTHIS,
    });
    setDoc(
      docRef,
      {
        lastUpdated: serverTimestamp(),
        firstName: bgyVarStore.firstName.BINDTHIS,
        middleInitial: bgyVarStore.middleInitial.BINDTHIS,
        lastName: bgyVarStore.lastName.BINDTHIS,
        suffixName: bgyVarStore.suffixName.BINDTHIS,
        address: bgyVarStore.address.BINDTHIS,
        birthdate: bgyVarStore.birthdate.BINDTHIS,
        gender: bgyVarStore.gender.BINDTHIS,
        height: bgyVarStore.height.BINDTHIS,
        weight: bgyVarStore.weight.BINDTHIS,
        dateOfAppointment: bgyVarStore.dateOfAppointment.BINDTHIS,
      },
      { merge: true }
    );
  };

  function fixDateFormat(originalDate) {
    // Split the date string into year, month, and day
    var parts = originalDate.split("-");

    // Pad the month part with leading zero if it's a single digit
    parts[1] = parts[1].length === 1 ? "0" + parts[1] : parts[1];

    // Pad the day part with leading zero if it's a single digit
    parts[2] = parts[2].length === 1 ? "0" + parts[2] : parts[2];

    // Reconstruct the date string with the corrected format
    var fixedDate = parts.join("-");

    return fixedDate;
  }

  //showModalComparison
  const editValueHandler = (data, index) => {
    compareIDvalue.set(index);
    setTimeout(function () {
      bgyVarStore.firstName.BINDTHIS = data.firstName;
      bgyVarStore.middleInitial.BINDTHIS = data.middleInitial;
      bgyVarStore.lastName.BINDTHIS = data.lastName;
      bgyVarStore.suffixName.BINDTHIS = data.suffixName;
      bgyVarStore.address.BINDTHIS = data.completeAddress;
      bgyVarStore.height.BINDTHIS = data.height;
      bgyVarStore.weight.BINDTHIS = data.weight;
      bgyVarStore.dateOfAppointment.BINDTHIS = fixDateFormat(
        data.dateOfAppointment
      );
      bgyVarStore.birthdate.BINDTHIS = fixDateFormat(data.birthdate);
    }, 100);
  };

  const handleSearch = () => {
    const q = query(
      colRef,
      orderBy("createdAt", "desc"),
      where("completeName", "==", bgyVarStore.kwiri)
    );
    onSnapshot(q, (snapshots) => {
      let fbData = [];
      snapshots.docs.forEach((doc) => {
        let data = { ...doc.data(), id: doc.id };
        fbData = [data, ...fbData];
      });
      onSnapsBgyID.set(fbData);
    });

    bgyVarStore.trigger = false;
  };

  const detectInputs = () => {
    if (bgyVarStore.kwiri.trim().length < 1) {
      const q = query(colRef, orderBy("createdAt", "desc"));
      onSnapshot(q, (snapshots) => {
        let fbData = [];
        snapshots.docs.forEach((doc) => {
          let data = { ...doc.data(), id: doc.id };
          fbData = [data, ...fbData];
        });
        onSnapsBgyID.set(fbData);
      });

      bgyVarStore.trigger = false;
    } else {
      bgyVarStore.trigger = true;
    }
  };

  async function printPdf(dataForPrint) {
    const mywindow = window.open(
      "",
      "PRINT",
      "height=891,width=649, initial-scale=1.0"
    );

    if (!mywindow) {
      console.error("Popup blocked. Please allow popups for this site.");
      return false;
    }
    const printContent = new IdContent({
      target: mywindow.document.body,
      props: {
        documentTitle: "hello world",
        mywindow: mywindow,
        ...dataForPrint,
        height: cmToFeetAndInches(dataForPrint.height),
      },
    });
    console.log(printContent);
    return true;
  }
  const headerSortAscending = {
    firstName: undefined,
    middleInitial: undefined,
    lastName: undefined,
    suffixName: undefined,
    precintNum: undefined,
    completeAddress: undefined,
    kwiri: undefined,
    trigger: undefined,
  };

  function cmToFeetAndInches(cm) {
    const inches = cm / 2.54;
    const feet = Math.floor(inches / 12);
    const remainingInches = Math.round(inches % 12);
    return `${feet}'${remainingInches}"`;
}

  const sortTable = (fieldName, isAscending) => {
    let q = query(colRef, orderBy(fieldName, isAscending ? "asc" : "desc"));
    onSnapshot(q, (snapshots) => {
      let fbData = [];
      snapshots.docs.forEach((doc) => {
        let data = { ...doc.data(), id: doc.id };
        fbData = [data, ...fbData];
      });
      onSnaps.set(fbData);
      console.log(fbData);

      bgyVarStore.trigger = false;
    });
  };

  const resetSort = (key, value) => {
    let keys = Object.keys(headerSortAscending);
    for (let i = 0; i < keys.length; i++) {
      headerSortAscending[keys[i]] = undefined;
    }
    headerSortAscending[key] = value;
  };


  const setDateIndex = writable(-1);

  const setDateHandler = (data, index) => {
    setDateIndex.set(index);
    setTimeout(function () {
      bgyVarStore.dateOfAppointment.BINDTHIS = fixDateFormat(
        data.dateOfAppointment
      );
    }, 500);
  };

  const setDate = async (data) => {
    const docRef = doc(colRef, data);
    await setDoc(
      docRef,
      {
        lastUpdated: serverTimestamp(),
        dateOfAppointment: bgyVarStore.dateOfAppointment.BINDTHIS,
      },
      { merge: true }
    );
    setDateIndex.set(-1);
  };
</script>

<div
  class="m-2 mx-auto text-xs"
  style="margin-bottom: {showPrintModel ? '20vh' : '0px'}"
>
  <div class="min-h-[50vh] p-10">
    <div class=" flex gap-2 items-center mb-2">
      <div class="w-full flex gap-2">
        <div class="">
          <Button TITLE="Add Barangay ID" on:click={toShowAddModal} />
        </div>

        <div class="">
          <Button
            TITLE="Print report"
            on:click={() => showPrintModel.set(true)}
          />
        </div>
      </div>

      <div class="flex flex-row-reverse items-center w-full">
        <button
          class="bg-blue-400 text-white absolute p-2 border-r-2 border-black hover:bg-blue-700 font-bold"
          on:click={handleSearch}>Search</button
        >
        <input
          type="text"
          placeholder="Complete Name Only"
          class="w-[40%] p-2 focus:outline-none border-2 border-black"
          on:keyup={detectInputs}
          bind:value={bgyVarStore.kwiri}
        />
      </div>
    </div>

    {#if $showPrintModel}
      <div class="fixed bottom-0 top-0 left-0 right-0 bg-white">
        <div class="mx-auto max-w-[1000px] mt-[20vh] p-10">
          <div
            class="fixed bottom-0 right-0 p-10"
            style="bottom: -10px !important;"
          >
            <div class="flex gap-2">
              {#if !$printing}
                <div class="">
                  <Button
                    TITLE="Print Now"
                    on:click={() => {
                      $printing = true;
                      // print();
                      // $printing = false;
                      setTimeout(() => print(), 100);
                      setTimeout(() => {
                        $printing = false;
                        $showPrintModel = false;
                      }, 2000);
                    }}
                  />
                </div>
                <div class="">
                  <Button
                    TITLE="Close"
                    on:click={() => showPrintModel.set(false)}
                  />
                </div>
              {/if}
            </div>
          </div>
        </div>
      </div>
    {/if}

    {#if $showAddModal}
      <div
        class="flex flex-col gap-2 bg-white p-4 max-w-fit mx-auto rounded-lg mt-2 absolute left-0 right-0 border-2 border-guiColor z-10"
      >
        <p class="text-xl text-center font-bold p-2 text-slate-500">
          New Barangay ID
        </p>
        <div class="">
          <Inputs
            TITLE="Complete Name:"
            PLACEHOLDER="Complete Name"
            bind:this={bgyVarStore.completeName}
          />
        </div>

        <div class="flex justify-center gap-2">
          <div class="">
            <Inputs
              TITLE="Height"
              PLACEHOLDER="Height"
              bind:this={bgyVarStore.height}
            />
          </div>
          <div class="">
            <Inputs
              TITLE="Weight"
              PLACEHOLDER="Weight"
              bind:this={bgyVarStore.weight}
            />
          </div>
        </div>

        <div class="flex justify-center gap-2">
          <div class="w-full">
            <Inputs
              TITLE="Birthdate:"
              TYPE="date"
              bind:this={bgyVarStore.birthdate}
            />
          </div>
          <div class="w-full">
            <Inputs
              TITLE="Gender:"
              TYPE="cordion"
              bind:this={bgyVarStore.gender}
            />
          </div>
        </div>

        <div class="">
          <Inputs
            TITLE="Complete Address:"
            PLACEHOLDER="Complete Address"
            bind:this={bgyVarStore.address}
          />
        </div>

        <div class="">
          <Inputs
            TITLE="Date Of Apointment:"
            TYPE="date"
            PLACEHOLDER="Complete Address"
            bind:this={bgyVarStore.dateOfAppointment}
          />
        </div>

        <div class="flex gap-2">
          <Button TITLE="Submit" on:click={submitData} />
          <Button
            TITLE="Cancel"
            on:click={() => {
              showAddModal.set(false);
            }}
          />
        </div>
      </div>
    {/if}
    <div class="" in:fly={{ x: -400, duration: 1000 }}>
      <div class="relative">
        {#if $showPrintModel}
          <img
            src={reportHeader}
            alt=""
            style="margin-top: -130px; margin-bottom: 100px"
          />
        {/if}
        <table class="w-full text-sm text-left text-gray-500">
          <thead class="text-xs text-gray-700 uppercase bg-gray-50">
            <tr>
              {#if !$showPrintModel}
                <th scope="col" class="px-6 py-3"> ID Image </th>
              {/if}
              <th
                scope="col"
                class="px-6 py-3"
                on:click={() => {
                  if (headerSortAscending.firstName == undefined) {
                    headerSortAscending.firstName = true;
                  } else {
                    headerSortAscending.firstName =
                      !headerSortAscending.firstName;
                  }
                  resetSort("firstName", headerSortAscending.firstName);
                  sortTable("firstName", headerSortAscending.firstName);
                }}
              >
                <div class="flex justify-center gap-2 items-center">
                  firstname
                  <span
                    class={`sort-indicator 
                ${
                  headerSortAscending.firstName == undefined
                    ? ""
                    : headerSortAscending.firstName
                      ? "asc"
                      : "desc"
                }
                `}
                  ></span>
                </div>
              </th>
              <th
                scope="col"
                class="px-6 py-3"
                on:click={() => {
                  if (headerSortAscending.middleInitial == undefined) {
                    headerSortAscending.middleInitial = true;
                  } else {
                    headerSortAscending.middleInitial =
                      !headerSortAscending.middleInitial;
                  }
                  resetSort("middleInitial", headerSortAscending.middleInitial);
                  sortTable("middleInitial", headerSortAscending.middleInitial);
                }}
              >
                <div class="flex justify-center gap-2 items-center">
                  MI
                  <span
                    class={`sort-indicator 
                ${
                  headerSortAscending.middleInitial == undefined
                    ? ""
                    : headerSortAscending.middleInitial
                      ? "asc"
                      : "desc"
                }
                `}
                  ></span>
                </div>
              </th>
              <th
                scope="col"
                class="px-6 py-3"
                on:click={() => {
                  if (headerSortAscending.lastName == undefined) {
                    headerSortAscending.lastName = true;
                  } else {
                    headerSortAscending.lastName =
                      !headerSortAscending.lastName;
                  }
                  resetSort("lastName", headerSortAscending.lastName);
                  sortTable("lastName", headerSortAscending.lastName);
                }}
              >
                <div class="flex justify-center gap-2 items-center">
                  lastName
                  <span
                    class={`sort-indicator 
                ${
                  headerSortAscending.lastName == undefined
                    ? ""
                    : headerSortAscending.lastName
                      ? "asc"
                      : "desc"
                }
                `}
                  ></span>
                </div>
              </th>
              <th
                scope="col"
                class="px-6 py-3"
                on:click={() => {
                  if (headerSortAscending.suffixName == undefined) {
                    headerSortAscending.suffixName = true;
                  } else {
                    headerSortAscending.suffixName =
                      !headerSortAscending.suffixName;
                  }
                  resetSort("suffixName", headerSortAscending.suffixName);
                  sortTable("suffixName", headerSortAscending.suffixName);
                }}
              >
                <div class="flex justify-center gap-2 items-center">
                  Suffix
                  <span
                    class={`sort-indicator 
                ${
                  headerSortAscending.suffixName == undefined
                    ? ""
                    : headerSortAscending.suffixName
                      ? "asc"
                      : "desc"
                }
                `}
                  ></span>
                </div>
              </th>
              <th
                scope="col"
                class="px-6 py-3"
                on:click={() => {
                  if (headerSortAscending.completeAddress == undefined) {
                    headerSortAscending.completeAddress = true;
                  } else {
                    headerSortAscending.completeAddress =
                      !headerSortAscending.completeAddress;
                  }
                  resetSort(
                    "completeAddress",
                    headerSortAscending.completeAddress
                  );
                  sortTable(
                    "completeAddress",
                    headerSortAscending.completeAddress
                  );
                }}
              >
                <div class="flex justify-center gap-2 items-center">
                  Address
                  <span
                    class={`sort-indicator 
                ${
                  headerSortAscending.completeAddress == undefined
                    ? ""
                    : headerSortAscending.completeAddress
                      ? "asc"
                      : "desc"
                }
                `}
                  ></span>
                </div>
              </th>
              <th scope="col" class="px-6 py-3"> birthdate </th>
              <th scope="col" class="px-6 py-3"> gender </th>
              <th scope="col" class="px-6 py-3"> height </th>
              <th scope="col" class="px-6 py-3"> weight </th>
              <th
                scope="col"
                class="px-6 py-3"
                on:click={() => {
                  if (headerSortAscending.dateOfAppointment == undefined) {
                    headerSortAscending.dateOfAppointment = true;
                  } else {
                    headerSortAscending.dateOfAppointment =
                      !headerSortAscending.dateOfAppointment;
                  }
                  resetSort(
                    "dateOfAppointment",
                    headerSortAscending.dateOfAppointment
                  );
                  sortTable(
                    "dateOfAppointment",
                    headerSortAscending.dateOfAppointment
                  );
                }}
              >
                <div class="flex justify-center gap-2 items-center">
                  Appointment
                  <span
                    class={`sort-indicator 
              ${
                headerSortAscending.dateOfAppointment == undefined
                  ? ""
                  : headerSortAscending.dateOfAppointment
                    ? "asc"
                    : "desc"
              }
              `}
                  ></span>
                </div>
              </th>
              <th scope="col" class="px-6 py-3"> status </th>
              {#if !$showPrintModel}
                <th scope="col" class="px-6 py-3"> action </th>
              {/if}
            </tr>
          </thead>
          <tbody>
            {#each $onSnapsBgyID as barangayId, i}
              <tr class="bg-white border-b">
                {#if !$showPrintModel}
                  <th
                    scope="row"
                    class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap"
                  >
                    <img
                      src={barangayId.IDpictureUrl}
                      alt="ID Picture"
                      class="h-10 w-10 rounded-full"
                    />
                  </th>
                {/if}
                <td class="px-6 py-4">
                  {barangayId.firstName}
                </td>
                <td class="px-6 py-4">
                  {barangayId.lastName}
                </td>
                <td class="px-6 py-4">
                  {barangayId.middleInitial}
                </td>
                <td class="px-6 py-4">
                  {barangayId.suffixName}
                </td>
                <td class="px-6 py-4">
                  {barangayId.address}
                </td>
                <td class="px-6 py-4">
                  {barangayId["birthdate"]}
                </td>
                <td class="px-6 py-4">
                  {barangayId.gender}
                </td>
                <td class="px-6 py-4">
                  {barangayId.height}
                </td>
                <td class="px-6 py-4">
                  {barangayId.weight}
                </td>
                <td class="px-6 py-4">
                  {#if barangayId.dateOfAppointment == undefined}
                    <button
                      class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-full"
                      on:click={() => {
                        setDateHandler(barangayId, i);
                      }}
                    >
                      Set Date
                    </button>
                  {:else}
                    <span
                      class="cursor-pointer"
                      on:click={() => {
                        setDateHandler(barangayId, i);
                      }}
                    >
                      {barangayId.dateOfAppointment}
                    </span>
                  {/if}
                  {#if $setDateIndex === i}
                    <div class="">
                      <div
                        class="flex bg-white flex-col gap-2 p-4 max-w-fit mx-auto rounded-lg mt-2 absolute left-0 right-0 border-2 border-slate-200 z-10"
                      >
                        <p
                          class="text-xl text-center font-bold p-2 text-slate-500"
                        >
                          Set Date
                        </p>
                        <div class="">
                          <Inputs
                            TITLE="Date Of Appointment:"
                            TYPE="date"
                            PLACEHOLDER=""
                            ONCLICK={disableWeekends}
                            bind:this={bgyVarStore.dateOfAppointment}
                          />
                        </div>

                        <div class="flex gap-2">
                          <button
                            class="bg-orange-300 px-4 py-2 rounded-lg w-1/2"
                            on:click={setDate(barangayId.id)}
                          >
                            Set
                          </button>
                          <button
                            class="bg-red-300 px-4 py-2 rounded-lg w-1/2"
                            on:click={() => {
                              setDateIndex.set(-1);
                            }}
                          >
                            Cancel
                          </button>
                        </div>
                      </div>
                    </div>
                  {/if}
                </td>
                {#if $showPrintModel}
                  <td class="px-6 py-4">
                    {barangayId.status}
                  </td>
                {/if}
                {#if !$showPrintModel}
                  <td class="px-6 py-4">
                    <select
                      class="bg-white"
                      bind:value={barangayId.status}
                      on:change={() =>
                        updateStatus(
                          barangayId.id,
                          barangayId.status,
                          barangayId.appointmentOwner
                        )}
                    >
                      <option value="Processing">On Process</option>
                      <option value="Ready for pickup">For Pickup</option>
                      <option value="Claimed">Completed</option>
                    </select>
                  </td>
                  <td class="px-6 py-4">
                    <div class="flex gap-2">
                      <button
                        class="hover:bg-orange-300 hover:scale-105 px-4 py-2 rounded-full duration-700 hover:text-white"
                        on:click={() => {
                          editValueHandler(barangayId, i);
                        }}><i class="ri-pencil-line text-base"></i></button
                      >
                      <button
                        class="hover:bg-red-300 hover:scale-105 px-4 py-2 rounded-full duration-700 hover:text-white"
                        on:click={removeData(barangayId.id)}
                        ><i class="ri-delete-bin-line"></i></button
                      >
                      <button
                        class="hover:bg-blue-500 rounded-full px-4 py-2 hover:scale-105 duration-700 text-blue-900 hover:text-white"
                        on:click={() => printPdf(barangayId)}
                        ><i class="ri-printer-line"></i></button
                      >
                    </div>
                  </td>
                {/if}
              </tr>

              {#if $compareIDvalue === i}
                <div
                  class=" flex flex-col gap-2 p-4 max-w-fit mx-auto rounded-lg mt-2 absolute left-0 right-0 border-2 bg-gray-white z-10 -top-32 bg-white"
                >
                  <p class="text-xl text-center font-bold p-2 text-slate-500">
                    Modify Values
                  </p>
                  <div class="">
                    <Inputs
                      TITLE="First Name"
                      PLACEHOLDER="First Name"
                      bind:this={bgyVarStore.firstName}
                    />
                  </div>
                  <div class="">
                    <Inputs
                      TITLE="M.I."
                      PLACEHOLDER="Middle Initial"
                      bind:this={bgyVarStore.middleInitial}
                    />
                  </div>

                  <div class="">
                    <Inputs
                      TITLE="Last Name"
                      PLACEHOLDER="Last Name"
                      bind:this={bgyVarStore.lastName}
                    />
                  </div>
                  <div class="">
                    <Inputs
                      TITLE="Suffix"
                      PLACEHOLDER="Suffix"
                      bind:this={bgyVarStore.suffixName}
                    />
                  </div>

                  <div class="flex justify-center gap-2">
                    <div class="">
                      <Inputs
                        TITLE="Height"
                        TYPE="number"
                        PLACEHOLDER="Height"
                        bind:this={bgyVarStore.height}
                      />
                    </div>
                    <div class="">
                      <Inputs
                        TITLE="Weight"
                        TYPE="number"
                        PLACEHOLDER="Weight"
                        bind:this={bgyVarStore.weight}
                      />
                    </div>
                  </div>

                  <div class="flex justify-center gap-2">
                    <div class="w-full">
                      <Inputs
                        TITLE="Birthdate:"
                        TYPE="date"
                        bind:this={bgyVarStore.birthdate}
                      />
                    </div>

                    <div class="w-full">
                      <Inputs
                        TITLE="Gender:"
                        TYPE="cordion"
                        bind:this={bgyVarStore.gender}
                      />
                    </div>
                  </div>

                  <div class="">
                    <Inputs
                      TITLE="Complete Address:"
                      PLACEHOLDER="Complete Address"
                      bind:this={bgyVarStore.address}
                    />
                  </div>

                  <div class="">
                    <Inputs
                      TITLE="Date Of Appointment:"
                      TYPE="date"
                      PLACEHOLDER="Date Of Appointment"
                      bind:this={bgyVarStore.dateOfAppointment}
                    />
                  </div>

                  <div class="flex gap-2">
                    <button
                      class="bg-orange-300 px-4 py-2 rounded-lg w-1/2"
                      on:click={updateData(barangayId.id)}>Update</button
                    >
                    <button
                      class="bg-red-300 px-4 py-2 rounded-lg w-1/2"
                      on:click={() => {
                        compareIDvalue.set("");
                      }}>Cancel</button
                    >
                  </div>
                </div>
              {/if}
            {/each}
          </tbody>
        </table>
      </div>
    </div>
  </div>
</div>

<style>
  /* Add some styling for the sorting indicator */
  .sort-indicator {
    display: inline-block;
    width: 0;
    height: 0;
    border-style: solid;
    vertical-align: middle;
    transition: transform 0.3s ease-out;
  }

  .asc {
    border-width: 8px 5px 0 5px;
    border-color: #000 transparent transparent transparent;
    transform: rotate(180deg);
  }

  .desc {
    border-width: 8px 5px 0 5px;
    border-color: #000 transparent transparent transparent;
    transform: rotate(0deg);
  }
</style>
