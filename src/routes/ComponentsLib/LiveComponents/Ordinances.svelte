<script>
  import { writable } from "svelte/store";
  import { auth, db } from "../../db/firebase";
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
  } from "firebase/firestore";

  //database loop data
  const onSnaps = writable([]);
  const colRef = collection(db, "ordinancesList");
  let q = query(colRef, orderBy("createdAt", "desc"));
  onSnapshot(q, (snapshots) => {
    let fbData = [];
    snapshots.docs.forEach((doc) => {
      let data = { ...doc.data(), id: doc.id };
      fbData = [data, ...fbData];
    });
    onSnaps.set(fbData);
  });
  import SvelteMarkdown from "svelte-markdown";
  
  const removeData = async (data) => {
    const docRef = doc(colRef, data);
    await deleteDoc(docRef);
  };
</script>

<div class="markdown-editor">
  <div class="w-full flex gap-2 m-5">
    <button
      class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-full"
    on:click={()=>{
      window.location.href = "./create"
    }}
      >Create</button
    >
  </div>
  <div class="flex justify-center">
    <table class="w-11/12 text-sm text-left text-gray-500 z-0 p-5">
      <thead class="text-xs text-gray-700 uppercase bg-gray-50">
        <tr>
          <th scope="col" class="px-6 py-3" on:click={() => {}}>Title</th>
          <th scope="col" class="px-6 py-3" on:click={() => {}}>Content</th>
          <th scope="col" class="px-6 py-3" on:click={() => {}}>Date</th>
          <th scope="col" class="px-6 py-3" on:click={() => {}}>Actions</th>
        </tr>
      </thead>

      <tbody>
        {#each $onSnaps as ordinance, i}
          <tr class="bg-white border-b">
            <td
              scope="row"
              class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap"
            >
              {ordinance.title}
            </td>
            <td class="px-6 py-4" style="max-width: 400px; overflow:hidden">
              {ordinance.content}
            </td>
            <td class="px-6 py-4">
              {ordinance.createdAt.toDate().toString()}
            </td>
            <td>
              <button
                class="hover:bg-orange-300 duration-700 px-4 p-2 rounded-full hover:text-black hover:font-bold hover:scale-105"
                on:click={() => {
                  window.location.href = `/edit?id=${ordinance.id}`;
                }}
              >
                <i class="ri-edit-box-line text-lg" />
              </button>
              <button
                        class="hover:bg-red-500 rounded-full px-4 py-2 hover:scale-105 duration-700 text-red-900 hover:text-white"
                        on:click={() => {
                          if (confirm('Are you sure you want to delete this ordinance?')) {
                            removeData(ordinance.id);
                          }
                        }}
                        ><i class="ri-delete-bin-line"></i></button
                      >
            </td>
          </tr>
        {/each}
      </tbody>
    </table>
    
  </div>
</div>
