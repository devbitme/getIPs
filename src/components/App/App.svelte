<script lang="ts">
  import AppWrapper from './AppWrapper.svelte';
  import axios from 'axios';
  import ipaddr from 'ipaddr.js';

  let ipv4 = '';
  let ipv6 = '';
  let ipServer = [];
  let canonicalName = '';

  // récupérer l'IPv4 client
  axios.get('https://api.ipify.org').then(function (response) {
    ipv4 = response.data;
  });

  // récupérer l'IPv6 client
  axios.get('https://api64.ipify.org').then(function (response) {
    ipv6 = response.data;
  });

  // récupérer les IPs server
  browser.tabs
    .query({
      currentWindow: true,
      active: true,
    })
    .then(function (tabInfo) {
      let domain = new URL(tabInfo[0].url).hostname.replace('www.', '');
      if (domain) {
        let resolving = browser.dns.resolve(domain, ['bypass_cache', 'canonical_name']);
        resolving.then((resolved) => {
          console.log(resolved);
          ipServer = resolved.addresses;
          canonicalName = resolved.canonicalName;
        });
      }
    });

  function copyIpOnClipboard(data) {
    navigator.clipboard.writeText(data.target.dataset.ip).then(
      function () {
        data.target.innerText = 'copied';
        data.target.className = 'btn-success';
        setTimeout(function () {
          data.target.innerText = 'copy';
          data.target.className = 'btn-primary';
        }, 3000);
      },
      function () {
        console.log('failed');
      }
    );
  }
</script>

<AppWrapper>
  <div class="root__wrapper">
    <h1 class="d-none">Get IPs</h1>
    <div class="kaomoji">°˖✧◝(⁰▿⁰)◜✧˖°</div>
    <h2>Your public IPs :</h2>
    {#if ipv4 !== ''}
      <strong>IPv4 :</strong>
      {ipv4}<button class="btn-primary" on:click={copyIpOnClipboard} data-ip={ipv4}>copy</button>
    {:else}
      <strong>IPv4</strong> unavailable
    {/if}
    {#if ipv6 !== ''}
      <br />
      <strong>IPv6 :</strong>
      {ipv6}<button class="btn-primary" on:click={copyIpOnClipboard} data-ip={ipv6}>copy</button>
    {:else}
      <br />
      <strong>IPv6</strong> unavailable
    {/if}
    <h2>The current URL IPs :</h2>
    {#if ipServer.length}
      <strong>Canonical name :</strong>
      {canonicalName}
      <br />
      {#each ipServer as ip}
        {#if ipaddr.IPv4.isValid(ip)}
          <strong>IPv4 :</strong>
        {:else if ipaddr.IPv6.isValid(ip)}
          <strong>IPv6 :</strong>
        {:else}
          <strong>IP no valid :</strong>
        {/if}
        {ip}<button class="btn-primary" on:click={copyIpOnClipboard} data-ip={ip}>copy</button><br />
      {/each}
    {:else}
      <strong>IP server</strong> unavailable
    {/if}
    <div class="version">getIPs <strong>1.2.0</strong></div>
  </div>
</AppWrapper>

<style>
  .root__wrapper {
    width: 100%;
  }
</style>
