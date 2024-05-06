<script>
    import { initializeApp, getApps, getApp } from "firebase/app";
    import { addDoc, deleteDoc, getFirestore } from "firebase/firestore";
    import { firebaseConfig } from "../lib/firebaseConfig";
    import { collection, onSnapshot, doc, updateDoc} from "firebase/firestore";

    //Make absolutely certain firebase is initialized
    if (!getApps().length) {
        const app = initializeApp(firebaseConfig);
    } else {
        getApp();
    }
    const db = getFirestore();

    //Constantly updates
    let garage = [];
    const colRef = collection(db, "garage");
    const unsubscribe = onSnapshot(colRef, (querySnapshot) => {
        garage = [];
        querySnapshot.forEach((doc) => {
            garage.push(doc.data());
            garage = garage;
        });
    });

    console.log(garage);

    let select = "brand";
    let newCar = {
        brand:"",
        model:"",
        year:"",
        owned:false,
        id:""
    };

    function seeModel() {
        select = "model";
    }

    function seeYear() {
        select = "year";
    }

    function seeBrand() {
        select = "brand";
    }

    async function add() {
        if (newCar.brand!="" && newCar.Brand!="" && newCar.year!="") {
            const addCar = newCar;
            newCar = {
                brand:"",
                model:"",
                year:"",
                owned:false,
                id:""
            };
            const newCarDoc = await addDoc(colRef, addCar);
            console.log("Document written with ID: ", newCarDoc.id);
            await updateDoc(doc(db, "garage", newCarDoc.id), {
                id: newCarDoc.id
            })
        }
    }

    async function deleteCar(id) {
        const docDelete = doc(db, "garage", id)
        await deleteDoc(docDelete);
    }

</script>
<style>
    .owned {
        background-color:rgb(218, 218, 218);
    }
    .notOwned {
        text-decoration:underline;
    }
    .delete button {
        background-color:rgb(255, 0, 0);
        color:white;
        border-style:solid;
    }
    .delete button:hover {
        background-color:rgb(255, 100, 100);
        cursor:pointer;
    }
    li {
        padding:3px;
    }

</style>

<h1>Our garage</h1>

<p>Welcome to the US Key Guard garage! Go ahead and add some cars. This website is connected to a database, so you can see what other people add too!</p>

<form>
<label for="brand">Brand:</label>
<input type="text" id="brand" bind:value={newCar.brand}>
<label for="model">Model:</label>
<input type="text" id="model" bind:value={newCar.model}>
<label for="year">Year:</label>
<input type="text" id="year" bind:value={newCar.year}>
<label for="owned">Owned?</label>
<input type="checkbox" id="owned" bind:value={newCar.owned}><br/><br/>
<button on:click={add}>Add car</button>
</form>

<ol>
    {#each garage as car}
        <li>
            <span class="car" class:owned={car.owned} class:notOwned={!car.owned}>
                {car.brand} {car.model} {car.year}
            </span>
            <span class="delete">
                <button on:click={deleteCar(car.id)}>Delete</button>
            </span>
        </li>
    {:else}
        <p>No cars :(</p>
    {/each}
</ol>
