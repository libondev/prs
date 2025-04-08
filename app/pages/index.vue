<script setup lang="ts">
import { useToggle } from '@vueuse/core'
import type { Contributions } from '~~/types'

const colorMode = useColorMode()
const { data: contributions } = await useFetch<Contributions>('/api/contributions')

if (!contributions.value) {
  throw createError('Could not load User activity')
}

interface Team {
  state: 'merged'
}

const selected = ref('')
const [collapse, toggleCollapse] = useToggle(false)

const url = useRequestURL()
const { user, prs } = contributions.value

const userUrl = `https://github.com/${user.username}`

const teams = (() => {
  const uniqTeams = prs.reduce((map, { repo }) => {
    const team = repo.split('/')[0]

    map.set(team, {
      name: team,
      repo: repo,
    })

    return map
  }, new Map<Team>())

  return [...uniqTeams.values()]
})()

const matchedPrs = computed(() => {
  const _team = selected.value

  if (!_team) {
    return prs
  }

  return prs.filter(it => it.repo.startsWith(_team))
})

const collapseTeams = computed(() => {
  if (collapse.value) {
    return teams.slice(0, 3)
  }

  return teams
})

useHead({
  link: [
    { rel: 'alternate', type: 'application/rss+xml', title: `${user.name}'s recent pull requests`, href: '/feed.xml' },
  ],
})

useSeoMeta({
  title: `${user.name} is Contributing`,
  description: `Discover ${user.name} recent pull requests on GitHub.`,
  ogTitle: `${user.name} is Contributing`,
  ogDescription: `Discover ${user.name} recent pull requests on GitHub.`,
  twitterCard: 'summary_large_image',
  ogImage: `${url.origin}/og.png`,
  twitterImage: `${url.origin}/og.png`,
})

// switch selected team
function onTeamSwitch(team: Team) {
  selected.value = team.name === selected.value ? '' : team.name
}
</script>

<template>
  <UContainer class="p-4 sm:p-6 lg:p-8 lg:pt-10 max-w-3xl">
    <div class="flex flex-col items-center gap-2">
      <a :href="userUrl" target="_blank"><UAvatar
        :src="user.avatar"
        :alt="user.name"
        size="xl"
        class="shadow-lg"
      />
      </a>

      <h1 class="text-2xl sm:text-3xl text-center">
        <a :href="userUrl" target="_blank">
          {{ user.name }}
        </a>
        is <span class="animate-pulse">Contributing...</span>
      </h1>

      <p class="text-center text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-300">
        <NuxtLink :to="userUrl" target="_blank">
          @{{ user.username }}'s recent pull requests on GitHub.
        </NuxtLink>
      </p>

      <div class="flex items-center justify-center gap-1 h-8 text-gray-700 dark:text-gray-300">
        <ClientOnly>
          <UButton
            :aria-label="`${user.name}'s GitHub profile`"
            :icon="colorMode.value === 'dark' ? 'i-lucide-moon' : 'i-lucide-sun'"
            color="gray"
            variant="link"
            @click="colorMode.preference = colorMode.value === 'dark' ? 'light' : 'dark'"
          />
          <template #fallback>
            <div class="w-8 h-8" />
          </template>
        </ClientOnly>

        <UButton
          :to="userUrl"
          external
          target="_blank"
          :aria-label="`${user.name}'s GitHub profile`"
          icon="i-lucide-github"
          color="gray"
          variant="link"
        />

        <UButton
          to="/feed.xml"
          external
          target="_blank"
          aria-label="RSS Feed"
          icon="i-lucide-rss"
          color="gray"
          variant="link"
        />
      </div>
    </div>

    <div class="group flex items-center flex-wrap gap-1.5 my-6 sm:mb-10">
      <div
        v-for="team of collapseTeams"
        :key="team.name"
        class="flex items-center gap-1 px-1.5 py-1 rounded-md group-hover:transition-colors text-sm cursor-default"
        :class="[team.name === selected ? 'bg-black text-white dark:bg-white dark:text-black' : 'bg-black/5 hover:bg-black/10 dark:bg-white/5 dark:hover:bg-white/10']"
        @click="onTeamSwitch(team)"
      >
        <img :src="`https://github.com/${team.name}.png`" :alt="team.repo" class="size-[1.1em] rounded-sm pointer-events-none">

        <span>{{ team.name }}</span>

        <a
          :href="`https://github.com/${team.repo}`"
          class="flex items-center opacity-35 hover:opacity-75"
          target="_blank"
          @click.stop
        >
          <UIcon name="i-material-symbols-attach-file-rounded" class="rotate-45" />
        </a>
      </div>

      <UButton
        variant="outline"
        class="px-1.5"
        @click="toggleCollapse()"
      >
        <UIcon name="i-solar-alt-arrow-left-line-duotone" :class="{ 'rotate-180': collapse }" />
      </UButton>
    </div>

    <UDivider class="mt-2 mb-6 sm:mb-10 w-1/2 mx-auto" />

    <div class="flex flex-col gap-6 sm:gap-10">
      <PullRequest v-for="pr of matchedPrs" :key="pr.url" :data="pr" />
    </div>

    <p class="text-center py-16 opacity-50">
      And more...
    </p>
  </UContainer>
</template>
