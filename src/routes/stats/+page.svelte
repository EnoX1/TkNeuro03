<script>
  import { supabase } from '$lib/supabase.js'
  import { onMount } from 'svelte'

  let date = new Date().toISOString().split('T')[0]
  let nhi_count = 0
  let self_pay_count = 0
  let notes = ''
  let message = ''
  let records = []

  onMount(async () => {
    await loadRecords()
  })

async function loadRecords() {
  const now = new Date()
  const firstDay = `${now.getFullYear()}-${String(now.getMonth()+1).padStart(2,'0')}-01`
  
  const { data, error } = await supabase
    .from('daily_stats')
    .select('*')
    .gte('date', firstDay)
    .order('date', { ascending: false })
  if (data) records = data
}


  async function saveStats() {
    const { error } = await supabase
      .from('daily_stats')
      .upsert({ date, nhi_count, self_pay_count, notes }, { onConflict: 'date' })
    if (error) {
      message = '儲存失敗：' + error.message
    } else {
      message = '儲存成功！'
      await loadRecords()
    }
  }

$: monthly_total = records.reduce((sum, r) => sum + (r.nhi_count || 0), 0)
</script>

<div class="p-6 max-w-2xl mx-auto">
  <h1 class="text-2xl font-bold mb-6">每日門診登錄</h1>

  <div class="bg-white border rounded-lg p-4 mb-6 space-y-4">
    <div>
      <label class="block text-sm font-medium mb-1">日期</label>
      <input type="date" bind:value={date}
        class="border rounded px-3 py-2 w-full" />
    </div>
    <div>
      <label class="block text-sm font-medium mb-1">健保人次</label>
      <input type="number" bind:value={nhi_count} min="0"
        class="border rounded px-3 py-2 w-full" />
    </div>
    <div>
      <label class="block text-sm font-medium mb-1">自費人次</label>
      <input type="number" bind:value={self_pay_count} min="0"
        class="border rounded px-3 py-2 w-full" />
    </div>
    <div>
      <label class="block text-sm font-medium mb-1">備註</label>
      <input type="text" bind:value={notes} placeholder="感冒季、假日診..."
        class="border rounded px-3 py-2 w-full" />
    </div>
    <button on:click={saveStats}
      class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700 w-full">
      儲存
    </button>
    {#if message}
      <p class="text-sm text-green-600">{message}</p>
    {/if}
  </div>

  <div class="bg-blue-50 border border-blue-200 rounded-lg p-4 mb-6">
    <p class="text-sm text-blue-800">本月健保累計：<span class="font-bold text-lg">{monthly_total}</span> 人次</p>
    <p class="text-sm text-blue-600">900人次門檻：{monthly_total >= 900 ? '✅ 達標' : `還差 ${900 - monthly_total} 人次`}</p>
  </div>

  <h2 class="text-lg font-semibold mb-3">最近30筆記錄</h2>
  <table class="w-full text-sm border-collapse">
    <thead>
      <tr class="bg-gray-100">
        <th class="border px-3 py-2 text-left">日期</th>
        <th class="border px-3 py-2">健保</th>
        <th class="border px-3 py-2">自費</th>
        <th class="border px-3 py-2 text-left">備註</th>
      </tr>
    </thead>
    <tbody>
      {#each records as r}
        <tr class="hover:bg-gray-50">
          <td class="border px-3 py-2">{r.date}</td>
          <td class="border px-3 py-2 text-center">{r.nhi_count}</td>
          <td class="border px-3 py-2 text-center">{r.self_pay_count}</td>
          <td class="border px-3 py-2">{r.notes || ''}</td>
        </tr>
      {/each}
    </tbody>
  </table>
</div>