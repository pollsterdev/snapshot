<template>
  <Block>
    <div class="d-flex flex-items-center mb-1">
      <UiAvatar
        class="mr-2 mb-2"
        :imgsrc="getLogoUrl(network.key)"
        :seed="network.key"
        :size="28"
      />
      <h3 v-text="network.name" />
      <div v-text="network.key" class="ml-1 text-gray" />
    </div>
    <div class="text-gray">
      {{ $tc('inSpaces', [_n(network.spaces.length)]) }}
    </div>
  </Block>
</template>

<script>
export default {
  props: ['network'],
  methods: {
    getLogoUrl(key) {
      return `https://raw.githubusercontent.com/pollsterdev/snapshot.js/master/src/networks/${key}.png`;
    }
  }
};
</script>
