<template>
    <div class="pb" :class="{ 'pb--full-screen': fullScreen }">
        <div class="pb__hidden-copiers">
            <textarea ref="copyCodeHTML" class="outline-none" :id="referenceId"></textarea>
            <textarea ref="copySetup" class="outline-none" :id="copySetupId"></textarea>
        </div>
        <div class="pb__container">
            <div class="pb__header">
                <h3 class="pb__header__heading">{{ title }}</h3>
                <div class="pb__header__inner">
                    <div class="pb__header__tabs">
                        <button type="button" class="ml-2 inline-block rounded-lg font-medium leading-none py-2 px-3 focus:outline-none text-gray-600">
                            <svg :class="{'animate-spin' : loading}" class="w-5 h-5 -scale-x-1" @click.prevent="refresh" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
                            </svg>
                        </button>
                        <button :class="{ 'pb__header__tab--active': activeTabIndex === 0 }" @click="showTab(0)" type="button" class="pb__header__tab">Setup</button>
                        <button :class="{ 'pb__header__tab--active': activeTabIndex === 1 }" @click="showTab(1)" type="button" class="pb__header__tab">Preview</button>
                        <button :class="{ 'pb__header__tab--active': activeTabIndex === 2 }" @click="showTab(2)" type="button" class="pb__header__tab">Code</button>
                    </div>
                    <button v-if="activeTabIndex !== 0" aria-label="Copy HTML" data-balloon-pos="left" type="button" :data-clipboard-target="'#' + referenceId" class="copy pb__header__button">
                        <svg class="h-6 w-6 fill-current" viewBox="0 0 24 24">
                            <path fill-rule="evenodd" clip-rule="evenodd" d="M13 2a1 1 0 0 1 1 1 1 1 0 0 0 1 1 1 1 0 1 1 0 2h-5a1 1 0 0 1 0-2 1 1 0 0 0 1-1 1 1 0 0 1 1-1h1zM8 5H7a1 1 0 0 0-1 1v13a1 1 0 0 0 1 1h11a1 1 0 0 0 1-1V6a1 1 0 0 0-1-1h-1a2 2 0 0 1-2 2h-5a2 2 0 0 1-2-2zm1 5h7v1H9v-1zm7 2H9v1h7v-1zm-7 3v-1h4v1H9z"></path>
                        </svg>
                    </button>
                    <button v-if="activeTabIndex === 0" aria-label="Copy CDN" data-balloon-pos="left" type="button" :data-clipboard-target="'#' + copySetupId" class="copy pb__header__button">
                        <svg class="h-6 w-6 fill-current" viewBox="0 0 24 24">
                            <path fill-rule="evenodd" clip-rule="evenodd" d="M13 2a1 1 0 0 1 1 1 1 1 0 0 0 1 1 1 1 0 1 1 0 2h-5a1 1 0 0 1 0-2 1 1 0 0 0 1-1 1 1 0 0 1 1-1h1zM8 5H7a1 1 0 0 0-1 1v13a1 1 0 0 0 1 1h11a1 1 0 0 0 1-1V6a1 1 0 0 0-1-1h-1a2 2 0 0 1-2 2h-5a2 2 0 0 1-2-2zm1 5h7v1H9v-1zm7 2H9v1h7v-1zm-7 3v-1h4v1H9z"></path>
                        </svg>
                    </button>
                    <a aria-label="Open in new tab" data-balloon-pos="left" :href="iframeSrc" target="_blank" class="pb__header__button">
                        <svg class="pb__header__svg" fill="currentColor" viewBox="0 0 24 24"><path d="M19 6.41L8.7 16.71a1 1 0 1 1-1.4-1.42L17.58 5H14a1 1 0 0 1 0-2h6a1 1 0 0 1 1 1v6a1 1 0 0 1-2 0V6.41zM17 14a1 1 0 0 1 2 0v5a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V7c0-1.1.9-2 2-2h5a1 1 0 0 1 0 2H5v12h12v-5z"></path></svg>
                    </a>
                    <a @click="toggleFullscreen" aria-label="Toggle fullscreen" data-balloon-pos="left" href="#" class="pb__header__button">
                        <svg class="pb__header__svg" fill="currentColor" viewBox="0 0 24 24"><path d="M4.207 17.207L7 20H0v-7l2.793 2.793 4.328-4.329 1.415 1.415-4.329 4.328zM15.793 2.793L13 0h7v7l-2.793-2.793-4.328 4.329-1.415-1.415 4.329-4.328zm0 14.414L13 20h7v-7l-2.793 2.793-4.328-4.329-1.415 1.415 4.329 4.328zM4.207 2.793L7 0H0v7l2.793-2.793L7.12 8.536 8.536 7.12 4.207 2.793z"/></svg>
                    </a>
                </div>
            </div>
            <div ref="container" class="pb__container__tabs">
                <div class="pb__container__tab pb__container__tab--setup" v-show="activeTabIndex === 0">
                    <slot name="setup"></slot>
                </div>
                <div ref="container" v-show="activeTabIndex === 1" class="pb__container__tab pb__container__tab--iframe">
                    <iframe @load="loading = false" ref="iframe" height="600" :src="src" class="pb__container__iframe"></iframe>
                    <div v-if="moving" class="pb__container__iframe-shield"></div>
                    <div v-if="showGuides">
                        <div v-for="(guide,index) in snapGuides" :key="index" :style="{left: guide+'px'}" class="absolute top-0 bottom-0 border-r border-dashed border-gray-300"></div>
                    </div>
                    <button ref="handle" v-on:mousedown="onMouseDown" class="pb__container__iframe-handle grabbable">
                        <svg class="pb__container__handle-svg" fill="currentColor" viewBox="0 0 24 24"><path d="M8 5h2v14H8zM14 5h2v14h-2z"></path></svg>
                    </button>
                    <div class="absolute left-0 bottom-0 bg-gray-200 m-4 rounded py-2 leading-none px-3 text-sm">{{ screenSize + 'px' }}</div>
                </div>
                <div class="pb__container__tab pb__container__tab--code" v-show="activeTabIndex === 2">
                    <div class="relative">
                        <pre class="pb__container__syntax-highlighter-outer" :class="'language-' + language">
                            <code ref="codeSnippet" class="pb__container__syntax-highlighter-inner"><slot name="code-snippet"></slot></code>
                        </pre>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import ClipboardJS from 'clipboard';
    import Prism from 'prismjs';
    import 'balloon-css';
    const d = window.document;

    export default {
        name: 'preview-box',
        props: {
            language: {
                required: false,
                type: String,
                default: 'html'
            },
            iframeSrc: {
                required: true,
                type: String
            },
            minScreenSize: {
                required: true,
                type: Number
            },
            title: {
                required: false,
                type: String,
                default: ''
            },
            snapGuides: {
                required: false,
                type: Array,
                default: () => {
                    return [380, 640, 768, 1024, 1280]
                }
            },
            snapToWidth: {
                required: false,
                type: Boolean,
                default: false
            },
            showGuides: {
                required: false,
                type: Boolean,
                default: true
            }
        },
        data () {
            return {
                referenceId: null,
                copySetupId: null,
                moving: false,
                activeTabIndex: 1,
                fullScreen: false,
                screenSize: 0,
                lastClickIsHandle: false,
                loading: true,
                cb: '',
                boundingBox: {
                    x1: 0,
                    x2: 0,
                    min_width: 0,
                    max_width: 0,
                    tolerance: 0,
                    handle_width: 0,
                    guidePoints: []
                },
            }
        },
        mounted() {
            this.setBoundingBox();
            this.trackEvents();
            this.setGuidePoints();

            this.$refs.copyCodeHTML.innerHTML = this.$slots['copy-code-snippet'][0].children[0].text;
            this.$refs.copySetup.innerHTML = this.$slots['copy-setup'][0].text;
            this.referenceId = Math.random().toString(36).replace(/[^a-z]+/g, '').substr(2, 10);
            this.copySetupId = Math.random().toString(36).replace(/[^a-z]+/g, '').substr(2, 10);
            this.setScreenSize();
            Prism.highlightAll();

            new ClipboardJS('.copy');
            d.querySelectorAll('.copy').forEach(function(btn){
                btn.addEventListener('click', function(){
                    this.setAttribute('aria-label', 'COPIED!');
                    this.classList.add('bounceIn');
                    setTimeout(function(){
                        this.setAttribute('aria-label', 'Copy HTML');
                        this.classList.remove('bounceIn');
                    }.bind(this), 1000);
                });
            });
        },
        computed: {
            src() {
                return this.iframeSrc + `?cb=${this.cb}`;
            }
        },
        methods: {
            refresh() {
              this.loading = true;
                this.cb = Math.round(new Date().getTime() / 1000);
            },
            setGuidePoints() {
                this.boundingBox.guidePoints = [];
                this.snapGuides.forEach((guide,index) => {
                    if(guide < this.boundingBox.min_width || guide > this.boundingBox.max_width) {
                        return false;
                    }
                    this.boundingBox.guidePoints.push({
                        guide,
                        x1: index === 0 ? guide : (guide-(guide-this.snapGuides[index-1])/2),
                        x2: this.snapGuides[index+1] === undefined ? guide : (guide+(this.snapGuides[index+1]-guide)/2),
                    });
                });
            },
            trackEvents() {
                const _this = this;
                window.addEventListener('mousemove', (e) => this.onMouseMove(e));
                window.addEventListener('mouseup', (e) => this.onMouseUp(e));
                document.addEventListener('click', (e) => this.trackMouseClicks(e));
                window.addEventListener('keyup', (e) => this.onKeyUp(e));
                window.addEventListener('resize', function(e) {
                    _this.setBoundingBox(e);
                });
            },
            setBoundingBox() {
                let bcr = this.$refs.container.getBoundingClientRect();
                let border = parseInt(window.getComputedStyle(document.querySelector('.pb__container')).borderLeftWidth.replace('px',''));
                this.boundingBox.x1 = bcr.x + border;
                this.boundingBox.x2 = bcr.x + bcr.width + border;
                this.boundingBox.min_width = this.minScreenSize;
                this.boundingBox.handle_width = this.$refs.handle.getBoundingClientRect().width;
                this.boundingBox.max_width = bcr.width - this.boundingBox.handle_width;
            },
            onKeyUp(e) {
                if(!this.lastClickIsHandle) {
                    return false;
                }
                switch(parseInt(e.keyCode)) {
                    case 37: this.moveIframeLeft(); break;
                    case 39: this.moveIframeRight(); break;
                }
            },
            moveIframeLeft() {
                let width = this.$refs.iframe.getBoundingClientRect().width - 1;
                width = width < this.minScreenSize ? this.minScreenSize : (width > this.boundingBox.max_width ? this.boundingBox.max_width : width);

                this.setWidthPosition(width);
            },
            moveIframeRight() {
                let width = this.$refs.iframe.getBoundingClientRect().width + 1;
                width = width < this.minScreenSize ? this.minScreenSize : (width > this.boundingBox.max_width ? this.boundingBox.max_width : width);

                this.setWidthPosition(width);
            },
            setWidthPosition(width) {
                this.$refs.iframe.style.width = width + 'px';
                this.$refs.handle.style.left = width + 'px';
                this.$refs.handle.style.right = 0;
                this.setScreenSize();
            },
            trackMouseClicks(e) {
                this.lastClickIsHandle = e.target === this.$refs.handle;
            },
            onMouseDown(e) {
                e.preventDefault();
                window.focus();
                this.setBoundingBox();
                this.boundingBox.tolerance = e.clientX - this.$refs.handle.getBoundingClientRect().x;
                this.moving = true;
                this.lastClickIsHandle = true;
            },
            onMouseMove(e) {
                e.preventDefault();
                this.boundingBox.handle = e.clientX <= this.boundingBox.x1 ? this.boundingBox.x1 : (e.clientX >= this.boundingBox.x2 ? this.boundingBox.x2 : e.clientX);
                this.boundingBox._width = this.boundingBox.handle - this.boundingBox.x1 - this.boundingBox.tolerance;
                this.boundingBox.width = this.boundingBox._width <= this.boundingBox.min_width ? this.boundingBox.min_width : (this.boundingBox._width >= this.boundingBox.max_width ? this.boundingBox.max_width : this.boundingBox._width);

                if(this.moving) {
                    if(this.snapToWidth) {
                        this.boundingBox.guidePoints.forEach(point => {
                            if(this.boundingBox._width >= point.x1 && this.boundingBox._width < point.x2) {
                                this.boundingBox.width = point.guide;
                                return;
                            }
                        });
                    }
                    this.setWidthPosition(this.boundingBox.width);
                }
            },
            onMouseUp() {
                this.moving = false;
            },
            showTab(index) {
                this.activeTabIndex = parseInt(index);
                this.$emit('openedTab', this.activeTabIndex);
            },
            toggleFullscreen(e) {
                e.preventDefault();
                this.$refs.iframe.focus();
                this.fullScreen = ! this.fullScreen;
                this.$nextTick(() => {
                    this.setBoundingBox();
                    this.setGuidePoints();
                    if(this.boundingBox._width > this.boundingBox.max_width) {
                        this.setWidthPosition(this.boundingBox.max_width);
                    }
                });
            },
            setScreenSize() {
                this.screenSize = this.$refs.iframe.getBoundingClientRect().width;
            }
        }
    }
