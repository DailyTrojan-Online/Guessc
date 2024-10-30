<script lang="ts">
    let showModal = false;


    import { onMount } from "svelte";
    let mousex , mousey;
    let mapRect;
    let map;
    let xfixed;
    let yfixed;
    let clickx, clicky;
    let dist;
    $: if(map != undefined) {
        mapRect = map.getBoundingClientRect();
    }
    $: console.log(mapRect);
    function mouseMove(event) {
        mapRect = map.getBoundingClientRect();
        mousex = (event.clientX - mapRect.left) / mapRect.width;
        xfixed = mousex.toFixed(2);
        mousey = (event.clientY - mapRect.top) / mapRect.height;
        yfixed = mousey.toFixed(2);
        if (mousex < 0 || mousey < 0) {
            mousex = 0;
            mousey = 0;
        }
    }
    function mouseclick(event) {
        clickx = mousex;
        clicky = mousey;
        
        dist = Math.sqrt((clickx-0.4355555216471354) ** 2 + (clicky-0.8228022643958655) ** 2);
        dist = dist.toFixed(2);
        dist = 5000-dist * 5000;
    }
    function buttonclick(event){
        showModal = true;
    }
</script>
<svelte:window on:pointermove={mouseMove}></svelte:window>
<div class="game-wrapper">
    <h1>Coordinates</h1>
    <p>X: {xfixed} y: {yfixed}</p>
    <p>X: {mousex} y: {mousey}</p>
    <p>clickX: {clickx} clickY: {clicky}</p>

    <div class="image">
        <img src="./images/test-image.jpg" alt="Guess-Image" />
    </div>
    <div class="map-wrapper">
        <div class="map">
            <div bind:this={map} on:click={mouseclick} class="image-wrapper">
                <img src="./Map.png" alt="Map" />
            </div>
            <button on:click={buttonclick}>Submit</button>
        </div>
    </div>
</div>
{#if showModal}
    <div class="modal">
        <div class="modal-content">
            <img src="./Map.png" alt="Map" />
            <p>Distance: {dist}</p>
            <button on:click={() => showModal = false}>Close</button>
        </div>
    </div>
{/if}
<style>
    .modal-content img{
        width: 100%;

    }
    .modal{
        position: fixed;
        top:50%;
        left:50%;
        transform: translate(-50%, -50%);
        width: 600px;
        z-index: 999;
        background-color: red;
    }
    .game-wrapper {
        width: 100%;
        height: 100%;
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        box-sizing: border-box;
    }
    .x-axis {
        width: 1px;
        height: 1px;
        background-color: red;
    }
    .y-axis {
        width: 1px;
        height: 1px;
        background-color: blue;
    }
    .image {
        width: 100%;
        height: 100%;
        border: 1px solid black;
        box-sizing: border-box;
    }
    .image img {
        width: 100%;
        height: 100%;
        position: absolute;
    }
    .map-wrapper {
        position: relative;
        border: 1px solid black;
    }
    .map {
        position: fixed;
        bottom: 0;
        right: 0;
        border: 1px solid black;
        background-color: white;
    }
    .map img {
        width: 300px;
    }
    .image-wrapper {
        transition: 0.2s;
        transform-origin: bottom right;
    }
    .image-wrapper:hover {
        cursor: crosshair;
        transform: scale(1.5);
    }
</style>
