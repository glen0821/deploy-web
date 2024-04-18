<script>
  import Button from "../GeneralComponents/Button.svelte";
  import Inputs from "../GeneralComponents/Inputs.svelte";
  import ListIcon from "../Images/SVGs/ListIcon.svg";
  import { fly } from "svelte/transition";

  import { showPrintModel, formattedDate } from "./stateStore";

  import {
    showAdd,
    onSnaps,
    compareValue,
    printing,
  } from "../BoundComponents/clickOutside";

  //database calls and hooks
  import { auth, db } from "../../db/firebase";
  import {
    addDoc,
    collection,
    serverTimestamp,
    onSnapshot,
    query,
    orderBy,
    increment,
    deleteDoc,
    doc,
    setDoc,
    where,
  } from "firebase/firestore";
  // Declare and initialize statuses
  import reportHeader from "./images/header_report.png";

  //toggle show modal of add voter
  const showAddModal = () => {
    showAdd.set(true);
  };

  //handler for send data of addVoter
  const listOfVotersStore = {
    firstName: "",
    middleInitial: "",
    lastName: "",
    suffixName: "",
    precintNum: "",
    completeAddress: "",
    kwiri: "",
    trigger: false,
  };
 
  const addVoter = async () => {
    const colRef = collection(db, "votersList");
    await addDoc(colRef, {
      createdAt: serverTimestamp(),
      firstName: listOfVotersStore.firstName.BINDTHIS,
      middleInitial: listOfVotersStore.middleInitial.BINDTHIS,
      lastName: listOfVotersStore.lastName.BINDTHIS,
      suffixName: listOfVotersStore.suffixName.BINDTHIS,
      precintNumber: listOfVotersStore.precintNum.BINDTHIS,
      completeAddress: listOfVotersStore.completeAddress.BINDTHIS,
      voterCounter: increment(1),
    }).then(() => {
      listOfVotersStore.firstName.BINDTHIS = "";
      listOfVotersStore.lastName.BINDTHIS = "";
      listOfVotersStore.middleInitial.BINDTHIS = "";
      listOfVotersStore.suffixName.BINDTHIS = "";
      listOfVotersStore.precintNum.BINDTHIS = "";
      listOfVotersStore.completeAddress.BINDTHIS = "";
    });
  };

  let sample = where("precintNumber", "==", 500);

  //database loop data
  const colRef = collection(db, "votersList");
  let q = query(colRef, orderBy("createdAt", "desc"));
  const handleSearch = () => {
    if (listOfVotersStore.trigger) {
      q = query(
        colRef,
        orderBy("createdAt", "desc"),
        where("precintNumber", "==", Number(listOfVotersStore.kwiri))
      );

      onSnapshot(q, (snapshots) => {
        let fbData = [];
        snapshots.docs.forEach((doc) => {
          let data = { ...doc.data(), id: doc.id };
          fbData = [data, ...fbData];
        });
        onSnaps.set(fbData);
      });

      listOfVotersStore.trigger = false;
    }
  };

  $: editModalId = null;

  const openEditModal = (voter) => {
    editModalId = voter.id;
    setTimeout(function () {
      listOfVotersStore.firstName.BINDTHIS = voter.firstName;
      listOfVotersStore.middleInitial.BINDTHIS = voter.middleInitial;
      listOfVotersStore.lastName.BINDTHIS = voter.lastName;
      listOfVotersStore.suffixName.BINDTHIS = voter.suffixName;
      listOfVotersStore.precintNum.BINDTHIS = voter.precintNumber;
      listOfVotersStore.completeAddress.BINDTHIS = voter.completeAddress;
    }, 100);
  };

  const detectInputs = () => {
    if (listOfVotersStore.kwiri.trim().length < 1) {
      let q = query(colRef, orderBy("createdAt", "desc"));
      onSnapshot(q, (snapshots) => {
        let fbData = [];
        snapshots.docs.forEach((doc) => {
          let data = { ...doc.data(), id: doc.id };
          fbData = [data, ...fbData];
        });
        onSnaps.set(fbData);

        listOfVotersStore.trigger = false;
      });
    } else {
      listOfVotersStore.trigger = true;
    }
  };

  onSnapshot(q, (snapshots) => {
    let fbData = [];
    snapshots.docs.forEach((doc) => {
      let data = { ...doc.data(), id: doc.id };
      fbData = [data, ...fbData];
    });
    onSnaps.set(fbData);
  });

  //database remove data
  const removeData = async (data) => {
    const docRef = doc(colRef, data);
    await deleteDoc(docRef);
  };

  //database update data
  const updateData = async (data) => {
    const docRef = doc(colRef, data);
    setDoc(
      docRef,
      {
        lastUpdated: serverTimestamp(),
        firstName: listOfVotersStore.firstName.BINDTHIS,
        middleInitial: listOfVotersStore.middleInitial.BINDTHIS,
        lastName: listOfVotersStore.lastName.BINDTHIS,
        suffixName: listOfVotersStore.suffixName.BINDTHIS,
        precintNumber: listOfVotersStore.precintNum.BINDTHIS,
        completeAddress: listOfVotersStore.completeAddress.BINDTHIS,
      },
      { merge: true }
    ).then(() => {
      listOfVotersStore.firstName.BINDTHIS = "Hello";
      listOfVotersStore.lastName.BINDTHIS = "";
      listOfVotersStore.middleInitial.BINDTHIS = "";
      listOfVotersStore.suffixName.BINDTHIS = "";
      listOfVotersStore.precintNum.BINDTHIS = "";
      listOfVotersStore.completeAddress.BINDTHIS = "";
    });
  };

  //showEditModal
  const showEditModal = (i) => {
    compareValue.set(i);
  };

  //print data to external platforms like pdf etc
  const printFunc = () => {
    print();
  };

  const updateStatus = async (docID, selectedStatus, ownerID) => {
    const docRef = doc(colRef, docID);

    const updatedData = {
      verified: selectedStatus,
      lastUpdated: serverTimestamp(),
    };

    await setDoc(docRef, updatedData, { merge: true });

    const notifDocRef = doc(collection(db, "notifications"));
    let message;
    if (selectedStatus == "verified") {
      message = "Your account has been verified";
      var currentDate = Date.now();
      const notificationData = {
        message: message,
        userID: ownerID,
        timestamp: currentDate,
      };
      console.log(notificationData);
      await setDoc(notifDocRef, notificationData);
    }
  };

  const headerSortAscending = {
    firstName: false,
    middleInitial: undefined,
    lastName: undefined,
    suffixName: undefined,
    precintNum: undefined,
    completeAddress: undefined,
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
      onSnaps.set(fbData);
      console.log(fbData);

      listOfVotersStore.trigger = false;
    });
  };

  const resetSort = (key, value) => {
    let keys = Object.keys(headerSortAscending);
    for(let i = 0; i < keys.length; i++){
      headerSortAscending[keys[i]] = undefined;
    }
    headerSortAscending[key] = value;
  }

  sortTable("firstName", false);
