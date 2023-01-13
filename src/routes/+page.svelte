<script lang="ts">
    import { onMount } from 'svelte';

    class Player{
        name: string;
        hands: number[];
        constructor(newName: string) {
            this.name = newName;
            this.hands = [1,1];
        }
    }

    let player1: Player = new Player("");
    let player2: Player = new Player("");

    let namesVisible: boolean = false;
    
    onMount(async () => {
        player1.name = ""
        player2.name = "Bob"
    })
</script>

<section>
    <h1>PolyDactyly</h1>

    <div id="player-1">
        <div class="hand" id="hand-1-1">
            <img src="hand.svg" alt="Hand">
        </div>
        <div class="hand" id="hand-1-2">
            <img src="hand.svg" alt="Hand">
        </div>
        {#if namesVisible}
            <h2 class="player-name" id="name-1">{player1.name}</h2>
        {/if}
    </div>

    <div id="player-2">
        <div class="hand" id="hand-2-1">
            <img src="hand.svg" alt="Hand">
        </div>
        <div class="hand" id="hand-2-2">
            <img src="hand.svg" alt="Hand">
        </div>
        {#if namesVisible}
            <h2 class="player-name" id="name-2">{player2.name}</h2>
        {/if}
    </div>

    <div id="center-interface">
        <h2>Enter Player 1's Name</h2>
        <input id="text-input" placeholder="Your Name" bind:value={player1.name}>
        <button id="confirm-button">Confirm</button>
    </div>
</section>

<style>
    @import url('https://fonts.googleapis.com/css2?family=Permanent+Marker&family=Rubik:wght@900&display=swap');
    h1 {
        text-align: center;
        font-family: 'Permanent Marker', cursive;
        font-size: xxx-large;
    }

    .player-name {
        font-family: 'Rubik', sans-serif;
        letter-spacing: 1vh;
        font-size: 2em;
        position: absolute;
        writing-mode: vertical-lr;
        text-orientation: upright;
        text-align: center;
        top: 56vh;
        translate: 0 -50% 0;
    }

    #name-1{
        color: #ef4143;
    }

    #name-2 {
        right: 0;
        color: #4191ef;
    }

    .hand {
        width: 50vmin;
        position: absolute;
        pointer-events: none;
    }    

    [id|="hand-1"] {
        rotate: 90deg;
        filter: invert(33%) sepia(57%) saturate(3626%) hue-rotate(338deg) brightness(98%) contrast(90%);
    }

    [id|="hand-2"] {
        rotate: -90deg;
        right: 0;
        scale: -1 1 1;
        filter: invert(46%) sepia(67%) saturate(2417%) hue-rotate(195deg) brightness(103%) contrast(87%);
    }

    [id|="hand"][id$="2"] {
        transform: scaleX(-1);
        bottom: 5vh;
    }

    #center-interface {
        position: absolute;
        top: 50%;
        left: 50%;
        translate: -50% -50%;
        text-align: center;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    #center-interface > h2 {
        font-family: 'Open Sans';
        font-size: xx-large;
        padding-bottom: 2vh;
    }

    #text-input{
        width: 18vw;
        height: 4vh;
        border: 2px solid black;
        border-radius: 8px;
        padding-left: 1vw;
        font-size: medium;
    }

    #confirm-button{
        margin-top: 3vh;
        width: 10vw;
        height: 4vh;
        border: 3px solid black;
        border-radius: 8px;
        background-color: black;
        color: white;
        font-size: medium;
        transition: 0.3s ease-in-out;
    }

    #confirm-button:hover{
        background-color: white;
        color: black;
        cursor: pointer;
    }

</style>
