<!-- THIS IS THE VERSION OF THE SVELTE RATING APP INTENDED FOR USE WITH MOTH VIDEOS -->

<!-- TODOs:
	examine weird build errors on JS console when using with MTurk (GET ERROR)
    Make sure to add changes at the bottom of if-else statement.
-->

<script>
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
    } from "./utils.js";

    import { onMount } from "svelte";
    import Intro from "./pages/Intro.svelte";
    import Botcheck from "./pages/Botcheck.svelte";
    import Consent from "./pages/Consent.svelte";
    import Instructions1 from "./pages/Instructions1.svelte";
    import Instructions2 from "./pages/Instructions2.svelte";
    import Demo from "./pages/Demo.svelte";
    import Task from "./pages/Task.svelte";
    import Debrief from "./pages/Debrief.svelte";
    import Complete from "./pages/Complete.svelte";
    import Loading from "./components/Loading.svelte";
    import Header from "./components/Header.svelte";
    import MTurkPreview from "./pages/MTurkPreview.svelte";
    import Debrief2 from "./pages/Debrief2.svelte";
    import Debrief3 from "./pages/Debrief3.svelte";
    import Prolific from "./pages/Prolific.svelte"



    // path details
    const ratingsPath = `${experiment}/ratings`;
    const ratingsDoc = db.doc(ratingsPath);
    const subjectGroupPath = `${experiment}/subjects/${userGroup}`;
    const subjectGroupCollection = db.collection(subjectGroupPath);
    const stimuliPath = `${experiment}/stimuli`;
    const stimuliDoc = db.doc(stimuliPath);

    // declare and set other necessary variables
    let currVid;
    let currVidSrc;
    let currRating;
    let subjectPath;
    let ratingDocPathway;
    let movieLinks = [];
    let currentState;
    let consentStatus;
    let alreadyWatched = [];
    let moviesRemaining = [];
    let numOptions;
    let time = 0;
    let initExperiment = false;

    // use to validate build type in JS console
    console.log(dev);

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
    

    const resetTestWorker = async () => {
        // Change to the new state within Svelte
        if (params.workerId === "test-worker") {
            currentState = "consent";
            let subjectRef = subjectGroupCollection.doc(params.workerId);
            subjectRef.get().then(function (doc) {
                try {
                    let currPath = `${ratingsPath}/${params.workerId}`;
                    db.collection(currPath)
                        .get()
                        .then(function (ratingList) {
                            // deletes all previous ratings
                            ratingList.forEach(function (doc) {
                                console.log("deleting: ", doc.id);
                                db.collection(currPath).doc(doc.id).delete();
                            });
                            // updates subject log
                            subjectRef.update({
                                startTime: serverTime,
                                consentStatus: "incomplete",
                            });
                            console.log("reset test-worker");
                        });
                } catch (error) {
                    console.error(error);
                }
            });
        } else {
            console.log(
                `Reset user requested but workerId is ${params.workerId}`
            );
        }
    };

    // sets movieIndex to 0 and ratingIndex to random, pushes & sorts all movies
    let CDN = "https://d1e3c04h84hf6s.cloudfront.net/";
    let fileType = ".mp4";
    let movieIndex = 0;
    let movieIndices = []
    let ratingIndex = Math.floor(Math.random() * ratingTypes.length);
    stimuliDoc.get().then(function (stimuliTable) {
        for (var field in stimuliTable.data()) {
            moviesRemaining.push(field);
            movieLinks.push(CDN + field + fileType);
        }
        moviesRemaining.sort();
        movieLinks.sort();
       // shuffle(moviesRemaining);
       // shuffle(movieLinks);

        console.log("move links")
        console.log(movieLinks)
        
        for (var field2 in movieLinks){
            let str = movieLinks[field2]
            str = str.replace(CDN, "");
            str = str.replace(fileType, "")
             var res = str.replace(/\D/g, "");
             res = parseInt(res)
             movieIndices.push(res)
        }
        console.log("movie indeces")
        console.log(movieIndices)
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
                console.log(movieLinks);
                // if any movie-rating pairings left, load and start
                if (numOptions > 0) {
                    // choose random movie and rating type
                    currVid = moviesRemaining[movieIndex];
                  
                    currRating = ratingTypes[ratingIndex];
                    
                    let vidPlusRating = `${currVid}-${currRating}`;
                    
                    ratingDocPathway = `${ratingsPath}/${params.workerId}/${vidPlusRating}`;
                    // grab URL for video sourcing
                    currVidSrc = movieLinks[movieIndex];
                    updateState("consent");
                } else {
                    console.log("no options left!");
                    updateState("complete");
                }
            });
    });

    // *****************************
    // main function
    // *****************************

    // Before we render anything see if we have a db entry for this subject based upon the URL parameters. If not
    // create an entry with a new random stimulus order and put them into the instructions state.
    // If we do, load their trial order and current experiment state

    if (params.assignmentId === "ASSIGNMENT_ID_NOT_AVAILABLE") {
        currentState = "mturk-preview";
    } else if (
        params.workerId &&
        params.assignmentId !== "ASSIGNMENT_ID_NOT_AVAILABLE" &&
        params.hitId
    ) {
        initExperiment = true;
    } else {
        currentState = "non-mturk";
    }

    onMount(async () => {
        // right when DOM is created
        if (initExperiment) {
            try {
                auth.onAuthStateChanged(async (user) => {
                    if (!user) {
                        // if no user
                        try {
                            // grab the worker and assignment ID and attempt login
                            await auth.signInWithEmailAndPassword(
                                `${params.workerId}@experiment.com`,
                                params.workerId
                            );
                            console.log(
                                "user found...signing in with credentials"
                            );
                            // then look for document
                        } catch (error) {
                            if (error.code === "auth/user-not-found") {
                                console.log(
                                    "no user found...creating new credentials"
                                );
                                // if login fails, create new user
                                await auth.createUserWithEmailAndPassword(
                                    `${params.workerId}@experiment.com`,
                                    params.workerId
                                );
                            } else {
                                console.log("not working");
                                console.error(error);
                            }
                        }
                    } else {
                        console.log("user authenticated...");
                        let currUser = auth.currentUser;
                        try {
                            // if user already signed in, grab relevant document
                            let subjectRef = subjectGroupCollection.doc(
                                params.workerId
                            );
                            subjectPath = `${subjectGroupPath}/${params.workerId}`; // setting for use in HTML below
                            subjectRef.get().then(function (doc) {
                                if (doc.exists) {
                                    // load old document
                                    console.log(
                                        "previous document found...loading state..."
                                    );
                                    // updates most recent login time
                                    subjectRef.update({
                                        mostRecentTime: serverTime,
                                    });
                                } else {
                                    // create a new document
                                    subjectGroupCollection
                                        .doc(params.workerId)
                                        .set({ name: "unknown" });
                                    console.log(
                                        "no previous documents found...creating new..."
                                    );
                                    subjectPath = `${subjectGroupPath}/${params.workerId}`; // setting for use in HTML below
                                    subjectRef.set({
                                        workerId: params.workerId,
                                        assignmentId: params.assignmentId,
                                        hitId: params.hitId,
                                        userId: currUser.uid,
                                        startTime: serverTime,
                                        consentStatus: "incomplete",
                                    });
                                }
                                // grab stimuli doc and add all movies to list
                            });
                        } catch (error) {
                            console.error(error);
                        }
                    }
                });
            } catch (error) {
                console.error(error);
            }
        }
    });

    // *****************************
    // handler functions
    // *****************************

    // this function updates the current state of the user to
    // dynamically render different parts of the experiment (i.e. instructions, quiz, etc)
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

    // registers user as a bot
    const failedBot = async () => {
        try {
            await db
                .doc(`${experiment}/subjects/${userGroup}/${params.workerId}`)
                .update({
                    botStatus: "bot",
                });
            console.log("user identified as bot");
        } catch (error) {
            console.error(error);
        }
    };

    // registers rejected consent form
    const failedConsent = async () => {
        try {
            await db
                .doc(`${experiment}/subjects/${userGroup}/${params.workerId}`)
                .update({
                    consentStatus: "failed",
                });
            console.log("user rejected consent");
        } catch (error) {
            console.error(error);
        }
    };

    // registers accepted consent form
    const agreedConsent = async () => {
        try {
            await db
                .doc(`${experiment}/subjects/${userGroup}/${params.workerId}`)
                .update({
                    consentStatus: "signed",
                });
            updateState("botcheck-instruct");
            console.log("user accepted consent");
        } catch (error) {
            console.error(error);
        }
    };
     // CHANGE THIS TO 0 (VIDEOINDEX)
    let debriefIndex = 0; // CHANGE THIS TO 0 (DEBRIEFINDEX)
    let debriefIndex2 = 0;
    const increment = async () => {
        movieIndex++;
        numOptions--;
        console.log(movieIndex);
    };

    const increment2 = async () => {
        debriefIndex++;
        console.log(debriefIndex);
    }

    const increment3 = async () => {
        debriefIndex2++;
        console.log(debriefIndex2);
    }
    // function used to remove previously watched videos from array
    function removeItemOnce(arr, value) {
        var index = arr.indexOf(value);
        if (index > -1) {
            arr.splice(index, 1);
        }
        return arr;
    }
