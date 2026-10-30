<script lang="ts">
    import debounce from "debounce";
    import { onMount, onDestroy } from "svelte";
    import { get } from "svelte/store";
    import { spring } from "svelte/motion";
    import { getMenuTransform, setMenuTransform } from "./hudVarsManager";
    import { showHud } from "../stores";
    
    export let name: string;

    // This whole file is a mess, but it's a mess that works*

    let transform = getMenuTransform(name);

    let x = transform?.x ?? 50;
    let y = transform?.y ?? 50;
    let width = transform?.width ?? 200;
    let height = transform?.height ?? 200;

    let lastWidth = width;
    let lastHeight = height;

    let coordSpring = spring(moveInbounds( x, y ), {
        stiffness: 0.1,
        damping: 0.5
    });

    let coords = get(coordSpring);
    let returnX: number = coords.x;
    let returnY: number = coords.y;

    function onResize() {
        let coords = get(coordSpring);
        coordSpring.set(moveInbounds(coords.x, coords.y));
    }

    function moveInbounds(x: number, y: number) {
        if(x < 0) x = 0;
        if(y < 0) y = 0;
        if(x + lastWidth > window.innerWidth) x = window.innerWidth - lastWidth;
        if(y + lastHeight > window.innerHeight) y = window.innerHeight - lastHeight;
        return { x, y };
    }

    let transitioning = false;

    // move out of the way when the hud is hidden
    showHud.subscribe((v) => {
        if(v) {
            coordSpring.set(moveInbounds(returnX, returnY));
            transitioning = false;
        } else {
            transitioning = true;
            let coords = get(coordSpring);

            // move over the nearest edge
            let cX = coords.x + lastWidth / 2;
            let cY = coords.y + lastHeight / 2;

            let endX = cX < window.innerWidth / 2 ? -lastWidth : window.innerWidth;
            let endY = cY < window.innerHeight / 2 ? -lastHeight : window.innerHeight;
            
            if(Math.abs(endX - coords.x) < Math.abs(endY - coords.y)) {
                coordSpring.set({ x: endX, y: coords.y });
            } else {
                coordSpring.set({ x: coords.x, y: endY });
            }
        }
    })

    let element: HTMLDivElement;

    let minimized = transform?.minimized ?? false;
    let dragState = 'waiting';
    let startX: number, startY: number;
    let dragDistance: number;

    function startDrag(event: MouseEvent) {
        dragState = 'checking';
        const coords = get(coordSpring)
        startX = event.clientX - coords.x;
        startY = event.clientY - coords.y;
        dragDistance = 0;
    }

    function saveTransform() {
        if(transitioning) return;
        let coords = get(coordSpring);
        setMenuTransform(name, {
            x: coords.x,
            y: coords.y,
            width: lastWidth,
            height: lastHeight,
            minimized
        })
    }

    const saveTransformDebounce = debounce(saveTransform, 100);
    coordSpring.subscribe(() => saveTransformDebounce())

    function drag(event: MouseEvent) {
        if(dragState == 'waiting') return;
        dragDistance += Math.abs(event.movementX) + Math.abs(event.movementY);

        if(dragState == 'checking' && dragDistance > 5) {
            dragState = 'dragging';
            return;
        }

        if(dragState == 'dragging') {
            let newX = event.clientX - startX;
            let newY = event.clientY - startY;

            returnX = newX;
            returnY = newY;

            coordSpring.set(moveInbounds(newX, newY));
            saveTransformDebounce();
        }
    }

    function stopDrag() {
        dragState = 'waiting';
    }

    let observer = new ResizeObserver((entries) => {
        dragState = 'waiting';
        let entry = entries[0];
        if(!minimized) {
            lastHeight = entry.contentRect.height;
        }
        lastWidth = entry.contentRect.width;
        saveTransformDebounce();
    });

    onMount(() => {
        observer.observe(element);
        element.style.height = `${Math.max(height, 38)}px`;
        element.style.width = `${Math.max(width, 240)}px`;
    });

    onDestroy(() => {
        observer.disconnect();
    });

    function toggleMinimized() {
        minimized = !minimized;
        saveTransformDebounce();
    }
</script>

<svelte:window on:mouseup={ stopDrag } on:mousemove={ drag } on:resize={onResize} />

<!-- svelte-ignore a11y-no-static-element-interactions-->
<div class="menu" class:minimized={ minimized } bind:this={ element }
on:mousedown={ startDrag }
style="left: { $coordSpring.x }px; top: { $coordSpring.y }px;">
    <div class="header">
        <span class="title">{name}</span>
        <button class="minimize" aria-label={minimized ? "Expand panel" : "Minimize panel"} on:click|stopPropagation={toggleMinimized}>
            {minimized ? "+" : "−"}
        </button>
    </div>
    <div class="children">
        <div class="groupContent open">
            <slot />
        </div>
    </div>
