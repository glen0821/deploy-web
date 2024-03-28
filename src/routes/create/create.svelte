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
        where,
    } from "firebase/firestore";
    let showPreview = true;
    let markdownContent = "";
    let ordinanceTitle = "";

    function togglePreview() {
        showPreview = !showPreview;
    }
    const submitOrdinance = async () => {
        const colRef = collection(db, "ordinancesList");
        await addDoc(colRef, {
            content: markdownContent,
            title: ordinanceTitle,
            createdAt: serverTimestamp()
        }).then(() => {
            window.location.href = "/";
        });
    };
</script>

<div class="markdown-editor">
    <h1>Create A New Ordinance</h1>
    <input type="text" id="title" placeholder="Title"
    bind:value={ordinanceTitle}>
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
</div>

<style>
</style>
