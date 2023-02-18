<script lang="ts">
    import { onMount, tick } from 'svelte';

    const timeout = async (ms: number) => new Promise(res => setTimeout(res, ms));

    class UI {
        visibility: {title: boolean, subtitle: boolean, textInput: boolean, button: boolean};
        title: string;
        subtitle: string;
        buttonText: string;
        inputTooltip: string;
        inputText: string;
        buttonIsPressed: boolean;
        constructor(title: string, subtitle: string, button: string, textInput: string) {
            this.visibility = {title: false, subtitle: false, textInput: false, button: false};
            this.title = title;
            this.subtitle = subtitle;
            this.buttonText = button;
            this.inputTooltip = textInput;
            this.inputText = "";
            this.buttonIsPressed = false;
        }
        buttonHandler() {
            this.buttonIsPressed = true;
        }
        async waitButtonPress() {
            while (this.buttonIsPressed === false) await timeout(50);
            this.buttonIsPressed = false;
        }
    }

    const fingerNames = ["pinky", "ring", "middle", "index", "thumb"]

    class Hand{
        fingers: {[pinky: string]: boolean, ring: boolean, middle: boolean, index: boolean, thumb: boolean};
        count: number;
        isAlive: boolean;
        style: string;
        constructor() {
            this.fingers = {pinky: false, ring: false, middle: false, index: true, thumb: false};
            this.count = 1;
            this.isAlive = true;
            this.style = "opacity: 1; pointer-events: all;";
        }
        setCount(count: number) {
            this.count = count
            if (this.count == 1) {
                this.fingers = {pinky: false, ring: false, middle: false, index: true, thumb: false};
            } else if (this.count == 2) {
                this.fingers = {pinky: false, ring: false, middle: true, index: true, thumb: false};
            } else if (this.count == 3) {
                this.fingers = {pinky: false, ring: true, middle: true, index: true, thumb: false};
            } else if (this.count == 4) {
                this.fingers = {pinky: true, ring: true, middle: true, index: true, thumb: false};
            } else if (this.count == 0) {
                this.isAlive = false
            } else {this.setCount(this.count - 5)}

            this.generateStyle();
        }
        generateStyle() {
            if (this.isAlive) {
                this.style = "opacity: 1; pointer-events: all;";
            } else {
                this.style = "opacity: 0; pointer-events: none;";
            }
        }
    }

    class Player{
        name: string;
        nameVisibility: boolean;
        hands: [Hand, Hand];
        constructor(newName: string) {
            this.name = newName;
            this.hands = [new Hand(), new Hand()];
            this.nameVisibility = false;
        }
        isAlive(): boolean {
            let handCount: number = 0
            for (let hand in this.hands) {
                if (this.hands[hand].isAlive) {
                    handCount += 1
                }
            }
            if (handCount == 0) {return false}
            else {return true}
        }
    }

    class Game {
        handValue: {player: number, hand: number};
        handPressed: boolean;
        players: Player[];
        uiContainer: UI;
        currentPlayer: number;

        constructor(players: Player[], ui: UI) {
            this.players = players;
            this.handValue = {player: 0, hand: 0};
            this.handPressed = false;
            this.uiContainer = ui;
            this.currentPlayer = 0;
        }
        handHandler(player: number, hand: number) {
            this.handValue = {player: player, hand: hand};
            this.handPressed = true;
        }
        async awaitUserChoice() {
            this.handPressed = false;
            while (this.handPressed == false) await timeout(50);
            this.handPressed = false;
        }
        hasEnded(): boolean {
            let playersAlive: number = 0;
            for (let player in this.players) {
                if (this.players[player].isAlive()){
                    playersAlive += 1;
                }
            }
            if (playersAlive == 1) {return true;} 
            else {return false;}
        }
        nextPlayer() {
            this.currentPlayer += 1;
            if (this.currentPlayer >= this.players.length) {
                this.currentPlayer = 0;
            }
        }
        async getHandSelect(condition: (a: number) => boolean): Promise<{player: number; hand: number;} | undefined>{
            let validHand = false;
            while (!validHand){
                await this.awaitUserChoice();
                let selectedHand = Object.assign({}, this.handValue);
                if (condition(selectedHand.player - 1) && this.players[selectedHand.player - 1].hands[selectedHand.hand - 1].isAlive) {
                    validHand = true;
                    return selectedHand;
                } else {
                    game.uiContainer.subtitle = "Selected hand is not valid"; // WARNING USING EXTERNAL VALUE TO CLASS!!!
                }
            }
        }
    }

    let players: Player[] = [new Player(""), new Player("")];
    
    let game: Game = new Game(players, new UI("Enter Player 1's Name", "","Confirm", "Your name"));

    onMount(async () => {
        game.uiContainer.visibility = {title: true, subtitle: false, textInput: true, button: true};
        for (let player = 0; player < 2; player++){
            game.uiContainer.title = "Enter Player " + (player + 1) + "'s Name";
            await game.uiContainer.waitButtonPress();
            if (game.uiContainer.inputText == "") {
                players[player].name = "Player " + (player + 1);
            } else {
                players[player].name = game.uiContainer.inputText;
            }
            players[player].nameVisibility = true;
        }
        game.uiContainer.visibility = {title: false, subtitle: false, textInput: false, button: false}
        while (!game.hasEnded()) {
            game.uiContainer.title = game.players[game.currentPlayer].name + "'s Turn";
            game.uiContainer.visibility.title = true;

            let handValues: {player: number, hand: number}[] = [];
            game.uiContainer.subtitle = "Select the hand you want to use";
            game.uiContainer.visibility.subtitle = true;
            await game.getHandSelect(function(a) {return a == game.currentPlayer}).then((hand) => {
                if (typeof hand != "undefined") {
                    handValues.push(hand);
                } else {
                    throw("Broken Promises");
                }
            });

            game.uiContainer.subtitle = "Select the hand you want to attack";
            await game.getHandSelect(function(a) {return a != game.currentPlayer}).then((hand) => {
                if (typeof hand != "undefined") {
                    handValues.push(hand);
                } else {
                    throw("Broken Promises");
                }
            });

            game.players[handValues[1].player - 1]
                .hands[handValues[1].hand - 1]
                .setCount( 
                    game.players[handValues[0].player - 1]
                    .hands[handValues[0].hand - 1]
                    .count + 
                    game.players[handValues[1].player - 1]
                    .hands[handValues[1].hand - 1]
                    .count
                );
            game.nextPlayer();
        }
    })

    function garbage() {
        game.players[0].hands[0].setCount(game.players[0].hands[0].count + 1);
    }
