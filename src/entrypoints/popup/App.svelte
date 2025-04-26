<script lang="ts">
    import ipaddr from "ipaddr.js";
    import { onMount } from 'svelte';

    // Constants
    const BUTTON_STATES = {
        COPY: "copy",
        COPIED: "copied"
    };

    // State variables
    let ipv4 = "";
    let ipv6 = "";
    let ipServer: string[] = [];
    let canonicalName = "";

    // Loading state tracking
    let loading = {
        ipv4: true,
        ipv6: true,
        server: true
    };

    // Variables to track the state of the buttons
    let ipv4ButtonText = BUTTON_STATES.COPY;
    let ipv6ButtonText = BUTTON_STATES.COPY;
    let serverButtonTexts: Record<string, string> = {};

    // Timer management
    let buttonTimers = new Map();

    async function fetchPublicIP(url: string): Promise<string> {
        try {
            const response = await fetch(url);
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            return await response.text();
        } catch (error) {
            console.error(`Error fetching IP from ${url}:`, error);
            return "Unavailable";
        }
    }

    async function fetchIPs() {
        loading.ipv4 = true;
        loading.ipv6 = true;
        
        try {
            // Fetch IPs in parallel for better performance
            const [ipv4Result, ipv6Result] = await Promise.all([
                fetchPublicIP("https://api.ipify.org"),
                fetchPublicIP("https://api64.ipify.org")
            ]);
            
            ipv4 = ipv4Result;
            ipv6 = ipv6Result;
        } catch (error) {
            console.error("Error fetching IPs:", error);
        } finally {
            loading.ipv4 = false;
            loading.ipv6 = false;
        }
    }

    async function fetchServerIPs() {
        loading.server = true;
        try {
            const tabs = await browser.tabs.query({
                currentWindow: true,
                active: true
            });
            if (tabs.length > 0) {
                const domain = new URL(tabs[0].url || "").hostname.replace("www.", "");
                if (domain) {
                    const resolved = await browser.dns.resolve(domain, ["bypass_cache", "canonical_name"]);
                    ipServer = resolved.addresses;
                    canonicalName = resolved.canonicalName || "Unavailable";

                    // Initialize button texts for server IPs
                    ipServer.forEach(ip => {
                        serverButtonTexts[ip] = BUTTON_STATES.COPY;
                    });
                }
            }
        } catch (error) {
            console.error("Error fetching server IPs:", error);
            ipServer = [];
            canonicalName = "Unavailable";
        } finally {
            loading.server = false;
        }
    }

    // Helper function to update button states
    function updateButtonState(type: "ipv4" | "ipv6" | "server", ip?: string) {
        // Reset all buttons
        ipv4ButtonText = BUTTON_STATES.COPY;
        ipv6ButtonText = BUTTON_STATES.COPY;
        Object.keys(serverButtonTexts).forEach(key => {
            serverButtonTexts[key] = BUTTON_STATES.COPY;
        });
        
        // Set the clicked button to "copied"
        if (type === "ipv4") {
            ipv4ButtonText = BUTTON_STATES.COPIED;
        } else if (type === "ipv6") {
            ipv6ButtonText = BUTTON_STATES.COPIED;
        } else if (ip) {
            serverButtonTexts[ip] = BUTTON_STATES.COPIED;
        }
    }

    // Helper function to reset button after delay
    function resetButtonAfterDelay(type: "ipv4" | "ipv6" | "server", ip?: string) {
        // Clear any existing timer for this button
        const timerId = type + (ip || "");
        if (buttonTimers.has(timerId)) {
            clearTimeout(buttonTimers.get(timerId));
        }
        
        // Set new timer
        const timer = setTimeout(() => {
            if (type === "ipv4") {
                ipv4ButtonText = BUTTON_STATES.COPY;
            } else if (type === "ipv6") {
                ipv6ButtonText = BUTTON_STATES.COPY;
            } else if (ip) {
                serverButtonTexts[ip] = BUTTON_STATES.COPY;
            }
            buttonTimers.delete(timerId);
        }, 3000);
        
        buttonTimers.set(timerId, timer);
    }

    async function copyIpOnClipboard(ip: string, type: "ipv4" | "ipv6" | "server" = "server") {
        try {
            // Copy the IP to the clipboard
            await navigator.clipboard.writeText(ip);

            // Update button states
            updateButtonState(type, type === "server" ? ip : undefined);
            
            // Set timer to reset button
            resetButtonAfterDelay(type, type === "server" ? ip : undefined);
        } catch (error) {
            console.error("Failed to copy IP to clipboard", error);
        }
    }

    // Lifecycle management
    onMount(() => {
        fetchIPs();
        fetchServerIPs();
        
        // Cleanup on component destruction
        return () => {
            buttonTimers.forEach(timer => clearTimeout(timer));
        };
    });
