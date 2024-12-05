<!-- this page handles the consent -->

<script>
       let pdfPath = 'https://imensa.tiiny.site/'
    import { createEventDispatcher } from "svelte";
    import { studyLocation, studyAim, studyTasks, email } from "../utils.js";
    const dispatch = createEventDispatcher();
    let consentRejected = false;

    function handleYes() {
        dispatch("finished");
    }

    function handleNo() {
        consentRejected = true;
        dispatch("return");
    }
    
</script>

<div class="container">
    {#if consentRejected}
        <p>Thank you for your time. You may exit the page.</p>
    {:else}
        <h1>We need your consent to proceed</h1>
        
            <p class="intro">
                Please read the following material that explains this research
                study. We want you to understand what you are being asked to do
                and what risks and benefits --if any-- are associated with the
                study. Consent with this form will indicate that you have been
                informed about the study and that you want to participate. Please wait
                for the consent form to load onscreen before reviewing the experiment protocol.
            </p>
            <iframe src={pdfPath} width="100%" height="600px" style="border: none;"></iframe>


        <h2>Do you understand and consent to these terms?</h2>

        <button class="yes" on:click={handleYes}>I agree</button>
        <button class="no" on:click={handleNo}
            >No thanks, I do not want to do this study</button
        >
    {/if}
</div>

<style>
    .container {
        margin: 0 auto !important;
        align-items: center;
        text-align: center;
        width: 50%;
    }
   
    .intro {
        font-style: italic;
    }
    .yes {
        background-color: lightblue;
    }
    .no {
        background-color: lightcoral;
    }
</style>
