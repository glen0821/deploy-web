<script>
  import Button from "../GeneralComponents/Button.svelte";
  import Inputs from "../GeneralComponents/Inputs.svelte";
  import {
    onSnapsBgyCert,
    showCertEditLogic,
    showCertAddModal,
    compareCertValue,
    printing,
  } from "../BoundComponents/clickOutside";
  import { showPrintModel } from "./stateStore";
  import {db } from "../../db/firebase";
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
  import { onMount } from "svelte";
  import CertificateContent from "./CertificateContent.svelte";
  import reportHeader from "./images/header_report.png";
  import { writable } from "svelte/store";

  let pdfElement

  const colRef = collection(db, "barangayCertificate");

  onMount(() => {
    pdfElement = document.getElementById("pdf-template");
  });

  async function printPdf(dataForPrint) {
    const mywindow = window.open(
      "",
      "PRINT",
      "height=891,width=649, initial-scale=1.0",
    );

    if (!mywindow) {
      console.error("Popup blocked. Please allow popups for this site.");
      return false;
    }

    new CertificateContent({
      target: mywindow.document.body,
      props: {
        documentTitle: "Barangay Certificate",
        ...dataForPrint,
      },
    });

    return true;
  }

  function fixDateFormat(originalDate) {
    var parts = originalDate.split("-");
    parts[1] = parts[1].padStart(2, "0");
    parts[2] = parts[2].padStart(2, "0");
    return parts.join("-");
  }

  const toShowAddModal = () => {
    showCertAddModal.set(true);
  };

  const toShowEditModal = () => {
    showCertEditLogic.set(true);
  };

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

  //submit data to database
  const submitData = async () => {
    const colRef = collection(db, "barangayCertificate");
    addDoc(colRef, {
      createdAt: serverTimestamp(),
      completeName: bgyVarStore.completeName.BINDTHIS,
      address: bgyVarStore.address.BINDTHIS,
      birthdate: bgyVarStore.birthdate.BINDTHIS,
      lengthOfStay: bgyVarStore.lengthOfStay.BINDTHIS,
      purpose: bgyVarStore.purpose.BINDTHIS,
      dateOfAppointment: bgyVarStore.dateOfAppointment.BINDTHIS,
      bgyCertificateCounter: increment(1),
    }).then(() => {
      showCertAddModal.set(false);
    });
  };

  const fetchData = () => {
    const q = query(colRef, orderBy("createdAt", "desc"));
    onSnapshot(q, (snapshots) => {
      const fbData = snapshots.docs.map((doc) => ({
        ...doc.data(),
        id: doc.id,
      }));
      onSnapsBgyCert.set(fbData);
    });
  };
  fetchData();

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
    const messages = {
      "Processing": "Your certificate is being processed",
      "Ready for pickup": "Your certificate is ready for pickup, get it before or on appointed date",
      "Claimed": "Your certificate has been claimed"
    };
    const message = messages[selectedStatus] || "Status updated";

    const notificationData = {
      message: message,
      userID: ownerID,
      timestamp: Date.now(),
    };
    await setDoc(notifDocRef, notificationData);
  };

  const updateGender = async (userID, selectedStatus) => {
    const docRef = doc(colRef, userID);
    const updatedData = {
      gender: selectedStatus,
      lastUpdated: serverTimestamp(),
    };
    await setDoc(docRef, updatedData, { merge: true });
  };

  //showModalComparison
  const editValueHandler = (data, index) => {
    toShowEditModal();
    compareCertValue.set(index);
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

        onSnapsBgyCert.set(filteredData);
      });

      bgyVarStore.trigger = false;
    }
  };

  const detectInputs = () => {
    bgyVarStore.trigger = bgyVarStore.kwiri.trim().length >= 1;
    if (!bgyVarStore.trigger) {
      const q = query(colRef, orderBy("createdAt", "desc"));
      onSnapshot(q, (snapshots) => {
        const fbData = snapshots.docs.map(doc => ({ ...doc.data(), id: doc.id }));
        onSnapsBgyCert.set(fbData);
      });
    }
  };
  
  const headerSortAscending = {
    firstName: undefined,
    middleInitial: undefined,
    lastName: undefined,
    suffixName: undefined,
    precintNum: undefined,
    completeAddress: undefined,
    age: undefined,
    kwiri: undefined,
    trigger: undefined,
  };

  const sortTable = (fieldName, isAscending) => {
    const orderDirection = isAscending ? "asc" : "desc";
    const q = query(colRef, orderBy(fieldName, orderDirection));
    onSnapshot(q, (snapshots) => {
      const fbData = snapshots.docs.map(doc => ({ ...doc.data(), id: doc.id }));
      onSnapsBgyCert.set(fbData);
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

  // sortTable("firstName", false);

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
    $showCertEditLogic = false;
  };
  const setDateIndex = writable(-1);

  const setDateHandler = (data, index) => {
    setDateIndex.set(index);
    setTimeout(function () {
      bgyVarStore.dateOfAppointment.BINDTHIS = fixDateFormat(
        data.dateOfAppointment,
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
      { merge: true },
    );
    setDateIndex.set(-1);
  };

  import flatpickr from "flatpickr";
  import "flatpickr/dist/flatpickr.css";
  const disableWeekends = (event) => {
    const element = event.target;
    if (element.getAttribute("picker_set") == "yes") {
      return;
    }
    flatpickr(element, {
      disable: [
        function (date) {
          return date.getDay() === 0 || date.getDay() === 6;
        },
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
    <div class=" flex gap-2 items-center mb-2">
      <div class="w-full flex gap-2">
        <div class="">
          <Button TITLE="Add Barangay Certificate" on:click={toShowAddModal} />
        </div>

        <div class="">
          <button
            class="text-xs py-2 px-4 bg-orange-300 rounded-lg hover:bg-orange-400 hover:scale-105 duration-700 font-bold hover:text-white"
            on:click={() => {
              $showPrintModel = true;
            }}
          >
            Print report
          </button>
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
                      TITLE="Print"
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

    {#if $showCertAddModal}
      <div
        class="flex flex-col gap-2 bg-white p-4 max-w-fit mx-auto rounded-lg mt-2 absolute left-0 right-0 border-2 border-guiColor z-10"
      >
        <p class="text-xl text-center font-bold p-2 text-slate-500">
          New Barangay Certificate
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
              TITLE="Birthdate:"
              TYPE="date"
              bind:this={bgyVarStore.birthdate}
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
            TITLE="Date Of Appointment"
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
              showCertAddModal.set(false);
            }}
          />
        </div>
      </div>
    {/if}

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
                  headerSortAscending.lastName = !headerSortAscending.lastName;
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
            <th scope="col" class="px-6 py-3"> quantity </th>
            <th scope="col" class="px-6 py-3"> status </th>
            {#if !$showPrintModel}
              <th scope="col" class="px-6 py-3"> action </th>
            {/if}
          </tr>
        </thead>
        <tbody>
          {#each $onSnapsBgyCert as cert, i}
            <tr class="bg-white border-b">
              <th
                scope="row"
                class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap"
              >
                {cert.firstName}
              </th>
              <td class="px-6 py-4">
                {cert.middleInitial}
              </td>
              <td class="px-6 py-4">
                {cert.lastName}
              </td>
              <td class="px-6 py-4">
                {cert.suffixName}
              </td>
              <td class="px-6 py-4"> {cert.address} </td>
              <td class="px-6 py-4">{cert.age} </td>
              <td class="px-6 py-4"> {cert.lengthOfStay} </td>
              <td class="px-6 py-4"> {cert.purpose} </td>
              <td class="px-6 py-4">
                <a href={cert.validIDUrl}>
                  <img
                    src={cert.validIDUrl}
                    alt="Valid ID"
                    class="h-10 w-10"
                  />
                </a>
              </td>
              <td class="px-6 py-4">
                <!-- {#if cert.dateOfAppointment == undefined}
                  <button
                    class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-full"
                    on:click={() => {
                      setDateHandler(cert, i);
                    }}
                  >
                    Set Date
                  </button>
                {:else} -->
                  <span
                    class="cursor-pointer"
                    on:click={() => {
                      setDateHandler(cert, i);
                    }}
                  >
                    {cert.dateOfAppointment ?? "Not set"}
                  </span>
                <!-- {/if} -->
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
                          ID="date_appointment"
                          ONCLICK={disableWeekends}
                          bind:this={bgyVarStore.dateOfAppointment}
                        />
                      </div>

                      <div class="flex gap-2">
                        <button
                          class="bg-orange-300 px-4 py-2 rounded-lg w-1/2"
                          on:click={setDate(cert.id)}
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
              <td class="px-6 py-4"> {cert.civilStatus} </td>
              {#if $showPrintModel}
                <td class="px-6 py-4"> {cert.gender} </td>
              {/if}
              {#if !$showPrintModel}
                <td class="px-6 py-4">
                  <select
                    class="bg-white"
                    bind:value={cert.gender}
                    on:change={() => updateGender(cert.id, cert.gender)}
                  >
                    <option value="Single">Male</option>
                    <option value="Married">Female</option>
                  </select>
                </td>
              {/if}
              <td class="px-6 py-4"> {cert.quantity ?? 1} </td>
              {#if $showPrintModel}
                <td class="px-6 py-4"> {cert.status} </td>
              {/if}

              {#if !$showPrintModel}
                <td class="px-6 py-4">
                  <select
                    class="bg-white"
                    bind:value={cert.status}
                    on:change={() =>
                      updateStatus(cert.id, cert.status, cert.appointmentOwner)}
                  >
                    <option value="Processing">On Process</option>
                    <option value="Ready for pickup">For Pickup</option>
                    <option value="Claimed">Completed</option>
                  </select>
                </td>
                <td class="px-6 py-4">
                  <div class="flex bg-slate-10 w-[30%] gap-2">
                    <button
                      class=" rounded-lg hover:scale-105 duration-700 text-black w-full p-2 hover:bg-orange-300 border-b-2 border-white"
                      on:click={() => {
                        editValueHandler(cert, i);
                      }}><i class="ri-edit-2-line" /></button
                    >

                    <button
                      class="font-bold rounded-lg hover:scale-105 duration-700 text-red-800 hover:text-white w-full p-2 hover:bg-red-600 border-b-2 border-white"
                      on:click={() => {
                        if (confirm('Are you sure you want to delete this item?')) {
                          removeData(cert.id);
                        }
                      }}
                      ><i class="ri-delete-bin-line" /></button
                    >

                    <button
                      class=" text-blue-800 w-full p-2 hover:bg-blue-600 border-b-2 hover:text-white rounded-lg duration-700"
                      on:click={() => printPdf(cert)}
                      ><i class="ri-printer-line"></i></button
                    >
                  </div>
                </td>
              {/if}
            </tr>
            {#if $showCertEditLogic && $compareCertValue == i}
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
                      on:click={updateData(cert.id)}
                    >
                      Update
                    </button>
                    <button
                      class="bg-red-300 px-4 py-2 rounded-lg w-1/2"
                      on:click={() => {
                        $showCertEditLogic = false;
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
