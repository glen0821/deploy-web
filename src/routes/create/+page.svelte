<script>
    import Login from "../ComponentsLib/Authentication/Login.svelte";
    import Create from "./create.svelte";

    //database calls and hooks
    import { auth } from "../db/firebase";
    import { onAuthStateChanged } from "firebase/auth";
    import { onMount } from "svelte";
    import { showNav } from "../ComponentsLib/BoundComponents/clickOutside";

    onMount(() => {
        onAuthStateChanged(auth, (userCred) => {
            if (userCred) {
                showNav.set(true);
            } else {
                showNav.set(false);
                window.location.href = "/";
            }
        });
    });
</script>

<main>
    {#if $showNav}
        <Create />
    {/if}
</main>
