<script>
  import P5 from 'p5-svelte';

  let selectedRegion = null;

  const regions = {
    frontal: {
      label: '額葉',
      x: 200, y: 120, w: 140, h: 80,
      functions: '執行功能、判斷、人格、語言表達',
      diseases: '額顳葉失智症、憂鬱症、躁鬱症',
      drugs: 'SSRIs、Memantine、Levetiracetam'
    },
    temporal: {
      label: '顳葉',
      x: 100, y: 200, w: 120, h: 80,
      functions: '記憶、語言理解、聽覺',
      diseases: '阿茲海默症、癲癇、語言障礙',
      drugs: 'Donepezil、Galantamine、Carbamazepine'
    },
    parietal: {
      label: '頂葉',
      x: 240, y: 180, w: 120, h: 80,
      functions: '感覺整合、空間感、注意力',
      diseases: '中風、感覺異常、忽略症候群',
      drugs: 'Aspirin、Clopidogrel'
    },
    occipital: {
      label: '枕葉',
      x: 340, y: 220, w: 100, h: 70,
      functions: '視覺處理',
      diseases: '視野缺損、視覺性癲癇',
      drugs: 'Valproate、Lamotrigine'
    },
    cerebellum: {
      label: '小腦',
      x: 320, y: 300, w: 110, h: 80,
      functions: '動作協調、平衡',
      diseases: '共濟失調、眩暈',
      drugs: 'Betahistine、Clonazepam'
    },
    brainstem: {
      label: '腦幹',
      x: 210, y: 290, w: 90, h: 70,
      functions: '呼吸、心跳、意識、眼球運動',
      diseases: '腦幹中風、意識障礙、帕金森',
      drugs: 'Levodopa、Amantadine'
    }
  };

  const sketch = (p) => {
    p.setup = () => {
      p.createCanvas(520, 420);
    };

    p.draw = () => {
      p.background(15, 20, 40);

      // 畫大腦輪廓
      p.stroke(60, 120, 200);
      p.strokeWeight(2);
      p.noFill();
      p.ellipse(250, 220, 380, 300);

      // 畫各腦區
      for (const [key, region] of Object.entries(regions)) {
        const isHover = p.mouseX > region.x && p.mouseX < region.x + region.w &&
                        p.mouseY > region.y && p.mouseY < region.y + region.h;
        const isSelected = selectedRegion === key;

        if (isSelected) {
          p.fill(100, 200, 255, 180);
        } else if (isHover) {
          p.fill(60, 140, 200, 150);
        } else {
          p.fill(30, 60, 120, 120);
        }

        p.stroke(80, 160, 255);
        p.strokeWeight(1.5);
        p.rect(region.x, region.y, region.w, region.h, 12);

        p.fill(220, 240, 255);
        p.noStroke();
        p.textAlign(p.CENTER, p.CENTER);
        p.textSize(14);
        p.text(region.label, region.x + region.w/2, region.y + region.h/2);
      }
    };

    p.mouseClicked = () => {
      for (const [key, region] of Object.entries(regions)) {
        if (p.mouseX > region.x && p.mouseX < region.x + region.w &&
            p.mouseY > region.y && p.mouseY < region.y + region.h) {
          selectedRegion = key;
          return;
        }
      }
      selectedRegion = null;
    };
  };
</script>
<div class="flex gap-6 p-6 min-h-screen bg-gray-900">
<div class="flex gap-6 p-6">
  <div>
    <h1 class="text-2xl font-bold text-white mb-4">腦區互動圖</h1>
    <P5 {sketch} />
    <p class="text-gray-400 text-sm mt-2">點擊腦區查看詳細資訊</p>
  </div>

  <div class="flex-1 min-w-64">
    {#if selectedRegion}
      {@const r = regions[selectedRegion]}
      <div class="bg-gray-800 rounded-xl p-6 text-white">
        <h2 class="text-xl font-bold text-blue-300 mb-4">{r.label}</h2>
        <div class="mb-3">
          <p class="text-gray-400 text-sm">功能</p>
          <p class="text-white">{r.functions}</p>
        </div>
        <div class="mb-3">
          <p class="text-gray-400 text-sm">相關疾病</p>
          <p class="text-yellow-300">{r.diseases}</p>
        </div>
        <div>
          <p class="text-gray-400 text-sm">常用藥物</p>
          <p class="text-green-300">{r.drugs}</p>
        </div>
      </div>
    {:else}
      <div class="bg-gray-800 rounded-xl p-6 text-gray-400">
        <p>點擊左側腦區查看詳細資訊</p>
      </div>
    {/if}
  </div>
</div></div>