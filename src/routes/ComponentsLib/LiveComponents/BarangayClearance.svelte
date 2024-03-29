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
  import { showPrintModel, formattedDate } from "./stateStore";

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
  } from "firebase/firestore";
  import { writable } from "svelte/store";
  export const setDateIndex = writable(-1);

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

    const notifDocRef = doc(collection(db, "notifications"));
    let message;
    if (selectedStatus == "Processing") {
      message = "Your clearance is being processed";
    } else if (selectedStatus == "Ready for pickup") {
      message =
        "Your clearance is ready for pickup, get it before or on appointed date";
    } else {
      message = "Your clearance has been claimed";
    }
    var currentDate = Date.now();
    const notificationData = {
      message: message,
      userID: ownerID,
      timestamp: currentDate,
    };
    console.log(notificationData);
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
    setTimeout(function () {
      bgyVarStore.firstName.BINDTHIS = data.firstName;
      bgyVarStore.middleInitial.BINDTHIS = data.middleInitial;
      bgyVarStore.lastName.BINDTHIS = data.lastName;
      bgyVarStore.suffixName.BINDTHIS = data.suffixName;
      bgyVarStore.address.BINDTHIS = data.address;
      bgyVarStore.lengthOfStay.BINDTHIS = data.lengthOfStay;
      bgyVarStore.purpose.BINDTHIS = data.purpose;
      bgyVarStore.dateOfAppointment.BINDTHIS = fixDateFormat(
        data.dateOfAppointment,
      );
    }, 100);
  };

  const setDateHandler = (data, index) => {
    setDateIndex.set(index);
    setTimeout(function () {
      bgyVarStore.dateOfAppointment.BINDTHIS = fixDateFormat(data.dateOfAppointment);
    }, 500);
  };

  const handlerSearch = () => {
    if (bgyVarStore.trigger) {
      const q = query(
        colRef,
        orderBy("createdAt", "desc"),
        where("completeName", "==", bgyVarStore.kwiri),
      );
      onSnapshot(q, (snapshots) => {
        let fbData = [];
        snapshots.docs.forEach((doc) => {
          let data = { ...doc.data(), id: doc.id };
          fbData = [data, ...fbData];
        });
        onSnapsClearance.set(fbData);
      });

      bgyVarStore.trigger = false;
    }
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
        onSnapsClearance.set(fbData);
      });
      bgyVarStore.trigger = false;
    } else {
      bgyVarStore.trigger = true;
    }
  };

  async function printPdf(dataForPrint) {
    // console.log({ dataForPrint });
    // printData.name = dataForPrint.completeName;
    // printData.activity = dataForPrint.purpose;
    // pdfElementHidden = false;
    // const pdf = await html2pdf().set(opt).from(pdfElement).save();
    // pdfElementHidden = true;
    // return pdf;

    printPdfTest(dataForPrint);

    return true;
  }
  function printPdfTest(dataForPrint) {
    const mywindow = window.open(
      "",
      "PRINT",
      "height=891,width=649, initial-scale=1.0",
    );
    console.log(dataForPrint);

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
    console.log(printContent);

    // mywindow.document.close(); // Necessary for IE >= 10
    // mywindow.focus(); // Necessary for IE >= 10

    mywindow.addEventListener("mousedown", function () {
      // Code to execute after the print operation is complete
      console.log("Print operation completed");
      // Optionally, close the window after printing
      // mywindow.print();
      // mywindow.close();
    });

    // mywindow.close();

    return true;
  }
  const headerSortAscending = {
    firstName: false,
  };

  const sortTable = (fieldName, isAscending) => {
    let q = query(colRef, orderBy(fieldName, isAscending ? "asc" : "desc"));
    onSnapshot(q, (snapshots) => {
      let fbData = [];
      snapshots.docs.forEach((doc) => {
        let data = { ...doc.data(), id: doc.id };
        fbData = [data, ...fbData];
      });
      onSnapsClearance.set(fbData);
      console.log(fbData);
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

  sortTable("firstName", false);

  const updateCivil = async (userID, selectedStatus) => {
    const docRef = doc(colRef, userID);
    const updatedData = {
      civilStatus: selectedStatus,
      lastUpdated: serverTimestamp(),
    };
    await setDoc(docRef, updatedData, { merge: true });
  };
  const updateGender = async (userID, selectedStatus) => {
    const docRef = doc(colRef, userID);
    const updatedData = {
      gender: selectedStatus,
      lastUpdated: serverTimestamp(),
    };
    await setDoc(docRef, updatedData, { merge: true });
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
</script>

<div
  class="m-2 mx-auto text-xs"
  style="margin-bottom: {showPrintModel ? '20vh' : '0px'}"
>
  <div class="min-h-[50vh] p-10">
    <div class="flex gap-2 items-center mb-2">
      <div class="w-full flex gap-2">
        <div class="">
          <Button TITLE="Add Barangay Clearance" on:click={toShowAddModal} />
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
    </div>

    {#if $showClearanceAddModal}
      <div
        class="flex flex-col gap-2 bg-white p-4 max-w-fit mx-auto rounded-lg mt-2 absolute left-0 right-0 border-2 border-guiColor z-10"
      >
        <p class="text-xl text-center font-bold p-2 text-slate-500">
          New Barangay Clearance
        </p>
        <div class="flex justify-center gap-2">
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
              PLACEHOLDER="M.I."
              bind:this={bgyVarStore.middleInitial}
            />
          </div>
        </div>
        <div class="flex justify-center gap-2">
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
              PLACEHOLDER="purpose"
              bind:this={bgyVarStore.purpose}
            />
          </div>
        </div>

        <div class="flex justify-center gap-2">
          <div class="w-full">
            <Inputs
              TITLE="Age"
              PLACEHOLDER="18"
              TYPE="number"
              bind:this={bgyVarStore.age}
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
            PLACEHOLDER=""
            bind:this={bgyVarStore.dateOfAppointment}
          />
        </div>

        <div class="flex gap-2">
          <Button TITLE="Submit" on:click={submitData} />
          <Button
            TITLE="Cancel"
            on:click={() => {
              showClearanceAddModal.set(false);
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
                    headerSortAscending.completeAddress,
                  );
                  sortTable("address", headerSortAscending.completeAddress);
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
              <th
                scope="col"
                class="px-6 py-3"
                on:click={() => {
                  if (headerSortAscending.age == undefined) {
                    headerSortAscending.age = true;
                  } else {
                    headerSortAscending.age = !headerSortAscending.age;
                  }
                  resetSort("age", headerSortAscending.age);
                  sortTable("age", headerSortAscending.age);
                }}
              >
                <div class="flex justify-center gap-2 items-center">
                  age
                  <span
                    class={`sort-indicator 
              ${
                headerSortAscending.age == undefined
                  ? ""
                  : headerSortAscending.age
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
                  if (headerSortAscending.lengthOfStay == undefined) {
                    headerSortAscending.lengthOfStay = true;
                  } else {
                    headerSortAscending.lengthOfStay =
                      !headerSortAscending.lengthOfStay;
                  }
                  resetSort("lengthOfStay", headerSortAscending.lengthOfStay);
                  sortTable("lengthOfStay", headerSortAscending.lengthOfStay);
                }}
              >
                <div class="flex justify-center gap-2 items-center">
                  length Of Stay
                  <span
                    class={`sort-indicator 
              ${
                headerSortAscending.lengthOfStay == undefined
                  ? ""
                  : headerSortAscending.lengthOfStay
                    ? "asc"
                    : "desc"
              }
              `}
                  ></span>
                </div>
              </th>

              <th scope="col" class="px-6 py-3"> purpose </th>
              <th scope="col" class="px-6 py-3"> Valid ID </th>
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
                    headerSortAscending.dateOfAppointment,
                  );
                  sortTable(
                    "dateOfAppointment",
                    headerSortAscending.dateOfAppointment,
                  );
                }}
              >
                <div class="flex justify-center gap-2 items-center">
                  date Of Appointment
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
              <th
                scope="col"
                class="px-6 py-3"
                on:click={() => {
                  if (headerSortAscending.civilStatus == undefined) {
                    headerSortAscending.civilStatus = true;
                  } else {
                    headerSortAscending.civilStatus =
                      !headerSortAscending.civilStatus;
                  }
                  resetSort("civilStatus", headerSortAscending.civilStatus);
                  sortTable("civilStatus", headerSortAscending.civilStatus);
                }}
              >
                <div class="flex justify-center gap-2 items-center">
                  civil Status
                  <span
                    class={`sort-indicator 
                ${
                  headerSortAscending.civilStatus == undefined
                    ? ""
                    : headerSortAscending.civilStatus
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
                  if (headerSortAscending.gender == undefined) {
                    headerSortAscending.gender = true;
                  } else {
                    headerSortAscending.gender = !headerSortAscending.gender;
                  }
                  resetSort("gender", headerSortAscending.gender);
                  sortTable("gender", headerSortAscending.gender);
                }}
              >
                <div class="flex justify-center gap-2 items-center">
                  gender
                  <span
                    class={`sort-indicator 
                ${
                  headerSortAscending.gender == undefined
                    ? ""
                    : headerSortAscending.gender
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
            {#each $onSnapsClearance as barangayClearance, i}
              <!-- {console.log(barangayClearance)} -->
              <tr class="bg-white border-b">
                <th
                  scope="row"
                  class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap"
                >
                  {barangayClearance.firstName}
                </th>
                <td class="px-6 py-4">
                  {barangayClearance.middleInitial}
                </td>
                <td class="px-6 py-4">
                  {barangayClearance.lastName}
                </td>
                <td class="px-6 py-4">
                  {barangayClearance.suffixName}
                </td>
                <td class="px-6 py-4">
                  {barangayClearance.address}
                </td>
                <td class="px-6 py-4">
                  {barangayClearance.age}
                </td>
                <td class="px-6 py-4">
                  {barangayClearance.lengthOfStay}
                </td>
                <td class="px-6 py-4">
                  {barangayClearance.purpose}
                </td>
                <td class="px-6 py-4">
                  <a href={barangayClearance.validIDUrl}>
                    <img
                      src={barangayClearance.validIDUrl}
                      alt="Valid ID"
                      class="h-10 w-10"
                    />
                  </a>
                </td>
                <td class="px-6 py-4">
                  {#if barangayClearance.dateOfAppointment == undefined}
                    <button
                      class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-full"
                      on:click={() => {
                        setDateHandler(barangayClearance, i);
                      }}
                    >
                      Set Date
                    </button>
                  {:else}
                    <span
                    class="cursor-pointer"
                      on:click={() => {
                        setDateHandler(barangayClearance, i);
                      }}
                    >
                      {barangayClearance.dateOfAppointment}
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
                            bind:this={bgyVarStore.dateOfAppointment}
                            ONCLICK={disableWeekends}
                          />
                        </div>

                        <div class="flex gap-2">
                          <button
                            class="bg-orange-300 px-4 py-2 rounded-lg w-1/2"
                            on:click={setDate(barangayClearance.id)}
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
                  <td class="px-6 py-4"> {barangayClearance.civilStatus} </td>
                {/if}
                {#if !$showPrintModel}
                  <td class="px-6 py-4">
                    <select
                      class="bg-white"
                      bind:value={barangayClearance.civilStatus}
                      on:change={() =>
                        updateCivil(
                          barangayClearance.id,
                          barangayClearance.civilStatus,
                        )}
                    >
                      <option value="Single">Single</option>
                      <option value="Married">Married</option>
                      <option value="Widowed">Widowed</option>
                      <option value="Divorced">Divorced</option>
                    </select>
                  </td>
                {/if}
                {#if $showPrintModel}
                  <td class="px-6 py-4"> {barangayClearance.gender} </td>
                {/if}
                {#if !$showPrintModel}
                  <td class="px-6 py-4">
                    <select
                      class="bg-white"
                      bind:value={barangayClearance.gender}
                      on:change={() =>
                        updateGender(
                          barangayClearance.id,
                          barangayClearance.gender,
                        )}
                    >
                      <option value="Single">Male</option>
                      <option value="Married">Female</option>
                    </select>
                  </td>
                {/if}
                {#if $showPrintModel}
                  <td class="px-6 py-4"> {barangayClearance.status} </td>
                {/if}
                {#if $showPrintModel}
                  <td class="px-6 py-4">
                    {barangayClearance.status}
                  </td>
                {/if}
                {#if !$showPrintModel}
                  <td class="px-6 py-4">
                    <select
                      class="bg-white"
                      bind:value={barangayClearance.status}
                      on:change={() =>
                        updateStatus(
                          barangayClearance.id,
                          barangayClearance.status,
                          barangayClearance.appointmentOwner,
                        )}
                    >
                      <option value="Processing">On Process</option>
                      <option value="Ready for pickup">For Pickup</option>
                      <option value="Claimed">Completed</option>
                    </select>
                  </td>
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
                          PLACEHOLDER="purpose"
                          bind:this={bgyVarStore.purpose}
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

      <!-- {#each $onSnapsClearance as value, i}
            <div class="flex justify-center items-center " in:fly={{x:-400, duration:1000}}>
                <p class="w-[25%] font-bold border-b-2 border-white bg-slate-300 p-2 ">Complete Name</p>
                <p class="w-full border-b-2 border-white bg-slate-100 p-2 ">{value.completeName}</p>

                <p class="w-[10%] font-bold border-b-2 border-white bg-slate-300 p-2 ">Address</p>
                <p class="w-full border-b-2 border-white bg-slate-100 p-2 ">{value.address}</p>

                <p class="w-[17%] font-bold border-b-2 border-white bg-slate-300 p-2 ">Birth date</p>
                <p class="w-[20%] border-2 border-white bg-slate-100 p-2 ">{value.birthdate}</p>

                <p class="w-[25%] font-bold border-b-2 border-white bg-slate-300 p-2 ">Length Of Stay</p>
                <p class="w-[20%] border-2 border-white bg-slate-100 p-2 ">{value.lengthOfStay}</p>

                <p class="w-[12%] font-bold border-b-2 border-white bg-slate-300 p-2 ">Purpose</p>
                <p class="w-[20%] border-2 border-white bg-slate-100 p-2 ">{value.purpose}</p>

                <p class="w-[33%] font-bold border-b-2 border-white bg-slate-300 p-2 ">Date Of Appointment</p>
                <p class="w-[20%] border-2 border-white bg-slate-100 p-2 ">{value.dateOfAppointment}</p>

                <div class="flex w-[30%] bg-slate-100">
                    <button class="bg-red-500 font-bold text-white w-full p-2 hover:bg-red-600 border-b-2 border-white"
                    on:click={removeData(value.id)}
                    >Delete</button>

                    <button class="bg-blue-500 font-bold text-white w-full p-2 hover:bg-blue-600 border-b-2 border-white"
                    on:click={()=>{editValueHandler(i)}}
                    >Update</button>

                    <button class="bg-red-500 font-bold text-white w-full p-2 hover:bg-red-600 border-b-2 border-white"
                    on:click={() => showPrintModel.set(true)}
                    >Print</button>
                    
                </div>
                {#if $compareClearanceValue === i}
                    <div class="">
                        <div class="flex flex-col gap-2 bg-guiColor p-4 max-w-fit mx-auto rounded-lg mt-2 absolute left-0 right-0 border-2 border-slate-200 z-10">
                            <p class="text-xl text-center font-bold p-2 text-slate-500">Modify Values</p>
                            <div class="">
                                <Inputs TITLE="Complete Name:" PLACEHOLDER="Complete Name" bind:this={bgyVarStore.completeName}/>
                            </div>
                
                            <div class="flex justify-center gap-2">
                                <div class="">
                                    <Inputs TITLE="Length of stay" PLACEHOLDER="Length of stay" bind:this={bgyVarStore.lengthOfStay}/>
                                </div>
                                <div class="">
                                    <Inputs TITLE="Purpose" PLACEHOLDER="purpose" bind:this={bgyVarStore.purpose}/>
                                </div>
                            </div>
                
                            <div class="flex justify-center gap-2">
                                <div class="w-full">
                                    <Inputs TITLE="Birthdate:" TYPE="date" bind:this={bgyVarStore.birthdate}/>
                                </div>
                            
                            </div>
                
                            <div class="">
                                <Inputs TITLE="Complete Address:" PLACEHOLDER="Complete Address" bind:this={bgyVarStore.address}/>
                            </div>
                            
                            <div class="">
                                <Inputs TITLE="Date Of Appointment:" TYPE="date" PLACEHOLDER="" bind:this={bgyVarStore.dateOfAppointment}/>
                            </div>

                            <div class="flex gap-2">
                                <Button TITLE="Confirm Edit" on:click={updateData(value.id)}/>
                                <Button TITLE="Cancel" on:click={() => {compareClearanceValue.set("")}}/>
                            </div>
                        </div>
                    </div>
                {/if}
            </div>
            {/each} -->
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