</script>

<style>
@import '~prismjs/themes/prism-tomorrow.css';

.grabbable {
    cursor: move;
    cursor: grab;
    cursor: -moz-grab;
    cursor: -webkit-grab;
}

.grabbable:active {
    cursor: grabbing;
    cursor: -moz-grabbing;
    cursor: -webkit-grabbing;
}

.pb--full-screen {
    position: fixed;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    padding: 0.5rem;
    background-color: #2d3748;
    z-index: 40;
}

.pb__hidden-copiers {
    width: 0.25rem;
    height: 0.25rem;
    overflow: hidden;
    position: absolute;
    left: 50%;
    top: 50%;
    z-index: -1;
}

.pb__container {
    border-bottom-width: 1px;
    border-color: #cbd5e0;
    border-top-width: 1px;
    margin-bottom: 2.5rem;
    overflow: hidden;
}


    .pb__container {
        border-width: 1px;
        border-radius: 0.5rem;
    }


.pb__container__tabs {
    position: relative;
    background-color: #a0aec0;
}

.pb__container__tab {
    background-color: #fff;
    display: block;
    position: relative;
    min-height: calc(100vh / 2);
}

.pb__container__tab--iframe {
    background-color: #a0aec0;
    padding-right: 1rem;
}

.pb__container__iframe {
    width: 100%;
    position: relative;
    overflow: auto;
    border-width: 0;
}

