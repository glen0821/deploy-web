<script>
  import Button from "../GeneralComponents/Button.svelte";
  import Inputs from "../GeneralComponents/Inputs.svelte";
  import { fly } from "svelte/transition";

  import {
    onSnapsClearance,
    showClearanceAddModal,
    compareClearanceValue,
    printing,
  } from "../BoundComponents/clickOutside";
  import { showPrintModel } from "./stateStore";
  import { showPrintModel } from "./stateStore";

  import bgyClearance from "../Images/bgyClearance.jpg";
  import PrintContent from "./PrintContent.svelte";

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
    or,
  } from "firebase/firestore";
  import { writable } from "svelte/store";
  export const setDateIndex = writable(-1);
  import flatpickr from "flatpickr";
  import 'flatpickr/dist/flatpickr.css'

  //handler to show add modal
  const toShowAddModal = () => {
    showClearanceAddModal.set(true);
  };

  //barangayID varStore
  const bgyVarStore = {
    firstName: "",
    lastName: "",
    middleInitial: "",
    suffixName: "",
    address: "",
    age: "",
    lengthOfStay: "",
    purpose: "",
    dateOfAppointment: "",
    kwiri: "",
    trigger: false,
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

  import reportHeader from "./images/header_report.png";

  //submit data to database
  const submitData = async () => {
    const colRef = collection(db, "barangayClearance");
    addDoc(colRef, {
      createdAt: serverTimestamp(),
      bgyClearanceCounter: increment(1),
      firstName: bgyVarStore.firstName.BINDTHIS,
      lastName: bgyVarStore.lastName.BINDTHIS,
      middleInitial: bgyVarStore.middleInitial.BINDTHIS,
      suffixName: bgyVarStore.suffixName.BINDTHIS,
      address: bgyVarStore.address.BINDTHIS,
      age: bgyVarStore.age.BINDTHIS,
      lengthOfStay: bgyVarStore.lengthOfStay.BINDTHIS,
      purpose: bgyVarStore.purpose.BINDTHIS,
      dateOfAppointment: bgyVarStore.dateOfAppointment.BINDTHIS,
    }).then(() => {
      showClearanceAddModal.set(false);
    });
  };

  //fetch data from database
  const colRef = collection(db, "barangayClearance");
  const q = query(colRef, orderBy("createdAt", "desc"));
  onSnapshot(q, (snapshots) => {
    let fbData = [];
    snapshots.docs.forEach((doc) => {
      let data = { ...doc.data(), id: doc.id };
      fbData = [data, ...fbData];
    });
    onSnapsClearance.set(fbData);
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

    const notifDocRef = doc(collection(db, "notifications"));
    const messages = {
      "Processing": "Your clearance is being processed",
      "Ready for pickup": "Your clearance is ready for pickup, get it before or on appointed date",
      "Claimed": "Your clearance has been claimed"
    };
    const message = messages[selectedStatus] || "Status updated";

    const notificationData = {
      message: message,
      userID: ownerID,
      timestamp: Date.now(),
    };
    await setDoc(notifDocRef, notificationData);
    await setDoc(docRef, updatedData, { merge: true });
  };

  //updateData from database
  const updateData = async (data) => {
    const docRef = doc(colRef, data);
    await setDoc(
      docRef,
      {
        lastUpdated: serverTimestamp(),
        firstName: bgyVarStore.firstName.BINDTHIS,
        middleInitial: bgyVarStore.middleInitial.BINDTHIS,
        lastName: bgyVarStore.lastName.BINDTHIS,
        suffixName: bgyVarStore.suffixName.BINDTHIS,
        address: bgyVarStore.address.BINDTHIS,
        lengthOfStay: bgyVarStore.lengthOfStay.BINDTHIS,
        purpose: bgyVarStore.purpose.BINDTHIS,
        dateOfAppointment: bgyVarStore.dateOfAppointment.BINDTHIS,
      },
      { merge: true },
    );
    compareClearanceValue.set(-1);
  };

  const setDate = async (data) => {
    const docRef = doc(colRef, data);
    await setDoc(
      docRef,
      {
        lastUpdated: serverTimestamp(),
        dateOfAppointment: bgyVarStore.dateOfAppointment.BINDTHIS,
      },
      { merge: true },
    );
    setDateIndex.set(-1);
  };

  //showModalComparison
  const editValueHandler = (data, index) => {
    compareClearanceValue.set(index);
    setTimeout(() => {
      bgyVarStore.firstName.BINDTHIS = data.firstName;
      bgyVarStore.middleInitial.BINDTHIS = data.middleInitial;
      bgyVarStore.lastName.BINDTHIS = data.lastName;
      bgyVarStore.suffixName.BINDTHIS = data.suffixName;
      bgyVarStore.address.BINDTHIS = data.address;
      bgyVarStore.lengthOfStay.BINDTHIS = data.lengthOfStay;
      bgyVarStore.purpose.BINDTHIS = data.purpose;
      bgyVarStore.dateOfAppointment.BINDTHIS = fixDateFormat(data.dateOfAppointment);
    }, 100);
  };

  
  const handlerSearch = () => {
    if (bgyVarStore.trigger) {
      console.log(bgyVarStore.kwiri);
      const q = query(colRef, orderBy("createdAt", "desc"));

      onSnapshot(q, (snapshots) => {
        const fbData = snapshots.docs.map((doc) => ({
          ...doc.data(),
          id: doc.id,
        }));

        const filteredData = fbData.filter((data) => {
          return Object.values(data).some((value) =>
            value.toString().toLowerCase().includes(bgyVarStore.kwiri.toLowerCase())
          );
        });

        onSnapsClearance.set(filteredData);
      });

      bgyVarStore.trigger = false;
    }
  };

  const detectInputs = () => {
    const querySnapshot = (q) => {
      onSnapshot(q, (snapshots) => {
        const fbData = snapshots.docs.map((doc) => ({
          ...doc.data(),
          id: doc.id,
        }));
        onSnapsClearance.set(fbData);
      });
    };

    if (bgyVarStore.kwiri.trim().length < 1) {
      const q = query(colRef, orderBy("createdAt", "desc"));
      querySnapshot(q);
      bgyVarStore.trigger = false;
    } else {
      bgyVarStore.trigger = true;
    }
  };

  async function printPdf(dataForPrint) {
    printPdfTest(dataForPrint);
    return true;
  }

  function printPdfTest(dataForPrint) {
    const mywindow = window.open(
      "",
      "PRINT",
      "height=891,width=649, initial-scale=1.0"
    );

    if (!mywindow) {
      console.error("Popup blocked. Please allow popups for this site.");
      return false;
    }

    const printContent = new PrintContent({
      target: mywindow.document.body,
      props: {
        documentTitle: "hello world",
        mywindow: mywindow,
        ...dataForPrint,
      },
    });

    mywindow.addEventListener("mousedown", function () {
      console.log("Print operation completed");
    });

    return true;
  }
  const headerSortAscending = {
    firstName: undefined,
  };

  const sortTable = (fieldName, isAscending) => {
    const q = query(colRef, orderBy(fieldName, isAscending ? "asc" : "desc"));
    onSnapshot(q, (snapshots) => {
      const fbData = snapshots.docs.map((doc) => ({ ...doc.data(), id: doc.id }));
      onSnapsClearance.set(fbData);
      bgyClearance.trigger = false;
    });
  };

  const resetSort = (key, value) => {
    let keys = Object.keys(headerSortAscending);
    for (let i = 0; i < keys.length; i++) {
      headerSortAscending[keys[i]] = undefined;
    }
    headerSortAscending[key] = value;
  };


  const updateGender = async (userID, selectedStatus) => {
    const docRef = doc(colRef, userID);
    const updatedData = {
      gender: selectedStatus,
      lastUpdated: serverTimestamp(),
    };
    await setDoc(docRef, updatedData, { merge: true });
  };

  
  
  const disableWeekends = (event) => {
    const element = event.target;
    if (element.getAttribute("picker_set") === "yes") {
      return;
    }
    flatpickr(element, {
      disable: [
        (date) => date.getDay() === 0 || date.getDay() === 6,
      ],
    });
    element.setAttribute("picker_set", "yes");
  };
</script>

<div
  class="m-2 mx-auto text-xs"
  style="margin-bottom: {showPrintModel ? '20vh' : '0px'}"
>
  <div class="min-h-[50vh] p-10">
    <div class="flex gap-2 items-center mb-2">
      <div class="w-full flex gap-2">
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
          on:click={handlerSearch}>Search</button
        >
        <input
          type="text"
          placeholder="Complete Name Only"
          class="w-[40%] p-2 focus:outline-none border-2 border-black"
          on:keyup={detectInputs}
          bind:value={bgyVarStore.kwiri}
        />
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
                  <Button
                    TITLE="Print Now"
                    on:click={() => handlePrint()}
                  />
                  <Button
                    TITLE="Close"
                    on:click={() => showPrintModel.set(false)}
                  />
                {/if}
              </div>

              <script>
                function handlePrint() {
                  $printing = true;
                  setTimeout(() => print(), 100);
                  setTimeout(() => {
                    $printing = false;
                    $showPrintModel = false;
                  }, 2000);
                }
              </script>
            </div>
          </div>
        </div>
      {/if}
    </div>
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
              {#each [
                { key: 'firstName', label: 'First Name' },
                { key: 'middleInitial', label: 'MI' },
                { key: 'lastName', label: 'Last Name' },
                { key: 'suffixName', label: 'Suffix' },
                { key: 'completeAddress', label: 'Address' },
                { key: 'age', label: 'Age' },
                { key: 'lengthOfStay', label: 'Length Of Stay' },
                { key: 'dateOfAppointment', label: 'Date Of Appointment' },
                { key: 'civilStatus', label: 'Civil Status' }
              ] as column}
                <th
                  scope="col"
                  class="px-6 py-3"
                  on:click={() => {
                    headerSortAscending[column.key] = headerSortAscending[column.key] === undefined ? true : !headerSortAscending[column.key];
                    resetSort(column.key, headerSortAscending[column.key]);
                    sortTable(column.key, headerSortAscending[column.key]);
                  }}
                >
                  <div class="flex justify-center gap-2 items-center">
                    {column.label}
                    <span
                      class={`sort-indicator 
                        ${
                          headerSortAscending[column.key] === undefined
                            ? ""
                            : headerSortAscending[column.key]
                              ? "asc"
                              : "desc"
                        }
                      `}
                    ></span>
                  </div>
                </th>
              {/each}
              <th scope="col" class="px-6 py-3"> Purpose </th>
              <th scope="col" class="px-6 py-3"> Valid ID </th>
              {#if !$showPrintModel}
                <th scope="col" class="px-6 py-3"> Action </th>
              {/if}
            </tr>
          </thead>
          <tbody>
            {#each $onSnapsClearance as barangayClearance, i}
              <tr class="bg-white border-b">
                {#each [
                  { key: 'firstName', value: barangayClearance.firstName },
                  { key: 'middleInitial', value: barangayClearance.middleInitial },
                  { key: 'lastName', value: barangayClearance.lastName },
                  { key: 'suffixName', value: barangayClearance.suffixName },
                  { key: 'address', value: barangayClearance.address },
                  { key: 'age', value: barangayClearance.age },
                  { key: 'lengthOfStay', value: barangayClearance.lengthOfStay },
                  { key: 'dateOfAppointment', value: barangayClearance.dateOfAppointment },
                  { key: 'civilStatus', value: barangayClearance.civilStatus },
                  { key: 'purpose', value: barangayClearance.purpose },
                  { key: 'validIDUrl', value: barangayClearance.validIDUrl, isLink: true }
                ] as column}
                  <td class="px-6 py-4">
                    {#if column.isLink}
                      <a href={column.value}>
                        <img src={column.value} alt="Valid ID" class="h-10 w-10" />
                      </a>
                    {:else if column.isSelect}
                      {#if column.key === 'gender' && !$showPrintModel}
                        <select
                          class="bg-white"
                          bind:value={column.value}
                          on:change={() => updateGender(barangayClearance.id, column.value)}
                        >
                          <option value="Male">Male</option>
                          <option value="Female">Female</option>
                        </select>
                      {:else}
                        {column.value}
                      {/if}
                    {:else}
                      {column.value}
                    {/if}
                  </td>
                {/each}
                
                {#if !$showPrintModel}
                  <td>
                    <div class="flex gap-2">
                      <button
                        class="hover:bg-orange-300 rounded-full px-4 py-2 hover:scale-105 duration-700"
                        on:click={() => {
                          editValueHandler(barangayClearance, i);
                        }}><i class="ri-pencil-line"></i></button
                      >

                      <button
                        class="hover:bg-red-500 rounded-full px-4 py-2 hover:scale-105 duration-700 text-red-900 hover:text-white"
                        on:click={removeData(barangayClearance.id)}
                        ><i class="ri-delete-bin-line"></i></button
                      >

                      <button
                        class="hover:bg-blue-500 rounded-full px-4 py-2 hover:scale-105 duration-700 text-blue-900 hover:text-white"
                        on:click={() => printPdf(barangayClearance)}
                        ><i class="ri-printer-line"></i></button
                      >
                    </div>
                  </td>
                {/if}
              </tr>
              {#if $compareClearanceValue === i}
                <div class="">
                  <div
                    class="flex bg-white flex-col gap-2 p-4 max-w-fit mx-auto rounded-lg mt-2 absolute left-0 right-0 border-2 border-slate-200 z-10"
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
                          TITLE="Length of stay"
                          PLACEHOLDER="Length of stay"
                          bind:this={bgyVarStore.lengthOfStay}
                        />
                      </div>
                      <div class="">
                        <Inputs
                          TITLE="Purpose"
                          PLACEHOLDER="Purpose"
                          bind:this={bgyVarStore.purpose}
                        />
                      </div>
                    </div>

                    <div class="">
                      <Inputs
                        TITLE="Complete Address:"
                        PLACEHOLDER="Complete Address"
                        bind:this={bgyVarStore.completeAddress}
                      />
                    </div>

                    <div class="">
                      <Inputs
                        TITLE="Date Of Appointment:"
                        TYPE="date"
                        PLACEHOLDER=""
                        bind:this={bgyVarStore.dateOfAppointment}
                      />
                    </div>

                    <div class="flex gap-2">
                      <button
                        class="bg-orange-300 px-4 py-2 rounded-lg w-1/2"
                        on:click={updateData(barangayClearance.id)}
                      >
                        Update
                      </button>
                      <button
                        class="bg-red-300 px-4 py-2 rounded-lg w-1/2"
                        on:click={() => {
                          compareClearanceValue.set("");
                        }}
                      >
                        Cancel
                      </button>
                    </div>
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
