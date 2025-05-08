<template>
  <div class="app">
    <h1>MLS Fixtures<br>(2025)</h1>
    <ul class="fixture-list">
      <li v-for="(fixture, index) in fixtures" :key="index" class="fixture-item">
        {{ fixture.date }} – {{ fixture.home_team }} vs {{ fixture.away_team }} ({{ fixture.score }})
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const fixtures = ref([])

onMounted(async () => {
  // Fetch the TXT file
  const response = await fetch('/data/2025_mls.txt')
  const textData = await response.text()

  // Split the file into lines
  const lines = textData.split('\n')

  let matchDate = ''
  const cutoffDate = new Date('2025-05-17')

  // Process each line
  lines.forEach(line => {
    // Skip lines that are headers or matchday indicators
    if (line.startsWith('»') || line.startsWith('Matchday') || line.startsWith('Date') || line.startsWith('#')) {
      return
    }

    // If the line looks like a date, update the matchDate
    if (line.match(/^Sat|Sun/)) {
      matchDate = line.trim()
      return
    }

    // Process match details
    const matchDetails = line.trim().split(/\s{2,}/) // Split by multiple spaces

    if (matchDetails.length === 4) {
      const [time, homeTeam, awayTeam, score] = matchDetails
      const matchDateObj = new Date(matchDate)

      // Only push fixtures up until the cutoff date (May 17, 2025)
      if (matchDateObj <= cutoffDate) {
        fixtures.value.push({
          date: matchDate,
          time: time,
          home_team: homeTeam,
          away_team: awayTeam,
          score: score || 'TBD'
        })
      }
    }
  })
})
</script>

<style>
.app {
  font-family: Arial, sans-serif;
  padding: 2rem;
}

.fixture-list {
  list-style: none;
  padding: 0;
}

.fixture-item {
  background: #f0f0f0;
  margin: 0.5rem 0;
  padding: 1rem;
  border-radius: 8px;
}
</style>
