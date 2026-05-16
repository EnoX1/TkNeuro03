<script>
  import { Img, Heading, P,AccordionItem, Accordion } from 'flowbite-svelte';
</script>
<div alignment="mx-auto" style="width: 390px;" >

<Img src="/images/svelte-welcome.jpg" alt="welcome" size="max-w-lg" alignment="mx-auto" />

<Heading class="p-8" tag="h1" customSize="text-3xl"alignment="mx-auto">歡迎光臨 永佳診所</Heading>

<Accordion>
		<AccordionItem>
	    <span slot="header">永佳使命</span>
	    <p class="mb-2 text-red-500 dark:text-gray-400">照顧鄰里鄉親健康</p>
		</AccordionItem>
		<AccordionItem>
	    <span slot="header">永佳願景</span>
			<p class="mb-2 text-red-500 dark:text-gray-400">發展神經科學 向下紮根</p>
			<p class="mb-2 text-red-500 dark:text-dark-400">建立數位化網路醫療環境</p>
		</AccordionItem>
		<AccordionItem>
	    <span slot="header">永佳門診</span>
		  <p class="mb-2 text-red-500 dark:text-gray-400">  健保醫療服務</p>
		  <p class="mb-2 text-red-500 dark:text-gray-400">  復健治療     </p>
		  <p class="mb-2 text-red-500 dark:text-gray-400">  專業神經檢查     </p>
		  <p class="mb-2 text-red-500 dark:text-gray-400">  成人健檢     </p>
		  </AccordionItem>
		  <AccordionItem>
	    <span slot="header">永佳服務</span>
			<p class="mb-2 text-red-500 dark:text-gray-400">   視病猶親      </p>
		  </AccordionItem>
	</Accordion>   
</div>

