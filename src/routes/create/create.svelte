<script>
  import SvelteMarkdown from "svelte-markdown";
  import { auth, db, storage } from "../db/firebase";
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
  import { onMount } from "svelte";
  import { ref, uploadBytes, getDownloadURL, listAll } from "firebase/storage";
  import { writable } from "svelte/store";

  let showPreview = true;
  let markdownContent = "";
  let ordinanceTitle = "";

  function togglePreview() {
    showPreview = !showPreview;
  }
  const submitOrdinance = async () => {
    const colRef = collection(db, "ordinancesList");
    const result = await addDoc(colRef, {
      content: markdownContent,
      title: ordinanceTitle,
      createdAt: serverTimestamp(),
    });
    window.location.href = `/edit?id=${result.id}`
  };

  let uploadedImages = writable([]);
  const handleFileUpload = async () => {
    const fileInput = document.getElementById("file_input");
    const file = fileInput.files[0];
    const storageRef = ref(storage, `images_ordinance/${file.name}`);
    const result = await uploadBytes(storageRef, file);
    console.log(result);
    const downloadURL = await getDownloadURL(result.ref);
    console.log(downloadURL);
    // uploadedImages = [...uploadedImages, downloadURL];
    uploadedImages.update((images) => [...images, downloadURL]);
  };

  onMount(async () => {
    const storageRef = ref(storage);
    const imagesFolderRef = ref(storageRef, "images_ordinance");
    const imageList = await listAll(imagesFolderRef);
    for (const imageRef of imageList.items) {
      const url = await getDownloadURL(imageRef);
      //   uploadedImages.push(url);
      uploadedImages.update((images) => [...images, url]);
    }
    console.log($uploadedImages);
  });

  const copyMarkdownToClipboard = (image) => {
    const markdownSyntax = `![alt_text](${image} "title")`;
    navigator.clipboard
      .writeText(markdownSyntax)
      .then(() => {
        // toast.success('Markdown copied to clipboard!', {
        //     timeout: 3000,
        // });
        showToast("Copied to clipboard");
      })
      .catch((error) => {
        console.error("Failed to copy Markdown to clipboard: ", error);
        // toast.error('Failed to copy Markdown to clipboard!');
      });
  };

  import {
    toasts,
    ToastContainer,
    FlatToast,
    BootstrapToast,
  } from "svelte-toasts";

  const showToast = (message) => {
    const toast = toasts.add({
      title: message,
      description: "Paste it on edit content",
      duration: 10000, // 0 or negative to avoid auto-remove
      placement: "bottom-right",
      type: "info",
      theme: "dark",
      placement: "bottom-right",
      type: "success",
      onClick: () => {},
      onRemove: () => {},
      // component: BootstrapToast, // allows to override toast component/template per toast
    });

    // toast.remove()
  };
</script>

<div class="markdown-editor">
  <h1>Create A New Ordinance</h1>
  <input
    type="text"
    id="title"
    placeholder="Title"
    bind:value={ordinanceTitle}
  />
  <button class="toggle-button" on:click={togglePreview}>
    {showPreview ? "Edit Content" : "Show Preview"}
  </button>
  {#if showPreview}
    <div class="preview">
      <SvelteMarkdown source={markdownContent} />
    </div>
  {:else}
    <div class="editor-container">
      <textarea
        bind:value={markdownContent}
        placeholder="Type your markdown here..."
      ></textarea>
    </div>
  {/if}
  <button class="cms-submit" on:click={submitOrdinance}> Submit </button>
  <div class="images" id="images_container">
    <div class="list">
      <!-- put the list of uploaded images here  -->
      {#each $uploadedImages as image}
        <img
          src={image}
          alt="Uploaded Image"
          on:click={() => copyMarkdownToClipboard(image)}
        />
      {/each}
    </div>
    <div class="upload">
      <input
        type="file"
        name=""
        id="file_input"
        accept="image/png, image/gif, image/jpeg"
      />
      <button on:click={handleFileUpload}>Upload</button>
    </div>
  </div>
  <ToastContainer placement="bottom-right" let:data>
    <FlatToast {data} />
  </ToastContainer>
</div>
