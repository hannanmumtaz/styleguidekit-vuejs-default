<template>
  <div>
    <!--Style Guide Header-->
    <header class="sg-header" role="banner">
      <a href="#sg-nav-container" class="sg-nav-toggle" v-on:click.prevent="toggleMenu">Menu</a>
      <div class="sg-nav-container" id="sg-nav-container">
        <!-- pattern lab nav will be inserted here -->
        <pattern-nav></pattern-nav>
        <div class="sg-controls" id="sg-controls">
          <!-- ish Controls will be inserted here -->
          <ish-control></ish-control>
        </div>
      </div>
    </header>
    <!--End Style Guide Header-->

    <!-- iFrame -->
    <div id="sg-vp-wrap">
      <div id="sg-cover"></div>
      <div id="sg-gen-container">
        <iframe id="sg-viewport" name="sg-viewport" sandbox="allow-same-origin allow-scripts allow-popups allow-forms"></iframe>
        <div id="sg-rightpull-container">
          <div id="sg-rightpull"></div>
        </div>
      </div>
    </div>
    <!--end iFrame-->

    <!-- modal window -->
    <div id="sg-modal-container" class="sg-modal anim-ready" style="bottom: -2000px">

      <!-- modal close btn -->
      <button id="sg-modal-close-btn" class="sg-modal-close-btn" title="Hide pattern info">
        <svg version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="12" height="12" viewBox="0 0 12 12">
          <title>Close</title>
          <path fill="#808080" d="M11.8905,9.6405 L11.8905,9.6405 L8.25,6 L11.8905,2.3595 L11.8905,2.3595 C11.9295,2.3205 11.958,2.27475 11.976,2.226 C12.0255,2.0925 11.997,1.9365 11.8905,1.82925 L10.17075,0.1095 C10.0635,0.00225 9.9075,-0.02625 9.774,0.024 C9.72525,0.042 9.6795,0.0705 9.6405,0.1095 L9.6405,0.1095 L6,3.75 L2.3595,0.1095 L2.3595,0.1095 C2.3205,0.0705 2.27475,0.042 2.226,0.024 C2.0925,-0.0255 1.9365,0.00225 1.82925,0.1095 L0.1095,1.82925 C0.00225,1.9365 -0.02625,2.0925 0.024,2.226 C0.042,2.27475 0.0705,2.3205 0.1095,2.3595 L0.1095,2.3595 L3.75,6 L0.1095,9.6405 L0.1095,9.6405 C0.0705,9.6795 0.042,9.72525 0.024,9.774 C-0.0255,9.9075 0.00225,10.0635 0.1095,10.17075 L1.82925,11.8905 C1.9365,11.99775 2.0925,12.02625 2.226,11.976 C2.27475,11.958 2.3205,11.9295 2.3595,11.8905 L2.3595,11.8905 L6,8.25 L9.6405,11.8905 L9.6405,11.8905 C9.6795,11.9295 9.72525,11.958 9.774,11.976 C9.9075,12.0255 10.0635,11.99775 10.17075,11.8905 L11.8905,10.17075 C11.99775,10.0635 12.02625,9.9075 11.976,9.774 C11.958,9.72525 11.9295,9.6795 11.8905,9.6405 L11.8905,9.6405 Z"></path>
        </svg>
      </button><!--end .sg-modal-close-btn-->

      <div id="sg-modal-content" class="sg-modal-content"></div><!--end .sg-modal-content-->

    </div><!--end .sg-modal-->
    <!-- end modal window -->

  </div>
</template>

<script>
  import { mapState } from 'vuex';
  import $ from 'jquery';
  import PatternNav from './PatternNav';
  import IshControl from './IshControl';
  import urlHandler from '../api/url-handler';

  export default {
    name: 'Home',

    components: {
      IshControl,
      PatternNav
    },

    // eslint-disable-next-line object-shorthand,func-names
    data: function () {
      return {
        sw: document.body.clientWidth, // Viewport Width
        sh: $(document).height(), // Viewport Height

        minViewportWidth: 240,
        maxViewportWidth: 2600,

        viewportResizeHandleWidth: 14, // Width of the viewport drag-to-resize handle
        $sgViewport: $('#sg-viewport'), // Viewport element
        $sizePx: $('.sg-size-px'), // Px size input element in toolbar
        $sizeEms: $('.sg-size-em'), // Em size input element in toolbar
        $bodySize: parseInt($('body').css('font-size'), 10), // Body size of the document,
        $headerHeight: $('.sg-header').height(),
        discoID: false,
        discoMode: false,
        fullMode: true,
        hayMode: false
      };
    },

    computed: {
      ...mapState(
        [
          'config'
        ]
      )
    },

    methods: {
      toggleMenu() {
        $('.sg-nav-container').toggleClass('active');
      }
    },

    mounted() {
      // get the request vars
      const oGetVars = urlHandler.getRequestVars();

      const baseIframePath = `${window.location.protocol}//${window.location.host}${window.location.pathname.replace('index.html', '')}`;
      let iFramePath = `${baseIframePath}styleguide/html/styleguide.html?${Date.now()}`;
      let patternName = ((this.$store.state.defaultPattern !== undefined) && (typeof this.$store.state.defaultPattern === 'string') && (this.$store.state.defaultPattern.trim().length > 0)) ? this.$store.state.defaultPattern : 'all';
      if ((oGetVars.p !== undefined) || (oGetVars.pattern !== undefined)) {
        patternName = (oGetVars.p !== undefined) ? oGetVars.p : oGetVars.pattern;
      }

      let patternPath;
      if (patternName !== 'all') {
        patternPath = urlHandler.getFileName(patternName);
        iFramePath = (patternPath !== '') ? `${baseIframePath}${patternPath}?${Date.now()}` : iFramePath;
        document.getElementById('title').innerHTML = `Pattern Lab - ${patternName}`;
        history.replaceState({ pattern: patternName }, null, null);
      }

      document.getElementById('sg-viewport').contentWindow.location.replace(iFramePath);


      // Viewport Height

      // set minimum and maximum viewport based on confg
      if (this.config.ishMinimum !== undefined) {
        this.minViewportWidth = parseInt(this.config.ishMinimum, 10); // Minimum Size for Viewport
      }
      if (this.config.ishMaximum !== undefined) {
        this.maxViewportWidth = parseInt(this.config.ishMaximum, 10); // Maxiumum Size for Viewport
      }

      // alternatively, use the ishViewportRange object
      if (this.config.ishViewportRange !== undefined) {
        this.minViewportWidth = this.config.ishViewportRange.s[0];
        this.maxViewportWidth = this.config.ishViewportRange.l[1];
      }

      // Body size of the document,
      this.$bodySize = (this.config.ishFontSize !== undefined) ? parseInt(this.config.ishFontSize, 10) : parseInt($('body').css('font-size'), 10);
    }
  };
</script>
