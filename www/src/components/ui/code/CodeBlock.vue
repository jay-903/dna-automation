<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue';
import { codeToHtml } from 'shiki';
import { format } from 'prettier/standalone';
import prettierPluginBabel from 'prettier/plugins/babel';
import prettierPluginEstree from 'prettier/plugins/estree';
import { Skeleton } from '@/components/ui/skeleton';

const props = withDefaults(
  defineProps<{
    code: string;
    lang?: string;
    theme?: string;
    format?: boolean;
  }>(),
  {
    lang: 'javascript',
    theme: 'github-dark',
    format: true
  }
);

const highlightedCode = ref('');
const isLoading = ref(true);

const formatCode = async (code: string): Promise<string> => {
  if (!props.format) return code;

  try {
    return await format(code, {
      parser: 'babel',
      plugins: [prettierPluginBabel, prettierPluginEstree],
      semi: true,
      singleQuote: true,
      tabWidth: 2,
      printWidth: 100,
      trailingComma: 'none',
      arrowParens: 'avoid'
    });
  } catch (error) {
    console.warn('Failed to format code with Prettier, using original:', error);
    return code;
  }
};

const highlight = async () => {
  try {
    isLoading.value = true;
    const formattedCode = await formatCode(props.code);

    highlightedCode.value = await codeToHtml(formattedCode, {
      lang: props.lang,
      theme: props.theme
    });
  } catch (error) {
    console.error('Failed to highlight code:', error);
    highlightedCode.value = `<pre><code>${props.code}</code></pre>`;
  } finally {
    isLoading.value = false;
  }
};

onMounted(() => {
  highlight();
});

watch(
  () => props.code,
  () => {
    highlight();
  }
);

const codeContent = computed(() => highlightedCode.value);
</script>

<template>
  <div class="code-block-wrapper">
    <Skeleton v-if="isLoading" />
    <div v-else v-html="codeContent" class="code-block"></div>
  </div>
</template>

<style lang="postcss" scoped>
.code-block :deep(pre) {
  font-size: var(--text-xs);
  padding-top: calc(var(--spacing) * 4);
  overflow-x: auto;
  border-radius: var(--radius);
}
</style>
