<!-- this page displays the task, consisting of a video and rating box -->

<script>
	import { createEventDispatcher } from "svelte";
	import RatingBox from "../RatingBox.svelte";
	import CustomVideo from "../CustomVideo.svelte";

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
	} from "../utils";

	import {
		App,
		currVid,
		currVidSrc,
		currRating,
		subjectPath,
		ratingDocPathway,
	} from "../App.svelte";

	const dispatch = createEventDispatcher();

	export let src;
	export let time;
	export let pathway;
	export let ratingType;
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

	function handlePause() {
		paused = true;
	}

	function handlePlay() {
		paused = false;
	}

	function handleEnd() {
		console.log(currVid);
		console.log(currVidSrc);
		console.log(currRating);
		console.log(subjectPath);
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

	// check to see which movies subject has already viewed (if any)
	let currPath = `${ratingsPath}/${params.workerId}`;
	db.collection(currPath)
		.get()
		.then(function (ratingList) {
			// removes already completed movies from option set
			ratingList.forEach(function (doc) {
				moviesRemaining = removeItemOnce(
					moviesRemaining,
					doc.id.split("-")[0]
				);
			});
			// see how many movies are left
			numOptions = moviesRemaining.length;
			console.log("moviesRemaining: ", moviesRemaining);
			// if any movie-rating pairings left, load and start
			if (numOptions > 0) {
				// choose random movie and rating type
				let movieIndex = Math.floor(
					Math.random() * moviesRemaining.length
				);
				let ratingIndex = Math.floor(
					Math.random() * ratingTypes.length
				);
				currVid = moviesRemaining[movieIndex];
				currRating = ratingTypes[ratingIndex];
				let vidPlusRating = `${currVid}-${currRating}`;
				ratingDocPathway = `${ratingsPath}/${params.workerId}/${vidPlusRating}`;
				// grab URL for video sourcing
				currVidSrc = stimuliTable.data()[currVid];
				updateState("consent");
			} else {
				console.log("no options left!");
				updateState("complete");
			}
		});
</script>

<main>
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
