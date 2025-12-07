<script lang="ts">
  import { goto } from '$app/navigation';
  import { page } from '$app/state';
  import {
    baseLocale,
    getLocale,
    isLocale,
    localizeHref,
    locales,
    setLocale
  } from '$lib/paraglide/runtime';
  import { browser } from '$app/environment';

  const currentLocale = $derived.by(() => {
    // tie updates to navigation changes
    page.url;

    if (browser) {
      const docLang = document.documentElement.lang;
      if (isLocale(docLang)) return docLang;
    }

    try {
      return getLocale();
    } catch (err) {
      console.warn('Could not read current locale', err);
      return baseLocale;
    }
  });

  const switchLocale = async (locale: string) => {
    if (!isLocale(locale) || locale === currentLocale) return;
    // Persist preference without forcing full reload
    await setLocale(locale, { reload: false });

    const target = localizeHref(`${page.url.pathname}${page.url.search}${page.url.hash}`, {
      locale
    });

    await goto(target, { invalidateAll: true });
  };
</script>

<label class="sr-only" for="locale">Language</label>
<select
  id="locale"
  class="border-border bg-muted text-foreground hover:border-primary focus:border-primary rounded-full border px-3 py-1 text-sm shadow-sm"
  value={currentLocale}
  onchange={(event) => switchLocale((event.target as HTMLSelectElement).value)}
>
  {#each locales as locale (locale)}
    <option value={locale} selected={locale === currentLocale}>{locale.toUpperCase()}</option>
  {/each}
</select>
