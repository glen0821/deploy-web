<script>
  import { writable } from "svelte/store";
  import { auth, db, storage } from "../../db/firebase";
  import Button from "../GeneralComponents/Button.svelte";

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
    Timestamp,
    updateDoc,
  } from "firebase/firestore";

  //database loop data
  const onSnaps = writable([]);
  const colRef = collection(db, "skofficialsList");
  let q = query(colRef, orderBy("name", "desc"));
  onSnapshot(q, (snapshots) => {
    let fbData = [];
    snapshots.docs.forEach((doc) => {
      let data = { ...doc.data(), id: doc.id };
      fbData = [data, ...fbData];
    });
    console.log(fbData);
    onSnaps.set(fbData);
  });

  const OfficialDetailsStore = {
    name: "",
    position: "",
    profileURL: "",
    file: null,
  };

  const showAddModal = writable(false);
  const showEditModal = writable(false);
  import Inputs from "../GeneralComponents/Inputs.svelte";
  import { ref, uploadBytes, getDownloadURL, listAll } from "firebase/storage";

  const handleFileUpload = async () => {
    const fileInput = document.getElementById("file_input");
    const file = fileInput.files[0];
    if(file == null){
      return null
    }
    const storageRef = ref(storage, `images_skofficial/${file.name}`);
    const result = await uploadBytes(storageRef, file);
    console.log(result);
    const downloadURL = await getDownloadURL(result.ref);
    return downloadURL;
  };

  let editID = "";

  const submitData = async () => {
    const colRef = collection(db, "skofficialsList");
    const profileURL = await handleFileUpload();
    const data = {
      name: OfficialDetailsStore.name.BINDTHIS,
      position: OfficialDetailsStore.position.BINDTHIS,
    };
    if(profileURL != null){
      data.profileURL = profileURL
    }
    console.log(data);
    if($showEditModal == true){
     await updateDoc(doc(colRef, editID), data).then(()=> {
        showAddModal.set(false);
      showEditModal.set(false);
      })
      return
    }
    await addDoc(colRef, data).then(() => {
      showAddModal.set(false);
      showEditModal.set(false);
    });
  };

  const editValueHandler = (data) => {
    showEditModal.set(true);
    editID = data.id;
    setTimeout(function () {
      OfficialDetailsStore.name.BINDTHIS = data.name;
      OfficialDetailsStore.position.BINDTHIS = data.position;
    }, 100);
  };
  const removeData = async (data) => {
    const docRef = doc(colRef, data);
    await deleteDoc(docRef);
  };
</script>

{#if ($showAddModal || $showEditModal)}
  <div
    class="flex flex-col gap-2 bg-white p-4 max-w-fit mx-auto rounded-lg mt-2 absolute left-0 right-0 border-2 border-guiColor z-10"
  >
    <p class="text-xl text-center font-bold p-2 text-slate-500">Add Official</p>
    <div class="">
      <Inputs
        TITLE="Complete Name:"
        PLACEHOLDER="Complete Name"
        bind:this={OfficialDetailsStore.name}
      />
    </div>
    <div class="">
      <Inputs
        TITLE="Position"
        PLACEHOLDER="Position"
        TYPE="off-positions"
        bind:this={OfficialDetailsStore.position}
      />
    </div>
    <div class="">
      <Inputs
        TITLE="Profile"
        PLACEHOLDER="Image.png"
        TYPE="file"
        ID="file_input"
      />
    </div>
    <div class="flex gap-2">
      <Button
        TITLE="Submit"
        on:click={() => {
          submitData();
        }}
      />
      <Button
        TITLE="Cancel"
        on:click={() => {
          showAddModal.set(false);
          showEditModal.set(false);
        }}
      />
    </div>
  </div>
{/if}

<div class="markdown-editor">
  <div class="w-full flex gap-2 m-5">
    <button
      class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-full"
      on:click={() => {
        showAddModal.set(true);
      }}>Add</button
    >
  </div>
  <div class="flex justify-center">
    <table class="w-11/12 text-sm text-left text-gray-500 z-0 p-5">
      <thead class="text-xs text-gray-700 uppercase bg-gray-50">
        <tr>
          <th scope="col" class="px-6 py-3" on:click={() => {}}>Profile</th>
          <th scope="col" class="px-6 py-3" on:click={() => {}}>Name</th>
          <th scope="col" class="px-6 py-3" on:click={() => {}}>Position</th>
          <th scope="col" class="px-6 py-3" on:click={() => {}}>Actions</th>
        </tr>
      </thead>
      <tbody>
        {#each $onSnaps as official, i}
          <tr class="bg-white border-b">
            <td
              scope="row"
              class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap"
            >
              <img
                src={official.profileURL}
                alt=""
                srcset=""
                class="w-24 h-24 rounded"
              />
            </td>
            <td class="px-6 py-4">
              {official.name}
            </td>
            <td class="px-6 py-4">
              {official.position.toUpperCase()}
            </td>
            <td>
              <button
                class="hover:bg-orange-300 duration-700 px-4 p-2 rounded-full hover:text-black hover:font-bold hover:scale-105"
                on:click={() => {
                  editValueHandler(official);
                }}
              >
                <i class="ri-edit-box-line text-lg" />
              </button>
              <button
                        class="hover:bg-red-500 rounded-full px-4 py-2 hover:scale-105 duration-700 text-red-900 hover:text-white"
                        on:click={removeData(official.id)}
                        ><i class="ri-delete-bin-line"></i></button
                      >
            </td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
</div>
