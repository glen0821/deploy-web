<script>
  import Button from "../GeneralComponents/Button.svelte";
  import NavSlider from "./NavSlider.svelte";
  import { showSlider, navSelections } from "./clickOutside";

  //live components
  import DashBoard from "../LiveComponents/DashBoard.svelte";

  //database calls and hooks
  import { auth } from "../../db/firebase";
  import { signOut } from "firebase/auth";
  import ListOfVoters from "../LiveComponents/ListOfVoters.svelte";
  import Analytics from "../LiveComponents/Analytics.svelte";
  import BarangayId from "../LiveComponents/BarangayID.svelte";
  import BarangayCertificate from "../LiveComponents/BarangayCertificate.svelte";
  import BarangayClearance from "../LiveComponents/BarangayClearance.svelte";
  import Complaint from "../LiveComponents/Complaint.svelte";
  import BarangayIndigency from "../LiveComponents/BarangayIndigency.svelte";
  import Ordinances from "../LiveComponents/Ordinances.svelte";
  import Officials from "../LiveComponents/Officials.svelte";

  //slide menu
  const showSliderMenu = () => {
    showSlider.set(true);
  };

  //logout user from system
  const logout = async () => {
    await signOut(auth).then(() => {
      localStorage.removeItem("uid");
    });
  };
</script>

<div class="">
  <div
    class="bg-white drop-shadow-lg border-b-slate-300 flex gap-2 items-center p-2"
  >
    <div class="">
      <div
        class="max-w-fit p-4 flex flex-col gap-2 rounded-full cursor-pointer hover:bg-orange-300 duration-700"
        on:keydown={() => {}}
        on:click={showSliderMenu}
      >
        <div class="w-6 border-b-black border-b-2"></div>
        <div class="w-6 border-b-black border-b-2"></div>
        <div class="w-6 border-b-black border-b-2"></div>
      </div>
    </div>

    <div class="w-full">
      <p class="font-bold sm:text-xl text-black">{$navSelections}</p>
    </div>

    <div class="w-[100px]">
      <Button TITLE="Logout" on:click={logout} />
    </div>
  </div>
  {#if $navSelections === "Dashboard"}
    <DashBoard />
  {:else if $navSelections === "Analytics"}
    <Analytics />
  {:else if $navSelections === "List of App users"}
    <ListOfVoters />
  {:else if $navSelections === "Barangay ID"}
    <BarangayId />
  {:else if $navSelections === "Barangay Certificate"}
    <BarangayCertificate />
  {:else if $navSelections === "Barangay Clearance"}
    <BarangayClearance />
  {:else if $navSelections === "Barangay Indigency"}
    <BarangayIndigency />
  {:else if $navSelections === "Complaints"}
    <Complaint />
  {:else if $navSelections === "Ordinances"}
    <Ordinances />
    {:else if $navSelections === "Officials"}
    <Officials />
  {/if}
</div>

<NavSlider />
