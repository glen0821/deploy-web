<script>
  import Button from "../GeneralComponents/Button.svelte";
  import Inputs from "../GeneralComponents/Inputs.svelte";
  import PdfTemplate from "./PdfTemplate.svelte";
  import { fly } from "svelte/transition";
  import html2pdf from "html2pdf.js";
  import {
    cordionLogic,
    onSnapsBgyCert,
    showCertEditLogic,
    showCertAddModal,
    compareCertValue,
    showEditModalLogic,
    printing
  } from "../BoundComponents/clickOutside";

  import { showPrintModel, formattedDate } from "./stateStore";
  import bgyCle from "../Images/bgyClearance.jpg";

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
  import { onMount } from "svelte";
  // import PrintContent from "./PrintContent.svelte";
  import CertificateContent from "./CertificateContent.svelte";
  import reportHeader from "./images/header_report.png";


  // $: console.log(html2pdf);

  const opt = {
    margin: 1,
    filename: "myfile.pdf",
    image: { type: "jpeg", quality: 1.0 },
    html2canvas: { scale: 2 },
    jsPDF: { unit: "in", format: "legal", orientation: "portrait" },
  };

  let pdfElement,
    pdfElementHidden = true,
    printData = {};

  onMount(() => {
    pdfElement = document.getElementById("pdf-template");
  });

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
    const printContent = new CertificateContent({
      target: mywindow.document.body,
      props: {
        documentTitle: "hello world",
        mywindow: mywindow,
        ...dataForPrint,
      },
    });
    console.log(printContent);
    return true;
  }
  

  //handler to show add modal
  const toShowAddModal = () => {
    showCertAddModal.set(true);
  };

  //handler to show edit modal
  const toShowEditModal = () => {
    showCertEditLogic.set(true);
  };

  //barangayID varStore
  const bgyVarStore = {
    completeName: "",
    address: "",
    birthdate: "",
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

  //fetch data from database
  const colRef = collection(db, "barangayCertificate");
  const q = query(colRef, orderBy("createdAt", "desc"));
  onSnapshot(q, (snapshots) => {
    let fbData = [];
    snapshots.docs.forEach((doc) => {
      let data = { ...doc.data(), id: doc.id };
      fbData = [data, ...fbData];
    });
    onSnapsBgyCert.set(fbData);
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
      message = "Your certificate is being processed";
    } else if (selectedStatus == "Ready for pickup") {
      message =
        "Your certificate is ready for pickup, get it before or on appointed date";
    } else {
      message = "Your certificate has been claimed";
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

  //updateData from database
  const updateData = async (data) => {
    const docRef = doc(colRef, data);
    setDoc(
      docRef,
      {
        lastUpdated: serverTimestamp(),
        completeName: bgyVarStore.completeName.BINDTHIS,
        address: bgyVarStore.address.BINDTHIS,
        birthdate: bgyVarStore.birthdate.BINDTHIS,
        lengthOfStay: bgyVarStore.lengthOfStay.BINDTHIS,
        purpose: bgyVarStore.purpose.BINDTHIS,
        dateOfAppointment: bgyVarStore.dateOfAppointment.BINDTHIS,
      },
      { merge: true }
    );
  };

  let editData = {};

  //showModalComparison
  const editValueHandler = (data) => {
    editData = { ...data };
    // console.log(data);
    toShowEditModal();
    // compareCertValue.set(data);

    console.log(editData);
  };

  const handlerSearch = () => {
    if (bgyVarStore.trigger) {
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
        onSnapsBgyCert.set(fbData);
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
        onSnapsBgyCert.set(fbData);
      });
      bgyVarStore.trigger = false;
    } else {
      bgyVarStore.trigger = true;
    }
  };
  const headerSortAscending = {
    firstName: false,
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
    let q = query(colRef, orderBy(fieldName, isAscending ? "asc" : "desc"));
    onSnapshot(q, (snapshots) => {
      let fbData = [];
      snapshots.docs.forEach((doc) => {
        let data = { ...doc.data(), id: doc.id };
        fbData = [data, ...fbData];
      });
      onSnapsBgyCert.set(fbData);
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

  sortTable("firstName", false);
</script>

<PdfTemplate
  printFunc={printPdf}
  isHidden={pdfElementHidden}
  {...printData}
  }
/>

<div class="m-2 mx-auto text-xs" style="margin-bottom: {showPrintModel? "20vh" : "0px"}">
  <div class="min-h-[50vh] p-10">
    <div class=" flex gap-2 items-center mb-2">
      <div class="w-full flex gap-2">
        <div class="">
          <Button TITLE="Add Barangay Certificate" on:click={toShowAddModal} />
        </div>

        <div class="">
          <button
            class="text-xs py-2 px-4 bg-orange-300 rounded-lg hover:bg-orange-400 hover:scale-105 duration-700 font-bold hover:text-white"
            on:click={() => {$showPrintModel = true;}}
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
          <div class="fixed bottom-0 right-0 p-10"  style="bottom: -10px !important;">
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

    {#if $showCertEditLogic}
      <div
        class="flex flex-col gap-2 bg-white p-4 max-w-fit mx-auto rounded-lg mt-2 absolute left-0 right-0 border-2 border-guiColor z-10"
      >
        <p class="text-xl text-center font-bold p-2 text-slate-500">
          Update Barangay Certificate
        </p>
        <div class="">
          <Inputs
            TITLE="Complete Name:"
            PLACEHOLDER={editData.completeName}
            bind:this={bgyVarStore.completeName}
          />
        </div>

        <div class="flex justify-center gap-2">
          <div class="">
            <Inputs
              TITLE="Length of stay"
              PLACEHOLDER={editData.lengthOfStay}
              bind:this={bgyVarStore.lengthOfStay}
            />
          </div>
          <div class="">
            <Inputs
              TITLE="Purpose"
              PLACEHOLDER={editData.purpose}
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
            PLACEHOLDER={editData.address}
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
          <Button TITLE="Submit" on:click={updateData(editData.id)} />
          <Button
            TITLE="Cancel"
            on:click={() => {
              showCertEditLogic.set(false);
            }}
          />
        </div>
      </div>
    {/if}

    <div class="relative ">
      {#if $showPrintModel}
      <img src={reportHeader} alt="" style="margin-top: -130px; margin-bottom: 100px" />
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
                  resetSort("completeAddress", headerSortAscending.completeAddress);
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
                  headerSortAscending.age =
                    !headerSortAscending.age;
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
                resetSort("dateOfAppointment", headerSortAscending.dateOfAppointment);
                sortTable("dateOfAppointment", headerSortAscending.dateOfAppointment);
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
            <th scope="col" class="px-6 py-3"> status </th>
            {#if !$showPrintModel}
              <th scope="col" class="px-6 py-3"> action </th>
              {/if}
          </tr>
        </thead>
        <tbody>
          {#each $onSnapsBgyCert as cert}
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
              <td class="px-6 py-4"> {cert.dateOfAppointment} </td>
              {#if $showPrintModel}
              <td class="px-6 py-4"> {cert.status} </td>
              {/if}
            {#if !$showPrintModel}
              <td class="px-6 py-4">
                <select
                  class="bg-white"
                  bind:value={cert.status}
                  on:change={() => updateStatus(cert.id, cert.status, cert.appointmentOwner)}
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
                      editValueHandler(cert);
                    }}><i class="ri-edit-2-line" /></button
                  >

                  <button
                    class="font-bold rounded-lg hover:scale-105 duration-700 text-red-800 hover:text-white w-full p-2 hover:bg-red-600 border-b-2 border-white"
                    on:click={removeData(cert.id)}
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
