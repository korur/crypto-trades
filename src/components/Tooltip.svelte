<script>
    export let title;
    export let x;
    export let y;
    export let xoffset;
    0;
</script>

<div
    style="
		width: fit-content;
		max-width: 250px;
		top: {y + 5}px;
		left: {x + 5}px;"
>
    {title}
</div>

<style>
    div {
        border: 1px solid black;
        box-shadow: 1px 1px 1px #ddd;
        background: white;
        border-radius: 4px;
        padding: 4px;
        position: absolute;
        font-weight: 400;
        color: black;
        margin-left: 0;
        margin-right: 0;
        font-size: 0.75rem;
        z-index: 30000;
    }
</style>
