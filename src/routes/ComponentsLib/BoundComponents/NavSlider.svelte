<svelte:options accessors />

<script>
  import { fly } from "svelte/transition";
  import { createEventDispatcher } from "svelte";
  import logo from "../Images/logo.png";
  import { showSlider, navSelections } from "./clickOutside.js";

  import dashboardIcon from "../Images/SVGs/dashb.svg";
  import analyticsIcon from "../Images/SVGs/analytics.svg";
  import listIcon from "../Images/SVGs/ListIcon.svg";
  import brgyID from "../Images/SVGs/brgyID.svg";
  import bgryCert from "../Images/SVGs/brgyCert.svg";
  import brgyClear from "../Images/SVGs/brgyClear.svg";
  import brgyIndi from "../Images/SVGs/brgyIndi.svg";
  import complaints from "../Images/SVGs/complaints.svg";
  import Button from "../GeneralComponents/Button.svelte";

  //slider list of menu
  let selections = [
    "Dashboard",
    "List of registered voters",
    "Barangay ID",
    "Barangay Certificate",
    "Barangay Clearance",
    "Barangay Indigency",
    "Complaints",
    "Ordinances",
  ];

  let newSelections = [
    {
      svg: dashboardIcon,
      nav: "Dashboard",
    },
    {
      svg: analyticsIcon,
      nav: "Analytics",
    },
    {
      svg: listIcon,
      nav: "List of App users",
    },

    {
      svg: brgyID,
      nav: "Barangay ID",
    },

    {
      svg: bgryCert,
      nav: "Barangay Certificate",
    },

    {
      svg: brgyClear,
      nav: "Barangay Clearance",
    },
    
    {
      svg: brgyClear,
      nav: "Barangay Indigency",
    },
    
    {
      svg: complaints,
      nav: "Complaints",
    },
    {
      svg: complaints,
      nav: "Ordinances",
    },
    {
      svg: complaints,
      nav: "Officials",
    },
    {
      svg: complaints,
      nav: "SK Officials",
    },
  ];

  //value grabber from menu
  const getValue = (data) => {
    navSelections.set(data);
    showSlider.set(false);
  };

  //close slider
  const back = () => {
    showSlider.set(false);
  };
</script>

{#if $showSlider}
  <div
    class="bg-white absolute min-w-[20vw] top-0 bottom-0 border-slate-400 border-2 z-20"
    in:fly={{ x: -200, duration: 1000 }}
  >
    <div class="flex space-x-4 p-2">
      <div>
        <img src={logo} alt="loading" class="w-10" />
      </div>

      <div class="flex justify-center text-center pt-2">
        <p id="title" class="text-base text-black italic">
          SN3 Appointment System
        </p>
      </div>
      <div class="pt-2">
        <button on:click={back}><i class="ri-close-circle-fill text-lg text-orange-600" /></button>
      </div>
    </div>

    <div class="flex justify-center">
      <p class="font-bold text-lg p-4">Hello, Administrator</p>
    </div>

    <div class="">
      {#each newSelections as value}
        <div class="p-2">
          <button
            class="flex gap-2 items-center w-full p-2 transition-all duration-700 hover:scale-105 hover:bg-orange-400 active:scale-105"
            class:active={$navSelections === value.nav}
            on:click={() => {
              getValue(value.nav);
            }}
          >
            <img src={value.svg} alt="loading" class="w-5" />
            <p class="font-semibold text-sm">{value.nav}</p>
          </button>
        </div>
      {/each}
    </div>
  </div>
{/if}

<style>
  .active {
    background-color: #fd951f;
  }
  #title {
    font-family: cursive;
  }
</style>
