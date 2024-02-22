---
title: 'About me'
---

I'm a **software engineer** with 12 years of experience developing apps and user interfaces for the web. I work across the full stack, and have always leaned toward product companies that build beautiful and stable software.

I have a strong preference for statically typed languages like **TypeScript** and **ReScript**. My library of choice for building UIs is **React**, and I've built backends and **GraphQL** servers with **node.js**. I've set up **CI/CD** pipelines and managed deployments to **GCP** (on Kubernetes), **AWS**, **Vercel**, and others.

I've worked at:

<ul>
  <li><a href="https://linear.app">Linear</a></li>
  <li><a href="https://amie.so">Amie</a></li>
  <li><a href="https://www.eyeem.com">EyeEm</a></li>
  <li><a href="https://www.native-instruments.com">Native Instruments</a></li>
  <li><a href="https://www.edenspiekermann.com/">Edenspiekermann</a></li>
  <li><a href="http://sunst-studio.com">Sunst Studio</a></li>
</ul>

<h2>Open source</h2>
<ul>
 <li>
    <a href="https://github.com/eschaefer/mubi-watch-party">
      Mubi Watch Party
    </a>
    - A web extension (Chrome + Firefox) to sync Mubi movies with friends.
 </li>
 <li>
    <a href="https://github.com/eschaefer/rescript-peerjs">
      rescript-peerjs
    </a>
    - ReScript bindings for PeerJS.
  </li>
  <li>
    <a href="https://github.com/eschaefer/bs-react-grid-dnd">
      bs-react-grid-dnd
    </a>
    - ReasonML binding for react-grid-dnd.
  </li>
  <li>
    <a href="https://github.com/eschaefer/babel-plugin-framer-x">
      babel-plugin-framer-x
    </a>
    - A Babel plugin to remove Framer X code from your React components.
  </li>
  <li>
    <a href="https://github.com/eschaefer/react-apollo-ssr-boilerplate">
      React + Apollo SSR Boilerplate
    </a>
    - Made for my talk at BerlinJS (unmaintained)
  </li>
  <li>
    <a href="https://github.com/eschaefer/dash-nightwatch-generator">
      Dash Nightwatch Generator</a
    >
    - Generates a Nightwatch.js docset for Dash
  </li>
  <li>
    <a href="https://github.com/eschaefer/tor-proxy-toggle">
      Tor Proxy Toggle
    </a>
    - Sets up an OS X command-line alias to toggle a system SOCKS proxy
    through Tor
  </li>
  <li>
    <a href="https://github.com/edenspiekermann/faster-react-tabs">
      faster-react-tabs</a
    >
    - A flexible and context-agnostic React component used to render
    accessible and simple tabs (unmaintained)
  </li>
</ul>

<h2>Talks</h2>
<ul>
  <li>
    <a href="https://www.youtube.com/watch?v=cmOSN_mZbEo">
      ReasonML for Skeptics</a
    >
    - React Day Berlin: 06.12.2019
  </li>
  <li>
    <a href="/blog/2017/10/22/apollo-graphql-a-step-toward-more-declarative-uis/">
      Apollo + GraphQL: A step toward more declarative UIs</a
    >
    - Berlin.JS: 19.10.2017
  </li>
</ul>
<!-- prettier-ignore-end -->

<script type="module">
  async function main() {
    async function getTracks() {
      const url =
        'https://ws.audioscrobbler.com/2.0/?method=user.getrecenttracks&user=twegen&api_key=c64ac6cffa22a119f22f856dcc646157&format=json';

      const response = await fetch(url).then((resp) => resp.json());
      return response.recenttracks.track;
    }

    const tracks = await getTracks();

    if (tracks && tracks.length) {
      document.querySelector(
        '.listening'
      ).innerHTML = `<h2>🎵 &nbsp;&nbsp;Lately I am listening to...</h2>
      <ul class="list lh-copy tracks"></ul>
    `;

      let el = document.querySelector('.tracks');

      tracks.slice(0, 25).forEach((track) => {
        let parent = document.createElement('li');
        parent.innerHTML = `<strong>${track.artist['#text']}</strong> - <span>${track.name}</span>`;
        parent.className = 'track';
        el.appendChild(parent);
      });
    }
  }

  main();
</script>

<div class="listening"></div>
