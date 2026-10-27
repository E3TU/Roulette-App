<script lang="ts">
    let items: Array<string> = ["Black", "Red", "Green"];
    let track: Array<any> = $state([]);
    let spinning: boolean = $state(false);
    let winner = $state(null);

    const itemWidth = 112;
    const visibleCount = 8;
</script>

<div class="container">
    <h1>Roulette App</h1>
    <div class="roulette">
        <div class="pointer"></div>

        <div class="track">

        </div>
    </div>
</div>

<style>
    .container{
        display: flex;
    }
    h1{
        color: var(--white);
        margin-top: 2rem;
        font-size: 3rem;
    }
</style>