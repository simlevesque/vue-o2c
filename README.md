# vue-o2c - Vue Options API to Composition API

**[Demo / Online Playground](https://tjk.github.io/vue-o2c/)**

**WORK IN PROGRESS** -- the following is not done:

- bunch of stuff still not implemented (working through case by case)
- publish package correctly (pretty important)
- data() preamble -- if there is premable maybe just create refs then use the function to set them
- handle setup() in options api
- allow options to configure (eg. no typescript)
- $el needs to try to rewrite part of template
- would like to maintain indentation

## Usage

### via CLI

*This is not working due to a publishing issue I need to fix...*

```bash
$ npx vue-o2c /path/to/sfc.vue
...
<transformed sfc code>
...
```

### Programmatically

```bash
$ pnpm add -D vue-o2c
```

*Please keep in mind, API is very experimental and likely will change!*

```typescript
import { transformPath, transform, type State } from "vue-o2c"

const s1: State = transformPath("/path/to/sfc.vue")
const s2: State = transform("<script>\nexport default {\n  props: {\n    a: String,\n  }\n}\n</script>")
// s1.transformed and s2.transformed will both contained transformed code
```

## Example

Given the following file:

```vue cat example.vue
<template lang="pug">
div
  p Wonderful
</template>

<script>
export default {
  props: {
    greeting: {
      type: String,
      default: "Hello",
    },
  },
  data() {
    // this.initializing = true -- uncommenting this breaks because data() becomes complex (need to improve)
    return {
      name: this.$route.query.name || 'John',
    };
  },
  methods: {
    doIt() {
      console.log(`${this["greeting"]} ${this.name} ${this.$el.clientHeight}`);
    },
  },
  mounted() {
    this.doIt();
    delete this.initializing // should not become `delete initializing` (so use $this)
  },
  watch: {
    watchMethod(v) {
      console.log("watchMethod", v)
    },
    watchObject: {
      deep: true,
      immediate: true,
      async handler(v, ov) {
        console.log("watchObject", v, ov)
      },
    },
  },
};
</script>

<style scoped>
:root {
  background: red;
}
</style>
```

```bash
$ git clone git@github.com:tjk/vue-o2c.git
$ cd vue-o2c
$ pnpm i
$ pnpm exec tsx index.ts ./example.vue
```

Will output the following:

```vue pnpm exec tsx src/cli.ts ./example.vue
<template lang="pug">
div(ref="$el")
  p Wonderful
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue"
import { useRoute } from "vue-router"

const props = withDefaults(defineProps<{
  greeting?: string
}>(), {
  greeting: "Hello",
})

const $route = useRoute()
const $this = {}

const $el = ref<HTMLElement | undefined>()
const name = ref($route.query.name || 'John')

onMounted(() => {
  doIt();
  delete this.initializing // should not become `delete initializing` (so use $this)
})

const watchMethod = watch((v) => {
  console.log("watchMethod", v)
})
const watchObject = watch(async (v, ov) => {
  console.log("watchObject", v, ov)
}, {
  deep: true,
  immediate: true,
})

function doIt() {
  console.log(`${props.greeting} ${name.value} ${$el.value.clientHeight}`);
}

</script>

<style scoped>
:root {
  background: red;
}
</style>

```