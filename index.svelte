<script>
    import { initializeApp , getApps , getApp } from "firebase/app";
    import { firebaseConfig } from "../lib/firebase.Config"
    import { getFirestore, collection, onSnapshot, doc , updateDoc , deleteDoc , addDoc } from "firebase/firestore";
    import { browser } from "$app/env";
    import { onMount } from "svelte";
    import { onDestroy } from "svelte";

    //Inilizing Firebase for the first time
    const firebaseApp =
        browser && 
        (getApps().length === 0? initializeApp(firebaseConfig) : getApp());

    const db = browser && getFirestore();    
   
    let cities = [];

    //Reading data from Database
    const colRef = browser && collection(db, "12345");

    onMount(() => {
        const unsubscribe = browser && onSnapshot(colRef, (querySnapshot) => {
                let fbCities = [];
                querySnapshot.forEach((doc) => {
                    let city = {...doc.data() , id: doc.id}
                    fbCities = [...fbCities, city]
                });
            cities = fbCities;
        });
        onDestroy(() => {unsubscribe;});
    });
  

    //Update function
    const markComplete = async (city) => {
        await updateDoc(doc(db, "12345" , city.id), {
        isComplete: !city.isComplete 
        });
    };

    let title = "";

    //Add function
    const addCities = async () => {
        const docRef = await addDoc(collection(db, "12345"), {
            name: title,
            isComplete: false,
            createdAt: new Date(),
        });
        title = "";
    }

    //Delete Function
    const onDelete = async (id) => {
        await deleteDoc(doc(db, "12345", id));
    };

</script>


    <span>Add Cities here</span>

    <input type="text" placeholder="Enter city name here" bind:value={title}>
    <button on:click={addCities}>Add</button>

    <ol>
        {#each cities as city }
            <li class:complete={city.isComplete}>
                {city.name}
            <button on:click={()=> markComplete(city)}>Update</button>  
            <button on:click={()=> onDelete(city.id)}>Delete</button>  
            </li>
        {/each}
    </ol>


<style>
    .complete {
        text-decoration: line-through;
    }
</style>
