<script>
  import { codeErrorStore } from '../code-error-store.js';
  import { link } from 'svelte-spa-router';
  import { Base64 } from 'js-base64';
  import { onMount } from 'svelte';
  import moment from 'moment';
  import { codeStore } from '../code-store.js';

  onMount(async () => {});

  const fixBrTag = (svg) => svg.replaceAll('<br>', '<br/>');

  export const onDownloadPNG = (event) => {
    const canvas = document.createElement('canvas');
    const container = document.getElementById('container');
    const svg = document.querySelector('#container svg');
    const box = svg.getBoundingClientRect();
    canvas.width = box.width;
    canvas.height = box.height;
    if (imagemodeselected === 'width') {
      const ratio = box.height / box.width;
      canvas.width = userimagewidth;
      canvas.height = userimagewidth * ratio;
    } else if (imagemodeselected === 'height') {
      const ratio = box.width / box.height;
      canvas.width = userimageheight * ratio;
      canvas.height = userimageheight;
    }
    const context = canvas.getContext('2d');
    context.fillStyle = 'white';
    context.fillRect(0, 0, canvas.width, canvas.height);

    const image = new Image();
    image.onload = function () {
      context.drawImage(image, 0, 0, canvas.width, canvas.height);

      var a = document.createElement('a');
      a.download = `mermaid-diagram-${moment().format('YYYYMMDDHHmmss')}.png`;
      a.href = canvas
        .toDataURL('image/png')
        .replace('image/png', 'image/octet-stream');
      a.click();
    };

    console.warn('SVG', fixBrTag(container.innerHTML));
    image.src = `data:image/svg+xml;base64,${Base64.encode(
      fixBrTag(container.innerHTML)
    )}`;
    event.stopPropagation();
    event.preventDefault();
    event.target.download = `mermaid-diagram-${moment().format(
      'YYYYMMDDHHmmss'
    )}.png`;
  };
  export const onDownloadSVG = (event) => {
    console.log('event', event.target);
    const container = document.getElementById('container');
    event.target.href = `data:image/svg+xml;base64,${Base64.encode(
      fixBrTag(container.innerHTML)
    )}`;
    event.target.download = `mermaid-diagram-${moment().format(
      'YYYYMMDDHHmmss'
    )}.svg`;
    console.log('event', event);
  };
  export const onCopyMarkdown = (event) => {
    event.target.select();
    document.execCommand('Copy');
  };

  let url = '/mermaid-live-editor/#/view';
  let b64Code;
  let iUrl;
  let svgUrl;
  let mdCode;
  let imagemodeselected = 'auto';
  let userimagewidth = 1920;
  let userimageheight = 1080;

  const unsubscribe = codeStore.subscribe((state) => {
    b64Code = Base64.encodeURI(JSON.stringify(state));
    url = `/mermaid-live-editor/#/view/${b64Code}`;
    iUrl = `https://mermaid.ink/img/${b64Code}`;
    svgUrl = `https://mermaid.ink/svg/${b64Code}`;
    mdCode = `[![](${iUrl})](${window.location.protocol}//${window.location.host}${window.location.pathname}#/edit/${b64Code})`;
  });
</script>

<style>
  #links {
    margin-bottom: 1rem;
    padding-bottom: 0.5rem;
    border-bottom: 1px solid lightgray;
  }
  #markdown {
    padding: 7px;
    font-family: monospace;
    font-size: 14px;
    width: 95%;
    margin: 1rem 0;
    border: 1px solid lightgray;
  }
  label[for='markdown'] {
    cursor: pointer;
    margin: 0 auto;
  }
  /*
    @media (prefers-color-scheme: dark) {
        label[for="markdown"] {
            color: #d8d8d8;
        }
    } */
  .button-style {
    background-color: #a2d9e2;
    color: #33a2c4;
    border-radius: 0.25rem;
    padding: 0.5rem;
    border: 1px solid #a2d9e2;
    margin: 0.25rem;
  }
  .button-style:hover {
    background-color: #fff;
    color: #33a2c4;
    border: 1px solid #33a2c4;
  }
  .button-style:focus {
    outline: none;
  }
  .link-style {
    text-decoration: none;
    color: #33a2c4;
  }
  #copy-section {
    padding-top: 1rem;
    text-align: center;
  }
</style>

<div id="links">
  <button class="button-style">
    <a class="link-style" href={url} use:link>Link to view</a>
  </button>
  <button class="button-style">
    <a class="link-style" href={url} download="" on:click={onDownloadSVG}>
      Download SVG
    </a>
  </button>
  <button class="button-style">
    <a class="link-style" href={iUrl}>Link to Image</a>
  </button>
  <button class="button-style">
    <a class="link-style" href={svgUrl}>Link to SVG</a>
  </button>
  <button class="button-style">
    <a class="link-style" href={url} download="" on:click={onDownloadPNG}>
      Download PNG
    </a>
  </button>
  (markdown is base64 encoded for these urls)
</div>
<div id="copy-section">
  <label for="markdown" class="button-style">Copy Markdown</label>
  <br />
  <input id="markdown" type="text" value={mdCode} on:click={onCopyMarkdown} />
</div>
<p>
  <label>PNG size:</label>
  <input
    type="radio"
    value="auto"
    id="autosize"
    bind:group={imagemodeselected} />
  <label for="autosize">auto</label>
  <input
    type="radio"
    value="width"
    id="width-active"
    bind:group={imagemodeselected} />
  <label for="width">width</label>
  <input
    id="width"
    type="number"
    min="3"
    max="10000"
    bind:value={userimagewidth}
    disabled={imagemodeselected !== 'width'} />
  <input
    type="radio"
    value="height"
    id="height-active"
    bind:group={imagemodeselected} />
  <label for="height">height</label>
  <input
    id="height"
    type="number"
    min="3"
    max="10000"
    bind:value={userimageheight}
    disabled={imagemodeselected !== 'height'} /><br />
</p>
