<script>
	let h = 160;
	let w = 70; 
  
  $: x = Math.pow((h / 100),2).toFixed(1);
	$: b = (w / x).toFixed(1);
  $: over = b > 27;
	function setBothFromH(value) {
		h = +value;  
	}
	function setBothFromW(value) {
	   w = +value; 
	}

	function setBothFromB(value) {
		b = +value;
		w = +(b * x).toFixed(1);
	}

</script>

<slot bmi={b}> </slot>
<div>
身體質量指數 (BMI) 計算器
<p>
BH: <input value={h} on:input="{e => setBothFromH(e.target.value)}" type=number step="0.1"> cm +
BW: <input value={w} on:input="{e => setBothFromW(e.target.value)}" type=number step="0.1"> kg =
BMI <input value={b} on:input="{e => setBothFromB(e.target.value)}" type=number step="0.1">  
</p>
<p>
  BH: {h} cm  BW:  <span class:over>{w}</span> kg = BMI: <span class:over> {b} </span>
</p>
</div>   

<style>
	input {
		width: 5em;
	}
	span 
    {
      color: blue;
    }
	.over {
		background-color: #ff3e00;
		color: white;
	}
</style>