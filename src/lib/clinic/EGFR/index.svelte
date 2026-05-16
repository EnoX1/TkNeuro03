<script>
 let scr = 1.0 
 let age = 60
 let gender = 1;
 let x;
 
 $: d = (186 * Math.pow(scr,  -1.154 ) * Math.pow(age, -0.203) * +gender).toFixed(2) ;
    
function setFromD(value) {
    d = +value
    let p = d / Math.pow(age, -0.203);
	  scr =p.toFixed(2); 
	}
</script>
<slot egfr={d}> </slot>
<div>
<h1>
  腎絲球過濾率計算
</h1>
<p>
  186 * 肌酸肝值Scr: <input style="width: 90px;"  bind:value={scr} type=number step="0.01">{@html "-1.154".sup()} * 年齡: <input style="width: 90px;"   bind:value={age} type=number>{@html "-0.203".sup()} = {d}
</p>
<p>
   肌酸肝值: {scr} 年齡: {age} 腎絲球過濾率= {d}  
</p>
<h2>性別</h2>
<label>
	<input type=radio bind:group={gender} value={1}>男生
</label>
<label>
	<input type=radio bind:group={gender} value={0.742}>女生 X 0.742
</label>

</div>
<style>
	input {
		width: 4em;
	}
</style>