.pb__container__iframe-shield {
    position: absolute;
    opacity: 0;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
}


    .pb__container__iframe-handle {
        height: auto;
        padding: 0;
        margin: 0;
        overflow: visible;
        clip: auto;
        white-space: normal;
        border-left-width: 1px;
        background-color: #f7fafc;
        position: absolute;
        right: 0;
        top: 0;
        bottom: 0;
        display: -webkit-box;
        display: -ms-flexbox;
        display: flex;
        -webkit-box-align: center;
        -ms-flex-align: center;
        align-items: center;
        width: 1rem;
    }


.pb__container__handle-svg {
    height: 1rem;
    width: 1rem;
    color: #718096;
    pointer-events: none;
}

.pb__container__syntax-highlighter-outer {
    padding: 0;
    margin: 0 !important;
}

.pb__container__syntax-highlighter-inner {
    display: block;
    padding: 1rem;
    -webkit-overflow-scrolling: touch;
    white-space: pre-wrap;
    margin: 0;
    min-height: calc(100vh / 2);
}

.pb__header {
    padding: 1rem 1.5rem;
    border-bottom-width: 1px;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-pack: justify;
    -ms-flex-pack: justify;
    justify-content: space-between;
    -webkit-box-align: baseline;
    -ms-flex-align: baseline;
    align-items: baseline;
    background-color: #fff;
}

