<!-- second instruction page -->

<script>
  // This is the Instructions page. It loops over the instructions array as a user reads and when click to the last page it notifies the main App.svelte component by dispatching a 'finished' event. When the last page of the instructions are reached the forward button turns into a "Take Quiz" button, but currently there is no quiz and it goes straight to the experiment
  import { createEventDispatcher } from "svelte";

  // Add/remove items here to create more instructions pages

 

  const instructions =
  ["If you'd like to practice the continuous rating box again, click the Back button. \
\
On the next page, you will watch a video lasting 3 minutes and rate one specific emotion moment by moment. Please adjust the rating level to reflect your current emotional experience as you watch. After the video, you will be asked to answer follow-up questions about the video. \
\
Please watch it without pausing to ensure better data quality. You can take a break before starting the next clip, after completing the follow-up questions. When you're ready to begin, click the Go to Task button to proceed. Click on the video panel on the next page to start the video.\
"];

  const dispatch = createEventDispatcher();
  let currentPage = 0;

  const backward = () => {
    dispatch("back");
  };
  const forward = () => {
    // Check if we're increasing the value of currentPage beyond the number of instructions, if so tell App.svelte we're ready to move to the quiz
    if (currentPage + 1 === instructions.length) {
      dispatch("finished");
    } else {
      currentPage += 1;
      currentPage = Math.min(currentPage, instructions.length - 1);
    }
  };
</script>

<div class="container">
  <h1>Instructions</h1>
  <div class="text-box">
    <div class="content">
      {@html instructions[currentPage]}
    </div>
  </div>
  <br />
  <button class="back" on:click={backward}>Back</button>
  <button class="next" on:click={forward}>
    {#if currentPage === instructions.length - 1}
      Go To Task
    {:else}
      Next
    {/if}
  </button>
</div>

<style>
  .container {
    margin: 0 auto !important;
    max-width: 800px;
    text-align: center;
  }

  .text-box {
    text-align: left;
    padding: 2%;
    background-color: rgba(255, 255, 255, 0.4);
    border: 2px solid grey;
    border-radius: 2px;
    box-shadow: 2px 2px 8px rgba(0, 0, 0, 0.1);
  }

  .next {
    background-color: lightblue;
  }

  .back {
    background-color: lightcoral;
  }
</style>
