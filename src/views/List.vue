<template lang="">
<input type="file" ref="file" @change="loadDatabase" id="upload" text="upload"/>
<p @change="listLoaded"></p>
    <div class="container" id="card-list">
        <div class="row"> 
            <div v-for="item in cardsOwned" class="col-md-4">{{item.count}}x  {{item.name}}</div>
        </div>
    </div>
</template>

<script setup>
    import {ref} from 'vue'
    import sets from "../data/sets.json"

    const setList = sets;
    const cardsOwned = ref("");

    const items = ref();

    created()
    {
        console.log(setList);
    }
    function parseFile(file)
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
                    let temp = {name: card.Name,number : card['Card Number'], count: card.Count};
                    if(card.Count > 0)
                        tempList.push(temp);
                }
                //console.log(tempList);
                cardsOwned.value = tempList;
                //csv.value = e.target.result;
            })
        }
    }
</script>
<style lang="">
    
</style>