.pb__header__heading {
    font-size: 1rem;
    line-height: 1.375;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

@media (min-width: 768px) {
    .pb__header__heading {
        font-size: 1.125rem;
    }
}

.pb__header__inner {
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -ms-flex-negative: 0;
    flex-shrink: 0;
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
}

.pb__header__tabs {
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    border-right-width: 1px;
    padding-left: 1rem;
    padding-right: 1rem;
    font-size: 0.875rem;
}

@media (min-width: 768px) {
    .pb__header__tabs {
        font-size: 1rem;
    }
}

.pb__header__tab {
    margin-left: 0.5rem;
    display: inline-block;
    border-radius: 0.5rem;
    font-weight: 500;
    line-height: 1;
    padding: 0.5rem 0.75rem;
    color: #718096;
}

.pb__header__tab:focus {
    outline: 0;
}

.pb__header__tab--active {
    background-color: #edf2f7;
    color: #7f9cf5;
}

.pb__header__button {
    margin-left: 0.75rem;
    color: #a0aec0;
}

.pb__header__button:focus {
    outline: 0;
}

.pb__header__button:hover {
    color: #7f9cf5;
}

.pb__header__svg {
    height: 1.25rem;
    width: 1.25rem;
    color: #a0aec0;
}

.pb__header__svg:hover {
    color: #7f9cf5;
}

.pb--full-screen .pb__container__tabs {
    height: calc(100vh - 84px);
}

.pb--full-screen .pb__container__tab,
.pb--full-screen .pb__container__iframe,
.pb--full-screen .pb__container__syntax-highlighter-outer,
.pb--full-screen .pb__container__tab--code > div {
    height:100%;
}
</style>

