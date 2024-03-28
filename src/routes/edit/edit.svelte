<script>
    import SvelteMarkdown from "svelte-markdown";
    import { auth, db } from "../db/firebase";
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
        getDoc,
        where,
        updateDoc
    } from "firebase/firestore";
    import { onMount } from "svelte";
    let showPreview = true;
    let markdownContent = "";
    let ordinanceTitle = "";
    let id = "";

    function togglePreview() {
        showPreview = !showPreview;
    }
    const submitOrdinance = async () => {
        const colRef = collection(db, "ordinancesList");
        await updateDoc(doc(colRef, id), {
            content: markdownContent,
            title: ordinanceTitle,
            createdAt: serverTimestamp(),
        }).then(() => {
            window.location.href = "/";
        });
    };

    const getDocById = async (id) => {
        const docRef = doc(db, "ordinancesList", id);
        const docSnap = await getDoc(docRef);
        if (docSnap.exists()) {
            return docSnap.data();
        } else {
            console.log("No such document!");
            return null;
        }
    };


    onMount(async () => {
        const urlParams = new URLSearchParams(window.location.search);
        id = urlParams.get("id");
        console.log(id);
        if (id == null) {
            window.location.href = "/";
        } else {
            const docData = await getDocById(id);
            ordinanceTitle = docData.title
            markdownContent = docData.content
        }
    });
</script>

<div class="markdown-editor">
    <h1>View/Edit Ordinance</h1>
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
    <button class="cms-submit" on:click={submitOrdinance}> Update </button>
</div>

<style>
</style>
