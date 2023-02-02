<script>
    import {
        AppStyle,
        Baseline as baseline,

        Button,

        vars,
    } from "svelte-doric"
    import { TronTheme as theme } from "svelte-doric/theme"

    let previewURL = ""
    let image = null
    let targetFile = ""
    function preview(evt) {
        const file = evt.target.files[0]
        console.log(file)
        targetFile = `${file.name}-bg.png`
        previewURL = URL.createObjectURL(file)
        image = new Image()
        image.addEventListener(
            "load",
            () => {
                scale = Math.max(
                    width / image.width,
                    height / image.height
                )
                pos = [0, 0]
                dist = [0, 0]
            }
        )
        image.src = previewURL
    }

    const aspectRatio = screen.height / screen.width

    const elemScale = 0.7
    const width = screen.width * elemScale
    const height = screen.height * elemScale

    let scale = 1

    const op = {
        add: (a, b) => [
            a[0] + b[0],
            a[1] + b[1]
        ],
        sub: (a, b) => [
            a[0] - b[0],
            a[1] - b[1]
        ],
    }
    function coords(pos) {
        return {
            x: `${pos[0]}px`,
            y: `${pos[1]}px`
        }
    }
    function snapCoord(coord, dist, max) {
        // console.log(coord, max, dist)
        if (Math.abs(coord) < dist) {
            return 0
        }
        if (Math.abs(coord - max) < dist) {
            return max
        }
        return coord
    }
    function edgeSnap(pos, snap) {
        if (image === null) {
            return pos
        }
        if (snap === false) {
            return pos
        }

        return [
            snapCoord(pos[0], 5, -(image.width * scale - width)),
            snapCoord(pos[1], 5, -(image.height * scale - height)),
        ]
    }

    let touchID = null
    let start = [-1, -1]
    let current = [-1, -1]
    $: dist = op.sub(current, start)

    let pos = [0, 0]
    let snap = true
    $: offset = edgeSnap(
        op.add(pos, dist),
        snap
    )
    function begin(evt) {
        start = [evt.clientX, evt.clientY]
        current = start
    }
    function move(evt) {
        current = [evt.clientX, evt.clientY]
    }
    function stop(evt) {
        pos = [...offset]
        dist = [0, 0]
    }

    let fillColor = "rgb(255, 255, 255)"
    let previewElem = document.createElement("canvas")
    function save() {
        const factor = devicePixelRatio
        previewElem.width = width * factor
        previewElem.height = height * factor

        const context = previewElem.getContext("2d")
        context.fillStyle = fillColor
        context.fillRect(0, 0, width * factor, height * factor)
        context.drawImage(
            image,
            pos[0] * factor,
            pos[1] * factor,
            image.width * factor * scale,
            image.height * factor * scale
        )

        const a = document.createElement("a")
        a.href = previewElem.toDataURL()
        a.download = targetFile
        a.click()
    }

    $: previewSettings = {
        scale,
        fillColor,
        width: `${width}px`,
        height: `${height}px`,
    }
</script>

<style>
    :global(body) {
        overflow: hidden;
        touch-action: none;
    }
    preview-area {
        display: block;
        width: var(--width);
        height: var(--height);
        background-color: var(--fillColor);

        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);

        overflow: hidden;
    }
    crop-border {
        border: 2px solid var(--primary);
        position: absolute;
        top: 0px;
        left: 0px;
        bottom: 0px;
        right: 0px;
    }

    img {
        pointer-events: none;
        transform-origin: 0% 0%;
        transform:
            translate(
                var(--x),
                var(--y)
            )
            scale(
                var(--scale)
            )
        ;
    }
</style>


<AppStyle {baseline} {theme} />

<preview-area use:vars={previewSettings}
on:pointerdown={begin}
on:pointermove={move}
on:pointerup={stop}
>
    <img src={previewURL} use:vars={coords(offset)} alt="" />
    <crop-border />
</preview-area>
<input type="file" on:change={preview} />
<Button on:tap={save}>
    Save?
</Button>
<input type="color" bind:value={fillColor} />
<!-- <canvas bind:this={previewElem} /> -->
