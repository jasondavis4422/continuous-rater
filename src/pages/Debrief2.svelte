<!-- this page is the debrief that collects post survey questions;
    there is a single button that saves responses to firebase and submits HIT  -->

    <script>
        import { db, params, serverTime, experiment, userGroup } from '../utils'
        import { createEventDispatcher } from 'svelte';

    	let value = [0];
        let value1 = [0];
        let value2 = [0];
        let value3 = [0];
        let value4 = [0];
        let value5 = [0];
        let value6 = [0];

        const ratingsPath = `${experiment}/ratings`;
	    const ratingsDoc = db.doc(ratingsPath);
	    const subjectGroupPath = `${experiment}/subjects/${userGroup}`;
	    const subjectGroupCollection = db.collection(subjectGroupPath);
	    const stimuliPath = `${experiment}/stimuli`;
	    const stimuliDoc = db.doc(stimuliPath);

        const dispatch = createEventDispatcher();
        
        // populating necessary variables
        
        export let email;
        export let labName;
    
        export let videoIndex;
        
        export let ratingType;
	    export let movies;
	    export let options;
	    export let links;
	    export let index; 

        let emailAddress = "mailto:" + email;
        let currID = params.assignmentId;
        let postURL = 'https://www.prolific.com/'
        let moviesRemaining = [];
        let numVideos = 9;

        let currVid;
	    let currVidSrc;
	    let ratingDocPathway;
        let botCheck = 2;

        if (options > 0) {
		// choose random movie and rating type
		currVid = movies[index];

		let vidPlusRating = `${currVid}-${ratingType}`;

		ratingDocPathway = `${ratingsPath}/${params.workerId}/${vidPlusRating}`;

		// grab URL for video sourcing
		currVidSrc = links[index];
	
	}
    
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
        var arr = ["joyful", "warm and tender", "inspired or uplifted", "sad", "disgusted", "ashamed", "horrified"];
        shuffle(arr);
        let Q1 = "How " + arr[0] + " did the previous video make you feel?";
        let Q2 = "How " + arr[1] + " did the previous video make you feel?";
        let Q3 = "How " + arr[2] + " did the previous video make you feel?";
        let Q4 = "How " + arr[3] + " did the previous video make you feel?";
        let Q5 = "How " + arr[4] + " did the previous video make you feel?";
        let Q6 = "How " + arr[5] + " did the previous video make you feel?";
        let Q7 = "How " + arr[6] + " did the previous video make you feel?";

 
        const submitHIT = async () => {
            
            try {
                let rating_info = [value, value1, value2, value3, value4, value5, value6];
                let dimensions = [arr[0], arr[1], arr[2], arr[3], arr[4], arr[5], arr[6]];
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
          
            
           if (videoIndex != numVideos){
            let rating_info = [value, value1, value2, value3, value4, value5, value6];
            let dimensions = [arr[0], arr[1], arr[2], arr[3], arr[4], arr[5], arr[6]];
                dispatch("finished")
                await db.doc(ratingDocPathway).update({
                    Ratings: rating_info,
                    Dimensions: dimensions,
                });    
            } 
            else
            {
                let rating_info = [value, value1, value2, value3, value4, value5, value6];
                let dimensions = [arr[0], arr[1], arr[2], arr[3], arr[4], arr[5], arr[6]];
                dispatch("complete")
                await db.doc(ratingDocPathway).update({
                    Ratings: rating_info,
                    Dimensions: dimensions,
                });   
            }
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
           width: 450%;
           height: 200%;
}
        .button {
            background-color: lightblue;
        }
    </style>
    
    <div class="container">
        <div class="form-box">
            <form name="mturk" action={postURL} method='POST'>
                <h2> We have a few questions about the previous video. Please rate your feelings on a scale from 1 (Barely at all) to 100 (Strongest imaginable).</h2>
                <em> Please answer all questions, then press “NEXT PAGE” to move on to the next question. </em>

                <input type="hidden" name="assignmentId" id="assignmentId" value={currID}>
                <input type="hidden" name="hidden_val_DONT_REMOVE" value="1">

                <label class="label"><h3>{Q1}</h3>
                    <div class="rangeslider">
                    <label class="radio2">
                        <input type="range" min = "1" max = "100" bind:value={value}>
                        <p> {arr[0]}: {value} <span id="demo2"> </span> </p>
                    </label>

                    <label class="label"><h3>{Q2}</h3>
                        <div class="rangeslider">
                        <label class="radio2">
                            <input type="range" min = "1" max = "100" bind:value={value1}>
                            <p> {arr[1]}: {value1} <span id="demo2"> </span> </p>
                        </label>

                        <label class="label"><h3>{Q3}</h3>
                            <div class="rangeslider">
                            <label class="radio2">
                                <input type="range" min = "1" max = "100" bind:value={value2}>
                                <p> {arr[2]}: {value2} <span id="demo2"> </span> </p>
                            </label>

                            <label class="label"><h3>{Q4}</h3>
                                <div class="rangeslider">
                                <label class="radio2">
                                    <input type="range" min = "1" max = "100" bind:value={value3}>
                                    <p> {arr[3]}: {value3} <span id="demo2"> </span> </p>
                                </label>

                                <label class="label"><h3>{Q5}</h3>
                                    <div class="rangeslider">
                                    <label class="radio2">
                                        <input type="range" min = "1" max = "100" bind:value={value4}>
                                        <p> {arr[4]}: {value4} <span id="demo2"> </span> </p>
                                    </label>

                                    <label class="label"><h3>{Q6}</h3>
                                        <div class="rangeslider">
                                        <label class="radio2">
                                            <input type="range" min = "1" max = "100" bind:value={value5}>
                                            <p> {arr[5]}: {value5} <span id="demo2"> </span> </p>
                                        </label>

                                        <label class="label"><h3>{Q7}</h3>
                                            <div class="rangeslider">
                                            <label class="radio2">
                                                <input type="range" min = "1" max = "100" bind:value={value6}>
                                                <p> {arr[6]}: {value6} <span id="demo2"> </span> </p>
                                            </label>
               
                        
                <div class="field-label">
                    <!-- Left empty for spacing -->
                </div> 
                <br>
                <button class="button is-success is-large" on:click={newPage}>NEXT PAGE</button>         
            </form>
        </div> 
    </div>
