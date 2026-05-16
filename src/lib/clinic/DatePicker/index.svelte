<script>
  import { DateInput, localeFromDateFnsLocale  } from 'date-picker-svelte'
  import { zhTW } from 'date-fns/locale'
  import { format } from 'date-fns';
  Date.prototype.addDays = function(days) {
    var date = new Date(this.valueOf());
    date.setDate(date.getDate() + days);
    return date;
  };
  let locale = localeFromDateFnsLocale(zhTW)

  let dateS1 = format(new Date(), "yyyy/MM/dd")
  let dateS2 = format(new Date(), "yyyy/MM/dd")
  let dateD1 = new Date()
  let dateD2 = new Date()

  $: dateS1 = format(dateD1, "yyyy/MM/dd") 
  $: dateS2 = format(dateD2, "yyyy/MM/dd") 

  $: d = Math.trunc((dateD2.getTime() - dateD1.getTime()) / (1000 * 60 * 60 * 24) );

	function setBothFromD(value) {
    d = +value;
	  dateD2 = dateD1.addDays(d);
    dateS2 = format(dateD2, "yyyy/MM/dd" )
	}
</script>
<div>
<p>
<tr>
  <td> Start Date </td>
  <td><DateInput {locale} format="yyyy/MM/dd" bind:value={dateD1} /></td>
  <td> End Date </td>
  <td><DateInput {locale} format="yyyy/MM/dd" bind:value={dateD2} /></td>
</tr>
</p>
<p>
{dateS1} + <input value={d} on:input="{e => setBothFromD(e.target.value)}" type=number step="1"> = {dateS2}
</p>

</div>

<style>
	input {
		width: 5em;
	}
</style>