</script>

<main class="flex w-[400px] flex-col p-4 text-[.8rem] select-none">
    <h1 class="sr-only">Get IPs</h1>

    <div class="text-center text-xl text-violet-700 select-none">°˖✧◝(⁰▿⁰)◜✧˖°</div>

    <h2 class="my-2 text-xl font-bold select-none">Your public IPs :</h2>

    {#if loading.ipv4}
        <div class="my-1">
            <strong>IPv4 : </strong> <span class="text-gray-500">loading...</span>
        </div>
    {:else if ipv4 !== "Unavailable"}
        <div class="my-1">
            <strong>IPv4 : </strong>
            <span>{ipv4}</span>
            <button
                class="ml-2 cursor-pointer rounded border px-1 text-[.6rem] transition-opacity hover:opacity-75 {ipv4ButtonText === BUTTON_STATES.COPIED ? 'bg-green-200 text-green-800 border-green-800' : 'bg-purple-200 text-violet-800 border-violet-800'}"
                on:click={() => copyIpOnClipboard(ipv4, "ipv4")}
                aria-label={ipv4ButtonText === BUTTON_STATES.COPIED ? `IPv4 address ${ipv4} copied to clipboard` : `Copy IPv4 address ${ipv4} to clipboard`}>
                {ipv4ButtonText}
            </button>
        </div>
    {:else}
        <div class="my-1">
            <strong>IPv4 : </strong> unavailable
        </div>
    {/if}
    
    {#if loading.ipv6}
        <div class="my-1">
            <strong>IPv6 : </strong> <span class="text-gray-500">loading...</span>
        </div>
    {:else if ipv6 !== "Unavailable"}
        <div class="my-1">
            <strong>IPv6 : </strong>
            <span>{ipv6}</span>
            <button
                class="ml-2 cursor-pointer rounded border px-1 text-[.6rem] transition-opacity hover:opacity-75 {ipv6ButtonText === BUTTON_STATES.COPIED ? 'bg-green-200 text-green-800 border-green-800' : 'bg-purple-200 text-violet-800 border-violet-800'}"
                on:click={() => copyIpOnClipboard(ipv6, "ipv6")}
                aria-label={ipv6ButtonText === BUTTON_STATES.COPIED ? `IPv6 address ${ipv6} copied to clipboard` : `Copy IPv6 address ${ipv6} to clipboard`}>
                {ipv6ButtonText}
            </button>
        </div>
    {:else}
        <div class="my-1">
            <strong>IPv6 : </strong> unavailable
        </div>
    {/if}

    <h2 class="my-2 text-xl font-bold select-none">The current URL IPs :</h2>

    {#if loading.server}
        <div class="my-1">
            <strong>Server IPs : </strong> <span class="text-gray-500">loading...</span>
        </div>
    {:else if ipServer.length > 0}
        <div class="my-1">
            <strong>Canonical name : </strong>
            <span>{canonicalName}</span>
        </div>
        {#each ipServer as ip}
            <div class="my-1">
                {#if ipaddr.IPv4.isValid(ip)}
                    <strong>IPv4 : </strong>
                {:else if ipaddr.IPv6.isValid(ip)}
                    <strong>IPv6 : </strong>
                {:else}
                    <strong>Invalid IP : </strong>
                {/if}
                <span>{ip}</span>
                <button
                    class="ml-2 cursor-pointer rounded border px-1 text-[.6rem] transition-opacity hover:opacity-75 {serverButtonTexts[ip] === BUTTON_STATES.COPIED ? 'bg-green-200 text-green-800 border-green-800' : 'bg-purple-200 text-violet-800 border-violet-800'}"
                    on:click={() => copyIpOnClipboard(ip, "server")}
                    aria-label={serverButtonTexts[ip] === BUTTON_STATES.COPIED ? `IP address ${ip} copied to clipboard` : `Copy IP address ${ip} to clipboard`}>
                    {serverButtonTexts[ip] || BUTTON_STATES.COPY}
                </button>
            </div>
        {/each}
    {:else}
        <div class="my-1">
            <strong>Server IPs : </strong> unavailable
        </div>
    {/if}
    <div class="absolute right-4 bottom-1 flex items-center gap-1 text-[.6rem] text-violet-700 select-none">
        <a class="transition-opacity hover:opacity-75" href="https://devbit.me" title="devbit.me" target="_blank" rel="noopener noreferrer">
            <img src="devbitme.png" alt="logo devbitme" />
        </a>
        <div>
            getIPs <a
                class="font-bold transition-opacity hover:opacity-75"
                href="https://github.com/devbitme/getIPs/releases"
                target="_blank"
                rel="noopener noreferrer"
                title="Releases">2.0.0</a
            >
        </div>
    </div>
</main>
