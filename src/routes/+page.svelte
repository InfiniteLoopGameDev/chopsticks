<script lang="ts">
    class Player{
        name: string;
        fingers: number;
        constructor(newName: string) {
            this.name = newName;
            this.fingers = 0;
        }
    }
    let player1: Player = new Player("John");
    let player2: Player = new Player("Joe");
</script>

<section>
    <h1>PolyDactyly</h1>

    <img class="hand" id="hand-1-1" src="hand.svg" alt="Hand">
    <img class="hand" id="hand-1-2" src="hand.svg" alt="Hand">
    <img class="hand" id="hand-2-1" src="hand.svg" alt="Hand">
    <img class="hand" id="hand-2-2" src="hand.svg" alt="Hand">

    <h2 class="player-name" id="name-1">{player1.name}</h2>
    <h2 class="player-name" id="name-2">{player2.name}</h2>

</section>

<style>
    @import url('https://fonts.googleapis.com/css2?family=Permanent+Marker&family=Rammetto+One&display=swap');
    h1 {
        text-align: center;
        font-family: 'Permanent Marker', cursive;
        font-size: xxx-large;
    }

    h2 {
        font-family: 'Rammetto One', cursive;
    }

    .player-name {
        letter-spacing: 1vh;
        font-size: xx-large;
        position: absolute;
        writing-mode: vertical-lr;
        text-orientation: upright;
        text-align: center;
        top: 58vh;
        translate: 0 -50% 0;
    }

    #name-2 {
        right: 0;
    }

    .hand {
        width: 50vmin;
        position: absolute;
        pointer-events: none;
    }    

    [id|="hand-1"] {
        rotate: 90deg;
    }

    [id|="hand-2"] {
        rotate: -90deg;
        right: 0;
        scale: -1 1 1;
    }

    [id|="hand"][id$="2"] {
        transform: scaleX(-1);
        bottom: 5vh;
    }
</style>
