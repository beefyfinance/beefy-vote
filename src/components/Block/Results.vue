<template>
  <Block :title="ts >= payload.end ? 'Results' : 'Current results'">
    <div v-for="(choice, i) in payload.choices" :key="i">
      <div class="text-white mb-1">
        <span v-text="_shorten(choice, 'choice')" class="mr-1" />
        <span v-if="results.totalScores[i]" class="mr-1">
          {{ _numeral(results.totalScores[i]) }}
          {{ _shorten(space.symbol, 'symbol') }}
        </span>

        <span
          class="float-right"
          v-text="
            $n(
              results.totalVoteScores
                ? results.totalScores[i] / results.totalVoteScores
                : 0,
              'percent'
            )
          "
        />
      </div>
      <UiProgress
        :value="results.totalScores[i]"
        :max="results.totalVoteScores"
        :titles="titles"
        class="mb-3"
      />
    </div>
    <UiButton
      @click="downloadReport"
      v-if="ts >= payload.end"
      class="width-full mt-2"
    >
      Download report
    </UiButton>
  </Block>
</template>

<script>
import * as jsonexport from 'jsonexport/dist';
import pkg from '@/../package.json';

export default {
  props: ['space', 'payload', 'results', 'votes'],
  computed: {
    ts() {
      return (Date.now() / 1e3).toFixed();
    },
    titles() {
      return [this.space.symbol];
    },
  },
  methods: {
    async downloadReport() {
      const obj = Object.entries(this.votes)
        .map(vote => {
          return {
            address: vote[0],
            choice: vote[1].msg.payload.choice,
            balance: vote[1].score,
            timestamp: vote[1].msg.timestamp,
            dateUtc: new Date(
              parseInt(vote[1].msg.timestamp) * 1e3
            ).toUTCString(),
            authorIpfsHash: vote[1].authorIpfsHash,
            relayerIpfsHash: vote[1].relayerIpfsHash,
          };
        })
        .sort((a, b) => a.timestamp - b.timestamp, 0);
      try {
        const csv = await jsonexport(obj);
        const link = document.createElement('a');
        link.setAttribute('href', `data:text/csv;charset=utf-8,${csv}`);
        link.setAttribute('download', `${pkg.name}-report-${this.id}.csv`);
        document.body.appendChild(link);
        link.click();
      } catch (e) {
        console.error(e);
      }
    },
  },
};
</script>
