---
dir: pages
title: BMI
icon: Envelope
---

<script>
  import BMI from '$lib/clinic/BMI/index.svelte';  
</script>

<BMI let:bmi={mybmi}>

<div>
<table class="myTable">
<tbody>
<tr>
  <th  class:active= {mybmi < 18.5}>
    BMI {"<"} 18.5
  </th>
  <th  class:active= {mybmi >= 18.5 && mybmi < 24}>
    18.5 ≤ BMI {"<"}  24
  </th>
  <th  class:active= {mybmi >= 24 && mybmi < 27 }>
    24 ≤ BMI {"<"}  27
  </th>
  <th  class:active= {mybmi >= 27}>
    BMI ≥ 27
  </th>
</tr>
<tr >
  <td >
    <p>「體重過輕」，需要多運動，均衡飲食，以增加體能，維持健康！</p>
  </td>
  <td >
    <p>恭喜！「健康體重」，要繼續保持！</p>
  </td>
  <td >
    <p>「體重過重」了，要小心囉，趕快力行「健康體重管理」！</p>
  </td>
  <td >
    <p>啊～「肥胖」，需要立刻力行「健康體重管理」囉！</p>
  </td>
</tr> 
</tbody>
</table>

</div>

<p>資料來源：衛生福利部國民健康署</p>

</BMI>

<style>

 	.active {
		background-color: #ff3e00;
		color: white;
	}

  table {
    border-collapse: collapse;
    text-align: center
  }
  th { background-color:#BDB76B } 
  .myTable { background-color:#eee; border:8px ; border-collapse:collapse; } 
  .myTable td, .myTable th { padding:5px;border:1px solid #000; } 
</style>
