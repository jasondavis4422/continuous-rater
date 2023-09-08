<!-- this page displays the task, consisting of a video and rating box -->

<script>
	import { createEventDispatcher } from "svelte";
	import RatingBox from "../RatingBox.svelte";
	import CustomVideo from "../CustomVideo.svelte";
	import Debrief2 from "../pages/Debrief2.svelte"

	import {
		db,
		auth,
		serverTime,
		params,
		ratingTypes,
		dev,
		experiment,
		userGroup,
		labName,
		email,
	} from "../utils"

	const dispatch = createEventDispatcher();

	export let src;
	export let time;
	export let pathway;
	export let ratingType;
	export let movies;
	export let options;
	console.log(options);
	let paused = true;
	let rating = 50.0;

	let currentState;
	let consentStatus;
	let alreadyWatched = [];
	let moviesRemaining = [];
	let numOptions;

	const ratingsPath = `${experiment}/ratings`;
	const ratingsDoc = db.doc(ratingsPath);
	const subjectGroupPath = `${experiment}/subjects/${userGroup}`;
	const subjectGroupCollection = db.collection(subjectGroupPath);
	const stimuliPath = `${experiment}/stimuli`;
	const stimuliDoc = db.doc(stimuliPath);

	let initExperiment = false;
	let movieLinks = [];
	let movieIndex = 0;
	let currVid;
	let currVidSrc;
	let ratingDocPathway;

	function handlePause() {
		paused = true;
	}

	function handlePlay() {
		paused = false;
	}

	function handleEnd() {
		movieIndex += 1;
		options -= 1;
		console.log(movieIndex);
		console.log(options);
		dispatch("finished");
	}

	const updateState = async (newState) => {
		// Change to the new state within Svelte
		currentState = newState;
		try {
			await db
				.doc(`${experiment}/subjects/${userGroup}/${params.workerId}`)
				.update({
					currentState,
				});
			console.log("updated user state");
		} catch (error) {
			console.error(error);
		}
	};

	function removeItemOnce(arr, value) {
		var index = arr.indexOf(value);
		if (index > -1) {
			arr.splice(index, 1);
		}
		return arr;
	}

   // sets movieIndex to 0 and ratingIndex to random, pushes & sorts all movies
    
	let CDN = 'https://d1h4dm5ishe8rs.cloudfront.net/';
	let fileType = ".mp4";
    let ratingIndex = Math.floor(Math.random() * ratingTypes.length);
    stimuliDoc.get().then(function (stimuliTable) {
        for (var field in stimuliTable.data()) {
            moviesRemaining.push(field);
			movieLinks.push(CDN + field + fileType);
        }
        moviesRemaining.sort();
		movieLinks.sort();
		console.log(moviesRemaining);
	    console.log(movieLinks);
	});

	if (options > 0) {
                    // choose random movie and rating type
                    currVid = movies[movieIndex];
					console.log(currVid);
                    let vidPlusRating = `${currVid}-${ratingType}`;
					console.log(vidPlusRating);
                    ratingDocPathway = `${ratingsPath}/${params.workerId}/${vidPlusRating}`;
					console.log(ratingDocPathway);
                    // grab URL for video sourcing
                    currVidSrc = movieLinks[movieIndex];
					console.log(currVidSrc);
                } 
                  





</script>

<main>
	<div class = "content">
	</div>
	<div class="container">
		<CustomVideo
			{src}
			bind:time
			on:pause={handlePause}
			on:play={handlePlay}
			on:finished={handleEnd}
		/>
		<RatingBox {pathway} {rating} bind:time bind:paused {ratingType} />
		<h2 style="text-align:center">
			Please rate how <strong>{ratingType}</strong> you feel
		</h2>
	</div>
</main>

<style>
	main {
		padding: 1em;
		margin: 0 auto;
		min-width: 400px !important;
		max-width: 1000px !important;
	}

	h2 {
		font-weight: normal;
		padding: none;
		width: 50%;
	}

	.container {
		display: flex;
		flex-direction: column;
		align-items: left;
		width: 200%;
	}
</style>
