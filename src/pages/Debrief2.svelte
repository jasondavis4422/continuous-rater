<!-- this page is the debrief that collects post survey questions;
    there is a single button that saves responses to firebase and submits HIT  -->

    <script>
        import { db, params, serverTime } from '../utils.js';
        import { createEventDispatcher } from 'svelte';

    	let value = [0];
        let value1 = [0];
        let value2 = [0];
        let value3 = [0];
        let value4 = [0];
        
        const dispatch = createEventDispatcher();
        
        // populating necessary variables
        
        export let email;
        export let labName;
        export let numOptions;
        export let pathway;
        let emailAddress = "mailto:" + email;
        let currID = params.assignmentId;
        let postURL = params.turkSubmitTo + '/mturk/externalSubmit';

     
        function shuffle(array) {
            let currentIndex = array.length, randomIndex;
            // While there remain elements to shuffle.
            while (currentIndex != 0) {
                // Pick a remaining element.
                randomIndex = Math.floor(Math.random() * currentIndex);
                currentIndex--;
                // And swap it with the current element.
                [array[currentIndex], array[randomIndex]] = [
                    array[randomIndex], array[currentIndex]];
            }
            return array;
        }
        // Used like so
        var arr = ["happy", "sad", "fearful", "disgusted", "angry"];
        shuffle(arr);
        let Q1 = "After watching the video, how " + arr[0] + " do you feel on a scale from 1-100?";
        let Q2 = "After watching the video, how " + arr[1] + " do you feel on a scale from 1-100?";
        let Q3 = "After watching the video, how " + arr[2] + " do you feel on a scale from 1-100?";
        let Q4 = "After watching the video, how " + arr[3] + " do you feel on a scale from 1-100?";
        let Q5 = "After watching the video, how " + arr[4] + " do you feel on a scale from 1-100?";
  
        const submitHIT = async () => {
            
            try {
                let rating_info = [value, value1, value2, value3, value4];
                let dimensions = [arr[0], arr[1], arr[2], arr[3], arr[4]];
                await db.doc(pathway).update({
                    Ratings: rating_info,
                    Dimensions: dimensions,
                });     
            }
            catch (error) {
                console.error(error);
            }
            dispatch("finished"); 
        };
        
        const newPage = async () =>{
                let rating_info = [value, value1, value2, value3, value4];
                let dimensions = [arr[0], arr[1], arr[2], arr[3], arr[4]];
                dispatch("finished");  
                await db.doc(pathway).update({
                    Ratings: rating_info,
                    Dimensions: dimensions,
                });     
        }

        const nextPage = async () =>{
            let rating_info = [value, value1, value2, value3, value4];
            let dimensions = [arr[0], arr[1], arr[2], arr[3], arr[4]];
            console.log(rating_info);
            console.log(dimensions);
            dispatch("finished");
        }

      
    </script>
   
    <style>
        .container {
            margin: 0 auto !important; 
            max-width: 800px;
            text-align: center;
        }
        .form-box {
            padding: 2%;
                background-color: rgba(255, 255, 255, 0.6);
            border-left: 2px solid #aaa;
                border-radius: 2px;
                box-shadow: 2px 2px 8px rgba(0, 0, 0, 0.2);   
            text-align: left;
        }
            .label {
            font-weight: bold;
        }
    
        a {
            font-weight: bold;
        }

        .rangeslider {
          overflow: hidden;
           width: 200%;
           height: 200%;
}
        .button {
            background-color: lightblue;
        }
    </style>
    
    <div class="container">
        <div class="form-box">
            <form name="mturk" action={postURL} method='POST'>
                <h2> After watching the video, we have a set of questions for you to answer.</h2>
                <em> Please answer all questions and press submit once you've finished for another video. </em>

                <input type="hidden" name="assignmentId" id="assignmentId" value={currID}>
                <input type="hidden" name="hidden_val_DONT_REMOVE" value="1">

                <label class="label"><h3>{Q1}</h3>
                    <div class="rangeslider">
                    <label class="radio2">
                        <input type="range" min = "1" max = "100" bind:value={value}>
                        <p> Rating: {value} <span id="demo2"> </span> </p>
                    </label>

                    <label class="label"><h3>{Q2}</h3>
                        <div class="rangeslider">
                        <label class="radio2">
                            <input type="range" min = "1" max = "100" bind:value={value1}>
                            <p> Rating: {value1} <span id="demo2"> </span> </p>
                        </label>

                        <label class="label"><h3>{Q3}</h3>
                            <div class="rangeslider">
                            <label class="radio2">
                                <input type="range" min = "1" max = "100" bind:value={value2}>
                                <p> Rating: {value2} <span id="demo2"> </span> </p>
                            </label>

                            <label class="label"><h3>{Q4}</h3>
                                <div class="rangeslider">
                                <label class="radio2">
                                    <input type="range" min = "1" max = "100" bind:value={value3}>
                                    <p> Rating: {value3} <span id="demo2"> </span> </p>
                                </label>

                                <label class="label"><h3>{Q5}</h3>
                                    <div class="rangeslider">
                                    <label class="radio2">
                                        <input type="range" min = "1" max = "100" bind:value={value4}>
                                        <p> Rating: {value4} <span id="demo2"> </span> </p>
                                    </label>
                <p>
                    If you have any questions or concerns, you can email <a href={emailAddress}>{labName}.</a> 
                   <br>
                    If you would like to <strong>repeat this HIT with a new video</strong>, 
                    please simply click the next button. There are {numOptions} videos left to watch. Thanks! <br>
                </p>
                        
                <div class="field-label">
                    <!-- Left empty for spacing -->
                </div> 
                <br>
                <button class="button is-success is-large" on:click={newPage}>NEXT PAGE</button>         
            </form>
        </div> 
    </div>
