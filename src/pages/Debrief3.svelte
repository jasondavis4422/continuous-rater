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
        let Main_question = ['1.Which animal was NOT rescued in the clip?', 
        '1.    	Which statement is NOT true about the animals mentioned in the clip?', 
        '1.    	Which of the following is NOT described in the clip?',
         '1.    	What is NOT described in the clip?',
         ' 1.    	Which of the following is NOT a behavior shown by the cow in the previous clip?',
           '1.    	What is NOT true about Esther, the pig, described in the clip?”', 
           '1.    	Which of the following animals was NOT described as enjoying hugs and petting from humans?',
           ' 1.    	Which of the following animals was NOT described as showing affection to humans?', 
           '1.    	Which animal did NOT appear in the previous clip?'];

          let Answer_a = ['A) Lamb',
           'A) The mother pig and her piglets were rescued from a slaughterhouse.',
           'A) Goats enjoying a slide.',' ​​A) Elephants playing with a tire.',
'A) Pumping water from a well',
' A) Esther was supposed to be a mini pig but grew to around 650 pounds.',
 ' ​A) Pig', 
  'A) Chicken',
   'A) Monkey'];

   let Answer_b = ['B) Cat',
    'B) Most pigs are raised in factory farms and live on concrete or metal floors.'
    , 'B) A bear scratching its back on a tree.'
    , 'B) Pigs playing with a ball.'
    , 'B) Caring for a sick cow',
     'B) Esther’s owners never wanted to give her up, even after realizing how large she would grow.', 
     '​​B) Cow'
     , 'B) Lion',
      'B) Donkey']
let Answer_c = ['C) Pigeon', 
'C) The rescued hens had been kept in large, spacious cages before their rescue.',
 'C) Cows running around in a pasture.'
 , 'C) Cows playing with a ball.'
 , '​​C) Opening a door with its mouth'
 , 'C) Esther has a diet managed with high-fiber, low-fat kibble.'
 , 'C) Sheep'
 , 'C) Sheep', 'C) Goat'];
let Answer_d = ['D) Whale'
, 'D) Calves rescued from dairy farms were given a chance to explore outside together.'
,'D) Chickens playing around', 'D) Goats riding in a cart.', 'D) Caring for its calf', 'D) Esther’s fans painted photos of her, which her owners received.', '​D) Chicken', 'D) Turkey', 'D) Duck'];
let Answer_e =['E) Koala', 'E) Beagles from a laboratory were seeing the outside world for the first time after their rescue.', 'E) Piglets playing in the mud', 'E) Sheep playing with a ball.', 'E) Spinning according to human’s command', 'E) Esther’s fans painted photos of her, which her owners received.', 'E) Horse', 'E) Cow', 'E) Elephant'];
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
            ><u>       {Main_question[videoIndex]}  </u>
            <div class="options">
                <label class="radio">
                    <input type="radio" bind:group={answer} value={"A"} />
                    {Answer_a[videoIndex]}  
                </label>
                <label class="radio">
                    <input type="radio" bind:group={answer} value={"B"} />
                    {Answer_b[videoIndex]}                </label>
                <label class="radio">
                    <input type="radio" bind:group={answer} value={"C"} />
                    {Answer_c[videoIndex]}  
                </label>
                <label class="radio">
                    <input type="radio" bind:group={answer} value={"D"} />
                    {Answer_d[videoIndex]}  
                </label>
                <label class="radio">
                    <input type="radio" bind:group={answer} value={"E"} />
                    {Answer_e[videoIndex]}  
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
