<script lang="ts">
    let t:number;
    let v:number;
    let res:number = null;
    let formattedRes:string;

    let t_units = ['years', 'days', 'hours', 'minutes', 'seconds']
    let v_units = ['m/s', 'km/h', '%c']
    let display_v_units:{[key:string]:string} = {'m/s': ' m/s', 'km/h': ' km/h', '%c': '% c'}

    let t_unit:string;
    let v_unit:string = '%c'

    let cByUnit: {[key: string]: number} = {'m/s': 299792458, 'kph': 299792458*3600, '%c': 100}


    $: t_unit, v_unit, t, v, updateTime();

    function updateTime(){
        if(v == null || t == null) return
        res = calculateTime()
        let time = formatTime(t_unit, res)
        formattedRes = ''
        for(let unit in time){
            if(time[unit] >= 2){
                formattedRes += `${time[unit]} ${unit}, `
            }
            else if(time[unit] > 0 && time[unit] < 2){
                formattedRes += `${time[unit]} ${unit.slice(0, -1)}, `
            }
        }
        formattedRes = formattedRes.slice(0, -2)
        return formattedRes;
        
    }

    function calculateTime():number {
        /*if(t == null || v == null){
            return
        }*/
        let time = t * Math.sqrt(1-(v**2/cByUnit[v_unit]**2))
        return Math.round(time * 1000000)/1000000;
    }

    function vInput(e){
        if(e.target.value < 0){
            v = 0;
        }
        console.log(v_unit)
        if(e.target.value > cByUnit[v_unit]){
            v = cByUnit[v_unit];
        }
    }

    function vUnitChange(e){
        let newUnit = e.target.value;
        if(v){
            v = convertVToNewUnit(v_unit, newUnit, v)
        }

        if(v > cByUnit[newUnit]){
            v = cByUnit[newUnit]
        }
        v_unit = e.target.value
        
    }

    function convertVToNewUnit(oldUnit:string, newUnit:string, value:number): number{
        let res;
        if(oldUnit == 'm/s'){
            if(newUnit == 'kph'){
                res = (value/1000)*3600
            }
            else {//if(newUnit == '%c'){
                res =(value/cByUnit['m/s'])*100
            }
        }
        else if(oldUnit == 'kph'){
            if(newUnit == 'm/s'){
                res = (value*1000)/3600
            }
            else {//if(newUnit == '%c'){
                res = (value/cByUnit['kph'])*100
            }
        }
        else {//if(oldUnit == '%c'){
            res = (value/100)*cByUnit[newUnit]
        }
        return Math.round(res * 10000)/10000
    }

    function formatTime(unit:string, value:number){
        let res:{[key:string]:number} = {years: 0, days: 0, hours: 0, minutes: 0, seconds: 0}
        let unitCoeff:{[key:string]:number} = {years: 365.25, days: 24, hours: 60, minutes: 60}
        let idx = t_units.indexOf(unit)
        for(let i = idx;  i < t_units.length; i++){
            if(t_units[i] == 'seconds'){
                res[t_units[i]] = Math.round(value)
            }
            else{
                res[t_units[i]] = Math.trunc(value)
            }
            value = (value % 1) * unitCoeff[t_units[i]]
        }
        return res
    }


</script>
<div id='body' class='h-screen p-6 flex flex-col items-center justify-center'>
    <div id='formula' class='flex justify-center items-center main-color'>
        <p class='text-5xl font-bold'>t = {t == null ? "t'" : t}</p>
        <p class='text-3xl font-bold mx-2'>x</p>
        <p class='text-8xl font-bold'>√</p>
        <div id='sqrt' class='border-t-8 flex items-center pr-1'>
            <p class='text-5xl font-bold'>1 -</p>
            <div class='ml-4 flex flex-col items-center'>
                <p class='text-3xl font-bold'>{v == null ? 'v' : v}<sup>2</sup></p>
                <hr id='divide-line' class='h-1 w-full'>
                <p class='text-3xl font-bold'>c<sup>2</sup></p>
            </div>
        </div>
        {#if res != null}
        <p class='text-5xl font-bold ml-2'>= {res} {t_unit}</p>
        {/if}
    </div>
    <div class="main-color mt-8 flex flex-col items-center">
        {#if res != null}
            <p class='text-xl mb-2'>Time experienced by a traveler while traveling at {v}{display_v_units[v_unit]}</p>
            <p class='text-3xl'>{formattedRes}</p>
        {/if}
    </div>
    <!--<div class="main-color mt-4">
        <p>t: Time experienced by traveler</p>
        <p>t': Time experienced on Earth</p>
        <p>v: Velocity of traveler relative to Earth</p>
    </div>-->
    <div class='flex mt-12'>
        <p class='text-xl w-56'>Time passed on Earth (t'):</p>
        <input type="number" class='border-b-2 border-slate-400 mx-2 px-2 text-center w-40 focus:outline-none bg-transparent' bind:value={t}>
        <select id="t-unit" class='border-2 border-slate-400 w-24 px-2 text-center bg-transparent' bind:value={t_unit}>
            {#each t_units as unit}
                <option value={unit}>{unit}</option>
            {/each}
        </select>
    </div>
    <div class='flex mt-3'>
        <p class='text-xl w-56'>Velocity of traveler (v):</p>
        <input type="number" id='v-input' min=0 class='border-b-2 border-slate-400 mx-2 px-2 text-center w-40 focus:outline-none bg-transparent' bind:value={v} on:input={(e) => {vInput(e)}}>
        <select id="v-unit" class='border-2 border-slate-400 w-24 px-2 text-center bg-transparent' value={v_unit} on:change={(e) => {vUnitChange(e)}}>
            {#each v_units as unit}
                <option value={unit}>{unit}</option>
            {/each}
        </select>
    </div>
</div>

<style lang="postcss">


    .main-color{
        color: rgb(80, 107, 128)
    }

    #divide-line{
        background-color: rgb(80, 107, 128);
    }

    #sqrt{
        border-color: rgb(80, 107, 128);
    }
  </style>