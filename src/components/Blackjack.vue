<template>
    <div class="container">
        <div class="bank">
            Bank: <span>{{bank}}</span>
        </div>
        <div class="bank bet">
            Bet: <span>{{betAmount}}</span>
        </div>
        <div>
            <h3 v-if="hand.value > 1">Player <span class="value">{{hand.value}}</span></h3>
            <div v-for="card in hand.cards" :key="card.id" :class="card.suit" class="card">
                <div>{{card.name}}</div>
                <img v-bind:src="card.img" /> 
                <button v-if="!card.selected" @click="setAce(1, card.id)">1</button>
                <button v-if="!card.selected" @click="setAce(11, card.id)">11</button>
            </div>
        </div>
        <div>
            <h3 v-if="dealer.value > 1">Dealer <span class="value">{{dealer.value}}</span></h3>
            <div v-for="card in dealer.cards" :key="card.id" class="card">
                <div>{{card.name}} </div>
                <img v-bind:src="card.img" /> 
            </div>
        </div>
        <form ref="form" v-if="!betPlaced">
            <input name="vitalInformation" v-model.number="betAmount">
            <button v-on:click="submit">Start</button>
            <p v-if="notEnoughFunds">You dont have enough funds to make that bet</p>
        </form>
        <button @click="hit" v-if="inGame">Hit</button>
        <button @click="stand" v-if="inGame">Stand</button>
        <button @click="restartGame" v-if="restart">Play again</button>
    </div>
</template>

<script>
import spadeImg from '../assets/spade.svg';
import clubImg from '../assets/clubs.svg';
import diamondImg from '../assets/diamond.svg';
import heartImg from '../assets/heart.svg';

