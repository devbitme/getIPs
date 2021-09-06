<script lang="ts">
  import AppWrapper from '../AppWrapper/AppWrapper.svelte';
  import axios from 'axios';

  const publicIp = require('public-ip');
  let ipv4 = '...';
  let ipv6 = '...';
  let textBtnV4 = 'copy';
  let textBtnV6 = 'copy';
  // let textBtnServer = 'copy';
  let classBtnV4 = 'btn-primary';
  let classBtnV6 = 'btn-primary';
  // let classBtnServer = 'btn-primary';

  (async () => {
    ipv4 = await publicIp.v4();
    ipv6 = await publicIp.v6();
  })();

  function copyIpOnClipboard(typeIP) {
    const type = typeIP.target.dataset.type;

    textBtnV4 = 'copy';
    textBtnV6 = 'copy';
    classBtnV4 = 'btn-primary';
    classBtnV6 = 'btn-primary';

    switch (type) {
      case 'IPv4':
        navigator.clipboard.writeText(ipv4).then(
          function () {
            textBtnV4 = 'copied';
            classBtnV4 = 'btn-success';
            setTimeout(function () {
              textBtnV4 = 'copy';
              classBtnV4 = 'btn-primary';
            }, 3000);
          },
          function () {
            console.log('failed');
          }
        );
        break;
      case 'IPv6':
        navigator.clipboard.writeText(ipv6).then(
          function () {
            textBtnV6 = 'copied';
            classBtnV6 = 'btn-success';
            setTimeout(function () {
              textBtnV6 = 'copy';
              classBtnV6 = 'btn-primary';
            }, 3000);
          },
          function () {
            console.log('failed');
          }
        );
        break;
      default:
        console.log(`Sorry, we are out of ${type}.`);
    }
  }
</script>

<AppWrapper>
  <div class="root__wrapper">
    <h1 class="d-none">Get IPs</h1>
    <div class="kaomoji">°˖✧◝(⁰▿⁰)◜✧˖°</div>
    <h2>Your public IPs :</h2>
    <strong>IPv4 :</strong>
    {ipv4}<button class={classBtnV4} on:click={copyIpOnClipboard} data-type="IPv4">{textBtnV4}</button>
    <br />
    <strong>IPv6 :</strong>
    {ipv6}<button class={classBtnV6} on:click={copyIpOnClipboard} data-type="IPv6">{textBtnV6}</button>
    <h2>The current URL IP :</h2>
    Soon...
    <div class="version">getIPs <strong>1.0.0</strong></div>
  </div>
</AppWrapper>

<style>
  .root__wrapper {
    width: 100%;
  }
</style>
