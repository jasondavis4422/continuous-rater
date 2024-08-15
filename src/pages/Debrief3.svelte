<!-- this page is the debrief that collects post survey questions;
    there is a single button that saves responses to firebase and submits HIT  -->

    <script>
        import { db, params, serverTime, experiment, userGroup } from '../utils'
        import { createEventDispatcher } from 'svelte';


        const ratingsPath = `${experiment}/ratings`;
	    const ratingsDoc = db.doc(ratingsPath);
	    const subjectGroupPath = `${experiment}/subjects/${userGroup}`;
	    const subjectGroupCollection = db.collection(subjectGroupPath);
	    const stimuliPath = `${experiment}/stimuli`;
	    const stimuliDoc = db.doc(stimuliPath);

        const dispatch = createEventDispatcher();
        
        // populating necessary variables
        
        export let email;
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
       let answer = '';

        if (options > 0) {
		// choose random movie and rating type
		currVid = movies[index];

		let vidPlusRating = `${currVid}-${ratingType}`;

		ratingDocPathway = `${ratingsPath}/${params.workerId}/${vidPlusRating}`;

		// grab URL for video sourcing
		currVidSrc = links[index];
	
	}
    console.log(ratingDocPathway)
     
     
        const newPage = async () =>{   
            if (videoIndex % botCheck == 0 && videoIndex != numVideos){
            
                dispatch("botcheck")
                await db.doc(ratingDocPathway).update({
                Comphrehension: answer
                });    
            }   else

                dispatch("finished")
                await db.doc(ratingDocPathway).update({
                   Comphrehension: answer
                
                });    
        
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


                                        <label class="label"
            ><u>What animals were featured in the video clip you just saw?</u>
            <div class="options">
                <label class="radio">
                    <input type="radio" bind:group={answer} value={"cows"} />
                    Cows
                </label>
                <label class="radio">
                    <input type="radio" bind:group={answer} value={"pigs"} />
                    Pigs
                </label>
                <label class="radio">
                    <input type="radio" bind:group={answer} value={"chicken"} />
                    Chicken
                </label>
                <label class="radio">
                    <input type="radio" bind:group={answer} value={"sheep"} />
                    Sheep
                </label>
                <label class="radio">
                    <input type="radio" bind:group={answer} value={"cats"} />
                    Cats
                </label>
                <br />
            </div>
        </label>

                 
                
                        
                <div class="field-label">
                    <!-- Left empty for spacing -->
                </div> 
                <br>
                <button class="button is-success is-large" on:click={newPage}>NEXT PAGE</button>         
            </form>
        </div> 
    </div>