export default {
    name: 'Blackjack',
    data: function() {
        return {
            deck: [],
            suits: [
                {name: 'spade', img: spadeImg},
                {name: 'club', img: clubImg},
                {name: 'diamond', img: diamondImg},
                {name: 'heart', img: heartImg}
            ],
            cards: [
                {id: null, value: [1, 11], name: "Ace", suit: "", selected: false},
                {id: null, value: 2, name: "2", suit: "", selected: true},
                {id: null, value: 3, name: "3", suit: "", selected: true},
                {id: null, value: 4, name: "4", suit: "", selected: true},
                {id: null, value: 5, name: "5", suit: "", selected: true},
                {id: null, value: 6, name: "6", suit: "", selected: true},
                {id: null, value: 7, name: "7", suit: "", selected: true},
                {id: null, value: 8, name: "8", suit: "", selected: true},
                {id: null, value: 9, name: "9", suit: "", selected: true},
                {id: null, value: 10, name: "10", suit: "", selected: true},
                {id: null, value: 10, name: "Jack", suit: "", selected: true},
                {id: null, value: 10, name: "Queen", suit: "", selected: true},
                {id: null, value: 10, name: "King", suit: "", selected: true}
            ],
            hand: {value: 0, cards: []},
            dealer: {value: 0, cards: []},
            gameStarted: false,
            inGame: false,
            restart: false,
            chooseAce: false,
            bank: 100,
            betPlaced: false,
            betAmount: 1,
            notEnoughFunds: false
        }
    },
    created() {
        this.buildDeck();
        this.shuffleDeck(this.deck);
    }, 
    methods: {
        submit : function(e){
            e.preventDefault();
            if (this.betAmount <= this.bank) {
                this.notEnoughFunds = false;
                this.bank -= this.betAmount;
                this.betPlaced = true;
                this.startGame();
            } else {
                this.notEnoughFunds = true;
            }
        },
        buildDeck() {
            let id = 1;
            for (let i = 0; i < this.suits.length; i++) { // 4 iterations
                for (let j = 0; j < this.cards.length; j++) { // 13 iterations
                    let card = {
                        id: id,
                        value: this.cards[j].value,
                        name: this.cards[j].name,
                        suit: this.suits[i].name,
                        selected: this.cards[j].selected,
                        img: this.suits[i].img
                    }
                    this.deck.push(card);
                    id++;
                }
            }
        },
        shuffleDeck(a) {
            var j, x, i;
            for (i = a.length - 1; i > 0; i--) {
                j = Math.floor(Math.random() * (i + 1));
                x = a[i];
                a[i] = a[j];
                a[j] = x;
            }
            return a;
        },
        getCard(i, loc) { // how many cards and where they're going
            for (let index = 0; index < i; index++) {
                let topCard = this.deck[1];
                if (loc == "player") {
                    this.hand.cards.push(topCard);
                    this.getValue();
                }
                if (loc == "dealer") {
                    this.dealer.cards.push(topCard);
                    this.getDealerValue();
                }
                this.deck.splice(0, 1);
            }
        },
        startGame() {
            this.getCard(2, "player");
            this.getCard(1, "dealer");
            this.gameStarted = true;
            this.inGame = true;
        },
        hit() {
            this.getCard(1, "player");
            this.checkBust();
        },
        stand() {
            this.runDealer();
        },
        checkBust() {
            if (this.hand.value > 21) {
                this.restart = true;
                this.inGame = false;
                this.betAmount = 0;
            }
        },
        restartGame() {
            this.deck = [];
            this.hand = {value: 0, cards: []};
            this.dealer = {value: 0, cards: []};
            this.gameStarted = false;
            this.inGame = false;
            this.restart = false;
            this.buildDeck();
            this.shuffleDeck(this.deck);
            this.betPlaced = false;
        },
        runDealer() {
            while (this.dealer.value < 17) {
                this.getCard(1, "dealer");
            }
            
            if (this.dealer.value == this.hand.value) {
                this.bank = this.bank + this.betAmount;
                this.betAmount = 0;
                this.restart = true;
                this.inGame = false;
            } else if (this.dealer.value < this.hand.value || this.dealer.value > 21) {
                this.bank = this.bank + this.betAmount * 2;
                this.betAmount = 0;
                this.restart = true;
                this.inGame = false;
            } else { // you lose :(
                this.betAmount = 0;
                this.restart = true;
                this.inGame = false;
            }
        },
        setAce(val, id) {
            let obj = this.hand.cards.find(obj => obj.id == id); // find object by id
            obj.value = val; // set value of ace to what option was passed from the button
            obj.selected = true; // set the ace to be selected so it can't be changed again
            let foundIndex = this.hand.cards.findIndex(x => x.id == id); // find index of ace that needs changing
            this.hand.cards[foundIndex] = obj; // change the ace to have correct value and selected
            this.getValue();
        },
        getValue() {
            this.hand.value = 0;
            for (let i = 0; i < this.hand.cards.length; i++) {
                if (!Array.isArray(this.hand.cards[i].value)) {// if not an ace
                    const cardVal = this.hand.cards[i].value;
                    this.hand.value += cardVal;
                }
            }
        },
        getDealerValue() {
            this.dealer.value = 0;
            for (let i = 0; i < this.dealer.cards.length; i++) {
                if (!Array.isArray(this.dealer.cards[i].value)) { // if not an ace
                    const cardVal = this.dealer.cards[i].value;
                    this.dealer.value += cardVal;
                } else { // else you got an ace big man
                    if (this.dealer.value < 11) { // if the dealer is on 10 or less the ace is worth 11.
                        const cardVal = this.dealer.cards[i].value[1]; // set value to 11
                        this.dealer.value += cardVal;
                    }
                    else { // if the dealer is on 11 or more the ace is worth 1.
                        const cardVal = this.dealer.cards[i].value[0]; // set value to 1
                        this.dealer.value += cardVal;
                    }
                }
            }
        }
    }
}
</script>

<style>
.container {
    max-width: 1200px;
    margin: 20px auto;
    position: relative;
}
.card {
    max-width: 200px;
    display: inline-block;
    padding: 20px;
}

.card img {
    width: 50px;
    height: 50px;
}
.bank {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 49%;
    background: black;
    color: white;
    padding: 20px 0;
}
.bank span {

}
.bet {
    right: 0;
    left: auto;
    border-right: none;
}
</style>