</script>

<div
  class="m-2 mx-auto text-xs w-full min-h-screen"
  style="margin-bottom: {showPrintModel ? '20vh' : '0px'}"
>
  <div class="h-full w-full p-10 relative">
    <div class=" flex gap-2 items-center mb-2">
      <div class="w-full flex gap-2">
        <div class="">
          <Button TITLE="Add Voter" on:click={() => showAdd.set(true)} />
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
          placeholder="Precint Number Only"
          class="w-[40%] p-2 focus:outline-none border-2 border-black"
          on:keyup={detectInputs}
          bind:value={listOfVotersStore.kwiri}
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
                    TITLE="Print report"
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

    <!--ADD voters-->
    {#if $showAdd}
      <div
        class="flex flex-col gap-2 bg-guiColor p-4 max-w-fit mx-auto rounded-lg mt-2 absolute left-0 right-0 border-2 border-slate-200"
      >
        <p class="text-xl text-center font-bold p-2 text-slate-500">
          Add New Voter
        </p>
        <div class="flex gap-2 justify-center">
          <div class="">
            <Inputs
              TITLE="Name:"
              PLACEHOLDER="Complete Name"
              bind:this={listOfVotersStore.completeName}
            />
          </div>
          <div class="">
            <Inputs
              TITLE="PRECINT#:"
              PLACEHOLDER="Precint Number"
              TYPE="number"
              bind:this={listOfVotersStore.precintNum}
            />
          </div>
          <div class="">
            <Inputs
              TITLE="ADDRESS:"
              PLACEHOLDER="Complete Address"
              bind:this={listOfVotersStore.completeAddress}
            />
          </div>
        </div>

        <div class="flex gap-2">
          <Button TITLE="Submit" on:click={addVoter} />
          <Button
            TITLE="Close"
            on:click={() => {
              showAdd.set(false);
            }}
          />
        </div>
      </div>
    {/if}
    <!--End of add voters-->

    <div class="w-full h-full" in:fly={{ x: 400, duration: 1000 }}>
      <div class="relative h-96 w-full">
        {#if $showPrintModel}
          <img
            src={reportHeader}
            alt=""
            style="margin-top: -130px; margin-bottom: 100px"
          />
        {/if}
        <table class="w-full text-sm text-left text-gray-500 z-0">
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
                  resetSort('firstName', headerSortAscending.firstName)
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
                  if (headerSortAscending.precintNum == undefined) {
                    headerSortAscending.precintNum = true;
                  } else {
                    headerSortAscending.precintNum =
                      !headerSortAscending.precintNum;
                  }
                  resetSort("precintNum", headerSortAscending.precintNum);
                  sortTable("precintNumber", headerSortAscending.precintNum);
                }}
              >
                <div class="flex justify-center gap-2 items-center">
                  Precint #
                  <span
                    class={`sort-indicator 
                ${
                  headerSortAscending.precintNum == undefined
                    ? ""
                    : headerSortAscending.precintNum
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
                  length of stay
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
                  sortTable("completeAddress", headerSortAscending.completeAddress);
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
              <th scope="col" class="px-6 py-3">
                Age
              </th>
              {#if !$showPrintModel}
                <th scope="col" class="px-6 py-3">
                  Action
                  <span class="sort-indicator"></span>
                </th>
              {/if}
            </tr>
          </thead>

          <tbody>
            {#each $onSnaps as voter, i}
              <tr class="bg-white border-b">
                <th
                  scope="row"
                  class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap"
                >
                  {voter.firstName}
                </th>
                <td class="px-6 py-4">
                  {voter.middleInitial}
                </td>
                <td class="px-6 py-4">
                  {voter.lastName}
                </td>
                <td class="px-6 py-4">
                  {voter.suffixName}
                </td>
                <td class="px-6 py-4">{voter.precintNumber} </td>
                <td class="px-6 py-4">{voter.lengthOfStay} </td>
                <td class="px-6 py-4"> {voter.completeAddress} </td>
                <td class="px-6 py-4"> {voter.age} </td>

                {#if !$showPrintModel}
                  <td class="flex">
                    <select
                      class="bg-white"
                      bind:value={voter.verified}
                      on:change={() => {
                        updateStatus(voter.id, voter.verified, voter.id);
                      }}
                    >
                      <option value="to_verify">For verification</option>
                      <option value="verified">Verified</option>
                    </select>
                    <div class="flex space-x-5">
                      <button
                        class="hover:bg-orange-300 duration-700 px-4 p-2 rounded-full hover:text-black hover:font-bold hover:scale-105"
                        on:click={openEditModal(voter)}
                        ><i class="ri-edit-box-line text-lg" /></button
                      >

                      <button
                        class="hover:bg-red-300 duration-700 px-4 p-2 rounded-full hover:text-black hover:font-bold hover:scale-105"
                        on:click={() => {
                          removeData(voter.id);
                        }}><i class="ri-delete-bin-line text-lg" /></button
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

    {#if editModalId !== null}
      <div
        class="flex flex-col w-96 h-auto bg-white gap-2 p-4 rounded-lg absolute left-1/2 right-1/2 -translate-x-1/2 -translate-y-3/4 border-2 border-slate-200 z-10"
      >
        <p class="text-xl text-center font-bold p-2 text-black">
          Modify Values
        </p>
        <div class="flex gap-2 justify-center flex-col">
          <div class="">
            <Inputs
              TITLE="First Name"
              PLACEHOLDER="First Name"
              bind:this={listOfVotersStore.firstName}
            />
          </div>
          <div class="">
            <Inputs
              TITLE="M.I."
              PLACEHOLDER="Middle Initial"
              bind:this={listOfVotersStore.middleInitial}
            />
          </div>

          <div class="">
            <Inputs
              TITLE="Last Name"
              PLACEHOLDER="Last Name"
              bind:this={listOfVotersStore.lastName}
            />
          </div>
          <div class="">
            <Inputs
              TITLE="Suffix"
              PLACEHOLDER="Suffix"
              bind:this={listOfVotersStore.suffixName}
            />
          </div>
          <div class="">
            <Inputs
              TITLE="Percint #:"
              PLACEHOLDER="Precint Number"
              TYPE="number"
              bind:this={listOfVotersStore.precintNum}
            />
          </div>
          <div class="">
            <Inputs
              TITLE="Address:"
              PLACEHOLDER="Complete Address"
              bind:this={listOfVotersStore.completeAddress}
            />
          </div>
        </div>

        <div class="flex gap-2">
          <button
            class="w-1/2 bg-orange-300 text-base hover:scale-105 rounded-lg duration-700"
            on:click={() => {
              updateData(editModalId);
            }}>Confirm</button
          >
          <button
            class="bg-red-300 text-base hover:scale-105 rounded-lg duration-700 w-1/2"
            on:click={() => {
              editModalId = null;
            }}
            >close
          </button>
        </div>
      </div>
    {/if}
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
