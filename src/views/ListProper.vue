<template lang="">
<form>
    <input type="file" ref="file" @change="parseFile" id="upload" text="upload"/>
    <button type="button" @click="this.showCategories = !this.showCategories">{{this.showCategories ? "Hide categories" : "Show Categories"}}</button>

    <div v-if="showCategories" class="container">
        <div class="row">
            <div v-for="cat in types" class="col-md-4">
                <input type="checkbox" @change="filterSetsByCategory" v-model="categoriesSelected" :value="cat.category" :id="'category_' + cat.id" />
                <label :for="'category_' + cat.id">{{cat.category}}<br /></label>
            </div>
        </div>
    </div>

    <div class="container">
        <select id="setSelected" v-model="setSelected">
            <option v-for="set in filteredSetList" :value="set.name"> {{set.name}} </option>
        </select>
        <button id="showCards" type="button" @click="showCards">Show cards</button>
    </div>

</form>
<p>Selected: {{categoriesSelected}} Set: {{setSelected}} </p>
    <div class="container" id="card-list">
        <div class="row" style="display:none"> 
            <div v-for="item in filteredCardsOwned" class="col-md-4">{{item.count}}x  {{item.name}}</div>
        </div>
    </div>

    <div class="container" id="setCardList">
        <div class="row" style="">
            <div v-for="card in cardsInSet" class="col-md-2" :style="isCollected(card) ? 'text-decoration : line-through; color: green; opacity: 0.5' : 'color:red' ">{{card.name}}
                <img class="img-thumbnail" :src="card.image" />

            </div>
        </div>
    </div>

    <button type="button" @click="minifyCards">!!!</button>
</template>

<script>
import sets from "../data/sets.json"
import defaultCards from "../data/default-cards-min.json"
import Papa from 'papaparse';
export default {
    created() {
        const setList = sets.sets;
        
        setList.sort(function (a,b)
        {
            let dateA = new Date(a.releaseDate);
            //console.log(dateA);
            let dateB = new Date(b.releaseDate);
            //console.log(dateB);
            //console.log(`is a > b? ${dateA > dateB}\n`);
            return dateB.getTime() -  dateA.getTime();
        });

        console.log(setList);
        for(let i = 0; i < setList.length; i++)
        {
            //console.log(this.types.indexOf(setList[i].type));
            let exists = false;
            if(setList[i].onlineOnly || setList[i].type == "funny" || setList[i].type == "memorabilia")
            {
                setList.splice(i,1);
            }
            else
            {
                for(let item in this.types)
                {
                    //console.log(`item type is ${this.types[item].category}, list type is ${setList[i].type}`);
                    if(this.types[item].category == setList[i].type)
                        exists = true;
                }
    
                if(!exists)
                {
                    let t = this.types;
                    let tempObj = {id: this.types.length,category:setList[i].type}
                    t.push(tempObj);
                    //console.log(t);
                    this.types = t;
                }
            }

        }
        console.log(setList);
        this.setList = setList;
        this.filteredSetList = setList;

        let defaultCategories = [];
        defaultCategories.push('core');
        defaultCategories.push('expansion');

        this.categoriesSelected = defaultCategories;
        this.filterSetsByCategory();
    },
    data() {
        return {
            cardsOwned : [], //unknown until user uploads their CSV string from deckbox
            filteredCardsOwned: [],

            setList : null, //set List
            filteredSetList: null,


            types: [], //this is really categories, shit
            categoriesSelected: [],
            showCategories: false,

            cardsInSet: [],
            setSelected: null,
        }
    },
    methods:
    {
        parseFile(file)
        {
            let list = file.target.files;
            console.log(list);

            if(file.target.files[0].type.includes("csv"))
            {
                console.log("is a csv file");
                let cvsText = "";

                const r = new FileReader();
                r.readAsText(file.target.files[0]);

                r.addEventListener('load', (e) => {
                    let parsed = Papa.parse(e.target.result,{header:true});
                    console.log(parsed);
                    
                    let tempList = [];
                    for(let i = 0; i < parsed.data.length; i++)
                    {
                        let card = parsed.data[i];
                        //console.log(parsed.data[i]);
                        //console.log(card);
                        let temp = {name: card.Name,number : card['Card Number'], count: card.Count, set: card.Edition};
                        if(card.Count > 0)
                            tempList.push(temp);
                    }
                    console.log(tempList);
                    this.cardsOwned = tempList;
                    //csv.value = e.target.result;
                })
            }
        },

        toggleShowCategories()
        {
            this.showCategories = !this.showCategories;
        },

        filterSetsByCategory()
        {
            let tempList = [];
            for(let set in this.setList)
            {
                let shouldDisplay = false;
                for(let item in this.categoriesSelected)
                {
                    //console.log(`category selected: ${this.categoriesSelected[item]}. Set type: ${this.setList[set].type}`);
                    if(this.categoriesSelected[item] == this.setList[set].type)
                        shouldDisplay = true;
                }
                //console.log(`Should display? ${shouldDisplay}`);
                if(shouldDisplay)
                {
                    tempList.push(this.setList[set]);
                    //console.log(this.setList[set]);
                }

            }
            //console.log(tempList);
            this.filteredSetList = tempList;
        },
        filterCardsOwnedBySet()
        {
            let tempList = [];

            for(let card in this.cardsOwned)
            {
                //console.log(`card: ${this.cardsOwned[card].set} is in ${this.setSelected}?`);
                if(this.cardsOwned[card].set == this.setSelected)
                {
                    tempList.push(this.cardsOwned[card]);
                    //console.log("hell yeah it is!");
                }
            }
            //console.log(tempList);
            this.filteredCardsOwned = tempList;
        },
        showCards()
        {
            let tempCardList = [];

            for(let card in defaultCards)
            {
                //console.log(defaultCards[card]);    
                if(defaultCards[card].set_name == this.setSelected)
                    tempCardList.push(defaultCards[card]);
            }
            //console.log(defaultCards);

            this.cardsInSet = tempCardList;
            this.filterCardsOwnedBySet();
            console.log(this.cardsInSet);
        },
        minifyCards()
        {
            console.log("working");
            let cardList = [];

            for(let card in defaultCards)
            {
                let tempCard = defaultCards[card];
                //console.log(tempCard.name);

                let image = typeof tempCard.image_uris == 'undefined' ? 'default.jpg' : tempCard.image_uris.normal;
                let temp = {
                    collector_number: tempCard.collector_number,
                    name: tempCard.name,
                    set: tempCard.set,
                    set_name: tempCard.set_name,
                    image: image
                };
                cardList.push(temp);
            }

            let jsonCardList = JSON.stringify(cardList);
            console.log(jsonCardList);
            //const fs = require('fs');
            //fs.writeFile("minifiedCards.json", jsonCardList,"utf-8");
        },
        isCollected(card)
        {
            //console.log(card);
            //console.log(`looking for ${card.name}`);
            for(let c in this.filteredCardsOwned)
            {
                //console.log(`current card: ${this.filteredCardsOwned[c].name}`);
                //for(let cardOwned in this.filteredCardsOwned)
                //{
                    //console.log(`looking for card ${this.cardsInSet[c].name} among ${this.cardsOwned[cardOwned].name}`);
                    if (this.filteredCardsOwned[c].name == card.name)
                    {
                        //console.log(`${this.cardsInSet[c].name} is collected!`);
                        return true;
                    }
                //}
            }
            return false;
        }
    },
    computed: {
    }
}
</script>

<style>

</style>