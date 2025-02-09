<template>
    <div class="flex flex-col gap-5 mx-auto max-w-2xl w-full p-2">
        <nuxt-link
            v-for="deck of decks"
            :key="deck.slug"
            :to="deck.slug"
            class="rounded-xl bg-[#2D282F] border-2 border-[#584F65] h-52 flex flex-row"
        >
            <div class="w-1/2 flex flex-col p-3 sm:p-6 gap-2 justify-center">
                <p class="w-full font-genshin text-xl sm:text-3xl">
                    {{ deck.name }}
                </p>
                <div class="w-full flex flex-row flex-wrap justify-start items-center gap-2">
                    <tag-bar
                        v-for="tag of deck.tags"
                        :key="tag"
                        :tag="tag"
                        class="rounded-md text-center font-bold px-2"
                    />
                </div>
            </div>
            <div class="w-1/2 flex flex-row mr-3">
                <card-component
                    v-for="(character, i) in deck.characters"
                    :key="i"
                    :name="character"
                    :modal="false"
                    class="w-full bg-[length:150%]"
                    style="clip-path: polygon(25% 0%, 100% 0%, 75% 100%, 0% 100%);"
                />
            </div>
        </nuxt-link>
    </div>
</template>

<script lang="ts">
import Vue from 'vue'
import CardComponent from '~/components/CardComponent.vue'

export default Vue.extend({
    name: 'IndexPage',
    components: { CardComponent },
    async asyncData ({ $content }) {
        const codeMapping: { [key: string]: string} = {}
        const cardcodes = await $content('cardcodes').fetch() as any
        for (const entry of cardcodes.body) {
            codeMapping[entry.Code] = entry.Card
        }

        const decks = (await $content('decks').fetch() as any[]).map((deck) => {
            const cards = deck.deck_code.replace(/[!=?]/g, '').match(/\.?.{2}/g)
            deck.characters = cards.slice(0, 3).map((code: string) => codeMapping[code])
            deck.deck_list = {}
            for (const card of cards.slice(3)) {
                deck.deck_list[codeMapping[card.replace('.', '')]] = card.includes('.') ? 2 : 1
            }

            return deck
        })

        return { decks }
    }
})
</script>
