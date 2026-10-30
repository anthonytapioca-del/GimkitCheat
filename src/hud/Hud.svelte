<script lang="ts">
    import Menu from "./Menu.svelte";
    import Group from "./Group.svelte";
    import ColorPicker from "./components/ColorPicker.svelte";

    import { showHud } from '../stores';
    import keybindManager from "../keybindManager";
    import ResetStyles from "./ResetStyles.svelte";

    import AutoAnswer from "../scripts/AutoAnswer.svelte";
    import InstantUse from "../scripts/2d/InstantUse.svelte";
    import CosmeticPicker from "../scripts/customization/CosmeticPicker.svelte";
    import CustomTheme from "../scripts/customization/customTheme/CustomTheme.svelte";
    import PlayerHighlighter from "../scripts/2d/PlayerHighlighter.svelte";
    import Freecam from "../scripts/2d/Freecam.svelte";
    import HideEnergyPopup from "../scripts/2d/HideEnergyPopup.svelte";
    import ClassicAutoPurchase from "../scripts/classic/AutoPurchase.svelte";
    import RichAutoPurchase from "../scripts/superRichMode/AutoPurchase.svelte";
    import ShowImposters from "../scripts/trustNoOne/ShowImposters.svelte";
    import Ctf from "../scripts/ctf/Ctf.svelte";
    import Tag from "../scripts/tag/Tag.svelte";
    import OneWayOut from "../scripts/oneWayOut/OneWayOut.svelte";
    import Snowbrawl from "../scripts/snowbrawl/Snowbrawl.svelte";
    import FloorIsLava from "../scripts/floorIsLava/FloorIsLava.svelte";
    import Farmchain from "../scripts/farmchain/Farmchain.svelte";
    import Movement from "../scripts/2d/Movement.svelte";
    import DigItUp from "../scripts/digItUp/DigItUp.svelte";
    import Knockback from "../scripts/knockback/Knockback.svelte";

    let activeTab = "General";

    keybindManager.addKeybind(new Set(["\\"]), () => {
        showHud.update(v => !v);
    })
</script>

<div id="gc-hud" style="display: {$showHud ? 'block' : 'none'}">
    <Menu name="Gimkit Hub">
        <div class="gc-tabs" role="tablist" aria-label="HUD sections">
            <button class:active={activeTab === "General"} role="tab" aria-selected={activeTab === "General"} on:click|stopPropagation={() => activeTab = "General"}>General</button>
            <button class:active={activeTab === "Gamemodes"} role="tab" aria-selected={activeTab === "Gamemodes"} on:click|stopPropagation={() => activeTab = "Gamemodes"}>Gamemodes</button>
            <button class:active={activeTab === "Customize"} role="tab" aria-selected={activeTab === "Customize"} on:click|stopPropagation={() => activeTab = "Customize"}>Customize</button>
        </div>

        {#if activeTab === "General"}
            <div class="gc-tab-content">
                <AutoAnswer />
                <InstantUse />
                <Movement />
                <PlayerHighlighter />
                <Freecam />
                <HideEnergyPopup />
            </div>
        {:else if activeTab === "Gamemodes"}
            <div class="gc-tab-content">
                <ClassicAutoPurchase />
                <RichAutoPurchase />
                <DigItUp />
                <Knockback />
                <Ctf />
                <Tag />
                <OneWayOut />
                <Snowbrawl />
                <FloorIsLava />
                <ShowImposters />
                <Farmchain />
            </div>
        {:else}
            <div class="gc-tab-content">
                <Group name="HUD Appearance">
                    <ColorPicker text="Text Color" bindValue="textColor" minOpactiy={0.3} />
                    <Group name="Menu Appearance">
                        <ColorPicker text="Background Color" bindValue="menuBackgroundColor" />
                        <ColorPicker text="Outline Color" bindValue="menuOutlineColor" />
                        <ColorPicker text="Header Color" bindValue="menuHeaderBackgroundColor" />
                        <ColorPicker text="Header Text Color" bindValue="menuHeaderTextColor" />
                    </Group>
                    <Group name="Button Appearance">
                        <ColorPicker text="Background Color" bindValue="buttonBackgroundColor" />
                        <ColorPicker text="Outline Color" bindValue="buttonBorderColor" />
                    </Group>
                </Group>
                <ResetStyles />
                <CosmeticPicker />
                <CustomTheme />
            </div>
        {/if}
    </Menu>
</div>

<style>
#gc-hud {
    position: absolute;
    top: 0;
    left: 0;
    z-index: 9999999999;
    width: 100vw;
    height: 100vh;
    pointer-events: none;
    color: var(--textColor);
}
:global(#gc-hud .gc-tabs) {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 5px;
    padding: 8px;
    border-bottom: 1px solid rgba(255,255,255,.09);
    background: rgba(255,255,255,.025);
}
:global(#gc-hud .gc-tabs button) {
    min-width: 0;
    padding: 8px 6px;
    border: 1px solid rgba(255,255,255,.1);
    border-radius: 8px;
    background: rgba(255,255,255,.045);
    color: inherit;
    font: inherit;
    font-weight: 600;
    cursor: pointer;
}
:global(#gc-hud .gc-tabs button.active) {
    background: rgba(129,145,255,.2);
    border-color: rgba(145,157,255,.55);
    color: #fff;
}
:global(#gc-hud .gc-tab-content) {
    display: flex;
    flex-direction: column;
    gap: 9px;
    padding: 4px 0 8px;
    width: 100%;
    box-sizing: border-box;
}
</style>