</script>

<div class="content">
    <div class="header">
        <Header on:resetTestWorker={resetTestWorker} />
    </div>
    {#if !currentState}
        <Loading>Loading...</Loading>
    {:else if currentState === "mturk-preview"}
        <MTurkPreview />
    {:else if currentState === "intro"}
        <Intro on:finished={() => updateState("consent")} />
    {:else if currentState === "consent"}
        <Consent
            on:finished={() => agreedConsent()}
            on:returned={() => failedConsent()}
        />
        {:else if currentState === "botcheck-instruct"}
        <Botcheck
        index={movieIndex}
            on:finished={() => updateState("prolific")}
            on:passed={() => updateState("instructions2")}
            on:failed={() => failedBot()}
        ></Botcheck>
    {:else if currentState === "botcheck-task"}
        <Botcheck      
        index={movieIndex}
            on:finished={() => updateState("prolific")}
            on:passed={() => updateState("instructions2")}
            on:failed={() => failedBot()}
        ></Botcheck>
        {:else if currentState === "prolific"}
        <Prolific subPath={subjectPath} {email} {labName} {numOptions}
        on:finished={() => updateState("instructions1")}></Prolific>

    {:else if currentState === "instructions1"}
        <Instructions1
            ratingType={currRating}
    
            on:finished={() => updateState("demo")}
        />
    {:else if currentState === "demo"}
        <Demo
            {time}
            ratingType={currRating}
            on:back={() => updateState("instructions1")}
            on:finished={() => updateState("instructions2")}
        />
    {:else if currentState === "instructions2"}
        <Instructions2
            on:back={() => updateState("demo")}
            on:finished={() => updateState("task")}
        />
    {:else if currentState === "task"}
        <Task
           
            ratingType={currRating}
            {time}
            movies={moviesRemaining}
            links={movieLinks}
            index={movieIndex}
            options={numOptions}
            on:finished={() => increment()}
            on:finished={() => updateState("debrief2")}
        />
    {:else if currentState === "debrief2"}
        <Debrief2
            {email}
            {labName}
            {numOptions}
            movies={moviesRemaining}
            links={movieLinks}
            index={debriefIndex}
            videoIndex = {movieIndex}
            options={numOptions}
            ratingType={currRating}
            on:finished={() => increment2()}
            on:complete={() => updateState("debrief")}
            on:finished={() => updateState("debrief3")}

        />

        {:else if currentState === "debrief3"}
        <Debrief3
            {email}
            {labName}
            {numOptions}
            movies={moviesRemaining}
            links={movieLinks}
            index={debriefIndex2}
            videoIndex = {movieIndex}
            movieIndices = {movieIndices}
            options={numOptions}
            ratingType={currRating}
            on:finished={() => increment3()}
            on:botcheck={() => updateState("botcheck-task")}
            on:finished={() => updateState("task")}

        />
        {:else if currentState === "debrief"}
        <Debrief subPath={subjectPath} {email} {labName} {numOptions}
        on:complete={() => updateState("complete")}
        >
        </Debrief>
    {:else if currentState === "complete"}
        <Complete />
    {/if}
</div>

<style>
    .content {
        position: relative;
    }
    .header {
        left: 0;
    }
</style>
