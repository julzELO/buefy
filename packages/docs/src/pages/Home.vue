<template>
    <section class="home">
        <TheNavbar light />
        <div class="hero is-fullheight is-primary">
            <div class="hero-body is-block">
                <div class="container has-text-centered">
                    <div class="logo-rounded home-hero">
                        <img src="../assets/buefy.png" alt="Buefy" />
                    </div>
                    <h2 class="subtitle home-hero">
                        <strong>Lightweight</strong> UI components for
                        <strong
                            ><a href="https://vuejs.org/" target="_blank"
                                >Vue 3</a
                            ></strong
                        >
                        based on
                        <strong
                            ><a href="http://bulma.io/" target="_blank"
                                >Bulma</a
                            ></strong
                        >
                    </h2>
                    <div class="home-hero">
                        <pre
                            class="npm"
                        ><code><span class="is-unselectable">$ </span>npm install buefy</code></pre>
                    </div>


                    <div class="buttons">
                        <router-link
                            class="button is-medium home-hero"
                            to="/documentation/start"
                        >
                            Get started
                        </router-link>
                        <a
                            class="button is-medium is-primary home-hero"
                            href="https://github.com/buefy/buefy/releases"
                            target="_blank"
                        >
                            Changelog
                        </a>
                    </div>

                    <div class="home-hero mb-6">
                        <p>
                            The Vue 2 version of Buefy is deprecated but will remain available as Version "0.x".
                        </p>
                    </div>



                    <div class="github-button home-hero">
                        <iframe
                            src="https://ghbtns.com/github-btn.html?user=buefy&repo=buefy&type=star&count=true&size=large"
                            frameborder="0"
                            scrolling="0"
                            width="160px"
                            height="30px"
                        />

                        <iframe
                            src="https://ghbtns.com/github-btn.html?user=buefy&repo=buefy&type=fork&count=false&size=large"
                            frameborder="0"
                            scrolling="0"
                            width="80px"
                            height="30px"
                        />
                    </div>

                    <small class="home-hero">v{{ version }}</small>
                </div>
            </div>
        </div>

        <section class="hero">
            <div class="hero-body">
                <div class="container">
                    <div class="columns">
                        <div class="column has-text-centered features">
                            <b-icon icon="github" size="is-large" />
                            <p class="title is-4"><strong>Free</strong></p>
                            <p class="subtitle">
                                Open source on
                                <strong
                                    ><a
                                        href="https://github.com/buefy/buefy"
                                        target="_blank"
                                        >GitHub</a
                                    ></strong
                                >
                            </p>
                        </div>
                        <div class="column has-text-centered features">
                            <b-icon icon="cellphone-link" size="is-large" />
                            <p class="title is-4">
                                <strong>Responsive</strong>
                            </p>
                            <p class="subtitle">
                                <b>Every</b> component is responsive
                            </p>
                        </div>
                        <div class="column has-text-centered features">
                            <b-icon icon="alert-decagram" size="is-large" />
                            <p class="title is-4"><strong>Modern</strong></p>
                            <p class="subtitle">
                                Built with
                                <strong
                                    ><a
                                        href="https://vuejs.org/"
                                        target="_blank"
                                        >Vue.js</a
                                    ></strong
                                >
                                and
                                <strong
                                    ><a href="http://bulma.io/" target="_blank"
                                        >Bulma</a
                                    ></strong
                                >
                            </p>
                        </div>
                        <div class="column has-text-centered features">
                            <b-icon
                                pack="far"
                                icon="paper-plane"
                                size="is-large"
                            />
                            <p class="title is-4">
                                <strong>Lightweight</strong>
                            </p>
                            <p class="subtitle">No other internal dependency</p>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <section class="hero is-light">
            <div class="hero-body">
                <div class="container" v-if="sponsors && sponsors.length">
                    <p class="content is-size-5 has-text-left">Sponsors</p>
                    <div class="columns">
                        <div
                            v-for="item in sponsors"
                            :key="item.title"
                            class="column has-text-centered is-one-quarter"
                        >
                            <a
                                :href="item.url"
                                target="_blank"
                                rel="noopener noreferrer"
                            >
                                <img
                                    class="image-has-shadow"
                                    :src="getSponsorImg(item.img)"
                                    :alt="item.title"
                                />
                            </a>

                            <p>
                                <a
                                    class="has-text-weight-semibold"
                                    :href="item.url"
                                    target="_blank"
                                    rel="noopener noreferrer"
                                >
                                    {{ item.title }}
                                </a>
                                <br />
                            </p>
                        </div>
                    </div>
                </div>
                <div class="container expo">
                    <p class="content is-size-5 has-text-centered">
                        Websites and apps
                        <a @click="tweet"
                            ><strong class="has-text-link"
                                >#MadeWithBuefy</strong
                            ></a
                        >
                        from the
                        <router-link to="/expo"
                            ><strong class="has-text-link"
                                >Expo page</strong
                            ></router-link
                        >
                    </p>
                    <div class="columns">
                        <div
                            v-for="item in expo"
                            :key="item.title"
                            class="column has-text-centered is-one-quarter"
                        >
                            <a
                                :href="item.url"
                                target="_blank"
                                rel="noopener noreferrer"
                            >
                                <img
                                    class="image-has-shadow"
                                    :src="getExpoImg(item.img)"
                                    :alt="item.title"
                                />
                            </a>

                            <p>
                                <a
                                    class="has-text-weight-semibold"
                                    :href="item.url"
                                    target="_blank"
                                    rel="noopener noreferrer"
                                >
                                    {{ item.title }}
                                </a>
                                <br />
                            </p>
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <TheFooter />
    </section>
</template>

<script lang="ts">
import { defineComponent } from "vue";

import { BIcon } from "buefy";

import ScrollReveal from "scrollreveal";
import TheNavbar from "../components/TheNavbar.vue";
import TheFooter from "../components/TheFooter.vue";
import Package from "../../../../package.json";
import expoData from "@/data/expo";
import sponsorsData from "@/data/sponsors";

export default defineComponent({
    name: "App",
    components: {
        BIcon,
        TheNavbar,
        TheFooter,
    },
    data() {
        return {
            version: Package.version,
            expo: expoData.filter((item) => {
                return item.featured;
            }),
            sponsors: sponsorsData,
        };
    },
    methods: {
        tweet() {
            const width = 575;
            const height = 400;
            const left = (window.screen.width - width) / 2;
            const top = (window.screen.height - height) / 2;
            const url = `https://twitter.com/share?url=${encodeURIComponent(
                document.location.protocol + "//" + document.location.host
            )}&text=My website made with Buefy!&hashtags=madewithbuefy`;
            const opts = `status=1,width=${width},height=${height},top=${top},left=${left}`;

            window.open(url, "", opts);
        },
        getExpoImg(img: string) {
            // require won't work on Vite
            // https://vitejs.dev/guide/assets.html#new-url-url-import-meta-url
            // see also https://stackoverflow.com/a/71514878
            return new URL(`/src/assets/expo/${img}`, import.meta.url).href;
        },
        getSponsorImg(img: string) {
            // require won't work on Vite
            // https://vitejs.dev/guide/assets.html#new-url-url-import-meta-url
            // see also https://stackoverflow.com/a/71514878
            return new URL(`/src/assets/sponsors/${img}`, import.meta.url).href;
        },
    },
    mounted() {
        const sr = ScrollReveal({
            duration: 400,
            delay: 200,
        });
        sr.reveal(".home-hero", { delay: 0, origin: "top" }, 200);
        sr.reveal(".features", 200);
    },
});
</script>
