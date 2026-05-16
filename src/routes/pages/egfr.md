---
dir: pages
title: eGFR
icon: Envelope
---

<script>
  import EGFR from '$lib/clinic/EGFR/index.svelte'; 
</script>
<EGFR let:egfr={gfr}>
<p>
  腎絲球過濾率計算 (使用MDRD公式)<br/>
慢性腎臟病的五個階段是以「eGFR」(腎絲球濾過率值)來判斷您的腎臟功能，而eGFR又必須綜合考量您的年齡、性別以及「CR」(血清肌酸酐)。您知道自己的「CR」(血清肌酸酐)多少嗎?知道的話就可以計算你的腎臟功能了：
</p> {gfr}
<table class="myTable">
  <tbody>
  <tr>
    <th>腎臟病階段</th>
    <th class:active= {gfr >=90 }>第一期</th>
    <th class:active= {gfr <90 && gfr >=60}>第二期</th>
    <th class:active= {gfr < 60 && gfr >=30}>第三期</th>
    <th class:active= {gfr <30 && gfr>= 15 }>第四期</th>
    <th class:active= {gfr <15 }>第五期</th>
  </tr>
  <tr>
    <td>腎絲球過濾率</td>
    <td>>=90</td>
    <td>60-89</td>
    <td>
        3A: 45-59<br/>
        3B: 30-44
    </td>
    <td>15-29</td>
    <td>{"<"}15</td>
  </tr>
  <tr>
    <td>類型</td>
    <td>腎功能正常，但出現蛋白尿、血尿</td>
    <td>輕度腎衰竭且出現蛋白尿、血尿</td>
    <td>中度腎衰竭</td>
    <td>重度腎衰竭</td>
    <td>末期腎臟病變</td>
  </tr>
  <tr>
    <td>腎臟功能</td>
    <td colspan="2">腎臟功能仍有正常人的 60﹪以上，且出現蛋白尿、血尿或水腫等症狀</td>
    <td colspan="2">腎臟功能僅有正常人的 15-59﹪以上，會有水腫、高
血壓、貧血和疲倦等症狀。</td>
    <td>腎臟功能剩下正常人的 15﹪以下，無法排除體內代謝廢物和水分。 </td>
  </tr>
  <tr>
    <td>治療方式</td>
    <td colspan="2" >
      <p>維持腎臟功能<br/>
            1.健康飲食何規律作息<br/>
            2.積極控制血糖和血壓<br/>
            3.定期做腎功能檢查<br/><br/><br/>
      </p>
    </td>
    <td colspan="2">
      <p>
        減緩進入末期衰竭<br/>
        1.積極配合醫師治療<br/>
        2.健康的生活習慣<br/>
        3.預防腎骨病變:限制高磷食物攝取<br/>
        4.改善水腫:避免喝過多湯汁及鹽份<br/>
        5.低蛋白飲食控制<br/>
      </p>
    </td>
    <td>
      <p>
        準備進入透析階段<br/>
        1.使用藥物改善食慾不振及噁心<br/>
        2.治療貧血:可注射紅血球生成素或鐵劑<br/>
        3.預防高血鉀<br/>
        4.減少心肺積水<br/>
        5.透析或移植的準備及自我心理調適<br/>
      </p>
    </td>
  </tr>
  </tbody>
</table>
</EGFR>

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
  p {
    text-align: left
  }
 .myTable { background-color:#eee; border:8px ; border-collapse:collapse; } 
 .myTable td, .myTable th { padding:5px;border:1px solid #000; } 
</style>