</div>

<style>
    .menu {
        --panel-bg: rgba(18, 22, 32, 0.97);
        --panel-border: rgba(255, 255, 255, 0.12);
        --panel-muted: rgba(255, 255, 255, 0.62);
        position: absolute;
        display: flex;
        flex-direction: column;
        box-sizing: border-box;
        background: var(--panel-bg);
        color: var(--textColor, #f4f6fb);
        border: 1px solid var(--panel-border);
        border-radius: 12px;
        box-shadow: 0 18px 50px rgba(0, 0, 0, 0.32), 0 2px 8px rgba(0, 0, 0, 0.2);
        resize: both;
        overflow: hidden;
        min-width: 240px;
        min-height: 38px;
        user-select: none;
        pointer-events: all;
        font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
        font-size: 13px;
        line-height: 1.45;
        backdrop-filter: blur(18px);
        -webkit-backdrop-filter: blur(18px);
        transition: box-shadow 160ms ease, border-color 160ms ease;
    }

    .menu:hover {
        border-color: rgba(255, 255, 255, 0.2);
        box-shadow: 0 20px 55px rgba(0, 0, 0, 0.38), 0 2px 8px rgba(0, 0, 0, 0.22);
    }

    .children {
        position: relative;
        flex: 1;
        min-height: 0;
        height: calc(100% - 44px);
        overflow-x: hidden;
        overflow-y: auto;
        padding: 10px;
        box-sizing: border-box;
        scrollbar-width: thin;
        scrollbar-color: rgba(255,255,255,.22) transparent;
    }

    .children::-webkit-scrollbar {
        width: 7px;
    }

    .children::-webkit-scrollbar-thumb {
        background: rgba(255,255,255,.2);
        border-radius: 999px;
    }

    :global(.groupContent) {
        position: absolute;
        top: 0;
        left: 0;
        display: flex;
        flex-direction: column;
        gap: 8px;
        width: 100%;
        box-sizing: border-box;
    }

    .groupContent {
        transform: translateX(0);
        opacity: 1;
        pointer-events: all;
    }

    .menu.minimized {
        overflow: hidden;
        resize: horizontal;
        height: 38px !important;
        min-height: 38px;
    }

    .header {
        display: flex;
        align-items: center;
        justify-content: space-between;
        flex: 0 0 42px;
        box-sizing: border-box;
        padding: 0 10px 0 14px;
        background: linear-gradient(180deg, rgba(255,255,255,.075), rgba(255,255,255,.025));
        border-bottom: 1px solid rgba(255,255,255,.09);
        color: var(--menuHeaderTextColor, #f8fafc);
        cursor: grab;
        text-align: left;
        font-size: 13px;
        font-weight: 650;
        letter-spacing: .01em;
    }

    .header:active {
        cursor: grabbing;
    }

    .title {
        overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
    }

    .minimize {
        display: inline-flex;
        align-items: center;
        justify-content: center;
        flex: 0 0 25px;
        width: 25px;
        height: 25px;
        margin-left: 12px;
        padding: 0;
        border: 1px solid rgba(255,255,255,.12);
        border-radius: 7px;
        background: rgba(255,255,255,.055);
        color: var(--menuHeaderTextColor, #f8fafc);
        font: inherit;
        font-size: 17px;
        line-height: 1;
        cursor: pointer;
        transition: background 140ms ease, border-color 140ms ease;
    }

    .minimize:hover {
        background: rgba(255,255,255,.13);
        border-color: rgba(255,255,255,.24);
    }

    .minimize:focus-visible,
    :global(#gc-hud button:focus-visible),
    :global(#gc-hud input:focus-visible),
    :global(#gc-hud select:focus-visible) {
        outline: 2px solid #8b9cff;
        outline-offset: 2px;
    }

    :global(#gc-hud button:not(.minimize)) {
        border-radius: 7px;
        transition: filter 140ms ease, background-color 140ms ease, border-color 140ms ease;
    }

    :global(#gc-hud button:not(.minimize):hover) {
        filter: brightness(1.12);
    }

    :global(#gc-hud input),
    :global(#gc-hud select),
    :global(#gc-hud textarea) {
        box-sizing: border-box;
        max-width: 100%;
        border: 1px solid rgba(255,255,255,.16);
        border-radius: 7px;
        padding: 7px 9px;
        background: rgba(8, 11, 18, .72);
        color: inherit;
        font: inherit;
    }

    :global(#gc-hud input[type="checkbox"]),
    :global(#gc-hud input[type="radio"]) {
        accent-color: #8b9cff;
    }

    @media (prefers-reduced-motion: reduce) {
        .menu, .minimize, :global(#gc-hud button) {
            transition: none;
        }
    }
</style>