</script>

<section>
    <h1>PolyDactyly</h1>
    {#each [1, 2] as player}
    <div id="player-{player}">
        {#each [1, 2] as hand}
        <div class="hand" id="hand-{player}-{hand}" on:click={() => {game.handHandler(player,hand)}} on:keypress={() => {game.handHandler(player,hand)}}
        style="{players[player - 1].hands[hand - 1].style}">
            <img src="hand.svg" alt="Hand">
            {#each fingerNames as finger}
                <button class="finger {finger}" style="opacity: {+ !players[player - 1].hands[hand - 1].fingers[finger]}" disabled></button>
            {/each}
        </div>
        {/each}
        {#if players[player - 1].nameVisibility}
            <h2 class="player-name" id="name-{player}">{players[player - 1].name}</h2>
        {/if}
    </div>
    {/each}

    <div id="center-interface">
        {#if game.uiContainer.visibility.title}
            <h2>{game.uiContainer.title}</h2>
        {/if}
        {#if game.uiContainer.visibility.subtitle}
            <h3>{game.uiContainer.subtitle}</h3>
        {/if}
        {#if game.uiContainer.visibility.textInput}
            <input id="text-input" placeholder={game.uiContainer.inputTooltip} name="inputText" bind:value={game.uiContainer.inputText} maxlength="16">
        {/if}
        {#if game.uiContainer.visibility.button}
            <button id="confirm-button" on:click={() => {game.uiContainer.buttonHandler()}}>{game.uiContainer.buttonText}</button>
        {/if}
        <!-- <button on:click={() => {garbage()}}>HAAH GARBAGE</button> -->
    </div>
</section>

<style>
    @import url('https://fonts.googleapis.com/css2?family=Open+Sans&family=Permanent+Marker&family=Rubik&display=swap');
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
        height: 39vh;
        position: absolute;
        pointer-events: none;
        pointer-events: all;
        cursor: pointer;
        z-index: 5;
        transition: 1s;
    }    

    [id|="hand-1"] {
        rotate: 90deg;
        border-color: #ee4345;
    }

    [id|="hand-1"] > img {
        filter: invert(33%) sepia(57%) saturate(3626%) hue-rotate(338deg) brightness(98%) contrast(90%);
    }

    [id|="hand-2"] {
        rotate: -90deg;
        right: 0;
        scale: -1 1 1;
        border-color: #4191ef;
    }

    [id|="hand-2"] > img {
        filter: invert(46%) sepia(67%) saturate(2417%) hue-rotate(195deg) brightness(103%) contrast(87%);
    }

    [id|="hand"][id$="2"] {
        transform: scaleX(-1);
        bottom: 5vh;
    }

    .finger {
        z-index: 0;
        background-color: white;
        border: 0;
        opacity: 1;
        border-right: 4px solid;
        border-color: inherit;
        pointer-events: inherit;
        cursor: inherit;
        transition: 1s;
    }

    .pinky {
        position: relative;
        top: -55%;
        height: 10%;
        right: -14.5%;
        width: 20%;
        rotate: 36deg;
    }

    .ring {
        position: relative;
        top: -71.8%;
        height: 9.8%;
        right: 0.8%;
        width: 25%;
        rotate: 61deg;
    }

    .middle {
        position: relative;
        top: -78.7%;
        height: 9.8%;
        right: 17.6%;
        width: 27%;
        rotate: 76deg;
    }

    .thumb {
        position: relative;
        top: -70.7%;
        height: 36%;
        right: -56.8%;
        width: 27%;
        rotate: 170deg;
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

    #center-interface > h2,h3 {
        font-family: 'Open Sans', sans-serif;
    }

    #center-interface > h2 {
        font-size: xx-large;
        padding-bottom: 2vh;
    }

    #center-interface > h3 {
        font-size: x-large;
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
